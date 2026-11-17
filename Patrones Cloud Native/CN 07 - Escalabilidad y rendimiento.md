---
tags: [cloud-native, patrones, escalabilidad, autoscaling, rendimiento]
aliases: [Deployment Stamps, Geode, Autoscaling, Compute Resource Consolidation, CDN, Stateless, Escalado horizontal, Multi-tenant]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 07 - Escalabilidad y rendimiento

> [!info] Objetivo del capítulo
> Cómo crecer en carga, usuarios, tenants y regiones sin rediseñar, y sin pagar por capacidad ociosa.

## Contenido
- [[#0. Conceptos]]
- [[#1. Stateless Services]]
- [[#2. Escalado horizontal y Autoscaling]]
- [[#3. Deployment Stamps]]
- [[#4. Geode]]
- [[#5. CDN y Edge Caching]]
- [[#6. Compute Resource Consolidation]]
- [[#7. Patrones multi-tenant]]
- [[#8. Rendimiento: dónde se pierde el tiempo]]

---

## 0. Conceptos

| Concepto | Definición |
|---|---|
| **Escalabilidad** | Capacidad de atender más carga añadiendo recursos, con un coste proporcional |
| **Elasticidad** | Añadir **y quitar** recursos automáticamente según la carga (pagar sólo lo que se usa) |
| **Escalado vertical** (*scale up*) | Máquina más grande. Sencillo, pero con techo, y suele requerir reinicio |
| **Escalado horizontal** (*scale out*) | Más instancias. Sin techo práctico, pero exige diseño *stateless* |
| **Latencia** | Tiempo de una operación. Se mide en **percentiles** (p50, p95, p99), nunca sólo en media |
| **Throughput** | Operaciones por unidad de tiempo |

> [!tip] Ley de Little
> `concurrencia = throughput × latencia`. Si atiendes 1.000 req/s con 200 ms de latencia, tienes ~200 peticiones en curso de media. Si la latencia sube a 2 s (una dependencia lenta), necesitas ~2.000 peticiones simultáneas: por eso la latencia agota pools de conexiones e hilos.

---

## 1. Stateless Services

**Problema.** Si una instancia guarda estado del usuario (sesión en memoria, ficheros subidos en disco local, caché local como única copia), las peticiones siguientes deben ir a **esa** instancia (*sticky sessions*). No puedes escalar libremente, ni reemplazar instancias sin perder datos.

**Solución.** Sacar el estado del proceso a **servicios de respaldo** externos:

| Estado | Dónde llevarlo |
|---|---|
| Sesión | Redis (`AddStackExchangeRedisCache` + `AddSession`), o tokens en el cliente (JWT/cookie) |
| Ficheros | Blob Storage / S3 |
| Caché | Redis / HybridCache con L2 distribuida (la L1 local es sólo una aceleración) |
| Claves de Data Protection de ASP.NET Core | Blob Storage + Key Vault (`PersistKeysToAzureBlobStorage`) — si no, las cookies de una réplica no las lee otra |
| Trabajos en curso | Una cola, no un `Task.Run` en memoria |

> [!warning] Data Protection: el fallo clásico al escalar ASP.NET Core
> Cada réplica genera sus propias claves de Data Protection por defecto. Con 2+ réplicas, las cookies de autenticación y los tokens antifalsificación emitidos por una réplica fallan en otra. Hay que persistir las claves en un almacén compartido.

Los servicios **con estado** (bases de datos, brokers) existen, pero se gestionan aparte: StatefulSet, Operators o, preferiblemente, servicios gestionados.

---

## 2. Escalado horizontal y Autoscaling

**Problema.** La carga varía (día/noche, campañas); dimensionar para el pico desperdicia dinero, dimensionar para la media provoca caídas en el pico.

**Solución.** Ajustar el número de instancias automáticamente según **métricas**.

| Tipo | Qué escala | Según qué | Herramienta Kubernetes / Azure |
|---|---|---|---|
| **Horizontal de Pods** | Réplicas | CPU, memoria, métricas personalizadas | HPA |
| **Por eventos** | Réplicas (incluso a 0) | Longitud de cola, lag de Kafka, cron, métricas de Prometheus | **KEDA**; Azure Container Apps (usa KEDA) |
| **Vertical de Pods** | Requests/limits | Uso histórico | VPA (no combinar con HPA sobre la misma métrica) |
| **Nodos** | Máquinas del clúster | Pods pendientes de planificar | Cluster Autoscaler, **Karpenter** (AKS: *Node Auto Provisioning*) |
| **Predictivo/programado** | Réplicas | Patrones conocidos | KEDA cron scaler, *scheduled scaling* |

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: api-pedidos }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: api-pedidos }
  minReplicas: 3            # mínimo para HA (una por zona)
  maxReplicas: 30
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 65 } }
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300   # evita "flapping": espera 5 min antes de reducir
```

**Claves de diseño:**
- **Escala por la métrica que refleja la saturación real**: para APIs suele ser CPU o peticiones por segundo; para workers, longitud/antigüedad de la cola.
- El autoscaling **tarda** (minutos si hay que crear nodos). Combínalo con Load Leveling, Throttling y un mínimo de réplicas suficiente.
- Escalar la app **no escala la base de datos**: 50 réplicas × 100 conexiones de pool = 5.000 conexiones. Revisa límites de las dependencias.
- **Arranque rápido** (factor IX): imágenes pequeñas, *ReadyToRun* o Native AOT en .NET si el arranque importa (p. ej. al escalar desde 0).

Detalle: [[08 - Escalado]].

---

## 3. Deployment Stamps

**Problema.** Un único despliegue compartido por todos los clientes acaba chocando con límites (de suscripción, de servicio, de BD), un fallo afecta a todos y algunos clientes necesitan sus datos en una región concreta o aislamiento.

**Solución.** Desplegar **varias copias independientes** de la aplicación completa (cómputo + datos), llamadas **stamps** (sellos) o *scale units* / *cells*. Cada stamp atiende a un subconjunto de clientes/tenants. Un **router global** dirige cada petición a su stamp.

```
                    ┌─► Stamp 1 (Europa Oeste) : tenants A–F  [AKS + SQL + Redis]
Front Door ─► Router┼─► Stamp 2 (Europa Oeste) : tenants G–P  [AKS + SQL + Redis]
(directorio tenant) └─► Stamp 3 (EE. UU. Este) : tenants Q–Z  [AKS + SQL + Redis]
```

**Ventajas:** escala casi lineal (añadir stamps), **radio de impacto** limitado (un fallo o un mal despliegue afecta sólo a un stamp), residencia de datos por región, despliegues progresivos stamp a stamp (*ring deployment*).
**Costes:** necesitas **IaC** total (cada stamp debe ser idéntico y reproducible), un **directorio** de tenants → stamp, y procesos para mover tenants entre stamps; las consultas globales cruzan stamps.

**Relación con la *cell-based architecture*** (AWS): es esencialmente el mismo concepto con otro nombre.

---

## 4. Geode

**Problema.** Usuarios repartidos por el mundo sufren latencia alta si todo está en una región, y una región caída deja sin servicio a todos.

**Solución.** Desplegar **nodos de backend completos en varias regiones** (*geodes*), todos **activos** y capaces de atender **cualquier** petición, con los datos **replicados globalmente**. Un balanceador global envía a cada usuario a la región más cercana y sana.

```
Usuario Europa ─┐                        ┌─► Geode Europa ─┐
Usuario EE. UU. ─┼─► Front Door (latencia)┼─► Geode EE. UU. ─┼─► Cosmos DB multi-región
Usuario Asia   ─┘                        └─► Geode Asia   ─┘    (replicación global)
```

**Diferencia con Deployment Stamps:** en Stamps cada stamp tiene **sus** datos y atiende **a sus** clientes; en Geode todos los nodos comparten datos replicados y atienden **a cualquiera**.

**El reto son los datos:** escrituras en varias regiones implican **conflictos** (resolución *last-writer-wins*, CRDTs o lógica propia) y consistencia eventual entre regiones. Requiere una BD diseñada para ello (Cosmos DB con escrituras multirregión, DynamoDB Global Tables, Spanner, CockroachDB). Muy caro y complejo: sólo cuando la latencia global o la disponibilidad lo justifican.

---

## 5. CDN y Edge Caching

**Problema.** Servir contenido desde el origen a usuarios lejanos es lento y consume capacidad del origen.

**Solución.** Una **Content Delivery Network** cachea contenido en **puntos de presencia** (*edge*) cerca de los usuarios. Productos: Azure Front Door, Cloudflare, Akamai, CloudFront.

- Contenido estático (ver Static Content Hosting en [[CN 05 - Patrones de gestión de datos]]): TTL largo + nombres versionados.
- Contenido dinámico cacheable (catálogo público): TTL corto + `Cache-Control`, `ETag`, `Vary`.
- Además de caché: terminación TLS cerca del usuario, WAF, protección DDoS, enrutamiento global.

**Cuidado:** nunca cachear respuestas personalizadas o con datos privados en una caché compartida (`Cache-Control: private` / `no-store`).

---

## 6. Compute Resource Consolidation

**Problema.** Muchos componentes pequeños, cada uno en su propia unidad de cómputo infrautilizada (una VM o un App Service Plan por microservicio al 5 % de CPU), disparan el coste y la gestión.

**Solución.** **Consolidar** varios componentes en la misma unidad de cómputo cuando tengan perfiles compatibles. Kubernetes hace esto por diseño (muchos Pods por nodo, *bin packing*); en PaaS, varias apps en el mismo App Service Plan o entorno de Container Apps.

**Criterios para agrupar:** ciclo de vida, requisitos de escalado, de seguridad y de aislamiento similares. **No** agrupes un componente crítico con uno ruidoso (ver *Noisy Neighbor* en [[CN 11 - Antipatrones]]) — aquí hay tensión con el patrón Bulkhead.

---

## 7. Patrones multi-tenant

En un SaaS, el modelo de *tenancy* determina coste, aislamiento y escalabilidad:

| Modelo | Descripción | Aislamiento | Coste por tenant |
|---|---|---|---|
| **Todo compartido** (*pool*) | Mismas instancias y misma BD; filtro por `TenantId` | Bajo (riesgo de fuga de datos por un bug en el filtro) | Muy bajo |
| **BD por tenant** | Cómputo compartido, BD propia | Medio-alto | Medio (Azure SQL Elastic Pools lo abarata) |
| **Stamp por tenant** (*silo*) | Despliegue completo dedicado | Máximo | Alto |
| **Híbrido** | Tenants pequeños en *pool*, grandes en *silo* | Según tier | Según tier — el más común en la práctica |

**Riesgos a gestionar:** *noisy neighbor* (un tenant consume todo), cuotas y rate limiting por tenant, filtrado obligatorio por tenant (filtros globales de EF Core, Row-Level Security en SQL), y medición del consumo por tenant para facturar.

---

## 8. Rendimiento: dónde se pierde el tiempo

Lista de revisión, de mayor a menor impacto típico:

1. **Llamadas remotas en serie** → paralelizar (`Task.WhenAll`), agregar, cachear.
2. **Consultas N+1** a la BD → `Include`, proyecciones, consultas por lotes.
3. **Chatty I/O** (muchas llamadas pequeñas) → operaciones por lotes, ver [[CN 11 - Antipatrones]].
4. **Sin caché** de datos que cambian poco → Cache-Aside.
5. **Payloads enormes** → paginación, compresión, campos seleccionados, gRPC/Protobuf.
6. **Crear clientes por petición** (`new HttpClient()`, conexiones de BD sin pool) → `IHttpClientFactory`, instancias reutilizadas (*Improper Instantiation*).
7. **Bloquear hilos** (`.Result`, `.Wait()`) → `async`/`await` de extremo a extremo.

> [!tip] Mide antes de optimizar
> Usa trazas distribuidas para ver **dónde** se va el tiempo de una petición antes de añadir caché o escalar. Ver [[CN 10 - Observabilidad]]. Herramientas de carga: k6, Azure Load Testing, NBomber (.NET).

---

## Puntos clave

- Escalar horizontalmente exige **stateless**; en ASP.NET Core, no olvides **Data Protection**.
- HPA por CPU/RPS para APIs; **KEDA** por cola para workers; Cluster Autoscaler/Karpenter para nodos.
- El autoscaling es lento: combínalo con colas y throttling.
- **Deployment Stamps**: copias independientes por grupo de clientes → escala lineal y radio de impacto limitado.
- **Geode**: todas las regiones activas atendiendo a cualquiera, datos replicados globalmente → muy complejo.
- CDN para todo lo cacheable; nunca datos privados en caché compartida.
- Mide con percentiles (p95/p99), no con medias.

## Errores comunes

- Sesión en memoria + escalar a 3 réplicas → usuarios desconectados aleatoriamente.
- HPA y VPA sobre la misma métrica.
- Escalar la API a 50 réplicas y tumbar la BD por exceso de conexiones.
- Multirregión activo-activo "por si acaso", sin necesidad de negocio ni estrategia de conflictos.
- Juzgar el rendimiento por la latencia media.

## 🎯 Para entrevistas

- *"¿Qué tiene que cumplir un servicio para escalar horizontalmente?"* → stateless, config externa, arranque rápido, apagado elegante, sin afinidad.
- *"Deployment Stamps vs Geode"* → datos por stamp y clientes asignados vs datos replicados y cualquier región atiende a cualquiera.
- *"¿Cómo escalarías un worker que consume de Service Bus?"* → KEDA con scaler de Service Bus (incluso a 0), vigilando el límite de la BD de destino.
- *"Diseña el modelo de tenancy de un SaaS B2B con clientes pequeños y algunos enterprise."* → híbrido: pool para pequeños, BD o stamp dedicados para enterprise; directorio de tenants; rate limit y métricas por tenant.

## Referencias

- Microsoft, *Deployment Stamps*, *Geode*, *Compute Resource Consolidation*, *Static Content Hosting*: https://learn.microsoft.com/azure/architecture/patterns/
- Microsoft, *Architect multitenant solutions on Azure*: https://learn.microsoft.com/azure/architecture/guide/multitenant/overview
- Kubernetes, *Horizontal Pod Autoscaling*: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
- KEDA: https://keda.sh/
- Microsoft, *Data Protection key management and lifetime in ASP.NET Core*: https://learn.microsoft.com/aspnet/core/security/data-protection/configuration/overview
- AWS, *Reducing the Scope of Impact with Cell-Based Architecture* (Well-Architected whitepaper).

---
⬅️ [[CN 06 - Resiliencia y tolerancia a fallos]] · ➡️ [[CN 08 - Despliegue, configuración y entrega]]
