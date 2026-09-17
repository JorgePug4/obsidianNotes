---
tags: [arquitectura, clean-architecture, dotnet, diseño]
up: "[[🗺️ Índice - Ingeniería de Software]]"
---

# Clean Architecture

> [!info] Propuesta por **Robert C. Martin (Uncle Bob)** en un artículo de 2012 y desarrollada en el libro *Clean Architecture* (2017). Es una síntesis de ideas anteriores: Arquitectura Hexagonal (Cockburn, 2005), Onion Architecture (Palermo, 2008) y DCI/BCE.

## ¿Qué es?

La **Clean Architecture** es un estilo de arquitectura de software cuyo objetivo es que **las reglas de negocio sean el centro de la aplicación** y que todo lo demás (frameworks, bases de datos, UI, APIs externas) sean **detalles reemplazables** que dependen del centro, y no al revés.

Busca sistemas:

- **Independientes de frameworks**: el framework es una herramienta, no la estructura.
- **Testeables**: las reglas de negocio se prueban sin UI, BD ni servidor web.
- **Independientes de la UI**: se puede cambiar una web por una consola sin tocar el negocio.
- **Independientes de la base de datos**: cambiar SQL Server por PostgreSQL o MongoDB no afecta al dominio.
- **Independientes de agentes externos**: el negocio no sabe nada del mundo exterior.

## ¿Para qué sirve?

- Proteger la lógica de negocio, **lo más valioso y duradero** del sistema, de los cambios tecnológicos, que son frecuentes.
- Facilitar **pruebas unitarias rápidas** del dominio y los casos de uso.
- Permitir que equipos trabajen en capas distintas con **contratos claros**.
- Servir de base para aplicar [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] y [[CQRS (Command Query Responsibility Segregation)|CQRS]].

## Conceptos relacionados

- [[Principios SOLID]] → la **Regla de Dependencia** es la Inversión de Dependencias (la D de SOLID) aplicada a toda la arquitectura.
- [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] → DDD dice *qué* modelar en el centro (Entidades, Agregados, Value Objects); Clean Architecture dice *cómo aislarlo*.
- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → los *handlers* de comandos y consultas son los casos de uso de la capa de Aplicación.
- [[🏗️ Diseño de Microservicios]] → cada microservicio suele estructurarse internamente con Clean Architecture.
- [[Bases de Datos SQL vs NoSQL]] → la base de datos es un detalle de Infraestructura; la elección no debe filtrarse al dominio.

## ¿Cómo funciona?

### Los círculos concéntricos

```
┌─────────────────────────────────────────────────────────────┐
│  Frameworks & Drivers   (Web, BD, UI, dispositivos)         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Interface Adapters   (Controllers, Presenters,       │  │
│  │                        Repositories, Gateways)        │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  Use Cases / Application   (reglas de la app)   │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │  Entities / Domain   (reglas de negocio)  │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
          Las dependencias SIEMPRE apuntan hacia adentro →
```

#### 1. Entities (Entidades / Dominio)

Contienen las **reglas de negocio de la empresa**, válidas incluso si no existiera la aplicación: una `Orden` no puede tener total negativo; un `Pedido` solo se cancela si no ha sido enviado.

- Ejemplos: `User`, `Order`, `Product`, `Money`.
- Son objetos del dominio **puro**: sin atributos del ORM, sin referencias a HTTP ni a la BD.
- En DDD corresponden a Entidades, Value Objects, Agregados, Domain Events e interfaces de Repositorio.

#### 2. Use Cases (Casos de uso / Aplicación)

Contienen las **reglas de negocio de la aplicación**: qué hace el sistema, orquestando entidades.

- Ejemplos: `CreateOrder`, `ProcessPayment`, `AuthenticateUser`.
- Reciben un *input* (DTO/comando), cargan entidades vía interfaces de repositorio, ejecutan la lógica y devuelven un *output*.
- **Definen las interfaces** que necesitan (`IOrderRepository`, `IEmailSender`, `IClock`) pero **no las implementan**.

#### 3. Interface Adapters (Adaptadores)

Traducen entre el formato conveniente para los casos de uso y el formato conveniente para el mundo exterior.

- **Controllers** (HTTP → comando), **Presenters/ViewModels** (resultado → JSON/HTML), **Repositories** (interfaz del dominio → SQL/EF Core), **Gateways** a APIs externas.
- Aquí viven las implementaciones de las interfaces definidas en el centro.

#### 4. Frameworks & Drivers

La capa más externa: **ASP.NET Core, Entity Framework, SQL Server, RabbitMQ, la UI**. Son detalles; el código aquí es mínimo y de configuración ("pegamento").

### La Regla de Dependencia (Dependency Rule)

> **El código de un círculo interno no puede saber nada de un círculo externo.** Ni nombres de clases, ni funciones, ni tipos de datos, ni frameworks.

```
Frameworks → Adapters → Use Cases → Entities
```

¿Cómo llama entonces un caso de uso a la base de datos, si no puede conocerla? Con **Inversión de Dependencias**: el caso de uso define la interfaz `IOrderRepository`; la Infraestructura la implementa con EF Core; la inyección de dependencias conecta ambas en tiempo de ejecución. El flujo de control va hacia fuera, pero la **dependencia del código fuente** apunta hacia dentro.

### Cruzar fronteras: DTOs

Los datos que cruzan una frontera deben ser **estructuras simples** (DTOs, records). Nunca pasar una entidad de EF Core al controlador, ni un `HttpRequest` a un caso de uso.

## Ejemplo en .NET

Estructura de solución habitual (plantillas de Jason Taylor, Ardalis y Milan Jovanović siguen este esquema):

```
MiTienda.sln
├── src/
│   ├── MiTienda.Domain/          ← Entities, Value Objects, Domain Events,
│   │                                interfaces de repositorio. SIN dependencias.
│   ├── MiTienda.Application/     ← Casos de uso (Commands/Queries + Handlers),
│   │                                DTOs, validadores, interfaces (IEmailSender…).
│   │                                Depende SOLO de Domain.
│   ├── MiTienda.Infrastructure/  ← EF Core DbContext, repositorios, email,
│   │                                clientes HTTP. Depende de Application y Domain.
│   └── MiTienda.WebApi/          ← Controllers / Minimal APIs, Program.cs,
│                                    DI. Depende de Application e Infrastructure.
└── tests/
    ├── MiTienda.Domain.Tests/
    └── MiTienda.Application.Tests/
```

```csharp
// Domain ─────────────────────────────────────────────
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id, CancellationToken ct);
    Task AddAsync(Order order, CancellationToken ct);
}

// Application ────────────────────────────────────────
public record CreateOrderCommand(Guid CustomerId, List<OrderLineDto> Lines);

public class CreateOrderHandler(IOrderRepository repo, IUnitOfWork uow)
{
    public async Task<Guid> Handle(CreateOrderCommand cmd, CancellationToken ct)
    {
        var order = Order.Create(cmd.CustomerId, cmd.Lines); // regla de negocio en Domain
        await repo.AddAsync(order, ct);
        await uow.SaveChangesAsync(ct);
        return order.Id;
    }
}

// Infrastructure ─────────────────────────────────────
public class EfOrderRepository(AppDbContext db) : IOrderRepository
{
    public Task<Order?> GetByIdAsync(Guid id, CancellationToken ct) =>
        db.Orders.Include(o => o.Lines).FirstOrDefaultAsync(o => o.Id == id, ct);

    public async Task AddAsync(Order order, CancellationToken ct) =>
        await db.Orders.AddAsync(order, ct);
}

// WebApi (Program.cs) ────────────────────────────────
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>();
```

Un test del handler usa un `IOrderRepository` en memoria: **sin base de datos, sin servidor web, en milisegundos**.

## Ventajas

- Lógica de negocio **aislada, testeable y duradera**.
- Cambiar tecnología (ORM, BD, framework web) afecta solo a la capa externa.
- Estructura **predecible**: todo el equipo sabe dónde va cada cosa.
- Encaja con DDD, CQRS y microservicios.

## Desventajas / Limitaciones

- **Más proyectos, más clases, más mapeos** (DTO ↔ entidad). Para un CRUD pequeño es sobreingeniería.
- Curva de aprendizaje y **disciplina**: la tentación de "solo esta vez" referenciar EF Core desde el dominio.
- Riesgo de **abstracciones vacías**: repositorios que solo envuelven `DbSet` sin aportar nada.
- No resuelve por sí sola un **dominio anémico**: si las entidades son bolsas de getters/setters y la lógica está en servicios, la estructura no ayuda.

## Comparación

| Arquitectura | Autor / año | Idea central | Relación |
|---|---|---|---|
| **Capas tradicional (N-Tier)** | — | UI → Lógica → Datos; las dependencias apuntan **hacia la BD** | El dominio depende de la infraestructura: es lo que Clean invierte |
| **Hexagonal (Ports & Adapters)** | Cockburn, 2005 | El núcleo expone **puertos** (interfaces) y los **adaptadores** los implementan | Misma idea; Clean añade la separación Entities / Use Cases |
| **Onion** | Palermo, 2008 | Capas concéntricas con el dominio en el centro | Prácticamente equivalente a Clean |
| **Clean Architecture** | Martin, 2012 | Síntesis de las anteriores + Regla de Dependencia explícita | — |
| **Vertical Slice** | Bogard, 2018 | Organizar por **funcionalidad** (cada *feature* con su request, handler y acceso a datos) en lugar de por capa | Alternativa o complemento: Vertical Slices *dentro* de la capa Application |

> [!tip] Hexagonal, Onion y Clean son la misma idea con distinto vocabulario
> En una entrevista, lo importante es explicar la **inversión de dependencias** y por qué el dominio no conoce la infraestructura.

## Puntos clave

- El **dominio en el centro**; frameworks y BD en el borde.
- **Regla de Dependencia**: el código apunta siempre hacia adentro.
- Se logra con **interfaces definidas dentro e implementadas fuera** + inyección de dependencias.
- 4 capas típicas en .NET: **Domain → Application → Infrastructure → WebApi**.
- Los datos cruzan fronteras como **DTOs** simples.

## Errores comunes

- Referenciar `Microsoft.EntityFrameworkCore` desde el proyecto Domain (atributos `[Table]`, `DbContext`).
- Devolver entidades de dominio directamente desde los controladores.
- Poner lógica de negocio en los controladores o en los repositorios.
- Crear una interfaz por cada clase "por si acaso", sin necesidad real.
- Aplicarla a un microservicio de 3 endpoints CRUD.

## 🎯 Para entrevistas y exámenes

- *"¿Qué es la Regla de Dependencia?"* → El código interno nunca depende del externo.
- *"¿Cómo accede el caso de uso a la BD sin conocerla?"* → Interfaz en el centro + implementación fuera + DI (Inversión de Dependencias).
- *"Diferencia entre Entities y Use Cases"* → Reglas de negocio de la empresa vs reglas de la aplicación.
- *"Clean vs Hexagonal vs Onion"* → Misma filosofía, vocabulario distinto.
- *"¿Cuándo no usarías Clean Architecture?"* → CRUD simple, prototipos, scripts.

## Referencias

- Robert C. Martin, *The Clean Architecture* (2012): https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- Robert C. Martin, *Clean Architecture: A Craftsman's Guide to Software Structure and Design* (Prentice Hall, 2017)
- Alistair Cockburn, *Hexagonal architecture*: https://alistair.cockburn.us/hexagonal-architecture/
- Jason Taylor, *Clean Architecture Solution Template* (.NET): https://github.com/jasontaylordev/CleanArchitecture
- Microsoft Learn, *Common web application architectures*: https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
