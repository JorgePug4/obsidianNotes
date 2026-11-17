---
tags: [dotnet, csharp, async, nivel/6, excepciones]
up: "[[00 - Async en .NET - Índice]]"
---

# 06 - Manejo de excepciones

> Nivel 🟡 6 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[04 - Composición de tareas]], [[05 - Cancelación y timeouts]].

## Contenido
- [[#Dónde vive una excepción async]]
- [[#await vs .Result vs .Wait() vs GetAwaiter().GetResult()]]
- [[#Excepciones con WhenAll]]
- [[#Excepciones no observadas]]
- [[#async void y el proceso que muere]]
- [[#Fire-and-forget seguro]]
- [[#finally, using y await]]
- [[#Pilas de llamadas async]]
- [[#🧠 Practica]]

---

## Dónde vive una excepción async

En un método `async`, una excepción **no se lanza al llamar** al método: se **guarda dentro del `Task`** (estado `Faulted`) y se **relanza al hacer `await`**.

```csharp
Task t = FailAsync();          // no lanza aquí
Console.WriteLine("sigo");     // se imprime
await t;                       // ← InvalidOperationException se lanza AQUÍ

static async Task FailAsync()
{
    await Task.Yield();
    throw new InvalidOperationException("boom");
}
```

Consecuencia: **si nunca haces `await` (ni observas) un `Task` que falla, la excepción se pierde silenciosamente.**

---

## await vs .Result vs .Wait() vs GetAwaiter().GetResult()

| Forma | ¿Bloquea el hilo? | Excepción que ves | Uso |
|---|---|---|---|
| `await t` | No | La **original** (primera si hay varias) | ✅ Siempre que puedas |
| `t.Result` / `t.Wait()` | Sí | `AggregateException` que la envuelve | ❌ Evitar (deadlocks, starvation) |
| `t.GetAwaiter().GetResult()` | Sí | La **original** | ⚠️ Solo cuando bloquear es inevitable (p. ej. un `Main` antiguo, un constructor de un framework sin async). Sigue siendo *sync-over-async* |

```csharp
try { var x = task.Result; }
catch (AggregateException ae) { var real = ae.InnerException; }   // incómodo

try { await task; }
catch (HttpRequestException ex) { /* directo */ }
```

---

## Excepciones con WhenAll

`Task.WhenAll` guarda **todas** las excepciones en `AggregateException`, pero `await` relanza **solo la primera**:

```csharp
Task a = Task.Run(() => throw new InvalidOperationException("A"));
Task b = Task.Run(() => throw new ArgumentException("B"));
Task all = Task.WhenAll(a, b);

try
{
    await all;
}
catch (Exception first)
{
    Console.WriteLine(first.Message);              // "A": la primera según el orden de las tareas pasadas a WhenAll

    // Para verlas todas, mira la tarea combinada:
    foreach (var ex in all.Exception!.InnerExceptions)
        Console.WriteLine($"- {ex.GetType().Name}: {ex.Message}");
}
```

Patrón alternativo: inspeccionar cada tarea individual tras `WhenAll`:

```csharp
var tasks = ids.Select(id => ProcessAsync(id, ct)).ToArray();
try { await Task.WhenAll(tasks); }
catch { /* ignorar aquí; se revisa abajo */ }

var failed = tasks.Where(t => t.IsFaulted).ToList();
var ok     = tasks.Where(t => t.IsCompletedSuccessfully).Select(t => t.Result).ToList();
_logger.LogWarning("{Failed} de {Total} fallaron", failed.Count, tasks.Length);
```

> [!tip] .NET 8+: `ConfigureAwaitOptions.SuppressThrowing`
> `await all.ConfigureAwait(ConfigureAwaitOptions.SuppressThrowing);` espera sin lanzar (solo para `Task` sin resultado). Después inspeccionas `all.Exception` o cada tarea. Evita el `try/catch` vacío.

---

## Excepciones no observadas

Si un `Task` falla y nadie lo observa (ni `await`, ni `.Exception`, ni `.Wait()`), cuando el GC lo recoge se dispara el evento `TaskScheduler.UnobservedTaskException`.

- En .NET Framework 4.0 esto **terminaba el proceso**. Desde .NET 4.5 y en todo .NET Core/.NET 5+, por defecto **no** lo termina: la excepción simplemente se pierde.
- Puedes suscribirte para **registrarlas** (diagnóstico), sabiendo que el evento llega tarde (cuando actúa el finalizador) y no siempre:

```csharp
TaskScheduler.UnobservedTaskException += (_, e) =>
{
    logger.LogError(e.Exception, "Excepción de Task no observada");
    e.SetObserved();
};
```

Esto es una red de seguridad, no una estrategia: la solución es **observar siempre** las tareas.

---

## async void y el proceso que muere

Una excepción no capturada en un método `async void` se relanza en el `SynchronizationContext` que había al empezar; si no había (ASP.NET Core, consola, servicios), se lanza en el **thread pool** → **excepción no controlada → el proceso termina**.

```csharp
// ❌ Este código puede tumbar toda tu API
public async void OnMessage(Message m)
{
    await _handler.HandleAsync(m); // si lanza, adiós proceso
}
```

Ni `AppDomain.CurrentDomain.UnhandledException` lo evita (solo te avisa antes de morir). Por eso: `async void` **solo** en manejadores de eventos y **con `try/catch` que cubra todo el cuerpo**.

---

## Fire-and-forget seguro

A veces quieres lanzar algo y no esperarlo (enviar una métrica, calentar una caché). Opciones, de peor a mejor:

```csharp
// ❌ 1. Descartar sin más: las excepciones se pierden, y en ASP.NET Core
//       los servicios scoped (DbContext) pueden estar ya liberados cuando se ejecute.
_ = SendTelemetryAsync(data);

// ⚠️ 2. Descartar con manejo de errores explícito (aceptable para cosas triviales, sin dependencias scoped)
_ = SafeFireAndForget(SendTelemetryAsync(data), _logger);

static async Task SafeFireAndForget(Task task, ILogger logger)
{
    try { await task; }
    catch (Exception ex) { logger.LogError(ex, "Fallo en tarea en segundo plano"); }
}

// ✅ 3. Encolar en una cola en memoria consumida por un BackgroundService
await _backgroundQueue.EnqueueAsync(new TelemetryWorkItem(data), ct);
```

La opción 3 da control de concurrencia, apagado ordenado y un *scope* de DI propio. Implementación completa en [[11 - Async en ASP.NET Core y servicios en segundo plano#Cola de trabajo en segundo plano|nivel 11]]. Si el trabajo **no puede perderse** (un correo de confirmación), ni siquiera eso basta: necesitas persistencia ([[Transactional Outbox]], [[02 - Queues y Topics|una cola de Service Bus]]).

> [!note] `Task.Run` en un controlador
> `Task.Run(() => DoWorkAsync())` dentro de un endpoint es un fire-and-forget con todos los problemas de la opción 1: si `DoWorkAsync` usa el `DbContext` de la petición, fallará con `ObjectDisposedException` cuando la petición termine.

---

## finally, using y await

`try/finally` y `using` funcionan con `await` como esperas: el `finally` se ejecuta cuando el método **realmente** termina (puede ser segundos después, en otro hilo).

```csharp
public async Task ImportAsync(string path, CancellationToken ct)
{
    await using var stream = File.OpenRead(path);   // DisposeAsync al terminar, aunque falle
    await _lock.WaitAsync(ct);
    try
    {
        await _importer.RunAsync(stream, ct);
    }
    finally
    {
        _lock.Release();                            // se ejecuta tras el await, ocurra lo que ocurra
    }
}
```

`await` está permitido dentro de `catch` y `finally` desde C# 6.

---

## Pilas de llamadas async

Desde .NET Core 2.1 las trazas de excepciones async son legibles: el runtime oculta los frames de la máquina de estados y muestra tus métodos en orden lógico. Aun así:
- Los métodos que **elidieron** `async` (devuelven el `Task` directamente) **no aparecen** en la traza.
- Para ver "quién está esperando a quién" en un proceso colgado, usa la ventana **Parallel Stacks / Tasks** de Visual Studio o `dotnet-dump` con `dumpasync` ([[10 - Rendimiento y diagnóstico#Herramientas de diagnóstico|nivel 10]]).

---

## Puntos clave

- La excepción vive en el `Task` y aparece en el `await`.
- `await` → excepción original; `.Result`/`.Wait()` → `AggregateException`.
- `WhenAll`: `await` muestra la primera; `task.Exception.InnerExceptions` todas.
- Las tareas no observadas pierden excepciones en silencio.
- `async void` sin `try/catch` puede **terminar el proceso**.
- Fire-and-forget: mejor una cola + `BackgroundService`; si no puede perderse, persistencia.

## Errores comunes

- `_ = DoAsync();` en código de negocio sin manejar errores.
- `catch (AggregateException)` alrededor de un `await` (nunca se lanzará ahí).
- Fire-and-forget que usa servicios *scoped* de la petición.
- Registrar solo la primera excepción de un `WhenAll` y perder las demás.

## 🎯 Para entrevistas y exámenes

- *"¿Qué pasa si un método async lanza y nadie hace await?"* → La excepción queda en el `Task`; si nunca se observa, se pierde (y se dispara `UnobservedTaskException` al recolectarlo). No tumba el proceso en .NET moderno.
- *"¿Y si es `async void`?"* → Se relanza en el contexto o el thread pool y normalmente termina el proceso.
- *"Con `await Task.WhenAll` y dos fallos, ¿qué ves?"* → Solo la primera excepción; todas están en `whenAllTask.Exception.InnerExceptions`.

## 🧠 Practica

1. Reproduce el ejemplo de `WhenAll` con dos fallos y muestra ambas excepciones.
2. Cambia `await` por `.Result` y observa el tipo de excepción.
3. En una app de consola, crea un `async void` que lance tras un `await Task.Delay(100)`, llama a él y haz `Console.ReadLine()`. Observa cómo muere el proceso.
4. Implementa `SafeFireAndForget` y pruébalo.

## Referencias

- Microsoft Learn, *Exception handling (Task Parallel Library)*: https://learn.microsoft.com/dotnet/standard/parallel-programming/exception-handling-task-parallel-library
- Stephen Cleary, *Async/Await - Best Practices in Asynchronous Programming* (MSDN Magazine, 2013): https://learn.microsoft.com/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
- David Fowler, *Async Guidance* (secciones async void y fire-and-forget): https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md

---
⬅️ [[05 - Cancelación y timeouts]] · ➡️ [[07 - SynchronizationContext, ConfigureAwait y deadlocks]] · ⬆️ [[00 - Async en .NET - Índice]]
