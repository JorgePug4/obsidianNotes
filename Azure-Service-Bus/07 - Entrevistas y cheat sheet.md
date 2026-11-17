---
tags: [azure-service-bus, cheatsheet, entrevistas, nivel/fundamentos, nivel/senior, nivel/architect]
---
# 07 - Entrevistas y cheat sheet

> Repaso rápido

## Contenido
- [[#Service Bus Cheat Sheet]]
- [[#Entrevistas - Basicas]]
- [[#Entrevistas - Intermedias]]
- [[#Entrevistas - Avanzadas]]
- [[#Entrevistas - Architect]]
- [[#Cheat sheets adicionales]]

## Service Bus Cheat Sheet
### Garantías en una línea
Peek-Lock = **at-least-once** → consumidor **idempotente**. Orden solo con **Sessions**. Sin 2PC con tu BD → **Outbox**.

### Límites y defaults (verifica en [quotas](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-quotas))
| Cosa | Valor |
|---|---|
| Mensaje Standard | 256 KB |
| Mensaje Premium | 1 MB default, hasta 100 MB (AMQP, sin batch) |
| LockDuration | default 1 min, máx. 5 min |
| MaxDeliveryCount | default 10 |
| Ventana Duplicate Detection | default 10 min, máx. 7 días |
| Tamaño entidad | 1–5 GB Standard (×16 particionada), hasta 80 GB Premium |
| Messaging Units Premium | 1, 2, 4, 8, 16 |
| Facturación Standard | por operación (bloques de 64 KB) + cuota base |
| Retiro SBMP / SDKs legacy | 30-sep-2026 |

### Inmutables tras crear
`RequiresSession` · `RequiresDuplicateDetection` · `EnablePartitioning`

### Settlement
| | Borra | DeliveryCount | Uso |
|---|---|---|---|
| Complete | ✔ | — | éxito / ya procesado |
| Abandon | ✘ | +1 | transitorio (sin backoff) |
| Defer | ✘ | — | fuera de orden; guarda SequenceNumber |
| DeadLetter | → DLQ | — | permanente |

### Rutas
- DLQ: `<queue>/$DeadLetterQueue` · `<topic>/subscriptions/<sub>/$DeadLetterQueue`
- SDK: `SubQueue.DeadLetter`

### .NET esencial
```csharp
// DI
services.AddAzureClients(b => {
    b.AddServiceBusClientWithNamespace("ns.servicebus.windows.net");
    b.UseCredential(new DefaultAzureCredential());
});
// Enviar
await sender.SendMessageAsync(new ServiceBusMessage(BinaryData.FromObjectAsJson(x)) { MessageId = id });
// Batch
using var batch = await sender.CreateMessageBatchAsync(); batch.TryAddMessage(m);
// Programar
long seq = await sender.ScheduleMessageAsync(m, DateTimeOffset.UtcNow.AddMinutes(5));
// Processor
var p = client.CreateProcessor("q", new() { MaxConcurrentCalls = 8, AutoCompleteMessages = false });
p.ProcessMessageAsync += async a => { /*...*/ await a.CompleteMessageAsync(a.Message); };
p.ProcessErrorAsync += a => { log(a.Exception); return Task.CompletedTask; };
await p.StartProcessingAsync();
```

### Roles RBAC
`Azure Service Bus Data Sender` · `Azure Service Bus Data Receiver` · `Azure Service Bus Data Owner` (evítalo en apps)

### Métricas a vigilar
ActiveMessages · DeadletteredMessages · ThrottledRequests · ServerErrors · UserErrors · IncomingMessages vs OutgoingMessages · (Premium) CPU y memoria por namespace

### Anti-patrones
`new ServiceBusClient` por request · ReceiveAndDelete para datos críticos · DLQ sin alerta · Prefetch alto con handler lento · Retry SDK × Polly × MaxDeliveryCount sin calcular · Confiar en DD en lugar de idempotencia · Publicar tras `SaveChanges` sin outbox

---

## Entrevistas - Basicas
*Entrevistas — Preguntas básicas (20)*

> Formato: respuesta corta que darías en voz alta + **el razonamiento** que demuestra comprensión. Tápate la respuesta y contesta primero.

**1. ¿Qué es Azure Service Bus?**
Broker de mensajería empresarial gestionado con queues y topics sobre AMQP.
*Razonamiento*: menciona para qué sirve (desacoplar en tiempo y carga), no solo qué es. → [[01 - Fundamentos y arquitectura#Que es Azure Service Bus|Que es Azure Service Bus]]

**2. ¿Por qué usar Service Bus en vez de HTTP entre servicios?**
Desacopla disponibilidad: el emisor no depende de que el receptor esté vivo; absorbe picos; reintentos y DLQ gestionados.
*Razonamiento*: menciona el coste — consistencia eventual y depuración más difícil. → [[01 - Fundamentos y arquitectura#Comunicacion sincrona vs asincrona|Comunicacion sincrona vs asincrona]]

**3. ¿Diferencia entre Queue y Topic?**
Queue: un mensaje lo procesa un consumidor. Topic: cada subscription recibe copia.
*Razonamiento*: las instancias de un mismo servicio compiten dentro de **una** subscription. → [[01 - Fundamentos y arquitectura#Namespace y entidades|Namespace y entidades]]

**4. ¿Qué es una Subscription?**
Una queue virtual colgada de un topic, con sus propias reglas, lock, MaxDeliveryCount y DLQ.

**5. ¿Qué es Peek-Lock?**
Modo en que el mensaje se bloquea (no se borra) al recibirlo, hasta que el consumidor hace settlement o expira el lock.
*Razonamiento*: consecuencia → at-least-once → idempotencia. → [[02 - Queues y Topics#Peek-Lock vs Receive-and-Delete|Peek-Lock vs Receive-and-Delete]]

**6. ¿Diferencia con Receive-and-Delete?**
Se borra al entregar; si el consumidor muere, se pierde. At-most-once.

**7. ¿Qué ocurre si un consumidor falla a mitad de procesar?**
El lock expira, `DeliveryCount` aumenta y el mensaje se reentrega. El trabajo parcial hecho no se revierte. → [[01 - Fundamentos y arquitectura#Ciclo de vida de un mensaje|Ciclo de vida de un mensaje]]

**8. ¿Qué es la DLQ?**
Subqueue donde van los mensajes que superan `MaxDeliveryCount`, expiran (si está activado) o se envían explícitamente. No se vacía sola. → [[03 - Confiabilidad y manejo de errores#Dead Letter Queue|Dead Letter Queue]]

**9. ¿Qué es `MaxDeliveryCount`?**
Número máximo de entregas con lock antes de mover a DLQ; por defecto 10.

**10. ¿Qué es el TTL de un mensaje?**
Tiempo tras el cual expira. Se aplica el menor entre el del mensaje y el default de la entidad. Por defecto el expirado se **borra**. → [[02 - Queues y Topics#TTL y expiracion|TTL y expiracion]]

**11. ¿Qué son Complete, Abandon, Defer y DeadLetter?**
Las cuatro formas de settlement. Abandon devuelve **sin espera**. → [[02 - Queues y Topics#Message Settlement|Message Settlement]]

**12. ¿Qué garantía de entrega ofrece Service Bus?**
At-least-once (en Peek-Lock). No exactly-once para efectos externos. → [[03 - Confiabilidad y manejo de errores#At-least-once delivery|At-least-once delivery]]

**13. ¿Qué es la idempotencia y por qué importa?**
Procesar N veces = procesar 1 vez. Porque habrá duplicados. → [[03 - Confiabilidad y manejo de errores#Idempotent Consumer|Idempotent Consumer]]

**14. ¿Qué es un namespace?**
Contenedor de entidades y unidad de configuración (tier, red, capacidad, autenticación).

**15. ¿Qué tiers existen?**
Basic (solo queues), Standard (topics, sessions, transacciones, DD; pago por operación), Premium (capacidad dedicada, 100 MB, VNet, geo-replication). → [[09 - Produccion y operacion#Tiers y facturacion|Tiers y facturacion]]

**16. ¿Tamaño máximo de mensaje?**
256 KB en Standard; en Premium 1 MB por defecto y hasta 100 MB con AMQP. Para más: claim-check.

**17. ¿Qué es un mensaje programado?**
Mensaje que se hace visible en `ScheduledEnqueueTime`; `ScheduleMessageAsync` devuelve un `SequenceNumber` para cancelarlo. → [[02 - Queues y Topics#Scheduled Messages|Scheduled Messages]]

**18. ¿Qué clases principales tiene el SDK .NET?**
`ServiceBusClient`, `ServiceBusSender`, `ServiceBusReceiver`, `ServiceBusProcessor`, `ServiceBusMessage`, `ServiceBusMessageBatch`.
*Razonamiento*: añade que el client es singleton y thread-safe. → [[04 - NET y entorno local#SDK moderno de NET|SDK moderno de NET]]

**19. ¿Service Bus, Event Grid o Event Hubs?**
Mensajes de negocio → Service Bus. Eventos discretos/reacciones → Event Grid. Streams/telemetría → Event Hubs. → [[01 - Fundamentos y arquitectura#Service Bus vs Storage Queue vs Event Grid vs Event Hubs|Service Bus vs Storage Queue vs Event Grid vs Event Hubs]]

**20. ¿Cómo te autenticas desde una app en Azure?**
Managed Identity + rol RBAC (`Azure Service Bus Data Sender`/`Receiver`), sin connection strings. → [[05 - Seguridad y performance#Seguridad con Entra ID y RBAC|Seguridad con Entra ID y RBAC]]

---

## Entrevistas - Intermedias
*Entrevistas — Preguntas intermedias (20)*

> Nivel: desarrollador que ya ha puesto Service Bus en producción. Se espera **un escenario de fallo** en cada respuesta.

**21. ¿Qué pasa si el handler tarda más que `LockDuration`?**
Si no hay renovación, el lock expira, el mensaje se entrega a otro consumidor y el `Complete` original falla con `MessageLockLost`. El processor renueva solo hasta `MaxAutoLockRenewalDuration` (5 min por defecto). → [[02 - Queues y Topics#Message Lock y Lock Renewal|Message Lock y Lock Renewal]]

**22. ¿Por qué un `PrefetchCount` alto puede provocar duplicados?**
Los mensajes se bloquean al entrar en el buffer local; el lock corre mientras esperan. Si esperan más que el lock, expiran antes de procesarse. → [[05 - Seguridad y performance#Performance tuning|Performance tuning]]

**23. Abandon vs Defer vs DeadLetter.**
Abandon: devuelve sin espera, `DeliveryCount++`. Defer: aparta el mensaje; solo se recupera por `SequenceNumber`. DeadLetter: a la DLQ con razón. *Razonamiento*: Abandon no es un retry con backoff. → [[02 - Queues y Topics#Message Settlement|Message Settlement]]

**24. ¿Cómo implementas reintentos con backoff si Abandon es inmediato?**
Completar + reprogramar una copia con `ScheduleMessageAsync` y un contador en `ApplicationProperties`, en una transacción. → [[03 - Confiabilidad y manejo de errores#Retry Policies|Retry Policies]]

**25. ¿Cómo se multiplican los reintentos y por qué importa?**
SDK × Polly × `MaxDeliveryCount`. Con 3 × 5 × 10 = 150 llamadas por mensaje a una dependencia caída: retry storm. → [[03 - Confiabilidad y manejo de errores#Retry Policies|Retry Policies]]

**26. ¿Qué es un mensaje venenoso y cómo lo tratas?**
Uno que siempre falla (dato inválido, bug). Clasificar: error permanente → DeadLetter inmediato con razón; no gastar los 10 intentos. → [[03 - Confiabilidad y manejo de errores#Poison Messages|Poison Messages]]

**27. ¿Cómo garantizas orden?**
Sessions con `SessionId` = clave de negocio y `MaxConcurrentCallsPerSession = 1`. Orden por clave, no global. → [[03 - Confiabilidad y manejo de errores#Sessions|Sessions]]

**28. ¿Qué ocurre si envías un mensaje sin `SessionId` a una entidad con sesiones?**
El envío se rechaza. Vía auto-forward acaba en DLQ con razón `Session ID is null`. → [[10 - Troubleshooting#Troubleshooting - Session lock problems|Session lock problems]]

**29. Explica duplicate detection y sus límites.**
Descarta envíos con el mismo `MessageId` dentro de la ventana (10 min por defecto, hasta 7 días). No cubre reentregas por lock perdido o crash. Inmutable tras crear. → [[03 - Confiabilidad y manejo de errores#Duplicate Detection|Duplicate Detection]]

**30. ¿Cómo haces idempotente un consumidor que escribe en SQL?**
Tabla `ProcessedMessages(Consumer, MessageId)` con PK, en la misma transacción que el efecto. → [[03 - Confiabilidad y manejo de errores#Idempotent Consumer|Idempotent Consumer]]

**31. ¿Cómo integras un processor en ASP.NET Core?**
`BackgroundService` con `StartProcessingAsync` en `ExecuteAsync`, `StopProcessingAsync` en `StopAsync`, cliente singleton vía `AddAzureClients`, handlers con scope de DI por mensaje. → [[04 - NET y entorno local#Integracion con ASP.NET Core|Integracion con ASP.NET Core]]

**32. ¿Qué es graceful shutdown y por qué afecta a la corrección?**
Dejar de recibir y terminar lo que está en curso antes de salir. Sin él, cada despliegue genera reentregas (y duplicados si no hay idempotencia). → [[09 - Produccion y operacion#Casos reales|Casos reales]]

**33. Correlation filter vs SQL filter.**
Correlation: igualdad sobre propiedades, evaluación barata, hasta 100 000 por topic. SQL: expresiones, más caro, hasta 2 000 por topic. Ninguno ve el body. → [[02 - Queues y Topics#Filters y Rules|Filters y Rules]]

**34. ¿Qué pasa si borras la regla `$Default` y no añades otra?**
La subscription no recibe nada. Los mensajes no se "pierden": nunca llegan. → [[10 - Troubleshooting#Troubleshooting - Mensajes perdidos|Mensajes perdidos]]

**35. ¿Qué es auto-forwarding y un uso típico?**
Encadenar una queue/subscription a otra entidad del mismo namespace. Uso: fan-in, o DLQ de varias subscriptions a una queue de incidencias. Máximo 4 saltos. → [[02 - Queues y Topics#Auto-forwarding|Auto-forwarding]]

**36. ¿Qué operaciones entran en una transacción de Service Bus?**
Envíos, programaciones, settlement y session state. **No** la recepción. Mismo namespace, 2 min de timeout, hasta 100 mensajes. → [[03 - Confiabilidad y manejo de errores#Transactions|Transactions]]

**37. ¿Por qué no debes crear un `ServiceBusClient` por petición?**
Cada uno abre una conexión AMQP (socket). Límite 5 000 conexiones por namespace y agotamiento de sockets. → [[10 - Troubleshooting#Troubleshooting - Connection failures|Connection failures]]

**38. ¿Qué métricas alertas siempre?**
DLQ > 0, `ActiveMessages` creciendo, edad del mensaje más viejo (propia), throttling, `Size` cerca del máximo. → [[09 - Produccion y operacion#Metricas clave|Metricas clave]]

**39. ¿Cómo reprocesas la DLQ de forma segura?**
Clasificar por razón, arreglar la causa, reenviar filtrado y con límite, Send + Complete en transacción, consumidores idempotentes, y tener en cuenta la ventana de DD. → [[03 - Confiabilidad y manejo de errores#Dead Letter Queue|Dead Letter Queue]]

**40. ¿Qué no puedes probar en el emulador?**
Entra ID/Managed Identity, VNet/Private Endpoints, métricas de Azure Monitor, Premium (tamaños grandes, geo-replication), SLA y rendimiento real. → [[04 - NET y entorno local#Emulador local|Emulador local]]

---

## Entrevistas - Avanzadas
*Entrevistas — Preguntas avanzadas (20)*

> Nivel senior. Se espera que discutas trade-offs y que digas "depende de…" **con** los factores concretos.

**41. Diseña la publicación de un evento tras guardar en SQL.**
Outbox en la misma transacción, relay con `READPAST`, `MessageId` = Id del outbox, DD activado, consumidores idempotentes. Explica qué duplicado sigue existiendo. → [[08 - Patrones de arquitectura#Outbox Pattern|Outbox Pattern]]

**42. "Usamos `TransactionScope` con EF Core y Service Bus, ya es atómico."**
Falso. Service Bus no participa en 2PC/MSDTC. O lanza excepción, o da falsa atomicidad. → [[03 - Confiabilidad y manejo de errores#Transactions|Transactions]]

**43. ¿Cómo funciona send-via y qué implica?**
Con `EnableCrossEntityTransactions`, los envíos pasan por la entidad de la primera operación y luego se transfieren. Importa el orden (recibir primero), se usa un cliente dedicado y cuenta como salto. → [[03 - Confiabilidad y manejo de errores#Transactions|Transactions]]

**44. Un `MessageLockLost` ocurre antes de que expire el lock. ¿Cómo es posible?**
El link AMQP se desconectó (red, timeout de inactividad de 10 min) entre la recepción y el settlement: el lock no puede usarse en el nuevo link. → [[10 - Troubleshooting#Troubleshooting - Lock expired|Lock expired]]

**45. La DLQ sube con `MaxDeliveryCountExceeded` pero el handler no lanza excepciones.**
Locks perdidos: prefetch, handler largo sin renovación, thread pool starvation, reloj desajustado, o settlement tras cerrar el receiver. → [[10 - Troubleshooting#Troubleshooting - Dead-letter increasing|Dead-letter increasing]]

**46. ¿Cómo dimensionas `MaxConcurrentCalls` y réplicas para 5 000 msg/s?**
Throughput ≈ instancias × concurrencia / duración. Con 100 ms: 500 slots concurrentes. Pero el límite real es la dependencia y los 5 000 receive requests por entidad; mide antes. → [[05 - Seguridad y performance#Performance tuning|Performance tuning]]

**47. Hot session: ¿qué es y cómo la evitas?**
Una clave de sesión con mucho más tráfico que el resto que se procesa en serie. Clave más fina; separar clientes grandes; aceptar que el orden por una clave es un límite de throughput. → [[03 - Confiabilidad y manejo de errores#Sessions|Sessions]]

**48. ¿Cómo implementas un circuit breaker en un consumidor?**
Pausar el processor (`StopProcessingAsync` fuera del handler) cuando la dependencia está caída, y reanudar tras un tiempo. Los mensajes esperan en la cola en lugar de agotar reintentos. → [[08 - Patrones de arquitectura#Circuit Breaker|Circuit Breaker]]

**49. Particionado: ventajas y costes.**
Paraleliza el broker y aumenta la disponibilidad/tamaño (80 GB en Standard). Pero no hay orden global, los batches y transacciones deben compartir partición (`PartitionKey`/`SessionId`) y DD funciona por partición. Inmutable. → [[05 - Seguridad y performance#Performance tuning|Performance tuning]]

**50. ¿Cómo versionas un evento con 6 consumidores que no controlas?**
Cambios aditivos + tolerant reader; para cambios incompatibles, nuevo tipo y publicación dual temporal; versión en `ApplicationProperties`; tests de contrato. → [[08 - Patrones de arquitectura#Versionado de mensajes|Versionado de mensajes]]

**51. Mensajes de 5 MB: ¿Premium con mensajes grandes o claim check?**
Claim check si necesitas Standard, batching o Geo-Replication (no soporta mensajes grandes). Premium si quieres simplicidad y aceptas peor throughput. → [[08 - Patrones de arquitectura#Claim Check Pattern|Claim Check Pattern]]

**52. ¿Cómo propagas la traza a través del outbox?**
Guardar `traceparent` en la fila y restaurarlo (o fijar `Diagnostic-Id`) al publicar. Si no, la traza termina en el `INSERT`. → [[09 - Produccion y operacion#Observabilidad con Application Insights|Observabilidad con Application Insights]]

**53. ¿Cómo eliminas por completo los secretos?**
Managed Identity (user-assigned) + roles con ámbito de entidad + `disableLocalAuth` + entidades por IaC (la app no tiene `Manage`). → [[05 - Seguridad y performance#Seguridad con Entra ID y RBAC|Seguridad con Entra ID y RBAC]]

**54. Private Endpoint configurado y el worker da timeout. ¿Qué miras primero?**
DNS: desde la VNet, `nslookup` debe resolver a IP privada. Zona `privatelink` vinculada a la VNet. → [[10 - Troubleshooting#Troubleshooting - Connection failures|Connection failures]]

**55. ¿Qué indica `ThrottledRequests` en Standard y en Premium?**
Standard: créditos de operaciones del namespace compartido (1 000/s). Premium: CPU/memoria de las MUs o checkpoints pendientes (`MessagingErrorSubCode`). → [[10 - Troubleshooting#Troubleshooting - Throttling y ServerBusy|Throttling y ServerBusy]]

**56. Un proyector CQRS recibe eventos desordenados. ¿Cómo lo haces correcto?**
Versión del agregado en el evento y en la proyección; descartar versiones ≤ la actual; o sesiones por agregado. → [[08 - Patrones de arquitectura#CQRS|CQRS]]

**57. Request/response sobre Service Bus: ¿cómo y cuándo?**
`ReplyTo` + `ReplyToSessionId` y una queue de respuestas con sesiones; TTL corto. Solo si HTTP no es viable (p. ej. el receptor está tras un firewall sin entrada). → [[08 - Patrones de arquitectura#Commands vs Events|Commands vs Events]]

**58. ¿Cómo pruebas un consumidor en CI?**
Lógica del handler aislada (unit), emulador en contenedor para integración, y pruebas de duplicado y de orden explícitas. Lo que dependa de Entra ID o red, en un entorno Azure. → [[04 - NET y entorno local#Emulador local|Emulador local]]

**59. Diferencia entre `UserErrors` y `ServerErrors`.**
User: errores del cliente (4xx), incluido `MessageLockLost`. Server: errores del servicio. Una subida de User suele ser tuya. → [[09 - Produccion y operacion#Metricas clave|Metricas clave]]

**60. ¿Cuándo usarías un framework (MassTransit, NServiceBus, Wolverine) y qué riesgo tiene?**
Muchos servicios, necesidad de sagas y outbox estándar. Riesgos: abstracción que oculta settlement, topología automática que choca con IaC/least privilege, cambios de licencia (MassTransit v9 comercial). → [[08 - Patrones de arquitectura#Frameworks de mensajeria NET|Frameworks de mensajeria NET]]

---

## Entrevistas - Architect
*Entrevistas — Preguntas de Software Architect (20)*

> Aquí no hay una respuesta correcta única. Se evalúa: qué preguntas haces antes de decidir, qué alternativas consideras y qué riesgos asumes conscientemente.

**61. ¿Deberíamos usar Service Bus aquí? (sistema CRUD con un consumidor)**
Probablemente no. Pregunta por requisitos de disponibilidad, picos, número de consumidores y si el usuario necesita la respuesta en la misma petición. La mensajería añade consistencia eventual y operación. → [[08 - Patrones de arquitectura#Event-Driven Architecture|Event-Driven Architecture]]

**62. Service Bus, Event Grid, Event Hubs o Kafka para: pedidos, cambios en Blob Storage, telemetría IoT.**
Pedidos → Service Bus. Blob → Event Grid. IoT → Event Hubs/Kafka. Justifica con retención, replay, DLQ por mensaje y volumen. → [[09 - Produccion y operacion#Service Bus vs otras tecnologias|Service Bus vs otras tecnologias]]

**63. Coreografía u orquestación para una saga de 7 pasos con timeouts?**
Orquestación: visibilidad del estado, timeouts y ramas. Coreografía para flujos cortos y estables. → [[08 - Patrones de arquitectura#Saga Pattern|Saga Pattern]]

**64. Topología: ¿un topic por bounded context o por tipo de evento?**
Por contexto: menos entidades, filtros por `Subject`. Por tipo: permisos y escalado más finos. Ten en cuenta los límites (2 000 subscriptions/topic, filtros) y quién es dueño de cada subscription. → [[08 - Patrones de arquitectura#Publisher Subscriber|Publisher Subscriber]]

**65. ¿Un namespace por servicio, por dominio o uno compartido?**
Compartido: barato (Standard: una cuota base por suscripción de Azure) pero radio de impacto común (throttling, cuotas de conexiones). Por dominio: aislamiento y costes asignables. En Premium, cada namespace son MUs: consolidar suele compensar. → [[09 - Produccion y operacion#Tiers y facturacion|Tiers y facturacion]]

**66. Standard o Premium para un sistema de pagos con requisito de red privada?**
Premium: Private Endpoints solo en Premium. Además aislamiento y latencia predecible. → [[05 - Seguridad y performance#Private Endpoints|Private Endpoints]]

**67. RPO 0 ante caída regional: ¿qué propones y qué cuesta?**
Geo-Replication síncrona: mayor latencia y dependencia de dos regiones, el doble de MUs y transferencia por GB. Además, los consumidores y BDs también deben ser multirregión. → [[09 - Produccion y operacion#Geo-Replication|Geo-Replication]]

**68. Geo-DR vs Geo-Replication.**
Geo-DR: solo metadatos, alias, failover de un sentido. Geo-Replication: datos y estado, promoción planned/forced. No combinables. → [[09 - Produccion y operacion#Geo-Replication|Geo-Replication]]

**69. ¿Cómo garantizas consistencia entre 4 servicios sin transacciones distribuidas?**
Outbox + inbox + idempotencia + sagas con compensación + reconciliación. Y diseño de UX para estados intermedios. → [[08 - Patrones de arquitectura#Eventual Consistency|Eventual Consistency]]

**70. ¿Quién es responsable de la DLQ en una organización con 10 equipos?**
El equipo dueño del consumidor. Alertas enrutadas a ese equipo, runbook, SLA de reprocesado y herramienta común de reprocesado. La DLQ sin dueño es pérdida de datos diferida. → [[03 - Confiabilidad y manejo de errores#Dead Letter Queue|Dead Letter Queue]]

**71. ¿Contratos compartidos en un NuGet común o duplicados?**
Trade-off entre tipado y autonomía de despliegue. Posición defendible: paquete solo con records, semver, tolerant reader y tests de contrato. → [[08 - Patrones de arquitectura#Versionado de mensajes|Versionado de mensajes]]

**72. ¿Qué te ata a Service Bus y cómo lo mitigas?**
Sessions, SQL filters, transacciones, scheduled messages, defer. Mitigación: puertos/adaptadores propios o framework; pero abstraer en exceso pierde las garantías que justifican elegirlo. → [[08 - Patrones de arquitectura#Frameworks de mensajeria NET|Frameworks de mensajeria NET]]

**73. ¿Cómo migrarías un sistema con `Microsoft.Azure.ServiceBus` tras el retiro?**
Inventario, tabla de equivalencias, migrar por servicio (el broker es el mismo, los mensajes son compatibles vía AMQP), cuidado con `UserProperties`→`ApplicationProperties` y serialización del body legacy. → [[00 - Indice y ruta de estudio#Estado del producto y cambios recientes|Estado del producto y cambios recientes]]

**74. ¿Cómo diseñas para picos de 50× durante 1 hora (Black Friday)?**
La cola absorbe; escala consumidores con KEDA hasta el límite de la dependencia; verifica tamaño de entidad (horas × tasa × tamaño), Premium con autoscale de MUs, prioridades separando queues críticas. → [[08 - Patrones de arquitectura#Competing Consumers|Competing Consumers]]

**75. ¿Prioridades de mensajes?**
Service Bus no tiene prioridad por mensaje. Queues separadas (alta/normal) con más consumidores en la alta, o subscriptions con filtros por prioridad. → [[02 - Queues y Topics#Filters y Rules|Filters y Rules]]

**76. ¿Multi-tenant: entidad por tenant o compartida?**
Compartida + `TenantId` en propiedades: simple pero con *noisy neighbor*. Por tenant: aislamiento, más entidades (límites 1 000/MU en Premium) y automatización. Sesiones por tenant si hace falta orden y equidad. → [[02 - Queues y Topics#Topics y Subscriptions|Topics y Subscriptions]]

**77. ¿Qué SLO definirías para un flujo asíncrono?**
Latencia de extremo a extremo (p99 de `now - EnqueuedTime` al terminar), tasa de DLQ, tiempo de reprocesado. Medidos con métricas propias, no solo de plataforma. → [[09 - Produccion y operacion#Metricas clave|Metricas clave]]

**78. ¿Cómo evalúas el coste de una arquitectura con mucho fan-out?**
Cuenta operaciones: 1 envío + N recepciones (+ N completes). Fan-out ancho en Standard puede superar a Premium; calcula con la calculadora. → [[09 - Produccion y operacion#Tiers y facturacion|Tiers y facturacion]]

**79. El negocio pide "exactly-once". ¿Qué respondes?**
Que no existe para efectos externos; se ofrece *effectively-once* mediante idempotencia (inbox, idempotency keys) y se documenta qué efectos (emails) pueden duplicarse. → [[03 - Confiabilidad y manejo de errores#At-least-once delivery|At-least-once delivery]]

**80. Presenta los riesgos de tu diseño de proyecto final en 2 minutos.**
Ejercicio: duplicados (y qué los neutraliza), DLQ (quién y cómo), orden (dónde importa), región (qué pasa si cae), coste (qué lo dispara), y la decisión que más te costaría revertir. → [[06 - Laboratorios#Proyecto final|Proyecto final]]

---

## Cheat sheets adicionales

### Cheat - Opciones del SDK
| Opción | Default | Cuándo tocarla |
|---|---|---|
| `ServiceBusProcessorOptions.MaxConcurrentCalls` | 1 | Siempre, según la dependencia |
| `PrefetchCount` | 0 | Handlers rápidos y alto volumen |
| `AutoCompleteMessages` | true | Ponlo a `false` para controlar el settlement |
| `MaxAutoLockRenewalDuration` | 5 min | Handlers largos |
| `ReceiveMode` | PeekLock | ReceiveAndDelete solo si la pérdida es aceptable |
| `SubQueue` | None | `DeadLetter` / `TransferDeadLetter` |
| `ServiceBusSessionProcessorOptions.MaxConcurrentSessions` | 8 | Paralelismo entre sesiones |
| `MaxConcurrentCallsPerSession` | 1 | Déjalo en 1 si el orden importa |
| `SessionIdleTimeout` | [Verificar default en tu versión] | Muchas sesiones con pocos mensajes |
| `ServiceBusClientOptions.TransportType` | AmqpTcp | `AmqpWebSockets` con proxies/443 |
| `EnableCrossEntityTransactions` | false | Transacciones entre entidades (cliente dedicado) |
| `RetryOptions` (Mode / MaxRetries / Delay / MaxDelay / TryTimeout) | Exponential / 3 / 0,8 s / 1 min / 60 s | Rara vez; no bajes a 0 |

### Cheat - Excepciones
`IsTransient` → deja que el SDK reintente · `MessageLockLost` → no reintentes el complete, idempotencia · `ServiceBusy` → throttling · `QuotaExceeded` → entidad llena/conexiones · `MessagingEntityNotFound` → nombre/borrado · `MessageSizeExceeded` → claim check · `UnauthorizedAccessException` → rol/ámbito/identidad. Detalle: [[10 - Troubleshooting#Mapa de ServiceBusFailureReason|Mapa de ServiceBusFailureReason]]

### Cheat - Azure CLI
```bash
az servicebus namespace create -g $RG -n $NS --sku Standard
az servicebus queue create -g $RG --namespace-name $NS -n orders \
  --max-delivery-count 10 --lock-duration PT1M --enable-dead-lettering-on-message-expiration true
az servicebus topic create -g $RG --namespace-name $NS -n order-events --enable-duplicate-detection true
az servicebus topic subscription create -g $RG --namespace-name $NS --topic-name order-events -n billing
az servicebus topic subscription rule create -g $RG --namespace-name $NS --topic-name order-events \
  --subscription-name billing -n paid --filter-sql-expression "sys.Label = 'OrderPaid'"
az servicebus queue show -g $RG --namespace-name $NS -n orders --query countDetails
az servicebus namespace failover --namespace-name $NS -g $RG --primary-location <region>   # Geo-Replication
```

### Cheat - KQL y alertas
```kusto
AzureMetrics | where ResourceProvider == "MICROSOFT.SERVICEBUS" and MetricName == "DeadletteredMessages" | summarize max(Maximum) by bin(TimeGenerated, 5m)
AZMSOperationalLogs | where EventName has "Delete" | project TimeGenerated, EventName, Caller, EventProperties
AZMSRuntimeAuditLogs | where Status == "Failure" | summarize sum(Count) by ActivityName, AuthType, AuthKey   // Premium
AZMSDiagnosticErrorLogs | summarize sum(ErrorCount) by OperationResult, EntityName, ErrorMessage
```
Recuerda: las dimensiones (`EntityName`) no se exportan en `AzureMetrics`; para alertas por entidad usa alertas de métricas.

### Cheat - Patrones (problema → patrón)
| Problema | Patrón |
|---|---|
| Guardar en BD y publicar | [[08 - Patrones de arquitectura#Outbox Pattern|Outbox]] |
| Procesar una sola vez el efecto | [[08 - Patrones de arquitectura#Inbox Pattern|Inbox]] / idempotencia |
| Proceso entre servicios | [[08 - Patrones de arquitectura#Saga Pattern|Saga]] |
| Dependencia caída | [[08 - Patrones de arquitectura#Circuit Breaker|Circuit breaker]] + backoff |
| Payload grande | [[08 - Patrones de arquitectura#Claim Check Pattern|Claim check]] |
| Escalar consumo | [[08 - Patrones de arquitectura#Competing Consumers|Competing consumers]] |
| Varios interesados | [[08 - Patrones de arquitectura#Publisher Subscriber|Pub/Sub]] |
| Orden por clave | [[03 - Confiabilidad y manejo de errores#Sessions|Sessions]] |
| Lecturas distintas de escrituras | [[08 - Patrones de arquitectura#CQRS|CQRS]] |

### Cheat - Elección de tecnología
Mensaje de negocio con DLQ/reintentos/orden → **Service Bus** · Reacción a cambios de recursos Azure → **Event Grid** · Telemetría, streams, replay → **Event Hubs / Kafka** · Cola simple y barata, > 80 GB → **Storage Queue** · On-prem / multi-cloud → **RabbitMQ**

### Cheat - Seguridad
MI user-assigned · `Data Sender`/`Data Receiver` con ámbito de entidad · nunca `Data Owner` en apps · `disableLocalAuth` · Private Endpoint + DNS privado (Premium) · TLS 1.2 · IaC crea entidades · datos sensibles fuera de propiedades

### Cheat - Checklist de producción
- [ ] Cliente singleton, `DisposeAsync` al apagar, graceful shutdown probado
- [ ] Consumidores idempotentes, `MessageId` determinista, outbox donde haya dual write
- [ ] Errores clasificados: permanente → DeadLetter con razón; transitorio → backoff
- [ ] Retries calculados (SDK × app × `MaxDeliveryCount`)
- [ ] Alertas: DLQ, Active, edad, throttling, size; runbooks enlazados
- [ ] Trazas de extremo a extremo
- [ ] Propiedades inmutables decididas (sesiones, DD, particionado)
- [ ] Coste estimado con la calculadora; tier justificado
- [ ] Plan regional (zonas / Geo-Replication) ensayado
- [ ] Herramienta y procedimiento de reprocesado de DLQ

