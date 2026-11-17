---
tags: [azure-service-bus, nivel/architect, nivel/produccion, observabilidad, costes, resiliencia]
---
# 09 - Produccion y operacion

> Métricas, trazas, costes, resiliencia regional, comparativa con otras tecnologías y casos reales.

## Contenido
- [[#Metricas clave]]
- [[#Observabilidad con Application Insights]]
- [[#Tiers y facturacion]]
- [[#Geo-Replication]]
- [[#Service Bus vs otras tecnologias]]
- [[#Casos reales]]

---

## Metricas clave
*Verificado contra [Monitoring data reference](https://learn.microsoft.com/azure/service-bus-messaging/monitor-service-bus-reference) — nov-2026*

### Las que importan (nombre REST entre paréntesis)
| Métrica | Qué indica | Alerta sugerida |
|---|---|---|
| **Active messages** (`ActiveMessages`) | Mensajes listos para entregar. Valor puntual por entidad | Crecimiento sostenido (p. ej. 15 min) o umbral por entidad |
| **Dead-lettered messages** (`DeadletteredMessages`) | Mensajes en DLQ | **> 0** en entidades críticas. Es la alerta más importante |
| **Scheduled messages** (`ScheduledMessages`) | Mensajes programados pendientes | Crecimiento inesperado (bucle de reprogramación) |
| **Incoming / Outgoing messages** | Ritmo de entrada y salida | Outgoing ≈ 0 con Incoming > 0 → consumidor parado |
| **Completed / Abandoned messages** (`CompleteMessage`, `AbandonMessage`) | Resultado del procesamiento | Ratio de abandon alto → fallos transitorios o retry storm |
| **Throttled requests** (`ThrottledRequests`) | Peticiones rechazadas por capacidad. Dimensión `MessagingErrorSubCode`: CPU, Storage, Namespace | > 0 sostenido |
| **Server errors** (`ServerErrors`) | Errores del servicio | Subida sostenida → Resource Health / soporte |
| **User errors** (`UserErrors`) | Errores del cliente: 4xx, **incluye `MessageLockLost`** | Subida → locks perdidos, permisos, entidades inexistentes |
| **Size** (`Size`) | Bytes en la entidad | > 70-80 % del máximo configurado |
| **Active connections** (`ActiveConnections`) | Conexiones AMQP | Cerca de 5 000 o crecimiento lineal (fuga de clientes) |
| **CPU / Memory** (`NamespaceCpuUsage`, `NamespaceMemoryUsage`) — Premium | Uso de las MUs | CPU > 75 % o memoria > 60 % → escalar MUs |
| **Server send latency** (`ServerSendLatency`) | Latencia del servicio al aceptar envíos | Comparar con la latencia que ve el cliente |
| **Replication lag duration** (`ReplicationLagDuration`) — Geo-Replication | RPO en segundos | Ver [[#Geo-Replication]] |

### Lo que la documentación aclara (y suele malinterpretarse)
- `Incoming ≠ Outgoing` **no** indica pérdida: duplicate detection, filtros que no coinciden, TTL, DLQ y auto-forwarding explican la diferencia.
- Las métricas de contadores (`ActiveMessages`, `DeadletteredMessages`…) son **valores puntuales**; agrégalos con Average/Max, no Sum.
- Si el namespace está siendo **throttled**, las consultas de información de entidad pueden devolver los contadores a **0**. Un "0 mensajes" durante throttling no es fiable.
- Las dimensiones (`EntityName`) **no** se exportan a Log Analytics vía diagnostic settings. Para alertas por entidad, usa alertas de métricas (no de logs).

### Lo que falta en las métricas de plataforma
**La antigüedad del mensaje más viejo** no existe como métrica. Es la mejor señal de "consumidor que no da abasto". Obtenerla:
```csharp
// Job periódico (o health check) con permisos de Receiver
var oldest = await receiver.PeekMessageAsync(cancellationToken: ct);
if (oldest is not null)
    meter.Record((DateTimeOffset.UtcNow - oldest.EnqueuedTime).TotalSeconds,
                 new KeyValuePair<string, object?>("entity", entityName));
```
Peek no bloquea ni cambia el `DeliveryCount`. Con sesiones, el peek puede devolver mensajes de cualquier sesión.

### Alertas mínimas por entidad crítica
```bash
az monitor metrics alert create -g rg-sb -n "dlq-orders" \
  --scopes $(az servicebus namespace show -g rg-sb -n sb-orders-prod --query id -o tsv) \
  --condition "max DeadletteredMessages > 0 where EntityName includes orders" \
  --window-size 5m --evaluation-frequency 1m --severity 2 \
  --action $ACTION_GROUP_ID
```
| Alerta | Severidad | Runbook |
|---|---|---|
| DLQ > 0 | 2 | [[10 - Troubleshooting#Troubleshooting - Dead-letter increasing|Dead-letter increasing]] |
| Active creciendo 15 min y Outgoing ≈ 0 | 1 | [[10 - Troubleshooting#Troubleshooting - Messages stuck|Messages stuck]] |
| Mensaje más viejo > SLO | 2 | [[10 - Troubleshooting#Troubleshooting - Consumer slow|Consumer slow]] |
| Throttled > 0 sostenido | 2 | [[10 - Troubleshooting#Troubleshooting - Throttling y ServerBusy|Throttling y ServerBusy]] |
| Size > 80 % | 2 | Consumidor parado o subscription abandonada |
| CPU Premium > 75 % | 3 | Escalar MUs / autoscale |

### Logs (diagnostic settings)
| Categoría | Qué contiene | Tier |
|---|---|---|
| `OperationalLogs` | Operaciones de **gestión** (crear/borrar entidades). Incluye borrados por `AutoDeleteOnIdle` | Todos |
| `RuntimeAuditLogs` | Operaciones del plano de datos agregadas por minuto: identidad, tipo de auth (SAS/Entra), red pública/privada, IP | **Premium** |
| `DiagnosticErrorLogs` | Errores de cliente, throttling y cuota con mensaje detallado | Todos [Verificar disponibilidad por tier en tu región] |
| `VNetAndIPFilteringLogs` | Conexiones aceptadas/rechazadas por reglas de red | Con reglas de red |
| `DataDRLogs` | Lag de Geo-Replication | Premium con Geo-Replication |

Los logs no registran mensajes individuales (volumen excesivo). Para seguir un mensaje concreto: **trazas del cliente** ([[#Observabilidad con Application Insights]]).

### Relación
[[05 - Seguridad y performance#Performance tuning|Performance tuning]] · [[03 - Confiabilidad y manejo de errores#Dead Letter Queue|Dead Letter Queue]] · [[06 - Laboratorios#Lab 11 - Observabilidad|Lab 11 - Observabilidad]]

---

## Observabilidad con Application Insights
### Qué hay que ver
1. **Trazas distribuidas**: la petición HTTP del usuario → envío → recepción → handler → llamadas a BD/APIs, como una sola transacción.
2. **Métricas de aplicación**: duración del handler, mensajes procesados/fallidos por tipo, antigüedad al procesar (`now - EnqueuedTime`).
3. **Logs correlacionados** con `MessageId`, `CorrelationId`, `DeliveryCount`, `SessionId`.

### Cómo se propaga el contexto
El SDK escribe la propiedad **`Diagnostic-Id`** en `ApplicationProperties` al enviar o programar (formato W3C `traceparent`). Al procesar, el span del consumidor se **enlaza** (link) al del productor mediante el span `Message`. No es parent-child directo: en Application Insights lo verás como una transacción enlazada.

### Configuración en .NET (OpenTelemetry + Azure Monitor)
```csharp
// El soporte de ActivitySource del SDK sigue marcado como EXPERIMENTAL en la documentación oficial.
AppContext.SetSwitch("Azure.Experimental.EnableActivitySource", true);

builder.Services.AddOpenTelemetry()
    .UseAzureMonitor()                                  // Azure.Monitor.OpenTelemetry.AspNetCore; usa APPLICATIONINSIGHTS_CONNECTION_STRING
    .WithTracing(t => t.AddSource("Azure.Messaging.ServiceBus.*")
                       .AddSource("Orders.*"));         // tus propias ActivitySource
```
Spans que emite el SDK (selección): `ServiceBusSender.Send`, `ServiceBusSender.Schedule`, `ServiceBusReceiver.Receive`, `ServiceBusReceiver.Complete`, `ServiceBusReceiver.DeadLetter`, `ServiceBusProcessor.ProcessMessage`, `ServiceBusSessionProcessor.ProcessSessionMessage`, `ServiceBusReceiver.RenewMessageLock`.

Fuente: [Troubleshooting guide – Distributed tracing](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-troubleshooting-guide#distributed-tracing) · [End-to-end tracing](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-end-to-end-tracing)

> [!warning] El outbox rompe la traza
> Si publicas desde un relay, el `Activity.Current` del relay no es el de la petición original. Guarda el `traceparent` en la fila del outbox y restáuralo al publicar (crea un `Activity` con ese padre, o añade `Diagnostic-Id` tú mismo), o la traza termina en el `INSERT`.

### Logging estructurado en el handler
```csharp
processor.ProcessMessageAsync += async args =>
{
    var m = args.Message;
    using var scope = logger.BeginScope(new Dictionary<string, object?>
    {
        ["MessageId"] = m.MessageId,
        ["CorrelationId"] = m.CorrelationId,
        ["Subject"] = m.Subject,
        ["DeliveryCount"] = m.DeliveryCount,
        ["Entity"] = args.EntityPath,
        ["AgeMs"] = (DateTimeOffset.UtcNow - m.EnqueuedTime).TotalMilliseconds
    });
    // ...
};
```
Logs del propio SDK: `AzureEventSourceListener` o, en ASP.NET Core, `Microsoft.Extensions.Azure` los redirige a `ILogger` (categorías `Azure.Messaging.ServiceBus*`). Ponlos en `Warning` en producción: en `Information` son muy verbosos.

### Métricas propias que merecen la pena
```csharp
static readonly Meter Meter = new("Orders.Messaging");
static readonly Histogram<double> HandlerDuration = Meter.CreateHistogram<double>("handler.duration", "ms");
static readonly Histogram<double> MessageAge = Meter.CreateHistogram<double>("message.age", "s");
static readonly Counter<long> Outcomes = Meter.CreateCounter<long>("message.outcome"); // tag: complete|abandon|deadletter|reschedule
```

### Consultas KQL útiles
```kusto
// Operaciones de Service Bus fallidas por tipo (dependencias)
dependencies
| where type == "Azure Service Bus" and success == false
| summarize count() by name, resultCode, bin(timestamp, 5m)

// Duración de procesamiento p95 por entidad
requests
| where name startswith "ServiceBusProcessor.ProcessMessage"
| summarize p95 = percentile(duration, 95), n = count() by tostring(customDimensions["messaging.destination.name"]), bin(timestamp, 15m)
```
[Verificar] Los nombres exactos de `type` y de las dimensiones dependen de la versión del exporter y de las convenciones semánticas de OpenTelemetry que emita tu versión del SDK. Comprueba una muestra en tu workspace antes de crear alertas sobre ellas.

### Relación
[[#Metricas clave]] · [[04 - NET y entorno local#Integracion con ASP.NET Core|Integracion con ASP.NET Core]] · [[06 - Laboratorios#Lab 11 - Observabilidad|Lab 11 - Observabilidad]]

---

## Tiers y facturacion
*Verificado contra [Pricing](https://azure.microsoft.com/pricing/details/service-bus/) y [Quotas](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-quotas) — nov-2026. Los precios no se copian aquí porque cambian por región y fecha: usa la [calculadora](https://azure.microsoft.com/pricing/calculator/?service=service-bus).*

### Comparativa funcional
| | Basic | Standard | Premium |
|---|---|---|---|
| Queues | ✔ | ✔ | ✔ |
| Scheduled messages | ✔ | ✔ | ✔ |
| Topics / subscriptions | ✘ | ✔ | ✔ |
| Sessions, transacciones, duplicate detection | ✘ | ✔ | ✔ |
| Auto-forwarding / send-via | ✘ | ✔ | ✔ |
| Tamaño de mensaje | 256 KB | 256 KB | 1 MB por defecto, hasta 100 MB (AMQP, sin batch) |
| Batch máximo | 256 KB | 256 KB | 1 MB |
| Recursos | Compartidos | Compartidos | **Dedicados** (messaging units) |
| Operaciones/s | 1 000 (créditos) | 1 000 (créditos) | Sin límite fijo; depende de MUs y carga |
| Entidades por namespace | 10 000 | 10 000 | 1 000 por MU, hasta 16 000 |
| Tamaño de entidad | 1–5 GB (80 GB particionada) | 1–5 GB (80 GB particionada) | 80 GB |
| Tamaño de namespace | 400 GB | 400 GB | 1 TB por MU |
| Private Endpoints / VNet | ✘ | ✘ | ✔ |
| Geo-Replication | ✘ | ✘ | ✔ |
| JMS 2.0 | ✘ | ✘ | ✔ |
| Availability zones | ✔ | ✔ | ✔ |

### Modelo de facturación
| Tier | Qué pagas |
|---|---|
| **Basic** | Por millón de operaciones |
| **Standard** | Cuota base por hora (una vez por suscripción de Azure, no por namespace) con **13 M operaciones/mes incluidas**, después por millón en tramos decrecientes. Conexiones brokered: 1 000 incluidas, después por conexión |
| **Premium** | Precio por **MU y hora**. Sin cargo por operación ni por conexión. Con particiones, MUs × particiones. Geo-Replication: MUs de cada réplica + GB replicados |

### Qué cuenta como operación (Standard/Basic)
- Toda llamada a la API: send, receive, complete, abandon, **renovación de lock**, peek, gestión (crear/listar/actualizar), session state.
- Mensajes > 64 KB cuentan como varias operaciones (por bloques de 64 KB).
- Un topic con 3 subscriptions: 1 envío + 3 recepciones = **4 operaciones** (más el complete de cada una si usas peek-lock).
- Recibir con peek-lock y completar = **2 operaciones**.

### Estimación: 5 M mensajes/día (ejercicio del día 28)
Supuestos: mensajes < 64 KB, queue con peek-lock, sin renovaciones.
```
Operaciones/mensaje ≈ 1 send + 1 receive + 1 complete = 3
5 M × 3 × 30 días ≈ 450 M operaciones/mes
Standard: 13 M incluidas + ~437 M facturadas en tramos → precio de la calculadora
Si el mensaje va a un topic con 4 subscriptions: 1 + 4×2 = 9 ops → ~1 350 M/mes
```
Con un patrón de fan-out ancho y tráfico constante, **Premium puede salir más barato que Standard** a partir de cierto volumen, además de ser predecible. Calcula ambos; no lo des por supuesto.

### Cuándo Premium (criterios, no precio)
- Necesitas aislamiento de red (Private Endpoints, desactivar acceso público).
- Latencia predecible / sin vecinos ruidosos; throttling en Standard que no puedes evitar.
- Mensajes > 256 KB sin claim-check.
- Geo-Replication con datos.
- Cumplimiento: CMK (claves gestionadas por el cliente).

### Migrar de Standard a Premium
- Existe migración **in-place** de namespace Standard → Premium que conserva el nombre de host ([docs](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-migrate-standard-premium)). Las **express entities** no se admiten en Premium: desactívalas antes.
- Revisa entidades particionadas: en Premium el particionado es a nivel de namespace y se decide al crearlo.
- Ensaya en un entorno no productivo; planifica la ventana.

### Autoscale de MUs (Premium)
Regla orientativa de Microsoft: subir con CPU > 75 % (o memoria > 60 %), bajar con CPU < 25 %. Configurable con Azure Monitor autoscale sobre `NamespaceCpuUsage`. Recuerda que el precio diario se calcula sobre el **máximo** de MUs asignadas en el periodo.

### Relación
[[05 - Seguridad y performance#Performance tuning|Performance tuning]] · [[#Geo-Replication]] · [[00 - Indice y ruta de estudio#Estado del producto y cambios recientes|Estado del producto y cambios recientes]]

---

## Geo-Replication
*Verificado contra [Geo-Replication](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-geo-replication) — nov-2026*

### Tres niveles de resiliencia
| Mecanismo | Protege frente a | Replica | Tier |
|---|---|---|---|
| **Availability zones** | Caída de un datacenter dentro de la región | Todo, síncrono, automático | Todos (según región) |
| **Geo-Disaster Recovery** (clásico) | Caída regional | **Solo metadatos** (entidades, configuración) vía un *alias*. Los mensajes **no**. Failover de un solo sentido; RBAC, red e identidades **no** se replican | Premium |
| **Geo-Replication** | Caída o degradación regional, migración de región | **Metadatos + datos + cambios de estado** de los mensajes | Premium |

Geo-DR y Geo-Replication **no se pueden combinar**. Para pasar de uno a otro, primero rompe el pairing de Geo-DR.

### Cómo funciona Geo-Replication
- Un **primario** que atiende a productores y consumidores; **un secundario** (actualmente solo uno) en *hot standby*: no puedes leer ni escribir en él.
- **Un único hostname**: siempre apunta al primario. Tras promover, el DNS cambia y los clientes reconectan sin cambiar código (puede tardar varios minutos).
- La **promoción** siempre la inicia el cliente; Azure nunca la dispara.

### Modos de replicación
| | Síncrono | Asíncrono |
|---|---|---|
| Ack al cliente | Tras confirmar en ambas regiones | Tras confirmar en el primario |
| Latencia | Mayor | Mínimo impacto |
| Disponibilidad | Depende de **ambas** regiones | La pérdida del secundario no afecta... hasta alcanzar el lag máximo configurado |
| RPO | 0 | Dentro del lag configurado; posible pérdida en promoción forzada |
| Si el secundario se atrasa o cae | El primario **throttlea** | El primario throttlea al superar el lag máximo |

Matiz: queues y topics (entidades de primer nivel) se replican siempre de forma síncrona; las **subscriptions** siguen el modo elegido.

### Promoción
| Tipo | Qué hace | Riesgo |
|---|---|---|
| **Planned** | Espera a que el lag llegue a 0; el namespace pasa a solo lectura mientras tanto | Ninguno de datos; tiempo de espera |
| **Forced** | Promueve ya | Pérdida de datos no replicados **y duplicados** (p. ej. un `Complete` no replicado → el mensaje se vuelve a entregar) |

Tras una promoción forzada, la recomendación actual es **eliminar la antigua región primaria** de la configuración y añadir un secundario nuevo, en lugar de dejar que se resincronice.

```bash
az servicebus namespace failover --namespace-name sb-orders-prod -g rg-sb --primary-location westeurope
```

### Limitaciones a conocer
- **Mensajes grandes (> 1 MB) no soportados** con Geo-Replication.
- El temporizador de `AutoDeleteOnIdle` se reinicia tras un failover.
- Integración con Event Grid: Event Grid replica a su región emparejada, no a tu secundario, y no hace failover con la promoción.
- Coste: cada réplica paga las mismas MUs que el primario + transferencia por GB replicado.

### Monitorización
- `ReplicationLagDuration` (segundos): para alertas de RPO. Microsoft recomienda alerta estática con agregación **Minimum** en ventana de 10-15 min para detectar lag **sostenido**.
- `ReplicationLagCount`: indicador auxiliar, no recomendado para alertas.

### Lo que Geo-Replication NO resuelve
- Tus consumidores, BDs y APIs también tienen que estar en la región secundaria y saber cuándo activarse.
- Duplicados tras promoción forzada → la idempotencia sigue siendo obligatoria.
- Decidir **cuándo** promover: la guía sugiere considerar la promoción tras 15-20 minutos sin conectividad. Ensaya el procedimiento al menos una vez.

### Private Endpoints y Geo-Replication
Los clientes vía private endpoint conectan al nuevo primario automáticamente. Para alta disponibilidad: un private endpoint en la VNet de cada región y una zona DNS privada **por región**.

### Relación
[[#Tiers y facturacion]] · [[05 - Seguridad y performance#Private Endpoints|Private Endpoints]] · [[00 - Indice y ruta de estudio#Estado del producto y cambios recientes|Estado del producto y cambios recientes]]

### Pregunta de entrevista
*Tenemos Geo-DR configurado, ¿estamos protegidos frente a la caída de una región?* → Solo parcialmente. Geo-DR replica metadatos: tras el failover tienes las entidades, pero **los mensajes que estaban en el primario no están**. Si eso es inaceptable, Geo-Replication (Premium) con modo síncrono para RPO 0, asumiendo latencia y dependencia de dos regiones.

---

## Service Bus vs otras tecnologias
### Comparativa (fuera de Azure y dentro)
| | Azure Service Bus | RabbitMQ | Apache Kafka / Event Hubs | AWS SQS | AWS SNS |
|---|---|---|---|---|---|
| Modelo | Broker de colas y topics | Broker (exchanges, colas) | **Log** distribuido particionado | Cola | Pub/sub push |
| Retención tras consumir | No (se borra al completar) | No (salvo streams) | **Sí** (por tiempo/tamaño); replay | No | No |
| Orden | Por sesión | Por cola con un consumidor | Por partición | FIFO queues por message group | Con FIFO topics |
| Entrega | At-least-once (peek-lock) | At-least-once con acks | At-least-once; exactly-once **dentro** de Kafka con transacciones | At-least-once (FIFO: dedup 5 min) | At-least-once |
| DLQ | Nativa por entidad | Dead letter exchanges | No nativa (patrón con topics) | Redrive policy | Hacia SQS |
| Sesiones / mensajes programados / defer | ✔ / ✔ / ✔ | ✘ / plugin / ✘ | ✘ / ✘ / ✘ | Message groups / delay ≤ 15 min / ✘ | ✘ |
| Transacciones | Dentro del namespace | Limitadas | Dentro de Kafka | ✘ | ✘ |
| Gestión | PaaS | Autogestionado o gestionado (CloudAMQP, Amazon MQ) | Autogestionado o gestionado (Confluent, Event Hubs con API Kafka) | PaaS | PaaS |
| Throughput | Alto | Alto | **Muy alto** (millones/s) | Alto | Alto |
| Protocolo | AMQP 1.0, JMS 2.0 (Premium) | AMQP 0-9-1 (y 1.0), MQTT, STOMP | Propio | HTTP | HTTP |

### Cuándo elegir cada uno
- **Service Bus**: mensajes de negocio en Azure con necesidad de DLQ, sesiones, transacciones, programación; equipos .NET; poca operación.
- **RabbitMQ**: on-premises o multi-cloud, enrutado flexible, latencia muy baja, control total (y operación propia).
- **Kafka / Event Hubs**: streams de eventos, telemetría, replay, múltiples consumidores que leen a su ritmo desde un offset, analítica. No es buena cola de trabajo con reintentos por mensaje.
- **SQS/SNS**: el equivalente en AWS. SNS → SQS (fan-out) ≈ topic + subscriptions.

### Errores de criterio frecuentes
- Elegir Kafka "porque escala más" para colas de comandos con reintentos individuales y DLQ: acabas reconstruyendo Service Bus encima.
- Elegir Service Bus para telemetría de millones de eventos/s o cuando necesitas replay.
- Suponer que "exactly-once" de Kafka cubre tu BD: solo cubre lecturas/escrituras dentro de Kafka.

### Relación
[[01 - Fundamentos y arquitectura#Service Bus vs Storage Queue vs Event Grid vs Event Hubs|Service Bus vs Storage Queue vs Event Grid vs Event Hubs]] · [[01 - Fundamentos y arquitectura#Message Broker|Message Broker]]

---

## Casos reales
> Escenarios **ilustrativos** construidos a partir de fallos típicos documentados en la guía de troubleshooting de Microsoft y en patrones conocidos. No son incidentes de empresas concretas.

### Caso 1 — La DLQ que nadie miraba
**Síntoma**: tres semanas después, Finanzas detecta 2 300 pedidos sin factura.
**Causa**: un cambio de contrato (`Total` pasó de `decimal` a objeto `Money`) hizo fallar la deserialización en Billing. Cada mensaje agotó `MaxDeliveryCount` y fue a la DLQ. No había alerta.
**Arreglo**: alerta `DeadletteredMessages > 0`, deserialización fallida → DeadLetter inmediato con razón `DeserializationFailed`, tolerant reader, reprocesado tras el fix (respetando la idempotencia).
**Lección**: la DLQ es una cola de incidentes, no un cubo de basura. [[03 - Confiabilidad y manejo de errores#Dead Letter Queue|Dead Letter Queue]] · [[08 - Patrones de arquitectura#Versionado de mensajes|Versionado de mensajes]]

### Caso 2 — Cobros duplicados tras un despliegue
**Síntoma**: clientes cobrados dos veces durante un despliegue.
**Causa**: el despliegue mató los pods sin graceful shutdown. Mensajes ya cobrados no llegaron a completarse; el lock expiró y otro pod volvió a cobrar. No había idempotency key con la pasarela.
**Arreglo**: `StopProcessingAsync` en `StopAsync`, `terminationGracePeriodSeconds` > duración máxima del handler, idempotency key = `MessageId`, tabla inbox.
**Lección**: at-least-once no es teoría; cada despliegue lo ejercita. [[03 - Confiabilidad y manejo de errores#Idempotent Consumer|Idempotent Consumer]] · [[04 - NET y entorno local#Integracion con ASP.NET Core|Integracion con ASP.NET Core]]

### Caso 3 — Prefetch 1 000 con un handler de 2 s
**Síntoma**: `MessageLockLost` constante, `DeliveryCount` alto, mensajes en DLQ con `MaxDeliveryCountExceeded` sin que el handler lance excepciones.
**Causa**: los mensajes esperaban en el buffer local más que el `LockDuration`.
**Arreglo**: `PrefetchCount = 0` (o ≈ 20 × tasa/s), escalar réplicas en lugar de prefetch.
[[05 - Seguridad y performance#Performance tuning|Performance tuning]] · [[10 - Troubleshooting#Troubleshooting - Lock expired|Lock expired]]

### Caso 4 — El topic lleno por una subscription huérfana
**Síntoma**: todos los productores de `order-events` reciben `QuotaExceeded`.
**Causa**: un servicio retirado meses atrás dejó su subscription; acumuló copias hasta llenar los 5 GB del topic.
**Arreglo**: borrar la subscription, alerta por `Size` y por `ActiveMessages` por subscription, `AutoDeleteOnIdle` en subscriptions temporales, checklist de baja de servicios.
[[08 - Patrones de arquitectura#Publisher Subscriber|Publisher Subscriber]]

### Caso 5 — Retry storm contra un proveedor caído
**Síntoma**: el proveedor de envíos cae 10 min; la API recibe 40× su tráfico normal al volver y vuelve a caer. Miles de mensajes en DLQ.
**Causa**: retries multiplicados (SDK × Polly × `MaxDeliveryCount`) sin backoff ni circuit breaker.
**Arreglo**: un único reintento corto, circuit breaker que pausa el processor, backoff por reprogramación, reprocesado controlado de la DLQ.
[[03 - Confiabilidad y manejo de errores#Retry Policies|Retry Policies]] · [[08 - Patrones de arquitectura#Circuit Breaker|Circuit Breaker]]

### Caso 6 — Funciona en dev, falla en producción con Private Endpoint
**Síntoma**: tras desactivar el acceso público, el worker en AKS falla con timeouts y errores de autorización.
**Causa**: la VNet del clúster no estaba enlazada a la zona `privatelink.servicebus.windows.net`; el nombre resolvía a la IP pública, que ya no aceptaba tráfico.
**Arreglo**: vincular la zona DNS a la VNet, verificar con `nslookup` desde un pod.
[[05 - Seguridad y performance#Private Endpoints|Private Endpoints]] · [[10 - Troubleshooting#Troubleshooting - Connection failures|Connection failures]]

### Caso 7 — Agotamiento de conexiones
**Síntoma**: errores intermitentes de conexión bajo carga, `ActiveConnections` creciendo con el tráfico.
**Causa**: `new ServiceBusClient(...)` por petición HTTP sin `DisposeAsync`: cada uno abre una conexión AMQP (y un socket).
**Arreglo**: cliente singleton vía `AddAzureClients`, senders cacheados.
[[04 - NET y entorno local#SDK moderno de NET|SDK moderno de NET]]

### Caso 8 — Una sesión caliente bloquea a un cliente grande
**Síntoma**: los pedidos del mayor cliente B2B tardan horas; el resto, segundos.
**Causa**: `SessionId = CustomerId`. Todos sus pedidos se procesaban en serie.
**Arreglo**: `SessionId = OrderId` (el orden solo importaba por pedido).
[[03 - Confiabilidad y manejo de errores#Sessions|Sessions]] · [[10 - Troubleshooting#Troubleshooting - Session lock problems|Session lock problems]]
