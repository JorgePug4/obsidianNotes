---
tags: [dotnet, csharp, async, rendimiento]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [ValueTask, Task vs ValueTask]
---

# 🚀 Task vs ValueTask en .NET

> [!info] `ValueTask<T>` llegó con .NET Core 2.0 (2017); la versión no genérica `ValueTask` e `IValueTaskSource<T>` con .NET Core 2.1. Es una **optimización**, no un reemplazo de `Task`.

## ¿Qué es?

Al diseñar sistemas backend en .NET de alto rendimiento, la elección entre `Task` y `ValueTask` importa. La diferencia fundamental está en la **asignación de memoria** y la **presión sobre el recolector de basura (GC)**.

### `Task` y `Task<T>`: el estándar (tipo de referencia)

`Task` es una **`class`**. Es el modelo asíncrono estándar desde la introducción de `async`/`await` (C# 5, 2012).

- **Asignación**: cada `Task` que no se completa de forma síncrona (o cuyo resultado no está en la caché interna del compilador) se reserva en el **heap** administrado.
- **El costo**: en APIs de alto tráfico, miles de asignaciones por segundo generan presión en el GC (generación 0), lo que provoca micro-pausas y aumenta la latencia en la cola (p99).
- **La flexibilidad**: al ser un objeto, permite **múltiples `await`**, `await` desde varios hilos, `.Result`/`.Wait()` y todos los combinadores (`Task.WhenAll`, `Task.WhenAny`).

### `ValueTask` y `ValueTask<T>`: el optimizador (tipo de valor)

`ValueTask<T>` es un **`struct`**. Fue introducido para optimizar escenarios donde una operación asíncrona **a menudo se completa de forma síncrona**.

- **El beneficio**: si el método termina síncronamente, **no hay asignación en el heap**; el resultado viaja dentro del propio `struct`.
- **Estructura interna**: es una **unión discriminada** que envuelve **una** de tres cosas: un resultado `T` ya disponible, un `Task<T>` (cuando la operación sí es asíncrona), o un `IValueTaskSource<T>` (objeto reutilizable de un *pool*, usado por librerías de bajo nivel como `System.IO.Pipelines` y sockets).

## ¿Para qué sirve?

Reducir asignaciones en **rutas críticas (*hot paths*)** que se ejecutan millones de veces y que **la mayoría de las veces no necesitan esperar**: lecturas que dan en caché, lecturas de un buffer ya lleno, validaciones que casi siempre pasan.

## Conceptos relacionados

- [[Go routines|Goroutines y Channels]] → el modelo de concurrencia de Go; comparación mental útil: `async`/`await` en .NET es cooperativo sobre el *thread pool*, las goroutines son hilos verdes gestionados por el runtime.
- [[Circuit Breaker]], [[Retry con Backoff Exponencial]] → las pipelines de Polly devuelven `ValueTask` internamente por rendimiento.
- [[Clean Architecture]] → las interfaces de repositorio en la capa de Aplicación suelen exponer `Task`; reserva `ValueTask` para implementaciones con caché.

## ¿Cómo funciona? Caso de uso clásico: la capa de caché

`ValueTask` brilla cuando una operación asíncrona **a menudo devuelve resultados de forma síncrona**, por ejemplo desde una caché en memoria.

```csharp
// ❌ CON TASK:
// Asigna un Task<User> en el heap INCLUSO SI el dato está en caché.
public async Task<User?> GetUserAsync(int id)
{
    if (_cache.TryGetValue(id, out User? user))
        return user; // el compilador tiene que envolverlo en un Task<User> nuevo

    user = await _dbContext.Users.FindAsync(id);
    if (user is not null) _cache.Set(id, user);
    return user;
}

// ✅ CON VALUETASK:
// CERO asignaciones en el heap si el dato está en caché.
public async ValueTask<User?> GetUserAsync(int id)
{
    if (_cache.TryGetValue(id, out User? user))
        return user; // viaja dentro del struct; sin asignación

    user = await _dbContext.Users.FindAsync(id); // asíncrono de verdad: aquí sí se asigna
    if (user is not null) _cache.Set(id, user);
    return user;
}
```

> [!note] Matices sobre las asignaciones
> - El compilador **cachea** algunos `Task<T>` completados: `Task<bool>` (`true`/`false`), `Task<int>` para valores pequeños y `Task.CompletedTask`. Para un `Task<User>` no hay caché, así que la asignación ocurre.
> - Cuando un método `async ValueTask<T>` **sí** espera (no completa síncronamente), internamente crea un `Task<T>` igual que siempre. El ahorro solo existe en el camino síncrono. Desde .NET 6 existe `PoolingAsyncValueTaskMethodBuilder`, que reutiliza objetos también en el camino asíncrono, pero es una optimización avanzada que hay que medir.
> - `FindAsync` de EF Core ya devuelve `ValueTask<T>` exactamente por esta razón: si la entidad está en el *change tracker*, la devuelve sin ir a la base de datos.

### Sin `async`: devolver el resultado directamente

Si el método no necesita `await` en el camino rápido, evita también la máquina de estados:

```csharp
public ValueTask<User?> GetUserAsync(int id)
{
    return _cache.TryGetValue(id, out User? user)
        ? new ValueTask<User?>(user)           // síncrono: cero asignaciones
        : new ValueTask<User?>(LoadFromDbAsync(id)); // asíncrono: envuelve el Task
}
```

## ⚠️ Los peligros de `ValueTask` (anti-patrones)

Debido a que `ValueTask` puede envolver un objeto **reutilizado de un pool** (`IValueTaskSource`), hay reglas estrictas. Romperlas produce **comportamiento indefinido**, no una excepción clara.

> [!danger] Lo que NUNCA debes hacer con un `ValueTask`
> 1. **Esperarlo más de una vez.** Tras el primer `await`, el objeto subyacente puede haberse reciclado. Un segundo `await` puede devolver el resultado de **otra operación**.
> 2. **Esperarlo desde varios hilos a la vez.** No está diseñado para múltiples consumidores.
> 3. **Bloquear con `.Result` o `.GetAwaiter().GetResult()`** sin comprobar antes `IsCompleted`. Con `Task` es un bloqueo (malo pero definido); con `ValueTask` puede ser indefinido.
> 4. **Guardarlo en un campo o lista para consumirlo después** sin convertirlo.

### ¿Necesitas funcionalidad de `Task`?

Si necesitas hacer cualquiera de las cosas anteriores, **convierte primero**:

```csharp
ValueTask<int> vt1 = GetValueAsync();
ValueTask<int> vt2 = GetValueAsync();

// ✅ Convertir a Task para usar combinadores
await Task.WhenAll(vt1.AsTask(), vt2.AsTask());

// ✅ Si necesitas esperarlo varias veces, "congela" el resultado
ValueTask<int> reutilizable = GetValueAsync().Preserve();
var a = await reutilizable;
var b = await reutilizable; // seguro tras Preserve()
```

- `AsTask()`: devuelve el `Task` interno o crea uno nuevo. Si el `ValueTask` ya se completó síncronamente, **asigna** (pierdes la optimización, pero recuperas la seguridad).
- `Preserve()`: devuelve un `ValueTask` seguro para múltiples `await`.

## 📊 Comparativa rápida

| Característica | `Task<T>` | `ValueTask<T>` |
|---|---|---|
| **Tipo** | `class` (referencia) | `struct` (valor) |
| **Asignación en heap** | Siempre (salvo resultados cacheados) | Solo si la operación es realmente asíncrona |
| **Tamaño** | Referencia de 8 bytes | 16 bytes (dos campos) → copiar cuesta un poco más |
| **Múltiples `await`** | ✅ Seguro | ❌ Indefinido (usar `Preserve()`) |
| **`await` concurrente** | ✅ Seguro | ❌ No |
| **Bloqueo (`.Result`)** | Permitido (desaconsejado) | ❌ Solo si `IsCompleted` |
| **Combinadores (`WhenAll`)** | ✅ Nativo | Requiere `AsTask()` |
| **Legibilidad / familiaridad** | Alta | Media; el equipo debe conocer las reglas |
| **Opción por defecto** | ✅ Sí | Solo tras medir |

## 🏗️ Regla general de arquitectura

1. **Por defecto usa `Task<T>`.** Es más seguro y fácil de razonar. El GC de .NET es muy bueno con objetos pequeños de corta vida.
2. **Usa `ValueTask<T>` solo cuando**:
   - el *profiling* (dotnet-counters, PerfView, BenchmarkDotNet) muestre que las asignaciones de `Task` en un *hot path* son un cuello de botella real, **y**
   - el método se complete **síncronamente la mayoría de las veces** (caché, buffers).
3. En **interfaces públicas** de librerías de bajo nivel (streams, pipelines, serializadores) `ValueTask` es razonable porque los implementadores pueden necesitar la optimización. En **interfaces de aplicación** (repositorios, servicios) suele ser ruido.
4. Documenta el uso de `ValueTask` para que quien consuma el método conozca las reglas.

> [!tip] Referencia obligada
> El artículo de Stephen Toub, *Understanding the Whys, Whats, and Whens of ValueTask*, sigue siendo la explicación definitiva y la fuente de estas reglas.

## Puntos clave

- `Task` = clase, siempre en el heap, flexible y seguro.
- `ValueTask` = struct, sin asignación **si completa síncronamente**, con reglas estrictas.
- **Un solo `await`**, nunca concurrente, nunca bloquear sin `IsCompleted`.
- `AsTask()` para combinadores; `Preserve()` para reutilizar.
- **Por defecto `Task`; `ValueTask` solo tras medir.**

## Errores comunes

- Cambiar todas las firmas a `ValueTask` "porque es más rápido" sin medir (y perder rendimiento por las copias del struct y los `AsTask()` posteriores).
- `await` dos veces el mismo `ValueTask` en un bucle de reintentos.
- Guardar `ValueTask` en una `List<ValueTask<T>>` y luego esperarlos "todos": hay que convertirlos con `AsTask()` primero.
- Usar `.Result` sobre un `ValueTask` en código síncrono.

## 🎯 Para entrevistas y exámenes

- *"¿Cuándo usarías `ValueTask`?"* → Hot paths donde la operación suele completarse síncronamente (caché) y el profiling muestra presión de GC.
- *"¿Cuál es el riesgo de `ValueTask`?"* → Esperarlo más de una vez o concurrentemente puede dar resultados indefinidos por el pooling de `IValueTaskSource`.
- *"¿Por qué `FindAsync` de EF Core devuelve `ValueTask`?"* → Porque suele resolver desde el *change tracker* sin ir a la BD.
- *"¿Cómo usas `Task.WhenAll` con `ValueTask`?"* → `AsTask()`.

## Referencias

- Stephen Toub, *Understanding the Whys, Whats, and Whens of ValueTask* (.NET Blog, 2018): https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/
- Microsoft Learn, `ValueTask<TResult>` Struct: https://learn.microsoft.com/dotnet/api/system.threading.tasks.valuetask-1
- Microsoft Learn, *Async guidance* (David Fowler): https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
