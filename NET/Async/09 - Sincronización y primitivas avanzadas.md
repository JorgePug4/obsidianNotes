---
tags: [dotnet, csharp, async, nivel/9, sincronizacion]
up: "[[00 - Async en .NET - Índice]]"
---

# 09 - Sincronización y primitivas avanzadas

> Nivel 🟠 9 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[07 - SynchronizationContext, ConfigureAwait y deadlocks]], [[08 - Streams asíncronos y Channels]].

## Contenido
- [[#Por qué lock no sirve con await]]
- [[#SemaphoreSlim como mutex asíncrono]]
- [[#Locks por clave]]
- [[#Qué primitivas son seguras con async]]
- [[#TaskCompletionSource]]
- [[#Envolver APIs antiguas]]
- [[#Inicialización perezosa async]]
- [[#Single-flight: deduplicar peticiones concurrentes]]
- [[#AsyncLocal]]
- [[#PeriodicTimer (.NET 6)]]
- [[#🧠 Practica]]

---

## Por qué lock no sirve con await

```csharp
lock (_sync)
{
    await SaveAsync();   // ❌ error CS1996: no se puede usar await en el cuerpo de una instrucción lock
}
```

El compilador lo prohíbe porque `lock` (`Monitor`) tiene **afinidad de hilo**: lo debe liberar el **mismo hilo** que lo adquirió, y tras un `await` puedes continuar en otro hilo. Además, mantener un lock de hilo durante una espera de I/O bloquearía a otros hilos sin motivo.

Lo mismo aplica al nuevo `System.Threading.Lock` de .NET 9: es más eficiente que `lock(object)`, pero **tampoco** admite `await` dentro.

`lock` sigue siendo correcto para secciones críticas **cortas y síncronas** dentro de código async (actualizar un contador o un diccionario sin esperar nada dentro).

---

## SemaphoreSlim como mutex asíncrono

`SemaphoreSlim` tiene `WaitAsync`, que espera **sin bloquear** un hilo. Con capacidad 1 actúa como un mutex async:

```csharp
private readonly SemaphoreSlim _mutex = new(1, 1);

public async Task<Token> GetTokenAsync(CancellationToken ct)
{
    await _mutex.WaitAsync(ct);
    try
    {
        if (_cached is { IsExpired: false }) return _cached;
        _cached = await _auth.RequestTokenAsync(ct);   // solo una tarea a la vez llega aquí
        return _cached;
    }
    finally
    {
        _mutex.Release();
    }
}
```

> [!warning] Reglas de `SemaphoreSlim`
> - **No es reentrante**: si dentro de la sección vuelves a llamar a `WaitAsync` sobre el mismo semáforo, te bloqueas a ti mismo (deadlock).
> - `Release()` **siempre en `finally`**, y solo si `WaitAsync` tuvo éxito (si `WaitAsync(ct)` lanza por cancelación, no has adquirido nada: por eso `WaitAsync` va **fuera** del `try`).
> - No hay orden FIFO garantizado entre esperadores.
> - Solo protege **dentro de un proceso**. Con varias réplicas (Kubernetes) necesitas un lock distribuido ([[Distributed locks con Redis]] / [[Redlock y sus limitaciones|Redlock]], *leases* de Blob Storage, `sp_getapplock` en SQL Server).

---

## Locks por clave

"Solo una operación a la vez **por cliente**", sin bloquear a los demás clientes:

```csharp
public sealed class KeyedLock<TKey> where TKey : notnull
{
    private readonly ConcurrentDictionary<TKey, SemaphoreSlim> _locks = new();

    public async Task<IDisposable> AcquireAsync(TKey key, CancellationToken ct)
    {
        var sem = _locks.GetOrAdd(key, _ => new SemaphoreSlim(1, 1));
        await sem.WaitAsync(ct);
        return new Releaser(sem);
    }

    private sealed class Releaser(SemaphoreSlim sem) : IDisposable
    {
        public void Dispose() => sem.Release();
    }
}

// Uso
using (await _keyedLock.AcquireAsync(customerId, ct))
{
    await RecalculateBalanceAsync(customerId, ct);
}
```

> [!note] Limitación
> Este ejemplo nunca elimina semáforos del diccionario: con claves ilimitadas (IDs de usuario) crece sin fin. Las implementaciones de producción usan conteo de referencias para retirar el semáforo cuando nadie lo usa (p. ej. el paquete `AsyncKeyedLock`).

---

## Qué primitivas son seguras con async

| Primitiva | ¿Válida a través de `await`? | Alternativa async |
|---|---|---|
| `lock` / `Monitor` / `System.Threading.Lock` | ❌ | `SemaphoreSlim(1,1)` |
| `Mutex` (de SO) | ❌ (afinidad de hilo) | `SemaphoreSlim`; entre procesos, diseño específico |
| `ReaderWriterLockSlim` | ❌ (afinidad de hilo) | `AsyncReaderWriterLock` (Nito.AsyncEx) o rediseño (inmutabilidad + `Interlocked.Exchange`) |
| `ManualResetEvent` / `AutoResetEvent` | ❌ (bloquean el hilo) | `TaskCompletionSource`, `SemaphoreSlim` |
| `SemaphoreSlim` | ✅ (`WaitAsync`) | — |
| `Interlocked` | ✅ (operación atómica, sin espera) | — |
| `ConcurrentDictionary`, `ConcurrentQueue` | ✅ (sin espera) | — |
| `Channel<T>` | ✅ | — |

> [!tip] Mejor que sincronizar: no compartir
> Antes de añadir un lock, pregúntate si puedes **evitar el estado compartido**: datos inmutables, un solo dueño que procesa mensajes de un `Channel` (patrón *actor*), o resultados combinados al final con `WhenAll`.

---

## TaskCompletionSource

`TaskCompletionSource<T>` (TCS) crea un `Task` que **tú completas manualmente**. Es el puente entre el mundo de eventos/callbacks y `await`. Existe versión no genérica `TaskCompletionSource` desde .NET 5.

```csharp
public Task<Message> WaitForReplyAsync(string correlationId, TimeSpan timeout, CancellationToken ct)
{
    var tcs = new TaskCompletionSource<Message>(TaskCreationOptions.RunContinuationsAsynchronously);
    _pending[correlationId] = tcs;
    return tcs.Task.WaitAsync(timeout, ct);
}

// Cuando llega la respuesta (en otro hilo, p. ej. un consumidor de cola):
public void OnReply(Message reply)
{
    if (_pending.TryRemove(reply.CorrelationId, out var tcs))
        tcs.TrySetResult(reply);
}
```

API: `SetResult`/`TrySetResult`, `SetException`/`TrySetException`, `SetCanceled`/`TrySetCanceled`. Usa las variantes `Try*` cuando puede haber carreras (timeout vs respuesta).

> [!danger] Usa siempre `RunContinuationsAsynchronously`
> Sin esa opción, `SetResult` puede **ejecutar síncronamente** las continuaciones de quien esperaba, en el hilo que llama a `SetResult`. Consecuencias: el productor queda atrapado ejecutando código ajeno, posibles deadlocks si `SetResult` se llama dentro de un `lock`, y stack overflows en cadenas largas. Con la opción, las continuaciones se encolan en el pool.

> [!note] Limpieza en el ejemplo
> Si salta el timeout, la entrada de `_pending` queda huérfana. En código real, elimínala también cuando `WaitAsync` lanza (p. ej. con un `try/finally` en un método async).

---

## Envolver APIs antiguas

| Patrón antiguo | Cómo convertirlo a `Task` |
|---|---|
| **APM** (`BeginXxx`/`EndXxx` + `IAsyncResult`) | `Task.Factory.FromAsync(obj.BeginRead, obj.EndRead, ...)` |
| **EAP** (`XxxAsync()` + evento `XxxCompleted`) | `TaskCompletionSource` suscrito al evento |
| Callback cualquiera | `TaskCompletionSource` |

```csharp
// EAP → TAP
public static Task<string> DownloadAsync(WebClientLike client, Uri uri)
{
    var tcs = new TaskCompletionSource<string>(TaskCreationOptions.RunContinuationsAsynchronously);
    void Handler(object? s, DownloadCompletedEventArgs e)
    {
        client.DownloadCompleted -= Handler;
        if (e.Error is not null) tcs.TrySetException(e.Error);
        else if (e.Cancelled)    tcs.TrySetCanceled();
        else                     tcs.TrySetResult(e.Result);
    }
    client.DownloadCompleted += Handler;
    client.DownloadStringAsync(uri);
    return tcs.Task;
}
```

---

## Inicialización perezosa async

Inicializar algo caro **una sola vez**, al primer uso, aunque muchas tareas lo pidan a la vez:

```csharp
private readonly Lazy<Task<Config>> _config;

public MyService(IConfigLoader loader)
{
    _config = new Lazy<Task<Config>>(() => loader.LoadAsync()); // thread-safe por defecto (ExecutionAndPublication)
}

public async Task<int> GetLimitAsync() => (await _config.Value).Limit;
```

> [!warning] Los fallos quedan cacheados
> Si `LoadAsync` falla, `Lazy` guarda **el `Task` fallido** y todos los usos posteriores verán el mismo error para siempre. Si el fallo puede ser transitorio, usa una implementación que reintente (p. ej. `AsyncLazy<T>` de Nito.AsyncEx con `AsyncLazyFlags.RetryOnFailure`) o una inicialización con `SemaphoreSlim` que solo guarde el resultado si tuvo éxito. Otra opción es inicializar en `IHostedService.StartAsync` y fallar el arranque si no se puede.

---

## Single-flight: deduplicar peticiones concurrentes

Si 500 peticiones piden a la vez el mismo dato que no está en caché, no quieres 500 consultas a la BD (*cache stampede*). Patrón: **compartir el mismo `Task`** entre quienes piden la misma clave.

```csharp
private readonly ConcurrentDictionary<string, Lazy<Task<Product>>> _inflight = new();

public async Task<Product> GetProductAsync(string id)
{
    var lazy = _inflight.GetOrAdd(id, key => new Lazy<Task<Product>>(() => LoadProductAsync(key)));
    try
    {
        return await lazy.Value;          // todos los llamadores concurrentes esperan el MISMO Task
    }
    finally
    {
        _inflight.TryRemove(new KeyValuePair<string, Lazy<Task<Product>>>(id, lazy)); // solo si sigue siendo el nuestro
    }
}
```

El `Lazy` es necesario porque `GetOrAdd` puede ejecutar la fábrica más de una vez en una carrera; con `Lazy` solo uno de ellos llega a llamar a `LoadProductAsync`.

> [!note] En .NET 9+ ya existe
> `HybridCache` (`Microsoft.Extensions.Caching.Hybrid`) implementa protección contra stampede de serie. Ver [[HybridCache]] en el curso de Redis.

---

## AsyncLocal

`AsyncLocal<T>` guarda un valor que **fluye con el flujo lógico async** (a través de `await`, `Task.Run`, etc.) gracias al `ExecutionContext`. Es la versión async de `[ThreadStatic]`.

```csharp
public static class CorrelationContext
{
    private static readonly AsyncLocal<string?> _id = new();
    public static string? Id { get => _id.Value; set => _id.Value = value; }
}

// Middleware
app.Use(async (ctx, next) =>
{
    CorrelationContext.Id = ctx.Request.Headers["X-Correlation-Id"].FirstOrDefault() ?? Guid.NewGuid().ToString();
    await next(ctx);                               // todo lo que se ejecute "dentro" ve este Id
});
```

Reglas:
- El valor fluye **hacia abajo** (a los métodos llamados y tareas creadas), **no hacia arriba**: si un método async hijo cambia el valor, el llamador **no** ve el cambio después del `await`.
- Lo usan internamente `Activity.Current` (OpenTelemetry), `IHttpContextAccessor` y los *scopes* de `ILogger`.
- Úsalo con moderación: es **estado global implícito**; complica tests y razonamiento. Prefiere pasar datos explícitamente o por DI.

---

## PeriodicTimer (.NET 6)

Temporizador **async-friendly** para trabajos periódicos. A diferencia de `System.Threading.Timer`, no hay solapamiento: el siguiente "tick" se espera solo cuando terminas el trabajo.

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    using var timer = new PeriodicTimer(TimeSpan.FromSeconds(30));   // .NET 8+: también acepta un TimeProvider
    try
    {
        while (await timer.WaitForNextTickAsync(stoppingToken))
        {
            await RefreshCacheAsync(stoppingToken);
        }
    }
    catch (OperationCanceledException) { /* apagado normal */ }
}
```

- Si el trabajo tarda más que el periodo, se pierden *ticks* (no se acumulan), lo que suele ser deseable.
- Solo un consumidor puede esperar a la vez en `WaitForNextTickAsync`.

---

## Puntos clave

- No se puede `await` dentro de `lock`; usa `SemaphoreSlim(1,1)` con `WaitAsync` y `Release` en `finally`.
- `SemaphoreSlim` no es reentrante y solo sirve dentro de un proceso.
- `TaskCompletionSource` + `RunContinuationsAsynchronously` para convertir callbacks/eventos en `Task`.
- `Lazy<Task<T>>` para inicialización única (cuidado: cachea fallos).
- Single-flight: compartir el `Task` en vuelo por clave.
- `AsyncLocal<T>` fluye hacia abajo, no hacia arriba.
- `PeriodicTimer` para bucles periódicos sin solapamiento.

## Errores comunes

- `WaitAsync` dentro del `try` (se libera un semáforo que no se adquirió).
- TCS sin `RunContinuationsAsynchronously`.
- `SetResult` en vez de `TrySetResult` cuando timeout y respuesta compiten (lanza `InvalidOperationException`).
- Usar `ReaderWriterLockSlim` o `Mutex` alrededor de un `await`.
- Esperar que un cambio en `AsyncLocal` hecho en un método hijo sea visible en el padre.

## 🎯 Para entrevistas y exámenes

- *"¿Cómo proteges una sección crítica que contiene `await`?"* → `SemaphoreSlim(1,1)`, `await WaitAsync()`, `Release()` en `finally`.
- *"¿Para qué sirve `TaskCompletionSource`?"* → Para crear un `Task` que se completa manualmente: adaptar eventos, callbacks o respuestas correlacionadas a `await`.
- *"¿Qué es `AsyncLocal`?"* → Almacenamiento ambiental que fluye con el `ExecutionContext` a través de `await`; base de `Activity.Current`.
- *"¿Cómo evitas que 1 000 peticiones simultáneas carguen el mismo dato 1 000 veces?"* → Single-flight / `HybridCache`: compartir el `Task` en vuelo.

## 🧠 Practica

1. Crea un contador compartido incrementado por 1 000 tareas con `await Task.Yield()` entre lectura y escritura. Observa la pérdida de incrementos y arréglalo con `SemaphoreSlim` y con `Interlocked.Increment`.
2. Implementa `WaitForReplyAsync` con limpieza de `_pending` en caso de timeout.
3. Escribe un `AsyncLocal<int>`, asígnale 1, llama a un método async que lo pone a 2, y comprueba su valor tras el `await` en el llamador.

## Referencias

- Microsoft Learn, *SemaphoreSlim*: https://learn.microsoft.com/dotnet/api/system.threading.semaphoreslim
- Sergey Teplyakov, *The danger of TaskCompletionSource<T> class*: https://devblogs.microsoft.com/premier-developer/the-danger-of-taskcompletionsourcet-class/
- Microsoft Learn, *Interop with Other Asynchronous Patterns and Types*: https://learn.microsoft.com/dotnet/standard/asynchronous-programming-patterns/interop-with-other-asynchronous-patterns-and-types
- Microsoft Learn, *AsyncLocal<T>*: https://learn.microsoft.com/dotnet/api/system.threading.asynclocal-1
- Stephen Cleary, Nito.AsyncEx: https://github.com/StephenCleary/AsyncEx

---
⬅️ [[08 - Streams asíncronos y Channels]] · ➡️ [[10 - Rendimiento y diagnóstico]] · ⬆️ [[00 - Async en .NET - Índice]]
