---
tags: [dotnet, csharp, async, nivel/13, entrevistas, cheatsheet]
up: "[[00 - Async en .NET - Índice]]"
---

# 13 - Anti-patrones, checklist y entrevistas

> Nivel 🏁 13 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Repaso de todo el curso.

## Contenido
- [[#Catálogo de anti-patrones]]
- [[#Checklist de code review]]
- [[#Cheat sheet de APIs]]
- [[#Preguntas de entrevista por nivel]]
- [[#Preguntas trampa]]

---

## Catálogo de anti-patrones

| # | Anti-patrón | Código | Consecuencia | Corrección | Nivel |
|---|---|---|---|---|---|
| 1 | Sync-over-async | `GetAsync().Result` | Deadlock (UI) o starvation (servidor) | `await` hasta arriba | [[07 - SynchronizationContext, ConfigureAwait y deadlocks\|7]] |
| 2 | `async void` | `async void Save()` | Excepciones que matan el proceso; no se puede esperar | `async Task` | [[02 - async y await - lo básico\|2]] |
| 3 | Lambda async a `Action` | `list.ForEach(async x => ...)`, `Parallel.ForEach(..., async x => ...)` | async void oculto | `foreach` + `await`, `WhenAll`, `Parallel.ForEachAsync` | [[04 - Composición de tareas\|4]] |
| 4 | Fake async | `Task<T> FooAsync() => Task.Run(Foo)` en librería | Consume hilos; engaña | Exponer solo la versión síncrona | [[07 - SynchronizationContext, ConfigureAwait y deadlocks\|7]] |
| 5 | `Task.Run` en ASP.NET Core | `await Task.Run(() => _repo.Get())` | Overhead sin beneficio | Llamada directa | [[02 - async y await - lo básico\|2]] |
| 6 | Tarea olvidada | `DoAsync();` (sin `await`, CS4014) | Excepciones perdidas, condiciones de carrera | `await` o cola explícita | [[06 - Manejo de excepciones\|6]] |
| 7 | Fire-and-forget con scope de la petición | `_ = Task.Run(() => _db.SaveAsync())` | `ObjectDisposedException`, datos de otra petición | `Channel` + `BackgroundService` + scope propio | [[11 - Async en ASP.NET Core y servicios en segundo plano\|11]] |
| 8 | Concurrencia ilimitada | `Task.WhenAll(10_000 items.Select(...))` | Agotamiento de sockets/conexiones, 429 | `SemaphoreSlim`, `Parallel.ForEachAsync` | [[04 - Composición de tareas\|4]] |
| 9 | `await` secuencial innecesario | `await A(); await B();` independientes | Latencia sumada | Iniciar ambas + `WhenAll` | [[04 - Composición de tareas\|4]] |
| 10 | `DbContext` compartido en paralelo | `WhenAll(db.A.ToListAsync(), db.B.ToListAsync())` | `InvalidOperationException` | `IDbContextFactory` o secuencial | [[04 - Composición de tareas\|4]] |
| 11 | No propagar `CancellationToken` | Aceptarlo y no pasarlo | Trabajo inútil tras cancelar | Pasar a todas las APIs (CA2016) | [[05 - Cancelación y timeouts\|5]] |
| 12 | CTS sin disponer | `CreateLinkedTokenSource` sin `using` | Fugas de registros/timers | `using var` | [[05 - Cancelación y timeouts\|5]] |
| 13 | Cancelación registrada como error | `catch (Exception) { LogError }` | Alertas falsas | `catch (OperationCanceledException) when (...)` | [[05 - Cancelación y timeouts\|5]] |
| 14 | `await` dentro de `lock` | No compila; "arreglo" con `Monitor` manual | Bloqueos, liberación en otro hilo | `SemaphoreSlim(1,1)` | [[09 - Sincronización y primitivas avanzadas\|9]] |
| 15 | TCS sin `RunContinuationsAsynchronously` | `new TaskCompletionSource<T>()` | Continuaciones inline, deadlocks | Pasar la opción | [[09 - Sincronización y primitivas avanzadas\|9]] |
| 16 | `ValueTask` esperado dos veces | `var v = FooAsync(); await v; await v;` | Comportamiento indefinido | Un solo `await`, `AsTask()` o `Preserve()` | [[Task vs ValueTask\|ValueTask]] |
| 17 | `Thread.Sleep` en código async | `Thread.Sleep(1000)` | Bloquea un hilo del pool | `await Task.Delay(1000, ct)` | [[10 - Rendimiento y diagnóstico\|10]] |
| 18 | `new HttpClient()` por petición | `using var c = new HttpClient()` | Agotamiento de puertos | `IHttpClientFactory` | [[10 - Rendimiento y diagnóstico\|10]] |
| 19 | Elidir `async` dentro de `using` | `using var x = ...; return DoAsync(x);` | Recurso liberado antes de tiempo | Mantener `async`/`await` | [[03 - Cómo funciona por dentro\|3]] |
| 20 | `Channel` sin `Complete()` | Productor termina sin completar | Consumidores colgados para siempre | `Writer.Complete()` en `finally` | [[08 - Streams asíncronos y Channels\|8]] |
| 21 | `Channel` ilimitado con productor rápido | `CreateUnbounded` | OOM | `CreateBounded` + `Wait` | [[08 - Streams asíncronos y Channels\|8]] |
| 22 | `BackgroundService` sin `try/catch` | Excepción en el bucle | Se detiene el host (.NET 6+) | `try/catch` por iteración | [[11 - Async en ASP.NET Core y servicios en segundo plano\|11]] |
| 23 | `SetMinThreads` como solución | Subir el mínimo del pool | Esconde la causa | Quitar el bloqueo | [[10 - Rendimiento y diagnóstico\|10]] |
| 24 | Devolver `null` como `Task` | `return null;` en método no async que devuelve `Task` | `NullReferenceException` al hacer `await` | `Task.CompletedTask` / `Task.FromResult` | [[02 - async y await - lo básico\|2]] |
| 25 | `ConfigureAwait(false)` y luego tocar UI | `await x.ConfigureAwait(false); label.Text = ...` | Excepción de acceso entre hilos | No usarlo en código de UI que toca controles | [[07 - SynchronizationContext, ConfigureAwait y deadlocks\|7]] |

---

## Checklist de code review

Úsala al revisar un PR con código asíncrono:

**Firmas**
- [ ] Ningún `async void` salvo *event handlers* (con `try/catch` completo).
- [ ] Métodos async con sufijo `Async` y `CancellationToken` como último parámetro (si hacen I/O).
- [ ] No se devuelve `null` desde métodos que devuelven `Task`.

**Bloqueos**
- [ ] Ningún `.Result`, `.Wait()`, `.GetAwaiter().GetResult()`, `Thread.Sleep` en código de servidor.
- [ ] Ninguna API de I/O síncrona donde existe la versión async.

**Tareas**
- [ ] Toda tarea creada se espera o se encola explícitamente (sin CS4014 ignorados, sin `_ =` injustificados).
- [ ] Operaciones independientes se ejecutan en concurrencia cuando aporta.
- [ ] Concurrencia sobre colecciones **limitada**.
- [ ] Recursos no thread-safe (`DbContext`, `HttpContext`, `Stream`) no se usan en paralelo.

**Cancelación y tiempo**
- [ ] El `CancellationToken` se propaga a todas las llamadas (CA2016).
- [ ] Los CTS (también los enlazados) se disponen.
- [ ] Cancelación no se registra como error.
- [ ] Hay un punto de no retorno claro en operaciones con efectos.
- [ ] El tiempo se obtiene de `TimeProvider` si hay lógica temporal que probar.

**Errores**
- [ ] Las excepciones de `WhenAll` se tratan de forma completa cuando importa.
- [ ] `BackgroundService` captura excepciones por iteración y respeta `stoppingToken`.

**Sincronización**
- [ ] Sin `lock`/`Mutex`/`ReaderWriterLockSlim` alrededor de `await`.
- [ ] `SemaphoreSlim.WaitAsync` fuera del `try`, `Release` en `finally`.
- [ ] `TaskCompletionSource` con `RunContinuationsAsynchronously` y `TrySet*` cuando hay carreras.

**Librerías**
- [ ] `ConfigureAwait(false)` en todos los `await` (o CA2007 activado).
- [ ] `ValueTask` solo con justificación medida y consumidores que conocen sus reglas.

---

## Cheat sheet de APIs

| Necesito… | API | Desde |
|---|---|---|
| Esperar sin bloquear | `await` | C# 5 |
| Resultado ya disponible | `Task.FromResult(x)`, `Task.CompletedTask`, `ValueTask.FromResult(x)` | — / .NET 5 |
| Ejecutar CPU en el pool | `Task.Run` | .NET 4.5 |
| Esperar todas / la primera | `Task.WhenAll` / `Task.WhenAny` | .NET 4.5 |
| Procesar en orden de llegada | `Task.WhenEach` | .NET 9 |
| Concurrencia limitada | `Parallel.ForEachAsync`, `SemaphoreSlim.WaitAsync` | .NET 6 / .NET 4.5 |
| Timeout sobre cualquier tarea | `task.WaitAsync(timeout, ct)` | .NET 6 |
| Cancelar | `CancellationTokenSource`, `CancelAfter`, `CreateLinkedTokenSource`, `CancelAsync` | — / .NET 8 (`CancelAsync`) |
| Esperar sin lanzar / forzar yield | `ConfigureAwait(ConfigureAwaitOptions.SuppressThrowing / ForceYielding)` | .NET 8 |
| Stream asíncrono | `IAsyncEnumerable<T>`, `await foreach`, `[EnumeratorCancellation]` | C# 8 / .NET Core 3.0 |
| LINQ async | `System.Linq.Async` (paquete) / `System.Linq.AsyncEnumerable` | — / .NET 10 |
| Productor-consumidor | `Channel.CreateBounded<T>` | .NET Core 3.0 |
| Completar un Task manualmente | `TaskCompletionSource<T>` (no genérico desde .NET 5) | .NET 4.0 |
| Temporizador periódico | `PeriodicTimer` | .NET 6 |
| Tiempo inyectable | `TimeProvider`, `FakeTimeProvider` | .NET 8 |
| Liberar async | `IAsyncDisposable`, `await using` | C# 8 |
| Mutex async | `SemaphoreSlim(1, 1)` | .NET 4.5 |
| Contexto ambiental | `AsyncLocal<T>` | .NET 4.6 |
| Trabajo en segundo plano | `BackgroundService`, `IHostedLifecycleService` | .NET Core 2.1 / .NET 8 |
| Pooling en métodos async | `[AsyncMethodBuilder(typeof(PoolingAsyncValueTaskMethodBuilder<>))]` | .NET 6 |

---

## Preguntas de entrevista por nivel

### 🟢 Junior
1. **¿Qué es un `Task`?** → Representación de una operación que puede no haber terminado; puede completar con resultado, excepción o cancelación. No es un hilo.
2. **¿Qué hace `await`?** → Si la tarea no ha terminado, suspende el método y devuelve el control; al completar, reanuda y devuelve el resultado o relanza la excepción.
3. **¿Por qué evitar `async void`?** → No se puede esperar ni capturar sus excepciones; puede terminar el proceso. Solo para event handlers.
4. **¿Async es lo mismo que multihilo?** → No. Async evita bloquear durante esperas; puede no usar ningún hilo adicional.
5. **¿Diferencia entre I/O-bound y CPU-bound?** → Esperar a algo externo vs calcular; `await` para el primero, `Task.Run`/`Parallel` para el segundo (en UI o batch).

### 🟡 Mid
6. **¿Cómo ejecutas varias llamadas en paralelo y esperas todas?** → Iniciarlas y `await Task.WhenAll(...)`.
7. **¿Qué excepción ves con `await WhenAll` si fallan dos?** → La primera; todas en `task.Exception.InnerExceptions`.
8. **¿Cómo implementas un timeout?** → `CancellationTokenSource` con `CancelAfter`, enlazado al token del llamador; o `WaitAsync` si la API no acepta token.
9. **¿Qué es `ConfigureAwait(false)`?** → Indica que la continuación no necesita el contexto capturado; útil en librerías.
10. **¿Por qué `.Result` puede causar deadlock?** → En contextos de un solo hilo, el hilo bloqueado es el que necesita la continuación.

### 🟠 Senior
11. **¿Qué genera el compilador para un método async?** → Una máquina de estados (`IAsyncStateMachine`) con `MoveNext`, campo de estado, locales como campos y un builder que crea el `Task`.
12. **¿Qué es *thread pool starvation* y cómo la diagnosticas?** → Hilos del pool bloqueados que impiden ejecutar continuaciones; `dotnet-counters` (cola del pool) y pilas (`dotnet-stack`, `dotnet-dump`).
13. **¿Cómo limitas la concurrencia?** → `SemaphoreSlim`, `Parallel.ForEachAsync`, canales con N consumidores.
14. **¿Cuándo usarías `ValueTask`?** → Hot paths que suelen completarse síncronamente, tras medir; reglas: un solo `await`, sin concurrencia, sin bloqueo.
15. **¿Cómo coordinas una sección crítica con `await`?** → `SemaphoreSlim(1,1)`; `lock` no admite `await`.
16. **¿Para qué sirve `TaskCompletionSource` y qué opción usarías siempre?** → Crear tareas completadas manualmente; `RunContinuationsAsynchronously`.

### 🔴 Architect / Staff
17. **Diseña el procesamiento de 1 M de registros desde una API paginada hacia una BD con límites de tasa.** → `IAsyncEnumerable` para paginar, `Channel` acotado entre etapas, N consumidores ajustados al límite de la API, inserción por lotes, cancelación de extremo a extremo, reintentos con backoff, idempotencia para reanudar.
18. **¿Trabajo en segundo plano dentro de la API o en un worker aparte?** → Dentro (cola en memoria) si es no crítico y ligero; fuera con cola persistente si debe sobrevivir a reinicios, escalar independientemente o no competir por recursos con las peticiones.
19. **¿Cómo evitas un *cache stampede*?** → Single-flight (compartir el `Task` en vuelo), `HybridCache`, expiración con *jitter*, refresco anticipado.
20. **¿Cómo garantizas un apagado ordenado en Kubernetes?** → Respetar `stoppingToken`/`RequestAborted`, `ShutdownTimeout` < `terminationGracePeriodSeconds`, operaciones idempotentes, nada crítico solo en memoria.

---

## Preguntas trampa

| Pregunta | Respuesta corta |
|---|---|
| "¿`await` crea un hilo nuevo?" | No. |
| "¿Un método `async` siempre se ejecuta en segundo plano?" | No: corre síncrono hasta el primer `await` incompleto, en el hilo del llamador. |
| "¿Después de `await` vuelvo al mismo hilo?" | Solo si hay un `SynchronizationContext` de un hilo (UI). En ASP.NET Core, no necesariamente. |
| "¿`ConfigureAwait(false)` evita que fluya `AsyncLocal`?" | No; afecta al `SynchronizationContext`, no al `ExecutionContext`. |
| "¿`Task.WhenAll` cancela las demás tareas si una falla?" | No; espera a todas. |
| "¿`WaitAsync(timeout)` cancela la operación?" | No; solo deja de esperarla. |
| "¿`cts.Cancel()` detiene la tarea inmediatamente?" | No; la cancelación es cooperativa. |
| "¿En ASP.NET Core `.Result` es seguro porque no hay deadlock?" | No; causa starvation bajo carga. |
| "¿Una excepción en una tarea no observada tumba el proceso?" | En .NET moderno no (sí en .NET 4.0). En `async void`, sí. |
| "¿`ValueTask` siempre es más rápido que `Task`?" | No; solo ahorra si completa síncronamente, y copiar el struct cuesta más. |

---
⬅️ [[12 - Testing de código asíncrono]] · ➡️ [[14 - Laboratorios y ejercicios]] · ⬆️ [[00 - Async en .NET - Índice]]
