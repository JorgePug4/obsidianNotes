---
tags: [microservicios, patrones, arquitectura, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
aliases: [CQRS]
---

# CQRS (Command Query Responsibility Segregation)

> [!info] Categoría: **Transaccionalidad y datos**. Acuñado por Greg Young (~2010) como evolución del principio CQS (*Command-Query Separation*) de Bertrand Meyer.

## ¿Qué es?

**CQRS** separa el modelo que **modifica el estado** (comandos) del modelo que **lee el estado** (consultas). En lugar de una sola clase o un solo modelo de datos que sirve para todo, hay dos caminos distintos, cada uno optimizado para su trabajo.

- **Command**: *"haz algo"*. Cambia estado, **no devuelve datos** (a lo sumo un identificador o un resultado de éxito/fallo). Ej.: `CrearPedido`, `CancelarReserva`.
- **Query**: *"dame algo"*. Devuelve datos, **no cambia nada**. Ej.: `ObtenerPedidosDelCliente`.

> [!important] CQRS **no** exige dos bases de datos
> La forma más simple de CQRS es tener dos conjuntos de clases (comandos y consultas) sobre la **misma** base de datos. Separar el almacenamiento es un paso opcional y avanzado.

## ¿Para qué sirve?

- **Cargas asimétricas**: la mayoría de sistemas tienen muchas más lecturas que escrituras. CQRS permite **escalar cada lado por separado**.
- **Modelos distintos para necesidades distintas**: la escritura necesita reglas de negocio e invariantes (un Agregado rico de [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]]); la lectura necesita datos planos y rápidos para una pantalla (un DTO).
- **Evitar el modelo único que no sirve a nadie**: un ORM con 15 `Include` para pintar una lista, o un DTO usado para validar reglas de negocio.
- **Seguridad**: es más fácil auditar y autorizar comandos que métodos genéricos de "guardar".

## Conceptos relacionados

- [[Event Sourcing]] → compañero habitual: los eventos del lado de escritura construyen los modelos de lectura. **No es obligatorio** combinarlos.
- [[Transactional Outbox]] → mecanismo fiable para propagar los cambios del lado de escritura al de lectura.
- [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] → los comandos operan sobre Agregados.
- [[Clean Architecture]] → los *handlers* de comandos y consultas viven en la capa de Aplicación (casos de uso).
- [[Bases de Datos SQL vs NoSQL]] → el lado de lectura puede usar un almacén distinto (Redis, Elasticsearch, [[Azure Cosmos DB]]).
- [[Teorema CAP y BASE|Consistencia eventual]] → aparece en cuanto se separan los almacenes.

## ¿Cómo funciona?

### Nivel 1: separación lógica (misma BD)

```
            ┌── Command ──► CommandHandler ──► Agregado ──► BD (escritura)
Cliente ────┤
            └── Query ────► QueryHandler ────► SQL / Dapper ──► BD (lectura)
```

Los *handlers* de comandos cargan Agregados, aplican reglas y persisten. Los de consultas hacen SQL directo (o vistas) y devuelven DTOs. **Sin eventos, sin sincronización, consistencia inmediata.** Este nivel ya aporta la mayor parte del valor.

### Nivel 2: almacenes separados (con sincronización)

```
Command ──► Modelo de escritura (SQL normalizado)
                      │  evento (Outbox / Event Sourcing)
                      ▼
            Proyección ──► Modelo de lectura (desnormalizado: Redis, Elastic, tabla plana)
                                    ▲
Query ──────────────────────────────┘
```

Los eventos de escritura se **proyectan** al modelo de lectura. Entre la escritura y la actualización de la proyección pasa un tiempo: **consistencia eventual**. La interfaz de usuario debe tolerarlo (ej. mostrar "procesando" o actualizar la vista de forma optimista).

## Ejemplo en .NET

Con el patrón *mediator* (librería **MediatR** u otras equivalentes como **Wolverine** o una implementación propia):

```csharp
// ── Lado de escritura ──────────────────────────────────────────
public record CrearPedidoCommand(Guid ClienteId, List<LineaDto> Lineas) : IRequest<Guid>;

public class CrearPedidoHandler : IRequestHandler<CrearPedidoCommand, Guid>
{
    private readonly IPedidoRepository _repo;

    public async Task<Guid> Handle(CrearPedidoCommand cmd, CancellationToken ct)
    {
        var pedido = Pedido.Crear(cmd.ClienteId, cmd.Lineas); // reglas de negocio aquí
        await _repo.AddAsync(pedido, ct);
        return pedido.Id;
    }
}

// ── Lado de lectura ────────────────────────────────────────────
public record ObtenerPedidosClienteQuery(Guid ClienteId) : IRequest<List<PedidoResumenDto>>;

public class ObtenerPedidosClienteHandler
    : IRequestHandler<ObtenerPedidosClienteQuery, List<PedidoResumenDto>>
{
    private readonly IDbConnection _conn;

    public Task<List<PedidoResumenDto>> Handle(ObtenerPedidosClienteQuery q, CancellationToken ct) =>
        _conn.QueryAsync<PedidoResumenDto>(
            "SELECT Id, Total, Estado, CreadoEn FROM vw_PedidosResumen WHERE ClienteId = @ClienteId",
            new { q.ClienteId })
            .ContinueWith(t => t.Result.ToList(), ct);
}
```

> [!note] Sobre MediatR
> Desde 2025 MediatR pasó a un modelo de licencia comercial para empresas grandes (sigue siendo gratuito para proyectos y empresas pequeñas). Verifica las condiciones actuales antes de adoptarlo en un proyecto nuevo.

## Ventajas

- Escalado **independiente** de lecturas y escrituras.
- Cada modelo es **más simple** que un modelo único que intenta servir a ambos.
- Consultas **rápidas** sobre modelos desnormalizados, sin JOINs complejos.
- Encaja de forma natural con DDD, Event Sourcing y arquitecturas orientadas a eventos.
- Facilita la **autorización** y la **auditoría** por tipo de comando.

## Desventajas / Limitaciones

- **Más código**: dos modelos, dos rutas, más clases.
- Con almacenes separados: **consistencia eventual**, proyecciones que mantener, posibilidad de leer datos "viejos".
- Sobreingeniería para aplicaciones **CRUD simples**.
- La curva de aprendizaje del equipo y el riesgo de aplicar el nivel 2 sin necesitarlo.

## Comparación

| Enfoque | Modelos | Consistencia | Cuándo |
|---|---|---|---|
| **CRUD clásico** | Uno | Inmediata | Apps simples, formularios |
| **CQS** (Meyer) | Métodos separados en la misma clase | Inmediata | Principio de diseño de código, siempre recomendable |
| **CQRS lógico** | Dos (clases), misma BD | Inmediata | Dominios con lógica de negocio real |
| **CQRS con almacenes separados** | Dos (clases y BD) | Eventual | Cargas muy asimétricas, búsquedas complejas, alta escala |

## Puntos clave

- Comandos **cambian** estado y no devuelven datos; consultas **devuelven** datos y no cambian nada.
- No requiere dos bases de datos ni Event Sourcing.
- El valor principal está en tener **modelos distintos** para escribir y leer.
- Al separar almacenes aparece la **consistencia eventual**.

## Errores comunes

- Empezar por dos bases de datos y sincronización antes de necesitarlo.
- Comandos que devuelven entidades completas (mezclan responsabilidades).
- Consultas que pasan por el Agregado y el repositorio en lugar de leer directo.
- Creer que CQRS = Event Sourcing.
- Aplicarlo a una aplicación CRUD de 5 tablas.

## 🎯 Para entrevistas y exámenes

- *"¿CQRS necesita dos bases de datos?"* → **No.** Es una separación de modelos; los almacenes separados son opcionales.
- *"¿Qué problema aparece al separar almacenes?"* → Consistencia eventual.
- *"¿Relación con Event Sourcing?"* → Se complementan, pero son independientes.
- *"¿Qué es CQS?"* → El principio original de Meyer: un método o cambia estado o devuelve datos, nunca ambos.

## Referencias

- Greg Young, *CQRS Documents* (2010): https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf
- Martin Fowler, *CQRS*: https://martinfowler.com/bliki/CQRS.html
- Microsoft Learn, *CQRS pattern*: https://learn.microsoft.com/azure/architecture/patterns/cqrs

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
