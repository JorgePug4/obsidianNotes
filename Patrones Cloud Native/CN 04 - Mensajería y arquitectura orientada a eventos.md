---
tags: [cloud-native, patrones, mensajeria, eventos, event-driven]
aliases: [Publisher-Subscriber, Pub/Sub, Competing Consumers, Queue-Based Load Leveling, Priority Queue, Claim Check, Pipes and Filters, Choreography, Dead Letter Queue, Inbox pattern, Sequential Convoy]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 04 - Mensajería y arquitectura orientada a eventos

> [!info] Objetivo del capítulo
> Desacoplar servicios **en el tiempo** con colas y eventos, y conocer los problemas que eso introduce: duplicados, orden, mensajes venenosos y consistencia eventual.
> Implementación concreta en Azure: [[Azure-Service-Bus/00 - Indice y ruta de estudio|curso de Azure Service Bus]].

## Contenido
- [[#0. Conceptos base]]
- [[#1. Publisher-Subscriber]]
- [[#2. Competing Consumers]]
- [[#3. Queue-Based Load Leveling]]
- [[#4. Priority Queue]]
- [[#5. Claim Check]]
- [[#6. Pipes and Filters]]
- [[#7. Coreografía vs Orquestación]]
- [[#8. Sequential Convoy]]
- [[#9. Dead Letter Queue y Poison Message]]
- [[#10. Idempotent Consumer e Inbox]]
- [[#11. Event-Carried State Transfer y otros estilos de eventos]]

---

## 0. Conceptos base

### Mensaje, comando y evento

| Tipo | Significado | Nombre | Destinatarios | Ejemplo |
|---|---|---|---|---|
| **Comando** | "Haz esto" — una intención | Imperativo | **Uno** (el dueño) | `CrearFactura` |
| **Evento** | "Esto ha pasado" — un hecho | Pasado | **Cero o muchos** | `PedidoConfirmado` |
| **Consulta** | "Dime esto" | Pregunta | Uno, espera respuesta | `ObtenerStock` |

El emisor de un comando **sabe y le importa** quién lo ejecuta; el emisor de un evento **no sabe** quién lo escucha. Esa diferencia determina el acoplamiento.

### Cola vs Topic vs Stream

| | Cola (*queue*) | Topic (pub/sub) | Log / stream |
|---|---|---|---|
| Entrega | Cada mensaje a **un** consumidor | Cada mensaje a **cada** suscripción | Los consumidores leen a su ritmo desde un *offset* |
| Tras consumir | Se elimina | Se elimina de esa suscripción | **Se conserva** (retención por tiempo/tamaño) |
| Releer el pasado | No | No | **Sí** (*replay*) |
| Ejemplos | Azure Service Bus Queue, Azure Storage Queue, RabbitMQ, SQS | Service Bus Topics, SNS, RabbitMQ *fanout* | Kafka, Azure Event Hubs, Kinesis, Redis Streams |

### Garantías de entrega

| Garantía | Qué significa | Consecuencia |
|---|---|---|
| **At-most-once** | Se entrega 0 o 1 vez | Puede perder mensajes |
| **At-least-once** | Se entrega 1 o más veces | **Puede duplicar** → el consumidor debe ser idempotente |
| **Exactly-once** | Exactamente 1 vez | No existe de forma general entre sistemas independientes; en la práctica se consigue **"efectivamente una vez" = at-least-once + idempotencia/deduplicación** |

> [!important] Regla de oro
> Diseña **siempre** asumiendo *at-least-once*: tu consumidor **va a** recibir duplicados. Ver [[Idempotencia]] y la sección 10.

### Orden

La mayoría de brokers **no garantizan orden global** con varios consumidores en paralelo. Si necesitas orden, lo normal es garantizarlo **por clave** (por pedido, por cliente): sesiones de Service Bus, particiones de Kafka/Event Hubs. Ver sección 8.

---

## 1. Publisher-Subscriber

**Problema.** Cuando ocurre algo (pedido confirmado), varios servicios deben reaccionar (facturación, envío, notificaciones, analítica). Si el emisor llama a cada uno, conoce a todos y cada nuevo interesado requiere modificarlo.

**Solución.** El emisor **publica un evento** en un canal (topic); cada interesado tiene su **suscripción** y recibe una copia. El emisor no conoce a los suscriptores.

```
                         ┌─► Suscripción "facturacion"  ─► Facturación
Pedidos ─► Topic         ├─► Suscripción "envios"       ─► Envíos
  "pedido-confirmado"    └─► Suscripción "notificaciones"─► Notificaciones
```

```csharp
// Publicar (Azure Service Bus)
await using var sender = client.CreateSender("pedido-confirmado");
var msg = new ServiceBusMessage(BinaryData.FromObjectAsJson(new PedidoConfirmado(pedidoId, total)))
{
    MessageId = pedidoId.ToString(),      // permite deduplicación en el broker
    Subject = nameof(PedidoConfirmado),
    CorrelationId = Activity.Current?.TraceId.ToString()
};
await sender.SendMessageAsync(msg);
```

**Cuándo usarlo.** Notificar hechos a un número variable de interesados; integración entre bounded contexts.
**Cuándo no.** Cuando el emisor necesita la respuesta para continuar (usa una llamada o Request-Reply), o cuando sólo hay un consumidor y siempre lo habrá (una cola basta).

**Trade-offs.** ✅ Desacoplamiento, extensibilidad (añadir suscriptores sin tocar al emisor). ❌ Consistencia eventual; difícil ver "quién reacciona a qué" (documenta los eventos: AsyncAPI, catálogo de eventos); **el evento es un contrato** y cambiarlo rompe a suscriptores que no conoces.

> [!tip] Versionado de eventos
> Cambios **aditivos** (añadir campos opcionales) son seguros. Para cambios incompatibles, publica una **nueva versión** (`PedidoConfirmado.v2`) durante un tiempo en paralelo. Usar **CloudEvents** (estándar CNCF) como sobre común facilita la interoperabilidad.

---

## 2. Competing Consumers

**Problema.** Llegan más mensajes de los que un consumidor puede procesar.

**Solución.** Varias instancias del **mismo** consumidor leen de **la misma cola**; el broker entrega cada mensaje a **una sola** de ellas. Para escalar, añades instancias.

```
               ┌─► Worker 1
Cola pedidos ──┼─► Worker 2      (cada mensaje lo procesa uno solo)
               └─► Worker 3
```

**Claves:**
- El broker usa un **bloqueo temporal** (*peek-lock* en Service Bus, *visibility timeout* en SQS). Si el worker no confirma (`Complete`) antes de que expire, el mensaje vuelve a la cola → **duplicados** si el worker era sólo lento.
- Escalado automático por **longitud de la cola**: **KEDA** en Kubernetes, reglas de escalado en Azure Container Apps.
- Se pierde el orden entre mensajes (ver Sequential Convoy si importa).

```yaml
# KEDA: escala el Deployment según los mensajes pendientes en Service Bus
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata: { name: worker-pedidos }
spec:
  scaleTargetRef: { name: worker-pedidos }
  minReplicaCount: 0          # escala a cero si no hay mensajes
  maxReplicaCount: 30
  triggers:
    - type: azure-servicebus
      metadata: { queueName: pedidos, messageCount: "50" }   # ~1 réplica por cada 50 mensajes
      authenticationRef: { name: servicebus-auth }
```

**Trade-offs.** ✅ Escalado horizontal sencillo, tolerancia a fallos de un worker. ❌ Duplicados, sin orden, la BD que hay detrás puede convertirse en el cuello de botella (30 workers = 30× conexiones).

---

## 3. Queue-Based Load Leveling

**Problema.** Picos de carga (Black Friday, un proceso batch) saturan un servicio o una base de datos que no puede escalar al mismo ritmo, y se producen timeouts y errores.

**Solución.** Poner una **cola entre productor y consumidor**: el productor escribe tan rápido como quiera; el consumidor procesa a su **ritmo sostenible**. La cola actúa como amortiguador.

```
Carga entrante:  ▁▂█████▇▃▁▁▁        Procesamiento:  ▃▃▃▃▃▃▃▃▃▃▃▃
                 (picos)                              (constante)
                        └── la cola absorbe la diferencia ──┘
```

**Diferencia con Competing Consumers:** Load Leveling trata de **proteger** al consumidor limitando su ritmo; Competing Consumers trata de **aumentar** el ritmo añadiendo consumidores. Se combinan a menudo.

**Cuándo no.** Si el cliente necesita la respuesta de inmediato (latencia baja), la cola añade espera. Combínalo con Async Request-Reply.

**Vigila:** la **antigüedad del mensaje más viejo** (no sólo la longitud) y define un TTL: un mensaje de "enviar SMS de verificación" que se procesa 2 horas tarde es inútil.

---

## 4. Priority Queue

**Problema.** Algunas peticiones son más importantes (clientes premium, pagos frente a informes) y no deben esperar detrás de las de baja prioridad.

**Solución.** Dos variantes:
1. **Una cola por prioridad** con más consumidores (o consumidores dedicados) para la alta. Es la más común porque la mayoría de brokers no ordenan por prioridad.
2. **Una cola con prioridad nativa** (RabbitMQ admite prioridades de mensaje; Service Bus no).

```
Alta  ─► cola-alta  ─► 8 workers
Baja  ─► cola-baja  ─► 2 workers
```

**Riesgo:** **inanición** (*starvation*) de los mensajes de baja prioridad si la alta nunca se vacía. Garantiza un mínimo de capacidad para la baja.

---

## 5. Claim Check

**Problema.** Quieres enviar un mensaje grande (un PDF, una imagen, un lote de 50 MB) pero el broker tiene un límite de tamaño (Service Bus Standard: 256 KB; Premium: hasta 100 MB) y los mensajes grandes encarecen y ralentizan el bus.

**Solución.** Guardar el contenido en un **almacén externo** (Blob Storage, S3) y enviar por el bus sólo una **referencia** (el "resguardo", como el ticket del guardarropa).

```
Productor ─1. sube fichero─► Blob Storage
          ─2. envía {blobUri}─► Cola ─► Consumidor ─3. descarga─► Blob Storage
```

**Detalles:** el consumidor necesita permiso de lectura (usa [[CN 09 - Seguridad|Managed Identity]] o un SAS de corta duración — patrón Valet Key); define quién **borra** el blob y cuándo (políticas de ciclo de vida); el mensaje y el blob pueden quedar desincronizados (mensaje sin blob o blob huérfano).

---

## 6. Pipes and Filters

**Problema.** Un procesamiento complejo (ingesta de un documento: validar → extraer texto → traducir → clasificar → indexar) metido en un único componente es difícil de escalar, probar y reutilizar. Cada paso tiene necesidades de recursos distintas.

**Solución.** Descomponerlo en **filtros** (pasos independientes) conectados por **tuberías** (colas). Cada filtro escala por separado.

```
Entrada ─► [Validar] ─cola─► [Extraer texto] ─cola─► [Traducir] ─cola─► [Indexar]
              2 inst.            10 inst. (CPU)          4 inst.          2 inst.
```

**Trade-offs.** ✅ Escalado y despliegue por paso, reutilización, aislamiento de fallos. ❌ Latencia total mayor, más piezas, y hay que pensar qué pasa si un paso intermedio falla (reintentos, DLQ, compensación). Si los pasos tienen que ser una unidad atómica, este patrón no encaja bien.

Variantes: Azure Durable Functions (*function chaining*), Kafka Streams, pipelines de datos (Azure Data Factory).

---

## 7. Coreografía vs Orquestación

Dos formas de coordinar un proceso que involucra varios servicios (por ejemplo, una [[Saga Pattern|Saga]]):

**Coreografía.** No hay coordinador: cada servicio **reacciona a eventos** y publica los suyos.

```
Pedidos ─PedidoCreado─► Pagos ─PagoRealizado─► Inventario ─StockReservado─► Envíos
```

**Orquestación.** Un **orquestador** envía comandos a cada servicio y decide el siguiente paso según las respuestas.

```
              ┌─► Pagos.Cobrar ──────┐
Orquestador ──┼─► Inventario.Reservar┤ (respuestas al orquestador)
              └─► Envíos.Programar ──┘
```

| Criterio | Coreografía | Orquestación |
|---|---|---|
| Acoplamiento | Bajo (sólo eventos) | El orquestador conoce a todos |
| Visibilidad del flujo | **Difícil**: el proceso está repartido | **Clara**: está en un sitio |
| Punto único de fallo | No | El orquestador (debe ser HA y persistente) |
| Añadir pasos | Fácil si sólo reaccionan | Hay que modificar el orquestador |
| Riesgo | Dependencias cíclicas, "¿quién hace qué?" | Orquestador "dios" con lógica de todos |
| Mejor para | Flujos sencillos, 2–4 pasos | Flujos largos, con muchas ramas o compensaciones |
| Herramientas | Broker de eventos | Temporal, Azure Durable Functions, AWS Step Functions, Dapr Workflow, MassTransit/NServiceBus sagas |

> [!tip] Criterio práctico
> Empieza con coreografía para reacciones simples; cuando no puedas explicar el flujo completo dibujándolo en una pizarra en 2 minutos, pasa a orquestación.

---

## 8. Sequential Convoy

**Problema.** Necesitas procesar mensajes **en orden** (los eventos de un mismo pedido: creado → pagado → cancelado), pero quieres seguir escalando con varios consumidores.

**Solución.** Agrupar los mensajes **por una clave** (ID de pedido) y garantizar que todos los de una misma clave los procesa un único consumidor en orden, mientras que claves distintas se procesan en paralelo.

- **Azure Service Bus:** *sessions* (`SessionId = pedidoId`); un receptor bloquea la sesión.
- **Kafka / Event Hubs:** la *partition key*; el orden se garantiza dentro de cada partición.

**Trade-offs.** El paralelismo está limitado por el número de claves/particiones; una clave "caliente" crea un cuello de botella; un mensaje que falla bloquea toda su sesión.

---

## 9. Dead Letter Queue y Poison Message

**Problema.** Un mensaje que **siempre** falla (formato inválido, bug, dato imposible) se reintenta infinitamente, consume recursos y bloquea a los demás. Es un *poison message*.

**Solución.** Tras **N intentos** (Service Bus: `MaxDeliveryCount`, 10 por defecto) o al expirar su TTL, el broker mueve el mensaje a una **Dead Letter Queue (DLQ)**. Allí se inspecciona, se corrige y se reenvía (o se descarta).

**Buenas prácticas:**
- **Alertar** cuando la DLQ tiene mensajes: una DLQ que nadie mira es una pérdida de datos silenciosa.
- Distinguir errores **transitorios** (reintentar) de **permanentes** (mandar directamente a la DLQ, sin agotar reintentos).
- Guardar el motivo (`DeadLetterReason`, `DeadLetterErrorDescription`).
- Tener una herramienta o *runbook* para el reenvío.

Detalle en Azure: [[Azure-Service-Bus/03 - Confiabilidad y manejo de errores|Service Bus — confiabilidad y manejo de errores]].

---

## 10. Idempotent Consumer e Inbox

**Problema.** Con *at-least-once*, un mensaje puede llegar dos veces. Si el consumidor cobra un pago o descuenta stock, el duplicado causa un error de negocio.

**Solución — Idempotent Consumer.** El consumidor registra los IDs de mensajes ya procesados y descarta los repetidos. Para que sea correcto, **el registro del ID y el efecto de negocio deben guardarse en la misma transacción**: es el **patrón Inbox** (el simétrico del [[Transactional Outbox]]).

```csharp
public async Task ProcesarAsync(ServiceBusReceivedMessage msg)
{
    await using var tx = await _db.Database.BeginTransactionAsync();

    // Clave primaria en MessageId: un duplicado provoca violación de unicidad
    var yaProcesado = await _db.Inbox.AnyAsync(i => i.MessageId == msg.MessageId);
    if (yaProcesado) return;   // el duplicado se descarta (y luego se completa el mensaje)

    var evt = msg.Body.ToObjectFromJson<PagoRealizado>();
    var pedido = await _db.Pedidos.FindAsync(evt.PedidoId);
    pedido!.MarcarComoPagado();

    _db.Inbox.Add(new InboxEntry(msg.MessageId, DateTime.UtcNow));
    await _db.SaveChangesAsync();
    await tx.CommitAsync();
}
```

> [!note] Deduplicación en el broker
> Service Bus ofrece *duplicate detection* por `MessageId` dentro de una ventana de tiempo configurable. Ayuda con los duplicados del **productor** (reenvíos), pero **no** sustituye la idempotencia del consumidor: el consumidor puede recibir dos veces el mismo mensaje si su bloqueo expira.

Alternativa sin tabla: hacer la operación **idempotente por naturaleza** (`UPDATE pedidos SET estado='Pagado' WHERE id=@id AND estado='Pendiente'`; *upserts*; versiones optimistas).

---

## 11. Event-Carried State Transfer y otros estilos de eventos

Martin Fowler distingue (*What do you mean by "Event-Driven"?*, 2017) varios estilos que conviene no confundir:

| Estilo | El evento contiene | Ventaja | Riesgo |
|---|---|---|---|
| **Event Notification** | Sólo "pasó X" + un ID | Eventos pequeños, poco acoplamiento de datos | El consumidor debe llamar al emisor para obtener detalles → acoplamiento en runtime |
| **Event-Carried State Transfer** | Los datos necesarios (el estado) | El consumidor mantiene su copia local y no llama a nadie | Datos duplicados, eventos más grandes, copias eventualmente consistentes |
| **Event Sourcing** | El evento **es** la fuente de verdad | Historial completo, auditoría | Complejidad alta; ver [[Event Sourcing]] |
| **CQRS** | — (patrón de lectura/escritura que suele usar eventos) | Modelos optimizados | Ver [[CQRS (Command Query Responsibility Segregation)\|CQRS]] |

---

## Puntos clave

- Comando = "haz esto", a uno; evento = "esto pasó", a muchos.
- Cola (un consumidor), topic (cada suscripción), stream (se conserva, se puede releer).
- Asume **at-least-once** → consumidores **idempotentes** (Inbox).
- **Load Leveling** protege; **Competing Consumers** acelera; **KEDA** escala por longitud de cola.
- Orden → por clave (sessions / particiones), no global.
- DLQ + alertas + runbook. Una DLQ sin vigilancia = pérdida silenciosa.
- Coreografía para flujos simples; orquestación cuando el flujo es largo o tiene compensaciones.

## Errores comunes

- Creer que el broker garantiza *exactly-once* y no implementar idempotencia.
- Procesar más tiempo que la duración del *lock* → el mensaje vuelve y se procesa dos veces (renueva el lock o ajusta la duración).
- Publicar el evento **antes** de confirmar la transacción de BD (o después, sin Outbox): se pierden o se inventan eventos. Ver [[Transactional Outbox]].
- Eventos con el modelo interno de la BD del emisor: cualquier cambio de esquema rompe a los consumidores.
- Usar eventos para algo que es realmente una petición con respuesta.

## 🎯 Para entrevistas

- *"¿Exactly-once existe?"* → En general no entre sistemas independientes; se logra efecto de una vez con at-least-once + idempotencia.
- *"Queue-Based Load Leveling vs Competing Consumers"* → proteger al consumidor vs escalarlo.
- *"¿Cómo mantienes el orden con varios consumidores?"* → particionar por clave: sessions en Service Bus, partition key en Kafka.
- *"¿Coreografía u orquestación para un checkout de 6 pasos con compensaciones?"* → orquestación (visibilidad, control de compensaciones); justificar.
- *"¿Qué es Claim Check?"* → mensaje con referencia a un blob para superar límites de tamaño.

## Referencias

- Microsoft, *Publisher-Subscriber*, *Competing Consumers*, *Queue-Based Load Leveling*, *Priority Queue*, *Claim Check*, *Pipes and Filters*, *Choreography*, *Sequential Convoy*: https://learn.microsoft.com/azure/architecture/patterns/
- Hohpe y Woolf, *Enterprise Integration Patterns* (2003): https://www.enterpriseintegrationpatterns.com/
- Martin Fowler, *What do you mean by "Event-Driven"?* (2017): https://martinfowler.com/articles/201701-event-driven.html
- Chris Richardson, *Idempotent Consumer*: https://microservices.io/patterns/communication-style/idempotent-consumer.html
- CloudEvents: https://cloudevents.io/
- KEDA, *Azure Service Bus scaler*: https://keda.sh/docs/latest/scalers/azure-service-bus/

---
⬅️ [[CN 03 - Comunicación, enrutamiento y APIs]] · ➡️ [[CN 05 - Patrones de gestión de datos]]
