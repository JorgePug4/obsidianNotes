---
tags: [azure-service-bus, cheatsheet, entrevistas, nivel/fundamentos]
---
# 07 - Entrevistas y cheat sheet

> Repaso rápido

## Contenido
- [[#Service Bus Cheat Sheet]]
- [[#Entrevistas - Basicas]]

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
Basic (solo queues), Standard (topics, sessions, transacciones, DD; pago por operación), Premium (capacidad dedicada, 100 MB, VNet, geo-replication). → [[Tiers y facturacion]]

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
Siguiente nivel: [[Entrevistas - Intermedias]] (🚧)
