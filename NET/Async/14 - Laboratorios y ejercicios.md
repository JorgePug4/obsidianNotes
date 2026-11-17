---
tags: [dotnet, csharp, async, laboratorios, ejercicios]
up: "[[00 - Async en .NET - Índice]]"
---

# 14 - Laboratorios y ejercicios

> Práctica progresiva del [[00 - Async en .NET - Índice|curso de Async en .NET]]. Cada lab indica su nivel. Las soluciones están plegadas: intenta resolverlo antes de abrirlas.

> [!info] Preparación
> ```bash
> dotnet new console -n AsyncLabs && cd AsyncLabs
> # Para los labs con ASP.NET Core:  dotnet new web -n AsyncApi
> # Para los labs de tests:          dotnet new xunit -n AsyncLabs.Tests
> ```
> Los ejemplos usan *top-level statements* y .NET 8 o superior.

## Contenido
- [[#Lab 1 - Secuencial vs concurrente]]
- [[#Lab 2 - ¿En qué hilo estoy?]]
- [[#Lab 3 - Descargador con concurrencia limitada]]
- [[#Lab 4 - Timeout que respeta la cancelación]]
- [[#Lab 5 - Todas las excepciones de WhenAll]]
- [[#Lab 6 - Paginación con IAsyncEnumerable]]
- [[#Lab 7 - Productor-consumidor con Channel]]
- [[#Lab 8 - Reproducir el deadlock clásico en consola]]
- [[#Lab 9 - Provocar y diagnosticar thread pool starvation]]
- [[#Lab 10 - Single-flight y su test]]
- [[#Lab 11 - Probar un reintento con FakeTimeProvider]]
- [[#Proyecto final - Pipeline de importación]]

---

## Lab 1 - Secuencial vs concurrente
**Nivel 🟢 2-4.** Simula tres llamadas de 300, 500 y 200 ms con `Task.Delay`. Mide con `Stopwatch` la versión secuencial y la concurrente.

> [!example]- Solución
> ```csharp
> using System.Diagnostics;
>
> static async Task<int> CallAsync(int ms) { await Task.Delay(ms); return ms; }
>
> var sw = Stopwatch.StartNew();
> await CallAsync(300); await CallAsync(500); await CallAsync(200);
> Console.WriteLine($"Secuencial: {sw.ElapsedMilliseconds} ms");   // ~1000
>
> sw.Restart();
> int[] results = await Task.WhenAll(CallAsync(300), CallAsync(500), CallAsync(200));
> Console.WriteLine($"Concurrente: {sw.ElapsedMilliseconds} ms");  // ~500
> ```
> Conclusión: la versión concurrente tarda lo que la **más lenta**, porque las tres tareas se inician antes de esperar.

---

## Lab 2 - ¿En qué hilo estoy?
**Nivel 🟢 2-3.** Imprime `Environment.CurrentManagedThreadId` antes y después de un `await Task.Delay(100)` y de un `await Task.CompletedTask`. Explica la diferencia.

> [!example]- Solución
> ```csharp
> Console.WriteLine($"Inicio: {Environment.CurrentManagedThreadId}");
> await Task.CompletedTask;
> Console.WriteLine($"Tras tarea completada: {Environment.CurrentManagedThreadId}");  // mismo hilo: no hubo suspensión
> await Task.Delay(100);
> Console.WriteLine($"Tras Delay: {Environment.CurrentManagedThreadId}");             // normalmente otro hilo del pool
> ```
> En consola no hay `SynchronizationContext`: tras una suspensión real, la continuación se ejecuta en cualquier hilo del pool. Con una tarea ya completada, `await` sigue de forma síncrona.

---

## Lab 3 - Descargador con concurrencia limitada
**Nivel 🟡 4.** Escribe `ProcessAllAsync(IEnumerable<int> ids, int maxConcurrency)` que "procese" cada id con `Task.Delay(Random.Shared.Next(50, 200))`, **nunca** más de `maxConcurrency` a la vez. Verifica el máximo observado con un contador `Interlocked`.

> [!example]- Solución
> ```csharp
> int current = 0, maxSeen = 0;
>
> async Task ProcessAsync(int id, CancellationToken ct)
> {
>     int now = Interlocked.Increment(ref current);
>     int seen;
>     while (now > (seen = Volatile.Read(ref maxSeen)) &&
>            Interlocked.CompareExchange(ref maxSeen, now, seen) != seen) { }
>     try { await Task.Delay(Random.Shared.Next(50, 200), ct); }
>     finally { Interlocked.Decrement(ref current); }
> }
>
> // Variante A: Parallel.ForEachAsync
> await Parallel.ForEachAsync(Enumerable.Range(1, 100),
>     new ParallelOptions { MaxDegreeOfParallelism = 5 },
>     async (id, ct) => await ProcessAsync(id, ct));
> Console.WriteLine($"Máximo simultáneo (A): {maxSeen}");   // 5
>
> // Variante B: SemaphoreSlim
> maxSeen = 0;
> using var gate = new SemaphoreSlim(5);
> await Task.WhenAll(Enumerable.Range(1, 100).Select(async id =>
> {
>     await gate.WaitAsync();
>     try { await ProcessAsync(id, CancellationToken.None); }
>     finally { gate.Release(); }
> }));
> Console.WriteLine($"Máximo simultáneo (B): {maxSeen}");   // 5
> ```

---

## Lab 4 - Timeout que respeta la cancelación
**Nivel 🟡 5.** Implementa `Task<string> FetchAsync(Func<CancellationToken, Task<string>> op, TimeSpan timeout, CancellationToken ct)` que:
- lance `TimeoutException` si vence el timeout,
- lance `OperationCanceledException` si cancela el llamador,
- disponga correctamente sus CTS.

> [!example]- Solución
> ```csharp
> static async Task<string> FetchAsync(Func<CancellationToken, Task<string>> op, TimeSpan timeout, CancellationToken ct)
> {
>     using var linked = CancellationTokenSource.CreateLinkedTokenSource(ct);
>     linked.CancelAfter(timeout);
>     try
>     {
>         return await op(linked.Token);
>     }
>     catch (OperationCanceledException) when (!ct.IsCancellationRequested)
>     {
>         throw new TimeoutException($"La operación superó {timeout.TotalMilliseconds} ms");
>     }
> }
>
> // Prueba 1: timeout
> try { await FetchAsync(async t => { await Task.Delay(2000, t); return "x"; }, TimeSpan.FromMilliseconds(200), default); }
> catch (TimeoutException e) { Console.WriteLine(e.Message); }
>
> // Prueba 2: cancela el llamador
> using var cts = new CancellationTokenSource(100);
> try { await FetchAsync(async t => { await Task.Delay(2000, t); return "x"; }, TimeSpan.FromSeconds(5), cts.Token); }
> catch (OperationCanceledException) { Console.WriteLine("Cancelado por el llamador"); }
> ```

---

## Lab 5 - Todas las excepciones de WhenAll
**Nivel 🟡 6.** Lanza 5 tareas donde las impares fallan. Muestra: (a) qué captura el `catch` de `await`, (b) todas las excepciones, (c) los resultados de las que sí terminaron.

> [!example]- Solución
> ```csharp
> var tasks = Enumerable.Range(1, 5).Select(async i =>
> {
>     await Task.Delay(10 * i);
>     if (i % 2 == 1) throw new InvalidOperationException($"falló {i}");
>     return i;
> }).ToArray();
>
> Task<int[]> all = Task.WhenAll(tasks);
> try { await all; }
> catch (Exception ex) { Console.WriteLine($"(a) catch: {ex.Message}"); }        // falló 1
>
> foreach (var e in all.Exception!.InnerExceptions)
>     Console.WriteLine($"(b) {e.Message}");                                    // falló 1, 3, 5
>
> var ok = tasks.Where(t => t.IsCompletedSuccessfully).Select(t => t.Result);
> Console.WriteLine($"(c) correctas: {string.Join(", ", ok)}");                 // 2, 4
> ```

---

## Lab 6 - Paginación con IAsyncEnumerable
**Nivel 🟠 8.** Simula una API que devuelve páginas de 10 elementos (5 páginas, 50 ms por página). Expón `IAsyncEnumerable<int> GetAllAsync(CancellationToken)` y consume solo los primeros 23 elementos. ¿Cuántas páginas se piden?

> [!example]- Solución
> ```csharp
> using System.Runtime.CompilerServices;
>
> int pagesRequested = 0;
>
> async Task<int[]> GetPageAsync(int page, CancellationToken ct)
> {
>     Interlocked.Increment(ref pagesRequested);
>     await Task.Delay(50, ct);
>     return Enumerable.Range(page * 10, 10).ToArray();
> }
>
> async IAsyncEnumerable<int> GetAllAsync([EnumeratorCancellation] CancellationToken ct = default)
> {
>     for (int page = 0; page < 5; page++)
>         foreach (var item in await GetPageAsync(page, ct))
>             yield return item;
> }
>
> int count = 0;
> await foreach (var item in GetAllAsync())
> {
>     if (++count == 23) break;
> }
> Console.WriteLine($"Páginas pedidas: {pagesRequested}");   // 3: la evaluación es perezosa
> ```

---

## Lab 7 - Productor-consumidor con Channel
**Nivel 🟠 8.** Un productor genera los números 1..10 000 en un canal acotado de capacidad 100; 3 consumidores los suman. Verifica que la suma es 50 005 000 y que el productor nunca tuvo más de 100 elementos pendientes.

> [!example]- Solución
> ```csharp
> using System.Threading.Channels;
>
> var channel = Channel.CreateBounded<int>(new BoundedChannelOptions(100)
> {
>     FullMode = BoundedChannelFullMode.Wait,
>     SingleWriter = true
> });
>
> var producer = Task.Run(async () =>
> {
>     try
>     {
>         for (int i = 1; i <= 10_000; i++)
>             await channel.Writer.WriteAsync(i);
>     }
>     finally
>     {
>         channel.Writer.Complete();
>     }
> });
>
> long total = 0;
> var consumers = Enumerable.Range(0, 3).Select(_ => Task.Run(async () =>
> {
>     long local = 0;
>     await foreach (var n in channel.Reader.ReadAllAsync())
>         local += n;
>     Interlocked.Add(ref total, local);
> })).ToArray();
>
> await Task.WhenAll(consumers.Append(producer));
> Console.WriteLine(total == 50_005_000 ? "✅ suma correcta" : $"❌ {total}");
> ```
> `channel.Reader.Count` (disponible en canales acotados) nunca supera 100 porque `WriteAsync` espera cuando el canal está lleno.

---

## Lab 8 - Reproducir el deadlock clásico en consola
**Nivel 🟠 7.** Una app de consola no tiene `SynchronizationContext`, así que `.Result` no produce deadlock. Crea un contexto de **un solo hilo** (como el de una UI), instálalo y reproduce el deadlock. Después arréglalo de dos formas.

> [!example]- Solución
> ```csharp
> using System.Collections.Concurrent;
>
> // 1. Deadlock: instalamos un contexto de un solo hilo (como una UI) y bloqueamos con Wait
> var ctx = new SingleThreadContext();
> SynchronizationContext.SetSynchronizationContext(ctx);
> Task<string> t = GetDataAsync();
> bool finished = t.Wait(TimeSpan.FromSeconds(2));   // Wait con timeout para no colgarnos para siempre
> Console.WriteLine(finished ? "Terminó" : "💀 Deadlock: la continuación espera al hilo que está bloqueado");
>
> // 2a. "Arreglo" con ConfigureAwait(false): funciona, pero es frágil
> Console.WriteLine(GetDataNoContextAsync().Result);
>
> // 2b. Arreglo correcto: no bloquear; bombear el contexto hasta que termine (lo que hace el bucle de una UI)
> var ctx2 = new SingleThreadContext();
> SynchronizationContext.SetSynchronizationContext(ctx2);
> Task<string> t2 = GetDataAsync();
> _ = t2.ContinueWith(_ => ctx2.Complete(), TaskScheduler.Default);
> ctx2.RunUntilComplete();                             // ejecuta las continuaciones en este hilo
> SynchronizationContext.SetSynchronizationContext(null);
> Console.WriteLine(t2.Result);                        // ya está completa: no bloquea
>
> static async Task<string> GetDataAsync()
> {
>     await Task.Delay(100);           // captura el contexto
>     return "datos";                  // necesita el contexto para continuar
> }
>
> static async Task<string> GetDataNoContextAsync()
> {
>     await Task.Delay(100).ConfigureAwait(false);
>     return "datos";
> }
>
> // Con top-level statements, las declaraciones de tipos van AL FINAL del fichero
> sealed class SingleThreadContext : SynchronizationContext
> {
>     private readonly BlockingCollection<(SendOrPostCallback Callback, object? State)> _queue = new();
>     public override void Post(SendOrPostCallback d, object? state) => _queue.Add((d, state));
>     public void RunUntilComplete()
>     {
>         foreach (var (callback, state) in _queue.GetConsumingEnumerable()) callback(state);
>     }
>     public void Complete() => _queue.CompleteAdding();
> }
> ```
>
> Reflexión: el problema no es `Task.Delay` ni `await`, sino **bloquear el único hilo que puede ejecutar la continuación**.
>
> Advertencia sobre el paso 1: el `Task` que quedó en deadlock sigue pendiente para siempre; en un programa real, eso sería la app congelada.

---

## Lab 9 - Provocar y diagnosticar thread pool starvation
**Nivel 🔴 10.** Crea una Minimal API con dos endpoints y somételos a carga.

```csharp
// Program.cs (dotnet new web -n AsyncApi)
var app = WebApplication.CreateBuilder(args).Build();

app.MapGet("/async", async () => { await Task.Delay(500); return "ok"; });
app.MapGet("/sync",  () => { Task.Delay(500).Wait(); return "ok"; });   // ❌ sync-over-async

app.Run();
```

Pasos:
1. `dotnet run -c Release`.
2. En otra terminal: `dotnet-counters monitor -n AsyncApi --counters System.Runtime`.
3. Carga con una herramienta HTTP, p. ej. `bombardier -c 500 -d 30s http://localhost:5000/async` y después `/sync`. (Alternativas: `k6`, `hey`, NBomber.)
4. Compara: peticiones/s, latencia p99, longitud de la cola del thread pool, número de hilos.
5. Durante la carga de `/sync`, ejecuta `dotnet-stack report -n AsyncApi` y busca `Task.Wait`.

> [!example]- Qué deberías observar
> | Métrica | `/async` | `/sync` |
> |---|---|---|
> | Peticiones/s | ≈ concurrencia / 0,5 s (≈ 1 000 con 500 conexiones) | Mucho menor al principio; mejora poco a poco |
> | Latencia p99 | ≈ 500 ms | Segundos |
> | Cola del pool | ≈ 0 | Alta y sostenida |
> | Hilos del pool | Estables y pocos | Suben lentamente (decenas → cientos) |
> | CPU | Baja | **Baja** (no es un problema de CPU) |
>
> Los valores exactos dependen de tu máquina y versión de .NET. Prueba también `ThreadPool.SetMinThreads(500, 500)` al inicio: `/sync` mejora, lo que demuestra que es starvation, pero a costa de 500 hilos. La corrección real es `/async`.

---

## Lab 10 - Single-flight y su test
**Nivel 🔴 9 y 12.** Implementa `SingleFlightCache.GetAsync(string key, Func<string, Task<T>> loader)` que garantice una sola carga concurrente por clave. Escribe un test con `TaskCompletionSource` que lo demuestre.

> [!example]- Solución
> ```csharp
> public sealed class SingleFlightCache<T>
> {
>     private readonly ConcurrentDictionary<string, Lazy<Task<T>>> _inflight = new();
>
>     public async Task<T> GetAsync(string key, Func<string, Task<T>> loader)
>     {
>         var lazy = _inflight.GetOrAdd(key, k => new Lazy<Task<T>>(() => loader(k)));
>         try { return await lazy.Value; }
>         finally { _inflight.TryRemove(new KeyValuePair<string, Lazy<Task<T>>>(key, lazy)); }
>     }
> }
>
> public class SingleFlightCacheTests
> {
>     [Fact]
>     public async Task Concurrent_requests_trigger_one_load()
>     {
>         int loads = 0;
>         var tcs = new TaskCompletionSource<string>(TaskCreationOptions.RunContinuationsAsynchronously);
>         var cache = new SingleFlightCache<string>();
>         Task<string> Loader(string _) { Interlocked.Increment(ref loads); return tcs.Task; }
>
>         var calls = Enumerable.Range(0, 50).Select(_ => cache.GetAsync("k", Loader)).ToArray();
>         tcs.SetResult("valor");
>         var results = await Task.WhenAll(calls).WaitAsync(TimeSpan.FromSeconds(5));
>
>         Assert.All(results, r => Assert.Equal("valor", r));
>         Assert.Equal(1, loads);
>     }
> }
> ```

---

## Lab 11 - Probar un reintento con FakeTimeProvider
**Nivel 🔴 12.** Toma el `RetryingSender` de [[12 - Testing de código asíncrono#Controlar el tiempo con FakeTimeProvider|nivel 12]] y escribe tests que verifiquen: (1) reintenta tras 10 s, (2) en el tercer fallo propaga la excepción, (3) si se cancela durante la espera lanza `OperationCanceledException`. Ningún test debe tardar más de 1 s.

> [!example]- Pista
> Para (3), crea un `CancellationTokenSource`, llama a `SendAsync`, comprueba que no ha terminado, cancela y haz `await Assert.ThrowsAnyAsync<OperationCanceledException>(() => send)`. No necesitas avanzar el reloj: `Task.Delay(…, time, ct)` observa el token.
> Para (2), configura el doble para fallar siempre y avanza el reloj 10 s y luego 20 s entre comprobaciones.

---

## Proyecto final - Pipeline de importación
**Nivel 🏁 todos.** Construye un Worker Service que importe productos de una API paginada a una base de datos.

Requisitos:
1. Lee la API con `IAsyncEnumerable<Product>` (páginas de 100).
2. Pipeline con `Channel` acotados: **descargar → validar/transformar → guardar por lotes de 500**.
3. Máximo 4 peticiones simultáneas a la API y 2 escrituras simultáneas a la BD.
4. Reintentos con backoff exponencial ante errores transitorios (Polly o `Microsoft.Extensions.Resilience`).
5. `CancellationToken` de extremo a extremo: Ctrl+C detiene el pipeline sin perder lotes a medio escribir (punto de no retorno en la escritura).
6. Los errores de un producto inválido se registran y no detienen el pipeline; los errores fatales sí, propagándose con `Writer.Complete(ex)`.
7. Métricas: productos/s y elementos pendientes por canal (`Reader.Count`), expuestas con `System.Diagnostics.Metrics`.
8. Tests: la etapa de transformación con datos de ejemplo; el reintento con `FakeTimeProvider`; el apagado con un `CancellationTokenSource`.
9. Idempotencia: si se reinicia a mitad, no duplica productos (upsert por SKU). Ver [[Idempotencia]].

Criterios de "terminado":
- [ ] Con la BD lenta (añade `Task.Delay` en la escritura), la memoria se mantiene estable (backpressure).
- [ ] Ctrl+C termina en menos de `ShutdownTimeout` y el log muestra un apagado limpio.
- [ ] `dotnet-counters` muestra la cola del thread pool cerca de 0 durante toda la ejecución.
- [ ] Ningún `.Result`, `.Wait()`, `async void` ni `Task.Run` innecesario (revisa con la [[13 - Anti-patrones, checklist y entrevistas#Checklist de code review|checklist]]).

---
⬅️ [[13 - Anti-patrones, checklist y entrevistas]] · ⬆️ [[00 - Async en .NET - Índice]]
