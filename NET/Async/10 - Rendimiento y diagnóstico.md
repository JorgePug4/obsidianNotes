---
tags: [dotnet, csharp, async, nivel/10, rendimiento, diagnostico]
up: "[[00 - Async en .NET - Índice]]"
---

# 10 - Rendimiento y diagnóstico

> Nivel 🔴 10 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[03 - Cómo funciona por dentro]], [[07 - SynchronizationContext, ConfigureAwait y deadlocks]]. Complemento: [[Task vs ValueTask]].

## Contenido
- [[#Primero, medir]]
- [[#Thread pool starvation]]
- [[#Diagnosticar starvation]]
- [[#Herramientas de diagnóstico]]
- [[#Reducir asignaciones]]
- [[#Pooling e IValueTaskSource]]
- [[#Otras trampas de rendimiento]]
- [[#🧠 Practica]]

---

## Primero, medir

En código de aplicación, el coste de `async`/`await` (decenas de nanosegundos y una asignación pequeña) es **irrelevante** frente a una llamada de red (milisegundos). Los problemas reales de rendimiento async casi nunca son "async es lento", sino:

1. **Bloqueos** (sync-over-async) → *thread pool starvation*.
2. **Concurrencia mal dimensionada** (sin límite o secuencial cuando podía ser concurrente).
3. **Asignaciones en hot paths** de librerías de muy alto rendimiento (solo aquí importan `ValueTask` y el pooling).

Herramientas para medir: **BenchmarkDotNet** con `[MemoryDiagnoser]` para microbenchmarks; **dotnet-counters**, **dotnet-trace** y métricas de OpenTelemetry en entornos reales.

---

## Thread pool starvation

### Qué es
El thread pool no tiene hilos libres para ejecutar el trabajo encolado (incluidas las **continuaciones de `await`**), porque los hilos existentes están **bloqueados** esperando algo. El pool añade hilos, pero **despacio** (del orden de uno o dos por segundo cuando detecta que la cola no avanza), así que bajo carga sostenida no alcanza.

### Por qué ocurre (el círculo vicioso)

```text
Petición A en hilo 1 ── .Result ──► espera a su Task
                                     │
La respuesta de red de A llega ──► su continuación se ENCOLA en el pool
                                     │
¿Hay un hilo libre para ejecutarla? ── No: todos están en .Result como el hilo 1
                                     │
            Más peticiones llegan, más hilos se bloquean, la cola crece...
```

### Síntomas típicos
- Latencia que se dispara (segundos) **con CPU baja**.
- Timeouts en cascada, *health checks* que fallan → Kubernetes reinicia los pods ([[16 - Troubleshooting|ver troubleshooting de K8s]]).
- Número de hilos del proceso **creciendo lentamente** y sin parar.
- Se "arregla solo" al bajar la carga.

### Causas habituales
- `.Result`, `.Wait()`, `.GetAwaiter().GetResult()` en el camino de una petición.
- APIs síncronas de I/O en código de servidor (`File.ReadAllText`, `HttpWebRequest` síncrono, drivers de BD sin async).
- `Thread.Sleep` en lugar de `await Task.Delay`.
- Locks muy contendidos con trabajo lento dentro.
- Librerías antiguas que bloquean internamente (clientes de caché o SDKs sin soporte async real).

### Cómo se arregla
1. **Eliminar el bloqueo** (async all the way). Es la única solución real.
2. Mitigación temporal mientras se corrige: `ThreadPool.SetMinThreads(workers, io)` para que el pool arranque con más hilos. Esconde el síntoma y aumenta el consumo de memoria; **no** es una solución.

> [!note] .NET 6+
> Desde .NET 6 el thread pool está implementado en código administrado y compensa algunos bloqueos conocidos (como esperar sobre un `Task`) inyectando hilos más rápido. Reduce el problema pero **no** lo elimina: bajo suficiente carga, bloquear sigue provocando starvation.

---

## Diagnosticar starvation

### 1. Contadores en vivo

```bash
dotnet tool install -g dotnet-counters
dotnet-counters ps                              # localizar el PID
dotnet-counters monitor -p <PID> --counters System.Runtime
```

Indicadores (los nombres exactos varían por versión: en .NET 8 aparecen como `ThreadPool Queue Length` / `ThreadPool Thread Count`; desde .NET 9 el *meter* `System.Runtime` también los expone como `dotnet.thread_pool.queue.length` y `dotnet.thread_pool.thread.count`):

| Métrica | Normal | Starvation |
|---|---|---|
| Longitud de la cola del pool | ~0, picos breves | Crece y se mantiene alta |
| Número de hilos del pool | Estable, cerca de núcleos | Sube lentamente sin parar |
| Uso de CPU | Proporcional a la carga | **Bajo** pese a la carga |

### 2. ¿Dónde están bloqueados los hilos?

```bash
dotnet tool install -g dotnet-stack
dotnet-stack report -p <PID>        # pilas de todos los hilos: busca muchos en Task.Wait / Monitor.Wait / GetResult
```

O con un volcado:

```bash
dotnet-dump collect -p <PID>
dotnet-dump analyze <fichero.dmp>
> clrstack -all          # pilas de todos los hilos administrados
> dumpasync              # máquinas de estado async pendientes y quién espera a quién
> threadpool             # estado del thread pool
```

Si ves decenas de hilos con `System.Threading.Tasks.Task.Wait`/`TaskAwaiter.GetResult` en la pila, encontraste el bloqueo: sube por la pila hasta tu código.

### 3. Reproducir en local
Un test de carga (k6, `bombardier`, NBomber) contra un endpoint que hace `.Result` de un `Task.Delay(500)` reproduce la starvation en segundos. Es un buen ejercicio ([[14 - Laboratorios y ejercicios#Lab 9 - Provocar y diagnosticar thread pool starvation|Lab 9]]).

---

## Herramientas de diagnóstico

| Herramienta | Para qué |
|---|---|
| `dotnet-counters` | Métricas en vivo: thread pool, GC, excepciones, peticiones/s |
| `dotnet-trace` | Trazas de eventos (CPU sampling, eventos de TPL `System.Threading.Tasks.TplEventSource`) para PerfView o Visual Studio |
| `dotnet-stack` | Pilas de todos los hilos de un proceso vivo |
| `dotnet-dump` + SOS (`clrstack`, `dumpasync`, `threadpool`) | Análisis post-mortem o de procesos colgados |
| Visual Studio: *Parallel Stacks*, *Tasks*, *Threads* | Depuración local de deadlocks y tareas pendientes |
| BenchmarkDotNet + `[MemoryDiagnoser]` | Medir tiempo y asignaciones de métodos concretos |
| OpenTelemetry / Application Insights | Trazas distribuidas: ver qué dependencia es lenta |

---

## Reducir asignaciones

Solo relevante en **hot paths** (millones de llamadas por segundo) y **tras medir**.

### 1. Camino rápido síncrono sin máquina de estados

```csharp
public ValueTask<int> ReadAsync(Memory<byte> buffer, CancellationToken ct = default)
{
    if (_buffered > 0)                                    // camino rápido: hay datos en el buffer
        return new ValueTask<int>(CopyFromBuffer(buffer.Span)); // sin asignación, sin máquina de estados
    return ReadSlowAsync(buffer, ct);                     // camino lento: método async separado
}

private async ValueTask<int> ReadSlowAsync(Memory<byte> buffer, CancellationToken ct)
{
    await FillBufferAsync(ct);
    return CopyFromBuffer(buffer.Span);
}
```

### 2. `ValueTask<T>` cuando suele completar síncronamente
Reglas y detalles en [[Task vs ValueTask]].

### 3. Tareas cacheadas
`Task.CompletedTask`, `Task.FromResult(true)`; o cachear tú mismo un `Task<T>` con un valor frecuente (`static readonly Task<int> s_zero = Task.FromResult(0);`).

### 4. Evitar *closures* en lambdas calientes
Las lambdas que capturan variables locales asignan un objeto de cierre por llamada. Usa `static` lambdas y sobrecargas con estado cuando existan:

```csharp
// Captura 'this' y 'id' → asignación de closure
await Task.Run(() => Process(id));

// Sin captura (lambda static + estado explícito); útil en código muy caliente
await Task.Factory.StartNew(static s => ((Worker)s!).Process(), this,
    CancellationToken.None, TaskCreationOptions.DenyChildAttach, TaskScheduler.Default);
```

(Este último patrón es para librerías de bajo nivel; en código de aplicación, la legibilidad gana.)

---

## Pooling e IValueTaskSource

### `PoolingAsyncValueTaskMethodBuilder` (.NET 6)
Hace que un método `async ValueTask`/`ValueTask<T>` reutilice de un pool el objeto que representa la operación **también en el camino asíncrono**:

```csharp
[AsyncMethodBuilder(typeof(PoolingAsyncValueTaskMethodBuilder<>))]
public async ValueTask<int> ReceiveAsync(CancellationToken ct)
{
    await _socket.ReceiveAsync(_buffer, ct);
    return Parse(_buffer);
}
```

Reduce asignaciones, pero aumenta la memoria retenida por el pool y **amplifica** los riesgos de usar mal un `ValueTask` (esperarlo dos veces devuelve datos de otra operación). Úsalo solo con benchmarks que lo justifiquen.

### `IValueTaskSource<T>` y `ManualResetValueTaskSourceCore<T>`
Permiten implementar un objeto **reutilizable** que respalda `ValueTask`s, de modo que una operación repetida (recibir de un socket) no asigne nada en estado estable. Así funcionan internamente `Socket`, `System.IO.Pipelines` y `Channel<T>`. Es código de muy bajo nivel: impleméntalo solo si escribes infraestructura de red o de colas, y estudia antes la implementación de referencia en `dotnet/runtime`.

---

## Otras trampas de rendimiento

| Trampa | Por qué duele | Alternativa |
|---|---|---|
| `await` en bucle sobre operaciones independientes | Latencia = suma | Concurrencia limitada ([[04 - Composición de tareas]]) |
| Concurrencia ilimitada | Agota conexiones/sockets, provoca 429/timeouts | `SemaphoreSlim`, `Parallel.ForEachAsync` |
| `new HttpClient()` por petición | Agota puertos (sockets en `TIME_WAIT`), sin reutilizar conexiones | `IHttpClientFactory` o cliente estático con `SocketsHttpHandler.PooledConnectionLifetime` |
| `Task.Delay` con intervalos muy cortos | Resolución del temporizador de ~1-15 ms según SO | No hacer *polling* fino; usar señales (TCS, `Channel`) |
| Bucle `while(true)` sin `await` real en un `BackgroundService` | Ocupa un hilo del pool para siempre y retrasa el arranque del host | `PeriodicTimer`, `await` real, o `TaskCreationOptions.LongRunning` para bucles síncronos |
| `Task.Run` dentro de un endpoint de ASP.NET Core | Overhead de encolar sin ganar nada | Llamada directa |
| `async` en un método trivial que solo reenvía | Una máquina de estados más | Elidir con cuidado ([[03 - Cómo funciona por dentro#Coste de un método async\|ver reglas]]) |
| Excepciones como control de flujo (cancelaciones masivas) | Lanzar excepciones es caro | Comprobar `IsCancellationRequested` en bucles muy calientes |

---

## Puntos clave

- Mide antes de optimizar; el problema real suele ser bloqueo o concurrencia, no `async` en sí.
- **Thread pool starvation** = hilos bloqueados + pool que crece despacio → latencia alta con CPU baja.
- Diagnóstico: `dotnet-counters` (cola del pool) + `dotnet-stack`/`dotnet-dump` (dónde se bloquea).
- `SetMinThreads` mitiga, **no** arregla.
- `ValueTask`, caminos rápidos y pooling: solo en hot paths medidos.

## Errores comunes

- Subir `SetMinThreads` y dar el problema por resuelto.
- Cambiar todo a `ValueTask` "por rendimiento" sin benchmark.
- Buscar la causa de la latencia en la BD cuando el problema son hilos bloqueados en la propia app.
- Crear un `HttpClient` por petición.

## 🎯 Para entrevistas y exámenes

- *"Una API tiene latencias de 30 s con CPU al 10 %. ¿Qué investigas?"* → Thread pool starvation: `dotnet-counters` (cola del pool creciente, hilos subiendo), luego `dotnet-stack`/dump para ver hilos bloqueados en `.Result`/`.Wait()`/I/O síncrona.
- *"¿Por qué `SetMinThreads` no es la solución?"* → Solo retrasa el problema y consume más memoria; la causa (bloqueo) sigue.
- *"¿Cuándo usarías `PoolingAsyncValueTaskMethodBuilder`?"* → En un hot path async de una librería, tras medir que las asignaciones del camino asíncrono importan.

## 🧠 Practica

1. Lab de starvation: [[14 - Laboratorios y ejercicios#Lab 9 - Provocar y diagnosticar thread pool starvation|Lab 9]].
2. Haz un benchmark con BenchmarkDotNet de `Task<int>` vs `ValueTask<int>` en un método que el 99 % de las veces devuelve de caché. Compara `Allocated`.
3. Toma un volcado de un proceso con una tarea colgada (`await new TaskCompletionSource().Task`) y localízala con `dumpasync`.

## Referencias

- Microsoft Learn, *Debug ThreadPool starvation*: https://learn.microsoft.com/dotnet/core/diagnostics/debug-threadpool-starvation
- Microsoft Learn, *dotnet-counters*: https://learn.microsoft.com/dotnet/core/diagnostics/dotnet-counters
- Microsoft Learn, *dotnet-dump*: https://learn.microsoft.com/dotnet/core/diagnostics/dotnet-dump
- Stephen Toub, *Understanding the Whys, Whats, and Whens of ValueTask*: https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/
- Stephen Toub, *Performance Improvements in .NET* (serie anual en el .NET Blog, secciones de threading y async).

---
⬅️ [[09 - Sincronización y primitivas avanzadas]] · ➡️ [[11 - Async en ASP.NET Core y servicios en segundo plano]] · ⬆️ [[00 - Async en .NET - Índice]]
