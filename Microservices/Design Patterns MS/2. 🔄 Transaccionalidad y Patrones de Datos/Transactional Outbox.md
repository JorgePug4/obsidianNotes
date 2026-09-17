---
tags: [microservicios, patrones, mensajeria, consistencia, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
---

# Transactional Outbox

> [!info] Categoría: **Transaccionalidad y datos**. Resuelve el problema de la **doble escritura** (*Dual-Write Problem*).

## ¿Qué es?

El **Transactional Outbox** es un patrón que garantiza que **un cambio en la base de datos y la publicación del evento que lo anuncia** ocurran ambos o ninguno, guardando el evento en una tabla `Outbox` **dentro de la misma transacción** que el cambio de negocio. Un proceso aparte lee esa tabla y entrega los mensajes al *broker*.

## ¿Para qué sirve?

### El problema de la doble escritura

Un servicio necesita hacer dos cosas cuando se crea un pedido:

1. Guardar el pedido en **su base de datos**.
2. Publicar `PedidoCreado` en **el broker** (RabbitMQ, Kafka, Azure Service Bus).

Son dos sistemas distintos y **no comparten transacción**. Cualquier orden falla:

| Orden | Qué puede salir mal |
|---|---|
| 1) Guardar en BD → 2) Publicar | La app se cae entre los dos pasos: el pedido existe pero **nadie se entera**. |
| 1) Publicar → 2) Guardar en BD | La BD rechaza la escritura: se anunció un pedido **que no existe**. |

El Outbox convierte los dos pasos en **una sola escritura transaccional**, y delega la publicación a un proceso que puede reintentar sin miedo.

## Conceptos relacionados

- [[Saga Pattern]] → cada paso de una Saga publica eventos; sin Outbox, la Saga puede quedarse a medias.
- [[Idempotencia]] → el Outbox garantiza **al menos una vez** (*at-least-once*); el consumidor debe tolerar duplicados.
- [[Event Sourcing]] → alternativa en la que el propio *event store* es la fuente de eventos; el Outbox no es necesario.
- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → los eventos del Outbox suelen alimentar los modelos de lectura.
- [[ACID en Bases de Datos|ACID]] → el patrón se apoya en la atomicidad de la transacción local.
- [[Retry con Backoff Exponencial]] → el publicador reintenta la entrega al broker.

## ¿Cómo funciona?

```
┌───────────────────── Transacción de BD ──────────────────────┐
│  INSERT INTO Pedidos  (...)                                  │
│  INSERT INTO Outbox   (Id, Tipo, Payload, CreadoEn, Enviado) │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌──────────────────────┐
                  │  Message Relay       │  lee Outbox WHERE Enviado = false
                  │  (worker / CDC)      │  publica al broker
                  └──────────┬───────────┘  marca Enviado = true (o borra la fila)
                             ▼
                        📨 Broker
```

### Dos formas de implementar el *relay*

| Estrategia | Cómo funciona | Pros | Contras |
|---|---|---|---|
| **Polling Publisher** | Un *background worker* consulta la tabla cada N ms | Simple, sin infraestructura extra | Latencia = intervalo de polling; carga sobre la BD |
| **Transaction Log Tailing (CDC)** | Una herramienta lee el *log* de transacciones de la BD (WAL, binlog) y publica los cambios | Latencia mínima, sin polling | Infraestructura adicional (Debezium + Kafka Connect) |

### Garantía de entrega

El *relay* puede publicar el mensaje y caerse **antes** de marcarlo como enviado. Al reiniciar, lo publicará **otra vez**. Por eso el Outbox ofrece **al menos una vez**, nunca *exactamente una vez*. Solución: los consumidores deben ser **idempotentes**, por ejemplo con un patrón **Inbox** (tabla de mensajes ya procesados, deduplicando por `MessageId`).

## Ejemplo en .NET con EF Core

```csharp
public async Task CrearPedidoAsync(CrearPedidoCommand cmd)
{
    var pedido = Pedido.Crear(cmd.ClienteId, cmd.Lineas);

    _db.Pedidos.Add(pedido);

    _db.OutboxMessages.Add(new OutboxMessage
    {
        Id = Guid.NewGuid(),
        Tipo = nameof(PedidoCreado),
        Payload = JsonSerializer.Serialize(new PedidoCreado(pedido.Id, pedido.Total)),
        CreadoEn = DateTime.UtcNow
    });

    await _db.SaveChangesAsync(); // UNA sola transacción: pedido + mensaje
}
```

Worker de publicación (versión simplificada):

```csharp
public class OutboxPublisher : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        while (!ct.IsCancellationRequested)
        {
            var pendientes = await _db.OutboxMessages
                .Where(m => m.EnviadoEn == null)
                .OrderBy(m => m.CreadoEn)
                .Take(50)
                .ToListAsync(ct);

            foreach (var msg in pendientes)
            {
                await _bus.PublishAsync(msg.Tipo, msg.Payload, ct);
                msg.EnviadoEn = DateTime.UtcNow;
            }

            await _db.SaveChangesAsync(ct);
            await Task.Delay(TimeSpan.FromMilliseconds(500), ct);
        }
    }
}
```

> [!tip] No lo escribas a mano en producción
> **MassTransit** (`UseEntityFrameworkOutbox`), **NServiceBus** (Outbox), **DotNetCore.CAP** y **Wolverine** implementan el patrón completo, incluyendo Inbox y limpieza de la tabla.

## Ventajas

- Elimina el problema de la doble escritura con **una sola base de datos**.
- No requiere 2PC ni que el broker soporte transacciones distribuidas.
- La tabla Outbox es también un **registro auditable** de los eventos emitidos.
- Funciona con cualquier BD relacional y con muchas NoSQL que soporten transacciones a nivel de documento/colección.

## Desventajas / Limitaciones

- **Latencia extra** entre el commit y la publicación (especialmente con polling).
- Entrega **al menos una vez**: obliga a consumidores idempotentes.
- La tabla Outbox **crece**: necesita purga periódica.
- El **orden** de los mensajes solo se garantiza si el *relay* publica en orden y con un único publicador por partición.
- Con CDC se añade infraestructura que hay que operar.

## Comparación

| Enfoque | ¿Resuelve la doble escritura? | Complejidad |
|---|---|---|
| Publicar directamente tras `SaveChanges` | ❌ No | Baja |
| 2PC entre BD y broker | ✅ Sí, pero con bloqueos y poco soporte | Alta |
| **Transactional Outbox** | ✅ Sí, con at-least-once | Media |
| **[[Event Sourcing]]** | ✅ Sí (el evento *es* la escritura) | Alta |
| Listen-to-yourself (publicar primero, la BD se actualiza al consumir) | ✅ Sí, con consistencia eventual en la propia BD | Media-alta |

## Puntos clave

- Evento y cambio de negocio en **la misma transacción local**.
- Un **relay** (worker o CDC) publica los eventos después.
- Garantía **at-least-once** → consumidores **idempotentes** (Inbox).
- Es la base para que las [[Saga Pattern|Sagas]] sean fiables.

## Errores comunes

- Publicar al broker dentro del `try` y confiar en que "casi nunca falla".
- No purgar la tabla Outbox y que crezca a millones de filas.
- Olvidar la deduplicación en el consumidor y procesar el mismo evento dos veces.
- Ejecutar varias instancias del worker de polling sin bloqueo y publicar duplicados masivamente (usar `SELECT ... FOR UPDATE SKIP LOCKED` o una sola instancia).

## 🎯 Para entrevistas y exámenes

- *"¿Qué es el Dual-Write Problem?"* → Escribir en dos sistemas sin transacción común; uno puede fallar.
- *"¿Qué garantía de entrega da el Outbox?"* → At-least-once; por eso hace falta idempotencia.
- *"¿Qué es CDC?"* → Change Data Capture: leer el log de transacciones para publicar cambios (Debezium).

## Referencias

- Chris Richardson, *Pattern: Transactional outbox*: https://microservices.io/patterns/data/transactional-outbox.html
- Microsoft Learn, *Transactional Outbox pattern with Azure Cosmos DB*: https://learn.microsoft.com/azure/architecture/databases/guide/transactional-outbox-cosmos
- Debezium, *Outbox Event Router*: https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
