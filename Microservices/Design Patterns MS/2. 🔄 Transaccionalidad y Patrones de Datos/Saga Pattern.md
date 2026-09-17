---
tags: [microservicios, patrones, transacciones-distribuidas, consistencia]
up: "[[🏗️ Diseño de Microservicios]]"
---

# Saga Pattern

> [!info] Categoría: **Transaccionalidad y datos**. Descrito originalmente por Hector Garcia-Molina y Kenneth Salem en 1987 para transacciones de larga duración; hoy es el patrón estándar para transacciones distribuidas en microservicios.

## ¿Qué es?

Una **Saga** es una **secuencia de transacciones locales**, cada una en un servicio distinto, que juntas completan una operación de negocio que abarca varios microservicios.

Si un paso falla, la Saga ejecuta **transacciones compensatorias** que deshacen (lógicamente) los pasos anteriores que ya se habían confirmado.

## ¿Para qué sirve?

En un monolito, "crear pedido + cobrar + reservar stock" cabe en **una transacción [[ACID en Bases de Datos|ACID]]**. En microservicios cada servicio tiene su propia base de datos ([[🏗️ Diseño de Microservicios|Database-per-Service]]), así que no existe una transacción que abarque a todos.

La alternativa clásica, el **commit en dos fases (2PC / XA)**, bloquea recursos en todos los participantes mientras coordina, escala mal y muchas bases NoSQL y brokers no lo soportan. La Saga resuelve el mismo problema **sin bloqueos distribuidos**, a cambio de aceptar **consistencia eventual**.

## Conceptos relacionados

- [[Transactional Outbox]] → garantiza que cada paso de la Saga publique su evento de forma fiable.
- [[Idempotencia]] → cada paso y cada compensación deben ser idempotentes, porque los mensajes pueden llegar duplicados.
- [[Event Sourcing]] → forma natural de persistir el estado de una Saga orquestada.
- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → suele convivir con Sagas en arquitecturas orientadas a eventos.
- [[Retry con Backoff Exponencial]] → los pasos y compensaciones se reintentan ante fallos transitorios.
- [[Teorema CAP y BASE|BASE / consistencia eventual]] → el modelo de consistencia que asume la Saga.
- [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] → los pasos de una Saga suelen corresponder a operaciones sobre Agregados de distintos Bounded Contexts.

## ¿Cómo funciona?

### Ejemplo de flujo: crear un pedido

```
T1: Pedidos       → crea el pedido en estado PENDIENTE
T2: Pagos         → cobra la tarjeta
T3: Inventario    → reserva el stock
T4: Pedidos       → marca el pedido como CONFIRMADO
```

Si **T3 falla** (no hay stock):

```
C2: Pagos         → devuelve el cobro       (compensa T2)
C1: Pedidos       → marca el pedido CANCELADO (compensa T1)
```

> [!important] Compensar no es hacer rollback
> Un rollback borra lo que nunca pasó. Una compensación es una **nueva operación de negocio** que revierte el efecto: el cobro existió y ahora existe un reembolso. Ambos quedan registrados.

### Tipos de transacciones dentro de una Saga

| Tipo | Descripción | Ejemplo |
|---|---|---|
| **Compensables** | Pueden deshacerse con una compensación | Reservar stock → liberar stock |
| **Pivote** | Punto de no retorno; si tiene éxito, la Saga debe completarse | Cobrar la tarjeta |
| **Reintentables** | Van después del pivote; se reintentan hasta que funcionan, nunca se compensan | Enviar el email de confirmación |

### Dos estilos de coordinación

#### 1. Coreografía (*Choreography*)

No hay coordinador. Cada servicio **publica eventos** y **reacciona** a los eventos de otros.

```
Pedidos ──PedidoCreado──► Pagos ──PagoAprobado──► Inventario ──StockReservado──► Pedidos
                            │                         │
                       PagoRechazado             StockInsuficiente
                            ▼                         ▼
                        Pedidos cancela          Pagos reembolsa → Pedidos cancela
```

- ✅ Simple para Sagas cortas (2 o 3 pasos), sin punto único de fallo, bajo acoplamiento.
- ❌ El flujo completo **no está escrito en ningún sitio**; con 6 servicios es muy difícil razonar sobre él o depurarlo. Riesgo de dependencias cíclicas entre servicios.

#### 2. Orquestación (*Orchestration*)

Un **orquestador** (una máquina de estados) **envía comandos** a cada servicio y decide el siguiente paso según la respuesta.

```
                    ┌──────────────────────┐
                    │  Orquestador Pedido  │
                    └──┬──────┬──────┬─────┘
          Cobrar ──────┘      │      └────── ReservarStock
                              ▼
                       ConfirmarPedido
```

- ✅ El flujo está **centralizado y explícito**; fácil de seguir, probar y modificar. Sin dependencias cíclicas.
- ❌ Riesgo de concentrar demasiada lógica de negocio en el orquestador ("orquestador dios"). Un componente más que desplegar y operar.

> [!tip] Regla práctica
> Coreografía para Sagas simples; **orquestación** cuando hay más de 3 o 4 pasos, ramas condicionales o necesidad de visibilidad del estado.

## Ejemplo en .NET

Con **MassTransit** se define el orquestador como una máquina de estados persistente:

```csharp
public class PedidoSaga : MassTransitStateMachine<PedidoEstado>
{
    public State EsperandoPago { get; private set; }
    public State EsperandoStock { get; private set; }
    public State Confirmado { get; private set; }

    public Event<PedidoCreado> PedidoCreado { get; private set; }
    public Event<PagoAprobado> PagoAprobado { get; private set; }
    public Event<StockReservado> StockReservado { get; private set; }
    public Event<StockInsuficiente> StockInsuficiente { get; private set; }

    public PedidoSaga()
    {
        InstanceState(x => x.EstadoActual);

        Initially(
            When(PedidoCreado)
                .Send(ctx => new CobrarTarjeta(ctx.Message.PedidoId, ctx.Message.Total))
                .TransitionTo(EsperandoPago));

        During(EsperandoPago,
            When(PagoAprobado)
                .Send(ctx => new ReservarStock(ctx.Message.PedidoId))
                .TransitionTo(EsperandoStock));

        During(EsperandoStock,
            When(StockReservado)
                .Publish(ctx => new PedidoConfirmado(ctx.Message.PedidoId))
                .TransitionTo(Confirmado),
            When(StockInsuficiente)
                .Send(ctx => new ReembolsarPago(ctx.Message.PedidoId)) // compensación
                .Publish(ctx => new PedidoCancelado(ctx.Message.PedidoId))
                .Finalize());
    }
}
```

Otras opciones: **NServiceBus Sagas**, **Azure Durable Functions** (orquestaciones duraderas), **Temporal** y **Camunda / Zeebe** (motores de workflow).

## Ventajas

- Permite operaciones de negocio multi-servicio **sin transacciones distribuidas ni bloqueos**.
- Cada servicio conserva la **autonomía sobre su base de datos**.
- Escala mejor que 2PC y funciona con cualquier tipo de almacenamiento.

## Desventajas / Limitaciones

- **Complejidad**: hay que diseñar y probar cada compensación, y pueden fallar.
- **Falta de aislamiento**: entre T1 y T4 otros procesos ven el pedido en estado PENDIENTE. Son las llamadas *anomalías de la Saga* (lecturas sucias, actualizaciones perdidas).
- **Depuración difícil**, sobre todo en coreografía.
- Requiere **mensajería fiable** ([[Transactional Outbox]]) e **idempotencia** en todos los participantes.

### Contramedidas a la falta de aislamiento

- **Bloqueo semántico**: marcar el registro como `PENDIENTE` para que otros no lo toquen hasta que la Saga termine.
- **Actualizaciones conmutativas**: diseñar operaciones cuyo orden no importe (sumar/restar saldo).
- **Vista pesimista**: reordenar pasos para minimizar la ventana de inconsistencia (reservar stock antes de cobrar).
- **Releer el valor** antes de actualizar y abortar si cambió.

## Comparación

| Criterio | Transacción ACID local | 2PC / XA | Saga |
|---|---|---|---|
| Alcance | Una base de datos | Varias, con coordinador | Varios servicios |
| Consistencia | Fuerte e inmediata | Fuerte, con bloqueos | **Eventual** |
| Bloqueos | Cortos | Largos y distribuidos | Ninguno |
| Escalabilidad | Alta dentro del servicio | Baja | Alta |
| Complejidad | Baja | Media | **Alta** (compensaciones) |

## Puntos clave

- Saga = secuencia de **transacciones locales** + **compensaciones**.
- **Coreografía** (eventos, sin coordinador) vs **Orquestación** (máquina de estados central).
- Sacrifica **aislamiento** y **consistencia inmediata** a cambio de autonomía y escalabilidad.
- Necesita **Outbox** e **Idempotencia** para ser fiable.
- Identifica la **transacción pivote**: antes se compensa, después se reintenta.

## Errores comunes

- Diseñar los pasos y olvidar las compensaciones (o no probarlas nunca).
- Pensar que una compensación puede fallar "en silencio": debe reintentarse o escalar a intervención manual.
- Usar coreografía para flujos largos y perder la trazabilidad.
- Ignorar los duplicados de mensajes y cobrar dos veces.
- Meter reglas de negocio de cada servicio dentro del orquestador.

## 🎯 Para entrevistas y exámenes

- *"¿Por qué no usar 2PC en microservicios?"* → Bloqueos distribuidos, punto único de fallo, no soportado por muchos brokers y bases NoSQL.
- *"Coreografía vs orquestación"* → Saber dar un pro y un contra de cada una.
- *"¿Qué es una transacción compensatoria?"* → Operación de negocio que revierte el efecto de un paso ya confirmado.
- *"¿Qué garantía de consistencia da una Saga?"* → Eventual, no inmediata.

## Referencias

- Chris Richardson, *Pattern: Saga*: https://microservices.io/patterns/data/saga.html
- Microsoft Learn, *Saga distributed transactions pattern*: https://learn.microsoft.com/azure/architecture/reference-architectures/saga/saga
- Garcia-Molina & Salem, *Sagas* (1987): https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf
- MassTransit, *Saga State Machines*: https://masstransit.io/documentation/patterns/saga/state-machine

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
