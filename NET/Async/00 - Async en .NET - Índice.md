---
tags: [dotnet, csharp, async, MOC]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [Curso Async .NET, Índice Async, MOC Async]
---

# ⚡ Async en .NET - Índice

> [!info] Qué es este curso
> Una ruta de **básico a máster** sobre programación asíncrona en C#/.NET: desde "¿por qué existe `await`?" hasta la máquina de estados que genera el compilador, `SynchronizationContext`, *thread pool starvation*, `Channels`, `IValueTaskSource` y diagnóstico en producción.
> Versiones de referencia: **.NET 8 (LTS)**, **.NET 9** y **.NET 10 (LTS)**. Cuando una API depende de versión, se indica.

## 📈 Roadmap

| Nivel | Nota | Qué aprendes | Al terminar sabes… |
|---|---|---|---|
| 🟢 1 | [[01 - Fundamentos - concurrencia, paralelismo y asincronía]] | Hilos, *thread pool*, I/O-bound vs CPU-bound, por qué async escala | Explicar **cuándo** async ayuda y cuándo no |
| 🟢 2 | [[02 - async y await - lo básico]] | `Task`, `Task<T>`, `async`/`await`, tipos de retorno, `async void`, `Task.Run` | Escribir métodos async correctos |
| 🟡 3 | [[03 - Cómo funciona por dentro]] | Máquina de estados, *awaiter pattern*, continuaciones, `ExecutionContext` | Explicar qué hace el compilador con `await` |
| 🟡 4 | [[04 - Composición de tareas]] | `WhenAll`, `WhenAny`, `WhenEach`, `Parallel.ForEachAsync`, limitar concurrencia | Ejecutar trabajo en paralelo de forma controlada |
| 🟡 5 | [[05 - Cancelación y timeouts]] | `CancellationToken`, tokens enlazados, `WaitAsync`, `TimeProvider` | Hacer operaciones cancelables y con tiempo límite |
| 🟡 6 | [[06 - Manejo de excepciones]] | Propagación, `AggregateException`, excepciones no observadas, *fire-and-forget* | No perder errores nunca |
| 🟠 7 | [[07 - SynchronizationContext, ConfigureAwait y deadlocks]] | Contextos, `ConfigureAwait(false)`, *sync-over-async*, deadlocks | Diagnosticar y evitar el deadlock clásico |
| 🟠 8 | [[08 - Streams asíncronos y Channels]] | `IAsyncEnumerable`, `await foreach`, `System.Threading.Channels` | Construir pipelines productor-consumidor |
| 🟠 9 | [[09 - Sincronización y primitivas avanzadas]] | `SemaphoreSlim`, `TaskCompletionSource`, `AsyncLocal`, `PeriodicTimer`, `Lazy` async | Coordinar tareas sin bloquear hilos |
| 🔴 10 | [[10 - Rendimiento y diagnóstico]] | Starvation, asignaciones, [[Task vs ValueTask]], `IValueTaskSource`, `dotnet-counters`/`dotnet-trace` | Encontrar y arreglar problemas async en producción |
| 🔴 11 | [[11 - Async en ASP.NET Core y servicios en segundo plano]] | Request pipeline, `BackgroundService`, colas en memoria, EF Core, `HttpClient`, apagado ordenado | Diseñar servicios async de producción |
| 🔴 12 | [[12 - Testing de código asíncrono]] | Tests async, `FakeTimeProvider`, determinismo, tests de concurrencia | Probar código async sin `Thread.Sleep` |
| 🏁 13 | [[13 - Anti-patrones, checklist y entrevistas]] | Catálogo de errores, checklist de *code review*, preguntas de entrevista | Revisar código async como un senior |
| 🧪 14 | [[14 - Laboratorios y ejercicios]] | Ejercicios progresivos con soluciones | Consolidar todo con práctica |

Nota complementaria (nivel 🔴): [[Task vs ValueTask]].

## 🧭 Cómo estudiar este curso

1. **Niveles 1-2 (base):** léelos completos y escribe los ejemplos. Sin esto, el resto no se entiende.
2. **Niveles 3-6 (intermedio):** aquí está el 80 % de lo que usas en el día a día. Haz los labs 1-6 de [[14 - Laboratorios y ejercicios]].
3. **Niveles 7-9 (avanzado):** lo que separa a quien "usa async" de quien lo **entiende**. Reproduce el deadlock del nivel 7 en una app de consola con un `SynchronizationContext` propio.
4. **Niveles 10-12 (máster):** diagnóstico, rendimiento y diseño. Útil para producción y entrevistas senior.
5. **Repaso:** [[13 - Anti-patrones, checklist y entrevistas]] antes de una entrevista o un *code review*.

## 🗺️ Mapa conceptual

```text
⚡ Async en .NET
├── 🧱 Fundamentos
│   ├── Concurrencia ≠ paralelismo ≠ asincronía
│   ├── Thread, ThreadPool, I/O completion
│   └── I/O-bound (await) vs CPU-bound (Task.Run / Parallel)
├── ✍️ Lenguaje
│   ├── async / await · Task / Task<T> / ValueTask
│   ├── async void (solo event handlers)
│   └── await using · await foreach · IAsyncEnumerable
├── ⚙️ Runtime
│   ├── Máquina de estados (IAsyncStateMachine)
│   ├── Awaiter pattern (GetAwaiter / IsCompleted / OnCompleted / GetResult)
│   ├── SynchronizationContext · TaskScheduler
│   └── ExecutionContext · AsyncLocal
├── 🔗 Composición y control
│   ├── WhenAll · WhenAny · WhenEach · Parallel.ForEachAsync
│   ├── CancellationToken · WaitAsync · TimeProvider
│   └── Excepciones · AggregateException · UnobservedTaskException
├── 🔒 Coordinación
│   ├── SemaphoreSlim · Channels · TaskCompletionSource
│   └── PeriodicTimer · Lazy<Task<T>>
└── 🚀 Producción
    ├── Thread pool starvation · sync-over-async
    ├── ValueTask · IValueTaskSource · pooling
    ├── ASP.NET Core · BackgroundService · EF Core · HttpClient
    └── dotnet-counters · dotnet-trace · dotnet-dump
```

## 🔗 Conexiones con otras notas del vault

- [[Go routines]]: comparación entre `async`/`await` (cooperativo, sin hilo durante la espera) y goroutines (hilos verdes M:N).
- [[Retry con Backoff Exponencial]], [[Circuit Breaker]], [[Bulkhead]]: resiliencia con Polly, que es 100 % async (`ResiliencePipeline.ExecuteAsync`).
- [[04 - NET y entorno local|Azure Service Bus con .NET]]: `ServiceBusProcessor` es un consumidor async con concurrencia configurable.
- [[13 - Kubernetes para .NET]]: apagado ordenado (`SIGTERM` → `CancellationToken` del host) y por qué el *thread pool starvation* se ve como "CPU baja y latencia alta".

## ✅ Checklist de dominio (autoevaluación)

- [ ] Sé explicar por qué `await` **no** crea un hilo.
- [ ] Sé cuándo usar `Task.Run` y cuándo es un anti-patrón.
- [ ] Puedo dibujar la máquina de estados de un método con dos `await`.
- [ ] Sé por qué `.Result` puede bloquear para siempre en WinForms/WPF/ASP.NET clásico y no en ASP.NET Core.
- [ ] Sé qué excepción ves al hacer `await Task.WhenAll(...)` con dos tareas fallidas, y cómo ver ambas.
- [ ] Sé propagar `CancellationToken` y combinarlo con un timeout.
- [ ] Sé limitar la concurrencia a N operaciones simultáneas de tres formas distintas.
- [ ] Sé construir un productor-consumidor con `Channel<T>` acotado.
- [ ] Sé diagnosticar *thread pool starvation* con `dotnet-counters`.
- [ ] Sé las 4 reglas de `ValueTask`.
- [ ] Sé probar un método con `Task.Delay` sin esperar el tiempo real.

## Referencias generales

- Microsoft Learn, *Asynchronous programming with async and await*: https://learn.microsoft.com/dotnet/csharp/asynchronous-programming/
- Microsoft Learn, *Task-based Asynchronous Pattern (TAP)*: https://learn.microsoft.com/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap
- Stephen Toub, *How Async/Await Really Works in C#* (.NET Blog, 2023): https://devblogs.microsoft.com/dotnet/how-async-await-really-works/
- Stephen Toub, *ConfigureAwait FAQ* (.NET Blog, 2019): https://devblogs.microsoft.com/dotnet/configureawait-faq/
- Stephen Cleary, *There Is No Thread* (2013): https://blog.stephencleary.com/2013/11/there-is-no-thread.html
- Stephen Cleary, *Concurrency in C# Cookbook*, 2.ª ed. (O'Reilly, 2019).
- David Fowler, *Async Guidance*: https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
