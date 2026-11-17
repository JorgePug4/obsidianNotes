---
tags: [cloud-native, patrones, api-gateway, bff, service-mesh, comunicacion]
aliases: [Backends for Frontends, BFF, Gateway Aggregation, Gateway Offloading, Gateway Routing, Service Mesh, Async Request-Reply, Strangler Fig, Anti-Corruption Layer]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 03 - Comunicación, enrutamiento y APIs

> [!info] Objetivo del capítulo
> Cómo **se encuentran, se exponen y se hablan** los servicios, y cómo migrar de un sistema antiguo a uno nuevo sin un *big bang*.

## Contenido
- [[#0. Síncrono vs asíncrono: la decisión previa]]
- [[#1. API Gateway (y sus tres variantes)]]
- [[#2. Backends for Frontends (BFF)]]
- [[#3. Service Discovery]]
- [[#4. Service Mesh]]
- [[#5. Asynchronous Request-Reply]]
- [[#6. API Composition]]
- [[#7. Anti-Corruption Layer]]
- [[#8. Strangler Fig]]
- [[#9. Messaging Bridge]]
- [[#Comparación]]

---

## 0. Síncrono vs asíncrono: la decisión previa

| | Síncrono (HTTP/REST, gRPC) | Asíncrono (colas, eventos) |
|---|---|---|
| Acoplamiento temporal | **Alto**: ambos deben estar vivos a la vez | **Bajo**: el receptor puede estar caído |
| Latencia percibida | Inmediata | Diferida |
| Modelo mental | Sencillo (como una llamada a función) | Más complejo (consistencia eventual, duplicados, orden) |
| Fallos | Se propagan en cadena | Se absorben en la cola |
| Cuándo | Consultas que el usuario espera *ahora*; validaciones | Procesos de negocio, notificaciones, integración entre dominios |

**Disponibilidad compuesta.** Si A llama de forma síncrona a B, C y D, y cada uno tiene 99,9 % de disponibilidad, la de A no puede superar ≈ 0,999⁴ ≈ **99,6 %** (asumiendo fallos independientes). Cada dependencia síncrona resta. Es el argumento matemático para la asincronía y para la resiliencia ([[CN 06 - Resiliencia y tolerancia a fallos]]).

**REST vs gRPC entre servicios:** gRPC (HTTP/2 + Protobuf) es más eficiente y tiene contratos fuertes; REST/JSON es más universal y fácil de depurar. Patrón habitual: REST hacia fuera, gRPC dentro. Ver [[Diseño Api Rest]].

---

## 1. API Gateway (y sus tres variantes)

> Nota detallada: [[API Gateway]].

**Problema.** Los clientes tendrían que conocer decenas de servicios, sus direcciones y protocolos, y cada servicio tendría que implementar autenticación, TLS, rate limiting y logging.

**Solución.** Un **único punto de entrada** que se interpone entre clientes y servicios. Microsoft lo descompone en tres patrones distintos que suelen convivir en el mismo producto:

| Patrón | Qué hace | Ejemplo |
|---|---|---|
| **Gateway Routing** | Enruta peticiones a varios servicios por una única URL, según ruta, cabecera o versión | `/api/pedidos/*` → servicio Pedidos; `/api/catalogo/*` → Catálogo |
| **Gateway Aggregation** | Una petición del cliente → varias peticiones internas → una respuesta combinada | La pantalla "detalle de pedido" necesita pedido + cliente + envío |
| **Gateway Offloading** | Descarga funcionalidad transversal de los servicios al gateway | Terminación TLS, validación de JWT, rate limiting, compresión, caché |

```
               ┌────────────── API Gateway ──────────────┐
Cliente ─HTTPS─► TLS · JWT · rate limit · routing · agreg.├──► Pedidos
               └─────────────────────────────────────────┘├──► Catálogo
                                                          └──► Clientes
```

**Ejemplo con YARP (.NET):**

```json
"ReverseProxy": {
  "Routes": {
    "pedidos":  { "ClusterId": "pedidos",  "Match": { "Path": "/api/pedidos/{**resto}" },
                  "AuthorizationPolicy": "default", "RateLimiterPolicy": "por-usuario" },
    "catalogo": { "ClusterId": "catalogo", "Match": { "Path": "/api/catalogo/{**resto}" } }
  },
  "Clusters": {
    "pedidos":  { "Destinations": { "d1": { "Address": "http://pedidos:8080" } } },
    "catalogo": { "Destinations": { "d1": { "Address": "http://catalogo:8080" } } }
  }
}
```

**Productos:** Azure API Management, Azure Application Gateway (L7, sin lógica de API avanzada), AWS API Gateway, Kong, Envoy, NGINX, Traefik, YARP; en Kubernetes, **Gateway API** (sucesor de Ingress).

**Cuándo no.** Si no hay lógica de negocio que agregar, no metas agregación en el gateway. Y nunca pongas **lógica de dominio** en él: se convierte en un monolito escondido.

**Trade-offs.** ✅ Simplifica clientes, centraliza seguridad. ❌ Punto único de fallo y cuello de botella (necesita HA y escalado), salto de red extra, riesgo de que un equipo central se convierta en cuello de botella organizativo.

---

## 2. Backends for Frontends (BFF)

**Problema.** Una app móvil, una web SPA y un partner B2B necesitan datos diferentes: el móvil quiere respuestas pequeñas, la web quiere más detalle, el partner un contrato estable. Un gateway único acaba lleno de condiciones `if (cliente == móvil)` y lo modifican todos los equipos.

**Solución.** **Un backend (gateway) por tipo de frontend**, idealmente propiedad del mismo equipo que el frontend. Patrón descrito por Sam Newman (2015), originado en SoundCloud.

```
App móvil ──► BFF móvil ──┐
Web SPA   ──► BFF web   ──┼──► Pedidos · Catálogo · Clientes
Partners  ──► API pública ┘
```

**BFF y seguridad en SPAs.** Hoy el BFF también se usa como patrón de **seguridad**: el BFF guarda los tokens OAuth en el servidor y el navegador sólo recibe una *cookie* de sesión `HttpOnly`/`Secure`/`SameSite`. Así los tokens no son accesibles desde JavaScript (mitiga robo por XSS). Es la recomendación del borrador IETF *OAuth 2.0 for Browser-Based Applications*. Ver [[CN 09 - Seguridad]].

**Cuándo usarlo.** Varios tipos de cliente con necesidades realmente distintas y equipos de frontend separados. **Cuándo no.** Un solo tipo de cliente, o clientes con necesidades casi iguales: un gateway basta.

**Trade-offs.** ✅ Cada frontend evoluciona a su ritmo; respuestas optimizadas. ❌ Más servicios que desplegar; riesgo de duplicar lógica entre BFFs (extraer lo común a servicios, no a una librería compartida enorme).

---

## 3. Service Discovery

> Nota detallada: [[Service Discovery]].

**Problema.** Las instancias aparecen y desaparecen con IPs dinámicas (falacia nº 5: "la topología no cambia").

**Solución.** Un **registro** de instancias disponibles y un mecanismo para consultarlo.

| Variante | Quién consulta el registro | Ejemplo |
|---|---|---|
| **Client-side discovery** | El cliente, que además balancea | Netflix Eureka + Ribbon; `Microsoft.Extensions.ServiceDiscovery` |
| **Server-side discovery** | Un balanceador/proxy intermedio | **Kubernetes Service** (DNS `pedidos.ns.svc.cluster.local` → ClusterIP → kube-proxy), AWS ALB |

En Kubernetes no tienes que implementarlo: el **Service** + **CoreDNS** es server-side discovery. Ver [[04 - Networking y tráfico]].

---

## 4. Service Mesh

**Problema.** Con decenas de servicios quieres mTLS entre todos, reintentos, timeouts, circuit breaking, *canary* por porcentaje y métricas de cada llamada… sin implementarlo en el código de cada servicio y cada lenguaje.

**Solución.** Una **capa de infraestructura dedicada a la comunicación servicio-a-servicio**, con dos planos:

- **Data plane:** proxies (Envoy, `linkerd2-proxy`) que interceptan todo el tráfico, normalmente como **sidecar** de cada Pod, o a nivel de nodo en los modos *sidecarless* (Istio *ambient*, Cilium).
- **Control plane:** configura los proxies (istiod, Linkerd control plane).

```
┌── Pod A ────────────┐        mTLS         ┌── Pod B ────────────┐
│ App A ─► Proxy (A)  ├────────────────────►│ Proxy (B) ─► App B  │
└─────────▲───────────┘                     └─────────▲───────────┘
          └────────────── Control plane (políticas, certificados) ┘
```

**Qué te da:** mTLS automático e identidad de servicio, políticas de autorización (quién puede llamar a quién), reintentos/timeouts/outlier detection, división de tráfico (canary, espejo), telemetría uniforme (métricas RED y trazas).

**Productos:** Istio, Linkerd, Cilium Service Mesh, Consul Connect. (El proyecto Open Service Mesh se archivó en 2023 y Microsoft anunció la retirada de su complemento para AKS; la alternativa recomendada es el complemento de AKS basado en Istio.)

**Cuándo usarlo.** Muchos servicios, varios lenguajes, requisitos de mTLS/Zero Trust o de gestión de tráfico avanzada.
**Cuándo no.** Pocos servicios o un solo lenguaje con buenas librerías: el coste operativo (actualizar el mesh, depurar el proxy, recursos de sidecars) suele superar el beneficio.

> [!warning] No dupliques resiliencia
> Si el mesh reintenta 3 veces y tu `HttpClient` también 3 veces, una llamada puede convertirse en 9 (o más, si hay varios saltos). Decide **una** capa responsable de los reintentos.

---

## 5. Asynchronous Request-Reply

**Problema.** El cliente hace una petición HTTP que tarda minutos (generar un informe, procesar un vídeo). Mantener la conexión abierta provoca timeouts en balanceadores y gateways y bloquea recursos.

**Solución.** Separar la petición de la respuesta:

```
1. POST /informes           → 202 Accepted
                              Location: /informes/estado/abc123
2. GET /informes/estado/abc123 → 200 { "estado": "en_proceso" }   (Retry-After: 10)
3. GET /informes/estado/abc123 → 303 See Other
                              Location: /informes/abc123
4. GET /informes/abc123      → 200 (el resultado)
```

Internamente, la API encola el trabajo y un worker lo procesa (ver [[CN 04 - Mensajería y arquitectura orientada a eventos]]).

```csharp
app.MapPost("/informes", async (SolicitudInforme req, ServiceBusSender sender, IEstadoStore estados) =>
{
    var id = Guid.NewGuid().ToString("N");
    await estados.GuardarAsync(id, "pendiente");
    await sender.SendMessageAsync(new ServiceBusMessage(BinaryData.FromObjectAsJson(req with { Id = id })));
    return Results.Accepted($"/informes/estado/{id}", new { id });
});

app.MapGet("/informes/estado/{id}", async (string id, IEstadoStore estados, HttpResponse res) =>
{
    var estado = await estados.ObtenerAsync(id);
    if (estado is null) return Results.NotFound();
    if (estado == "completado") return Results.Redirect($"/informes/{id}"); // o 303 explícito
    res.Headers.RetryAfter = "10";
    return Results.Ok(new { id, estado });
});
```

**Alternativas al *polling*:** WebSockets/SignalR, Server-Sent Events, o un **webhook** (*callback*) que el servidor llama al terminar.

**Trade-offs.** ✅ Sin timeouts, la API sigue respondiendo bajo carga. ❌ Más complejidad en el cliente; hay que limpiar estados antiguos; el polling genera tráfico (mitígalo con `Retry-After`).

---

## 6. API Composition

**Problema.** Con *database per service*, una consulta que necesita datos de varios servicios no puede hacer un `JOIN`.

**Solución.** Un **compositor** (el gateway, un BFF o un servicio) llama a cada servicio dueño de los datos y combina el resultado en memoria (Richardson). Es la misma idea que Gateway Aggregation, aplicada a consultas.

```csharp
var pedidoTask  = pedidos.ObtenerAsync(id);
var clienteTask = clientes.ObtenerAsync(clienteId);
var envioTask   = envios.ObtenerAsync(id);
await Task.WhenAll(pedidoTask, clienteTask, envioTask);   // en paralelo, no en serie
```

**Límites.** Lento con muchos servicios o datos grandes; disponibilidad reducida (necesitas todos); sin consistencia entre fuentes. Si la consulta es frecuente o compleja, pasa a **CQRS / Materialized View** ([[CN 05 - Patrones de gestión de datos]]).

---

## 7. Anti-Corruption Layer

**Problema.** Tu nuevo servicio debe integrarse con un sistema *legacy* (o un proveedor externo) con un modelo de datos y una semántica distintos. Si usas su modelo directamente, "contamina" tu dominio.

**Solución.** Una **capa de traducción** entre ambos modelos (concepto de DDD, Eric Evans 2003). Tu dominio sólo ve tus conceptos; la ACL convierte en ambas direcciones.

```
[Nuevo servicio: Cliente{Id, Nombre, Email}] ◄── ACL ──► [ERP legacy: KUNNR, NAME1, SMTP_ADDR]
```

Puede ser una clase *adapter* dentro del servicio o un servicio independiente. Ver [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]].

**Trade-offs.** ✅ Protege el modelo, facilita retirar el legacy después. ❌ Otra pieza que mantener y escalar; latencia si es un servicio separado.

---

## 8. Strangler Fig

**Problema.** Reescribir un monolito de golpe (*big bang rewrite*) es muy arriesgado: meses sin entregar valor y un corte final peligroso.

**Solución.** Patrón nombrado por Martin Fowler (2004) por las higueras estranguladoras, que crecen sobre un árbol hasta reemplazarlo. Se pone una **fachada** (proxy/gateway) delante del sistema antiguo y se van **redirigiendo funcionalidades una a una** al sistema nuevo.

```
Fase 1:  Cliente ─► Fachada ─► Monolito (100 %)
Fase 2:  Cliente ─► Fachada ─┬► Nuevo: /catalogo
                             └► Monolito: resto
Fase 3:  Cliente ─► Fachada ─► Nuevo (100 %)  →  se retira el monolito
```

**Claves para que funcione:**
- Empieza por funcionalidades de **bajo riesgo y alto valor**, o por las que más cambian.
- Los **datos** son lo difícil: durante la transición puede hacer falta sincronizar BD antigua y nueva (CDC, eventos) con una **Anti-Corruption Layer**.
- Usa **feature flags** o la fachada para volver atrás rápido.
- Define cuándo se retira cada parte; si no, acabas manteniendo dos sistemas para siempre.

YARP (.NET) tiene una guía específica de migración incremental de ASP.NET (Framework) a ASP.NET Core con este patrón.

---

## 9. Messaging Bridge

**Problema.** Dos sistemas usan brokers de mensajería distintos (RabbitMQ on-premise y Azure Service Bus; Kafka y una cola legacy) y necesitan intercambiar mensajes.
**Solución.** Un componente intermedio que **consume de un sistema y publica en el otro**, traduciendo protocolo y formato (patrón de *Enterprise Integration Patterns*, incluido en el catálogo de Microsoft). Útil en migraciones y entornos híbridos; añade latencia y un punto que debe ser idempotente y monitorizado.

---

## Comparación

| Patrón | Resuelve | Dónde vive |
|---|---|---|
| API Gateway | Un punto de entrada, funciones transversales | Borde (edge) |
| BFF | Necesidades distintas por tipo de cliente | Borde, uno por cliente |
| Service Discovery | Encontrar instancias dinámicas | Plataforma (K8s DNS) |
| Service Mesh | Comunicación interna segura, resiliente y observable | Plataforma (proxies) |
| Async Request-Reply | Operaciones largas detrás de HTTP | API + cola + worker |
| API Composition | Consultas que cruzan servicios | Gateway/BFF/servicio |
| Anti-Corruption Layer | Proteger el modelo frente a sistemas ajenos | Frontera entre contextos |
| Strangler Fig | Migración incremental | Fachada delante del legacy |

**API Gateway vs Service Mesh:** el gateway gestiona tráfico **norte-sur** (cliente → sistema); el mesh, tráfico **este-oeste** (servicio → servicio). Se complementan.

## Puntos clave

- Cada dependencia **síncrona** reduce la disponibilidad compuesta.
- Gateway = Routing + Aggregation + Offloading. **Sin lógica de negocio.**
- BFF = un gateway por tipo de cliente, del equipo del frontend; también patrón de seguridad para SPAs.
- En Kubernetes, Service Discovery viene de serie.
- Service Mesh: potente pero con coste operativo; no lo adoptes por moda.
- Operaciones largas → `202 Accepted` + `Location` + polling o callback.
- Migrar → Strangler Fig + ACL, nunca *big bang* si puedes evitarlo.

## Errores comunes

- Cadenas largas de llamadas síncronas (A→B→C→D→E).
- Lógica de negocio en el gateway.
- Reintentos configurados a la vez en cliente, mesh y gateway.
- Agregar en serie lo que podría ir en paralelo (`await` uno tras otro en vez de `Task.WhenAll`).
- Strangler Fig sin plan de datos ni fecha de retirada.

## 🎯 Para entrevistas

- *"API Gateway vs BFF"* → gateway único genérico vs uno por tipo de cliente; el BFF evita el gateway "lleno de ifs".
- *"API Gateway vs Service Mesh"* → norte-sur vs este-oeste.
- *"¿Cómo expondrías una operación de 10 minutos por HTTP?"* → Async Request-Reply: 202 + Location + estado + Retry-After, o webhook.
- *"¿Cómo migrarías un monolito .NET Framework a microservicios?"* → Strangler Fig con YARP como fachada, ACL para el modelo legacy, empezar por un bounded context de bajo riesgo.

## Referencias

- Microsoft, *Gateway Routing / Aggregation / Offloading*: https://learn.microsoft.com/azure/architecture/patterns/gateway-routing (y `/gateway-aggregation`, `/gateway-offloading`)
- Microsoft, *Backends for Frontends*: https://learn.microsoft.com/azure/architecture/patterns/backends-for-frontends
- Microsoft, *Asynchronous Request-Reply*: https://learn.microsoft.com/azure/architecture/patterns/async-request-reply
- Microsoft, *Anti-corruption Layer*: https://learn.microsoft.com/azure/architecture/patterns/anti-corruption-layer
- Microsoft, *Strangler Fig*: https://learn.microsoft.com/azure/architecture/patterns/strangler-fig
- Microsoft, *Messaging Bridge*: https://learn.microsoft.com/azure/architecture/patterns/messaging-bridge
- Sam Newman, *Backends For Frontends* (2015): https://samnewman.io/patterns/architectural/bff/
- Martin Fowler, *StranglerFigApplication*: https://martinfowler.com/bliki/StranglerFigApplication.html
- Chris Richardson, *API Composition*: https://microservices.io/patterns/data/api-composition.html
- IETF, *OAuth 2.0 for Browser-Based Applications* (borrador): https://datatracker.ietf.org/doc/draft-ietf-oauth-browser-based-apps/

---
⬅️ [[CN 02 - Patrones de contenedores y Kubernetes]] · ➡️ [[CN 04 - Mensajería y arquitectura orientada a eventos]]
