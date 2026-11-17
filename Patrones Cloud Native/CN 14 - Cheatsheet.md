---
tags: [cloud-native, patrones, cheatsheet, repaso]
aliases: [Cheatsheet patrones cloud native, Chuleta cloud native]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 14 - Cheatsheet de patrones cloud native

> [!tip] Formato
> **Patrón** — problema → solución en una línea. ⚠️ = coste o riesgo principal. 📍 = capítulo.

## 🧱 Fundamentos · 📍 [[CN 01 - Fundamentos Cloud Native|01]]
- **12 factores** — app apta para la nube → config externa, stateless, logs a stdout, arranque/apagado rápido, build/release/run separados.
- **Infraestructura inmutable** — deriva de configuración → reemplazar, no modificar. ⚠️ requiere pipeline automatizado.
- **Cattle, not pets** — servidores irreemplazables → instancias idénticas y desechables.

## 📦 Contenedores y Kubernetes · 📍 [[CN 02 - Patrones de contenedores y Kubernetes|02]]
- **Sidecar** — funcionalidad transversal sin tocar la app → contenedor auxiliar en el Pod. ⚠️ recursos × réplicas.
- **Ambassador** — conexión compleja a servicios externos → proxy de salida en `localhost`.
- **Adapter** — interfaces heterogéneas → sidecar que normaliza hacia fuera (exporter).
- **Init Container** — tareas previas al arranque → contenedores secuenciales que deben terminar con éxito.
- **Health Probe** — saber si vive/está listo → startup, liveness (reinicia), readiness (saca del balanceo). ⚠️ nunca dependencias en liveness.
- **Predictable Demands** — scheduling a ciegas → requests/limits.
- **Managed Lifecycle** — cortes en despliegues → manejar SIGTERM, `preStop`.
- **Operator** — operar software con estado → controller + CRD + bucle de reconciliación. ⚠️ automatiza también los errores.
- **Leader Election** — una sola instancia debe actuar → lease distribuido. ⚠️ split brain → fencing tokens.

## 🌐 Comunicación · 📍 [[CN 03 - Comunicación, enrutamiento y APIs|03]]
- **API Gateway** (Routing · Aggregation · Offloading) — clientes acoplados a muchos servicios → entrada única. ⚠️ sin lógica de negocio; HA.
- **BFF** — clientes con necesidades distintas → un backend por tipo de frontend.
- **Service Discovery** — IPs dinámicas → registro + DNS (K8s Service).
- **Service Mesh** — mTLS/retries/telemetría uniforme → proxies + control plane. ⚠️ coste operativo alto.
- **Async Request-Reply** — operaciones largas tras HTTP → `202` + `Location` + polling/callback.
- **API Composition** — consultas entre servicios → llamar en paralelo y combinar.
- **Anti-Corruption Layer** — modelo ajeno contamina el tuyo → capa de traducción.
- **Strangler Fig** — reescritura big-bang → fachada y migración ruta a ruta.
- **Messaging Bridge** — brokers distintos → puente que consume de uno y publica en otro.

## 📨 Mensajería · 📍 [[CN 04 - Mensajería y arquitectura orientada a eventos|04]]
- **Publisher-Subscriber** — el emisor conoce a todos sus interesados → topic + suscripciones. ⚠️ el evento es un contrato.
- **Competing Consumers** — un consumidor no da abasto → N instancias de la misma cola. ⚠️ duplicados, sin orden.
- **Queue-Based Load Leveling** — picos saturan al consumidor → cola amortiguadora.
- **Priority Queue** — lo urgente espera tras lo trivial → colas por prioridad. ⚠️ inanición.
- **Claim Check** — mensajes demasiado grandes → payload en blob, referencia en el mensaje.
- **Pipes and Filters** — procesamiento monolítico → pasos independientes unidos por colas.
- **Coreografía / Orquestación** — coordinar procesos → eventos sin coordinador / coordinador con comandos.
- **Sequential Convoy** — orden con paralelismo → agrupar por clave (sessions, particiones).
- **Dead Letter Queue** — mensajes venenosos → apartar tras N intentos. ⚠️ alertar siempre.
- **Idempotent Consumer / Inbox** — duplicados at-least-once → registrar IDs procesados en la misma transacción.

## 💾 Datos · 📍 [[CN 05 - Patrones de gestión de datos|05]]
- **Database per Service** — acoplamiento por BD compartida → cada servicio dueño de sus datos. ⚠️ sin JOINs ni ACID global.
- **Saga** — transacción entre servicios → transacciones locales + compensaciones. ⚠️ sin aislamiento. → [[Saga Pattern]]
- **Transactional Outbox** — dual write → evento en la misma transacción + relay. → [[Transactional Outbox]]
- **CQRS** — un modelo malo para leer y escribir → modelos separados. ⚠️ eventual. → [[CQRS (Command Query Responsibility Segregation)|CQRS]]
- **Event Sourcing** — se pierde el historial → los eventos son la verdad. ⚠️ complejidad alta. → [[Event Sourcing]]
- **CDC** — propagar cambios sin tocar la app → leer el log de la BD. ⚠️ esquema interno como contrato.
- **Cache-Aside** — lecturas repetidas lentas → caché gestionada por la app con TTL; invalidar al escribir. ⚠️ stampede, obsoletos.
- **Materialized View** — consultas costosas → vistas precalculadas y desechables.
- **Sharding** — límites de un único almacén → particionar por clave. ⚠️ la clave lo es todo.
- **Index Table** — buscar por campos no clave → tablas de índice secundario.
- **Valet Key** — la API hace de intermediario de ficheros → token temporal de mínimo privilegio (SAS).
- **Static Content Hosting** — servir estáticos desde cómputo → storage + CDN.

## 🛡️ Resiliencia · 📍 [[CN 06 - Resiliencia y tolerancia a fallos|06]]
- **Timeout** — esperas indefinidas → límite por intento y total. *Lo lento mata más que lo caído.*
- **Retry** — fallos transitorios → reintentos limitados, backoff + jitter, sólo idempotentes, una capa. → [[Retry con Backoff Exponencial]]
- **Circuit Breaker** — insistir contra lo que falla → cerrado/abierto/medio abierto, fail fast. → [[Circuit Breaker]]
- **Bulkhead** — un fallo agota recursos compartidos → compartimentar pools. → [[Bulkhead]]
- **Fallback** — error duro → respuesta degradada pensada de antemano (caché, por defecto, ocultar).
- **Rate Limiting** — abuso / cliente ruidoso → cuota por clave, `429` + `Retry-After`.
- **Throttling** — sobrecarga → limitar consumo según la carga del sistema.
- **Load Shedding** — todo falla bajo sobrecarga → rechazar parte rápido para servir al resto.
- **Compensating Transaction** — no hay rollback distribuido → deshacer semánticamente. ⚠️ también falla; idempotente.
- **Scheduler Agent Supervisor** — workflows que se quedan a medias → orquestar + ejecutar + supervisar (Durable Functions, Temporal).
- **Pipeline Polly:** rate limiter → timeout total → retry → circuit breaker → timeout por intento.

## 📈 Escalabilidad · 📍 [[CN 07 - Escalabilidad y rendimiento|07]]
- **Stateless** — afinidad a instancias → estado fuera del proceso (incl. Data Protection en ASP.NET Core).
- **Autoscaling** — capacidad fija → HPA (CPU/RPS), KEDA (colas, a 0), Cluster Autoscaler/Karpenter. ⚠️ tarda; escala también las conexiones a la BD.
- **Deployment Stamps** — límites y radio de impacto de un único despliegue → copias completas por grupo de clientes.
- **Geode** — latencia global + región caída → backends activos en todas las regiones con datos replicados. ⚠️ conflictos, coste.
- **CDN** — origen lejano → caché en el edge. ⚠️ nunca datos privados.
- **Compute Resource Consolidation** — infraestructura infrautilizada → agrupar componentes compatibles.

## 🚀 Entrega · 📍 [[CN 08 - Despliegue, configuración y entrega|08]]
- **IaC** — infraestructura manual → código versionado (Terraform, Bicep).
- **GitOps** — push con credenciales potentes, deriva → Git como verdad + reconciliación pull (Argo CD, Flux).
- **Rolling / Blue-Green / Canary / Shadow** — despliegues arriesgados → gradual / conmutación con rollback instantáneo / % creciente con métricas / copia del tráfico.
- **Progressive Delivery** — canary manual → promoción/rollback automáticos por métricas.
- **Feature Flags** — desplegar = lanzar → toggles en runtime. ⚠️ deuda si no se retiran.
- **External Configuration Store** — config dentro del paquete → almacén central (App Configuration), validada al arrancar.
- **Expand and Contract** — cambios de esquema que rompen el rollback → añadir, migrar, cambiar, eliminar.

## 🔐 Seguridad · 📍 [[CN 09 - Seguridad|09]]
- **Zero Trust** — confiar en la red interna → verificar siempre, mínimo privilegio, asumir brecha.
- **Federated Identity** — usuarios en cada app → IdP + OIDC/OAuth (Auth Code + PKCE). Valida `aud`.
- **Gatekeeper** — servicios con datos expuestos → intermediario que valida, sin secretos.
- **Workload Identity / Managed Identity** — secretos para acceder a recursos → identidad de plataforma con tokens cortos.
- **External Secret Store** — secretos en código/config → Key Vault + rotación.
- **mTLS** — tráfico interno legible/suplantable → certificados mutuos con identidad SPIFFE (mesh).
- **Quarantine** — artefactos externos no verificados → escanear y aprobar antes de usar.

## 🔭 Observabilidad · 📍 [[CN 10 - Observabilidad|10]]
- **Log Aggregation** — logs perdidos en contenedores efímeros → stdout estructurado + almacén central.
- **Correlation ID** — no se puede seguir una petición → W3C `traceparent` en HTTP **y** mensajes.
- **Distributed Tracing** — ¿dónde se va el tiempo? → spans encadenados; tail sampling.
- **Health Endpoint Monitoring** — enterarse antes que el usuario → endpoints de salud + pruebas sintéticas.
- **RED / USE / Golden Signals** — qué medir → servicios / recursos / de cara al usuario. Percentiles, no medias.
- **SLO + error budget** — fiabilidad sin criterio → objetivo medible y presupuesto de fallo; alertas por burn rate.

## 🧮 Números útiles
| Dato | Valor |
|---|---|
| 99,9 % al mes | ≈ 43 min de caída |
| 99,99 % al mes | ≈ 4,3 min |
| Disponibilidad compuesta (síncrona) | producto de las disponibilidades: 0,999³ ≈ 99,7 % |
| Ley de Little | concurrencia = throughput × latencia |
| Timeout por defecto de `HttpClient` | 100 s (¡cámbialo!) |
| `terminationGracePeriodSeconds` por defecto | 30 s |
| Service Bus `MaxDeliveryCount` por defecto | 10 |
| Tamaño máximo de mensaje Service Bus | 256 KB (Standard) · hasta 100 MB (Premium) |

## 🧠 Las 5 frases que hay que recordar
1. **Todo falla; lo lento es peor que lo caído.**
2. **At-least-once siempre → idempotencia siempre.**
3. **Cada servicio es dueño de sus datos; la consistencia es eventual.**
4. **Reintenta en una sola capa, con backoff y jitter.**
5. **El patrón mínimo suficiente: cada patrón tiene un coste.**

---
⬅️ [[CN 13 - Preguntas de entrevista]] · 🏠 [[CN 00 - Patrones Cloud Native - Índice|Índice]]
