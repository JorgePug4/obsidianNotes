---
tags: [microservicios, patrones, persistencia, eventos, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
---

# Event Sourcing

> [!info] Categoría: **Transaccionalidad y datos**. Patrón de **persistencia**: en lugar de guardar el estado actual, se guarda la historia completa de cambios.

## ¿Qué es?

**Event Sourcing** consiste en persistir el estado de una entidad como una **secuencia inmutable de eventos** que describen todo lo que le ha ocurrido, en lugar de guardar solo su estado actual.

El estado actual se obtiene **reproduciendo** (*replay*) los eventos desde el principio.

```
Modelo tradicional (estado):        Event Sourcing (historia):
┌────────────────────────┐          1. CuentaAbierta      (saldo inicial 0)
│ Cuenta 123             │          2. DineroDepositado   (+100)
│ saldo: 70              │          3. DineroRetirado     (-50)
└────────────────────────┘          4. DineroDepositado   (+20)
                                    ───────────────────────────
                                    Estado derivado: saldo = 70
```

## ¿Para qué sirve?

- **Auditoría completa** y gratuita: sabes no solo *qué* estado hay, sino *cómo* y *por qué* se llegó a él.
- **Depuración y análisis temporal**: reconstruir el estado en cualquier instante del pasado ("¿cuánto saldo tenía el 3 de marzo?").
- **Integración por eventos**: los eventos ya existen; otros servicios pueden suscribirse sin [[Transactional Outbox]].
- **Nuevas vistas retroactivas**: crear un informe nuevo y calcularlo sobre toda la historia.
- **Dominios donde la historia es el negocio**: banca, contabilidad, seguros, logística, control de versiones.

## Conceptos relacionados

- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → casi siempre acompaña a Event Sourcing: los eventos alimentan los modelos de lectura (proyecciones).
- [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] → los eventos son **Domain Events** emitidos por un **Agregado**; cada *stream* suele corresponder a un Agregado.
- [[Saga Pattern]] → el estado de una Saga orquestada puede persistirse como eventos.
- [[Transactional Outbox]] → alternativa cuando **no** se usa Event Sourcing; con Event Sourcing el *event store* ya es la fuente de eventos.
- [[Idempotencia]] → las proyecciones deben tolerar reprocesar el mismo evento.
- [[ACID en Bases de Datos|ACID]] → el *append* de eventos de un mismo stream es atómico, con control de concurrencia optimista.

## ¿Cómo funciona?

### 1. El Agregado emite eventos en lugar de mutar campos

```csharp
public class CuentaBancaria : AggregateRoot
{
    public decimal Saldo { get; private set; }

    public void Retirar(decimal monto)
    {
        if (monto > Saldo)
            throw new SaldoInsuficienteException();

        Emitir(new DineroRetirado(Id, monto)); // no toca Saldo directamente
    }

    // El evento se aplica para actualizar el estado en memoria
    private void Apply(DineroRetirado e) => Saldo -= e.Monto;
    private void Apply(DineroDepositado e) => Saldo += e.Monto;
    private void Apply(CuentaAbierta e) { Id = e.CuentaId; Saldo = 0; }
}
```

### 2. Los eventos se guardan en un *stream* (append-only)

```
Stream "cuenta-123"
┌───┬───────────────────┬──────────────────────────┬─────────────────────┐
│ # │ Tipo              │ Datos                    │ Timestamp           │
├───┼───────────────────┼──────────────────────────┼─────────────────────┤
│ 1 │ CuentaAbierta     │ {clienteId: "c-9"}       │ 2026-01-10 09:00:00 │
│ 2 │ DineroDepositado  │ {monto: 100}             │ 2026-01-10 09:05:00 │
│ 3 │ DineroRetirado    │ {monto: 50}              │ 2026-01-11 14:30:00 │
└───┴───────────────────┴──────────────────────────┴─────────────────────┘
```

Los eventos **nunca se modifican ni se borran**. Un error se corrige con un nuevo evento compensatorio (`DepositoRevertido`), igual que en contabilidad.

### 3. Cargar el Agregado = reproducir su stream

```csharp
var eventos = await _eventStore.LeerStreamAsync("cuenta-123");
var cuenta = new CuentaBancaria();
foreach (var e in eventos) cuenta.Apply(e);   // estado reconstruido
```

### 4. Concurrencia optimista

Al guardar se indica la **versión esperada** del stream. Si otro proceso añadió eventos entre la lectura y la escritura, el *event store* rechaza el *append* y el comando se reintenta.

### 5. Snapshots (optimización)

Un stream con 100 000 eventos tarda en reproducirse. Cada N eventos se guarda una **instantánea** del estado; al cargar, se parte del último snapshot y solo se reproducen los eventos posteriores.

### 6. Proyecciones (lado de lectura)

Suscriptores que escuchan los eventos y actualizan **modelos de lectura** optimizados: una tabla `SaldosPorCliente`, un índice en Elasticsearch, una caché en Redis. Se pueden **reconstruir desde cero** reproduciendo todos los eventos.

## Ejemplo de herramientas

| Ecosistema | Herramienta |
|---|---|
| **.NET** | **Marten** (sobre PostgreSQL, muy popular), **EventStoreDB / KurrentDB** (cliente oficial), Eventuous |
| **Multiplataforma** | **KurrentDB** (antes EventStoreDB, renombrado en 2025), **Axon Server** (Java), Apache Kafka con retención infinita (con matices) |
| **Cloud** | Azure Cosmos DB con *Change Feed*, DynamoDB Streams |

> [!warning] Kafka no es un event store completo
> Kafka funciona bien como bus de eventos y para *replay* global, pero no ofrece de forma nativa lectura eficiente de un stream por Agregado ni concurrencia optimista por stream. Se puede usar, pero no es su caso de uso principal.

## Ventajas

- **Historial completo** e inmutable: auditoría y trazabilidad perfectas.
- Permite **viajar en el tiempo** y depurar reproduciendo eventos.
- **Proyecciones reconstruibles**: si un modelo de lectura se corrompe o cambia, se regenera.
- Modelo muy alineado con **DDD** y arquitecturas orientadas a eventos.
- Elimina el problema de la doble escritura.

## Desventajas / Limitaciones

- **Curva de aprendizaje alta**: cambia la forma de pensar la persistencia.
- **Versionado de eventos**: los eventos antiguos viven para siempre; si cambia su esquema hay que hacer *upcasting* (transformar versiones viejas al leer).
- **Consultas ad hoc difíciles**: no puedes hacer un `SELECT WHERE` sobre el estado; necesitas proyecciones (y por tanto CQRS).
- **Consistencia eventual** en los modelos de lectura.
- **Privacidad / GDPR**: el "derecho al olvido" choca con la inmutabilidad. Soluciones: *crypto-shredding* (cifrar datos personales con una clave por usuario y destruir la clave) o eventos con datos personales fuera del stream.
- **Volumen**: streams muy largos requieren snapshots.

## Comparación

| Aspecto | Persistencia de estado (CRUD/ORM) | Event Sourcing |
|---|---|---|
| Qué se guarda | Estado actual | Historia de cambios |
| Historial | Solo si se añade tabla de auditoría | Inherente |
| Consultas ad hoc | Fáciles (SQL) | Requieren proyecciones |
| Corrección de errores | `UPDATE` | Evento compensatorio |
| Complejidad | Baja | Alta |
| Cuándo | Mayoría de aplicaciones | Dominios donde la historia importa o la auditoría es requisito |

## Puntos clave

- Se guardan **eventos**, no estado; el estado se **deriva**.
- Eventos **inmutables**, en tiempo pasado, ordenados por stream.
- Casi siempre va con **CQRS** y **proyecciones**.
- **Snapshots** para streams largos; **upcasting** para eventos que cambian de versión.
- No es la opción por defecto: elígelo cuando la **historia sea parte del negocio**.

## Errores comunes

- Usar Event Sourcing en un CRUD sencillo por moda.
- Diseñar eventos técnicos (`FilaActualizada`) en lugar de eventos de negocio (`PedidoPagado`).
- Modificar o borrar eventos "para arreglar un dato".
- No planificar el versionado de eventos desde el principio.
- Meter datos personales en claro en eventos inmutables.

## 🎯 Para entrevistas y exámenes

- *"¿Qué diferencia hay entre Event Sourcing y CQRS?"* → Event Sourcing es cómo **persistes**; CQRS es cómo **separas lectura y escritura**. Se complementan pero son independientes.
- *"¿Cómo corriges un error en Event Sourcing?"* → Con un evento compensatorio, nunca editando la historia.
- *"¿Qué es un snapshot?"* → Instantánea del estado para no reproducir todos los eventos.
- *"¿Cómo manejas el derecho al olvido?"* → Crypto-shredding o separar datos personales del stream.

## Referencias

- Martin Fowler, *Event Sourcing*: https://martinfowler.com/eaaDev/EventSourcing.html
- Microsoft Learn, *Event Sourcing pattern*: https://learn.microsoft.com/azure/architecture/patterns/event-sourcing
- Greg Young, *Versioning in an Event Sourced System* (libro gratuito): https://leanpub.com/esversioning
- Marten (.NET): https://martendb.io/

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
