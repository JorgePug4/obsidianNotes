---
tags: [arquitectura, diseño, solid, dotnet, clean-code]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [SOLID]
---

# Principios SOLID

> [!info] Cinco principios de diseño orientado a objetos recopilados por **Robert C. Martin** (años 2000); el acrónimo lo acuñó Michael Feathers. Son la base conceptual de [[Clean Architecture]].

## ¿Qué es?

**SOLID** es un acrónimo de cinco principios que ayudan a escribir código **fácil de mantener, extender y probar**, reduciendo el acoplamiento y aumentando la cohesión.

| Letra | Principio | En una frase |
|---|---|---|
| **S** | Single Responsibility | Una clase debe tener **una sola razón para cambiar** |
| **O** | Open/Closed | Abierta a **extensión**, cerrada a **modificación** |
| **L** | Liskov Substitution | Una subclase debe poder **sustituir** a su clase base sin romper nada |
| **I** | Interface Segregation | Interfaces **pequeñas y específicas**, no una gigante |
| **D** | Dependency Inversion | Depender de **abstracciones**, no de implementaciones concretas |

## ¿Para qué sirve?

Sin estos principios, el código tiende a la **rigidez** (un cambio obliga a tocar muchas partes), la **fragilidad** (un cambio rompe cosas no relacionadas) y la **inmovilidad** (no se puede reutilizar nada sin arrastrar dependencias). SOLID ataca esas tres enfermedades.

## Conceptos relacionados

- [[Clean Architecture]] → la Regla de Dependencia es el principio **D** aplicado a toda la arquitectura.
- [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] → Agregados y Value Objects bien diseñados aplican **S** y encapsulación.
- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → separar comandos y consultas es una aplicación de **S** e **I**.
- [[Líder Técnico (Tech Lead)]] → promover estos principios en las revisiones de código es parte del rol.

## ¿Cómo funciona? Principio por principio

### S — Single Responsibility Principle (SRP)

> "Una clase debe tener una, y solo una, razón para cambiar."

La "razón para cambiar" es un **actor** o área del negocio. Si Contabilidad y Recursos Humanos pueden pedir cambios en la misma clase, tiene dos responsabilidades.

```csharp
// ❌ Tres razones para cambiar: reglas de negocio, persistencia y notificaciones
public class Pedido
{
    public decimal CalcularTotal() { /* ... */ }
    public void GuardarEnBaseDeDatos() { /* SQL aquí */ }
    public void EnviarEmailConfirmacion() { /* SMTP aquí */ }
}

// ✅ Cada clase, una responsabilidad
public class Pedido               { public decimal CalcularTotal() { /* ... */ } }
public class PedidoRepository     { public void Guardar(Pedido p) { /* ... */ } }
public class NotificadorPedidos   { public void EnviarConfirmacion(Pedido p) { /* ... */ } }
```

> [!warning] SRP no significa "una clase hace una sola cosa pequeña"
> Significa que responde a **un solo actor**. Una clase con 10 métodos cohesionados sobre el mismo concepto cumple SRP; una clase con 2 métodos que sirven a departamentos distintos no.

### O — Open/Closed Principle (OCP)

> "Las entidades de software deben estar abiertas a extensión pero cerradas a modificación." (Bertrand Meyer)

Añadir un comportamiento nuevo debería hacerse **añadiendo código**, no editando el existente que ya funciona y está probado.

```csharp
// ❌ Cada nuevo método de pago obliga a modificar esta clase
public decimal CalcularComision(string metodo, decimal monto) => metodo switch
{
    "tarjeta" => monto * 0.03m,
    "paypal"  => monto * 0.04m,
    _ => throw new NotSupportedException()
};

// ✅ Nuevo método de pago = nueva clase, sin tocar las existentes
public interface IMetodoPago { decimal CalcularComision(decimal monto); }
public class PagoTarjeta : IMetodoPago { public decimal CalcularComision(decimal m) => m * 0.03m; }
public class PagoPayPal  : IMetodoPago { public decimal CalcularComision(decimal m) => m * 0.04m; }
public class PagoCripto  : IMetodoPago { public decimal CalcularComision(decimal m) => m * 0.01m; } // añadido
```

Herramientas: polimorfismo, patrón Strategy, inyección de dependencias.

### L — Liskov Substitution Principle (LSP)

> "Si S es un subtipo de T, los objetos de tipo T pueden reemplazarse por objetos de tipo S sin alterar el comportamiento correcto del programa." (Barbara Liskov, 1987)

Una subclase **no debe** lanzar excepciones inesperadas, reforzar precondiciones ni debilitar postcondiciones respecto a la clase base.

```csharp
// ❌ Viola LSP: un Cuadrado "es un" Rectángulo matemáticamente, pero no en comportamiento
public class Rectangulo
{
    public virtual int Ancho { get; set; }
    public virtual int Alto  { get; set; }
    public int Area => Ancho * Alto;
}
public class Cuadrado : Rectangulo
{
    public override int Ancho { set { base.Ancho = value; base.Alto = value; } }
    public override int Alto  { set { base.Ancho = value; base.Alto = value; } }
}
// Quien reciba un Rectangulo, ponga Ancho=2, Alto=3 y espere Area=6 obtendrá 9.

// ✅ Modelar sin herencia forzada
public interface IFigura { int Area { get; } }
public record Rectangulo(int Ancho, int Alto) : IFigura { public int Area => Ancho * Alto; }
public record Cuadrado(int Lado) : IFigura { public int Area => Lado * Lado; }
```

Señal clásica de violación: un método que hace `if (obj is TipoConcreto)` o una implementación que lanza `NotImplementedException`.

### I — Interface Segregation Principle (ISP)

> "Ningún cliente debería verse forzado a depender de métodos que no usa."

Es preferible tener varias interfaces pequeñas y específicas que una "interfaz gorda".

```csharp
// ❌ Una impresora básica se ve obligada a implementar Fax y Escanear
public interface IMultifuncion { void Imprimir(); void Escanear(); void EnviarFax(); }

// ✅ Interfaces por capacidad; cada clase implementa las que le corresponden
public interface IImpresora { void Imprimir(); }
public interface IEscaner   { void Escanear(); }
public interface IFax       { void EnviarFax(); }

public class ImpresoraBasica : IImpresora { /* ... */ }
public class Multifuncion : IImpresora, IEscaner, IFax { /* ... */ }
```

En .NET, `IReadOnlyList<T>` frente a `IList<T>`, o `IAsyncEnumerable<T>` son ejemplos de segregación.

### D — Dependency Inversion Principle (DIP)

> "Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones. Las abstracciones no deben depender de detalles; los detalles deben depender de abstracciones."

```csharp
// ❌ El caso de uso (alto nivel) depende de SQL Server (detalle)
public class CrearPedidoHandler
{
    private readonly SqlServerPedidoRepository _repo = new();  // acoplado
}

// ✅ Ambos dependen de la abstracción; la implementación se inyecta
public interface IPedidoRepository { Task GuardarAsync(Pedido p); }

public class CrearPedidoHandler(IPedidoRepository repo)   // solo conoce la interfaz
{
    public Task Handle(CrearPedidoCommand cmd) => repo.GuardarAsync(Pedido.Crear(cmd));
}

public class SqlServerPedidoRepository : IPedidoRepository { /* EF Core */ }

// Program.cs: la composición ocurre en el borde
builder.Services.AddScoped<IPedidoRepository, SqlServerPedidoRepository>();
```

> [!important] DIP ≠ Inyección de Dependencias
> **DIP** es el *principio* (depender de abstracciones). **Inyección de dependencias (DI)** es una *técnica* para cumplirlo (pasar las dependencias desde fuera). **Contenedor de DI** (el de ASP.NET Core, Autofac) es una *herramienta* que automatiza la técnica. Se pueden cumplir DIP sin contenedor.

## Ventajas

- Código más **fácil de probar** (dependencias sustituibles por dobles de prueba).
- Cambios **localizados**: menos efecto dominó.
- Facilita el trabajo en **paralelo** de varios desarrolladores.
- Base para arquitecturas limpias y sistemas extensibles.

## Desventajas / Limitaciones

- **Sobreingeniería**: una interfaz por cada clase "por si acaso", capas de abstracción que nadie necesita.
- **Más archivos y más indirección**: seguir el flujo puede ser más difícil para quien llega nuevo.
- Aplicar OCP de forma prematura para variaciones que **nunca llegan**.
- Son **heurísticas**, no leyes: en scripts, prototipos o código trivial pueden ignorarse.

## Comparación con otros principios

| Principio | Idea | Relación con SOLID |
|---|---|---|
| **DRY** (Don't Repeat Yourself) | No duplicar **conocimiento** | Complementa SRP; duplicar código a veces es preferible a un acoplamiento incorrecto |
| **KISS** (Keep It Simple) | La solución más simple que funcione | Freno contra la sobreingeniería de SOLID |
| **YAGNI** (You Aren't Gonna Need It) | No construir lo que no se necesita hoy | Freno contra aplicar OCP prematuramente |
| **Ley de Demeter** | Habla solo con tus vecinos directos (`a.B().C().D()` es sospechoso) | Reduce acoplamiento como ISP y DIP |
| **Composición sobre herencia** | Preferir componer objetos a heredar | Evita muchas violaciones de LSP |

## Puntos clave

- **S**: una razón para cambiar (un actor).
- **O**: extender añadiendo código, no editando.
- **L**: las subclases no rompen el contrato de la base.
- **I**: interfaces pequeñas por capacidad.
- **D**: alto nivel y bajo nivel dependen de abstracciones.
- Aplicarlos con criterio: **KISS y YAGNI** son el contrapeso.

## Errores comunes

- Confundir SRP con "métodos de 5 líneas".
- Crear `IServicioX` para cada `ServicioX` aunque solo exista una implementación y nunca se pruebe con dobles.
- Heredar para reutilizar código en lugar de componer (viola LSP).
- Creer que usar el contenedor de DI de ASP.NET Core ya garantiza DIP.
- Un `switch` sobre tipos que crece cada sprint (viola OCP).

## 🎯 Para entrevistas y exámenes

- Tener **un ejemplo de código propio** para cada letra; el del Rectángulo/Cuadrado para LSP es el clásico.
- *"Diferencia entre DIP e inyección de dependencias"* → Principio vs técnica.
- *"¿Qué principio viola una clase con 40 métodos que usan 5 clientes distintos?"* → ISP (y probablemente SRP).
- *"¿Cómo se relaciona SOLID con Clean Architecture?"* → La Regla de Dependencia es DIP a escala de sistema.

## Referencias

- Robert C. Martin, *Agile Software Development: Principles, Patterns, and Practices* (2002) y *Clean Architecture* (2017, parte III)
- Robert C. Martin, *The Principles of OOD*: http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod
- Barbara Liskov, *Data Abstraction and Hierarchy* (1987)
- Microsoft Learn, *Architectural principles*: https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/architectural-principles

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
