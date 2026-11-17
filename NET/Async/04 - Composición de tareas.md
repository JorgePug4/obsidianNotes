---
tags: [dotnet, csharp, async, nivel/4, concurrencia]
up: "[[00 - Async en .NET - Índice]]"
---

# 04 - Composición de tareas

> Nivel 🟡 4 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[02 - async y await - lo básico]].

## Contenido
- [[#Secuencial vs concurrente]]
- [[#Task.WhenAll]]
- [[#Task.WhenAny]]
- [[#Task.WhenEach (.NET 9)]]
- [[#Limitar la concurrencia]]
- [[#Parallel.ForEachAsync (.NET 6)]]
- [[#Recursos que no admiten concurrencia]]
- [[#Elegir la herramienta]]
- [[#🧠 Practica]]

---

## Secuencial vs concurrente

```csharp
// Secuencial: ~300 ms (100 + 200)
var user   = await GetUserAsync(id);     // 100 ms
var orders = await GetOrdersAsync(id);   // 200 ms

// Concurrente: ~200 ms (el máximo de los dos)
Task<User>         userTask   = GetUserAsync(id);    // se INICIA aquí
Task<List<Order>>  ordersTask = GetOrdersAsync(id);  // se INICIA aquí
var user2   = await userTask;
var orders2 = await ordersTask;
```

Clave: **un `Task` empieza a ejecutarse cuando llamas al método, no cuando haces `await`**. `await` solo espera el resultado. Para ejecutar en concurrencia, **inicia primero y espera después**.

> [!warning] Solo si son independientes
> Si `GetOrdersAsync` necesita el resultado de `GetUserAsync`, la versión secuencial es la correcta. Y si ambas usan el mismo `DbContext`, **no puedes** ejecutarlas a la vez (ver [[#Recursos que no admiten concurrencia]]).

---

## Task.WhenAll

Devuelve una tarea que termina cuando **todas** terminan.

```csharp
Task<User>        u = GetUserAsync(id);
Task<List<Order>> o = GetOrdersAsync(id);
await Task.WhenAll(u, o);
// Ambas completadas: leer .Result aquí es seguro (no bloquea) aunque es más claro usar await
var user = await u;
var orders = await o;

// Con una colección del mismo tipo: devuelve T[] en el mismo orden de entrada
string[] pages = await Task.WhenAll(urls.Select(url => client.GetStringAsync(url, ct)));
```

Comportamiento:
- **Espera a todas**, incluso si alguna falla pronto (no cancela las demás).
- Si alguna falla, la tarea resultante queda `Faulted` con **todas** las excepciones en su `AggregateException`, pero `await` relanza **solo la primera**. Cómo ver todas: [[06 - Manejo de excepciones#Excepciones con WhenAll|nivel 6]].
- Si ninguna falla pero alguna se cancela, el resultado queda `Canceled`.

> [!danger] `urls.Select(...)` sin límite
> `Task.WhenAll(urls.Select(...))` lanza **todas** las operaciones a la vez. Con 10 000 URLs abres 10 000 peticiones simultáneas: agotas sockets, te bloquea el servidor remoto (429) o saturas la BD. Para colecciones grandes, [[#Limitar la concurrencia|limita la concurrencia]].

---

## Task.WhenAny

Devuelve una tarea que termina cuando **la primera** termina (bien, mal o cancelada). El resultado es **la tarea que terminó**, no su valor.

```csharp
Task<string> primary   = GetFromPrimaryAsync(ct);
Task<string> secondary = GetFromSecondaryAsync(ct);

Task<string> winner = await Task.WhenAny(primary, secondary);
string value = await winner; // ← segundo await: obtiene el valor o relanza la excepción del ganador
```

Usos típicos:
- **Timeout manual** (aunque hoy es mejor `WaitAsync`, ver [[05 - Cancelación y timeouts#WaitAsync (.NET 6)|nivel 5]]).
- **Hedging**: pedir a dos réplicas y quedarte con la más rápida (cancela la otra después).
- Procesar resultados en orden de llegada (en .NET 9+ mejor `Task.WhenEach`).

> [!warning] Las perdedoras siguen ejecutándose
> `WhenAny` no cancela nada. Si no las cancelas tú (con un `CancellationTokenSource`), las otras tareas siguen consumiendo recursos y sus excepciones quedan sin observar.

```csharp
using var cts = CancellationTokenSource.CreateLinkedTokenSource(ct);
var tasks = new[] { GetFromPrimaryAsync(cts.Token), GetFromSecondaryAsync(cts.Token) };
var first = await Task.WhenAny(tasks);
cts.Cancel();               // detén a las perdedoras
return await first;
```

---

## Task.WhenEach (.NET 9)

Procesa tareas **a medida que terminan**, como `IAsyncEnumerable`:

```csharp
Task<Report>[] tasks = sources.Select(s => BuildReportAsync(s, ct)).ToArray();

await foreach (Task<Report> done in Task.WhenEach(tasks))
{
    Report r = await done;             // ya está completa; await obtiene valor o excepción
    Console.WriteLine($"Listo: {r.Name}");
}
```

Antes de .NET 9 el patrón equivalente era un bucle con `WhenAny` y `tasks.Remove(done)`, que es **O(n²)** y propenso a errores.

---

## Limitar la concurrencia

Tres formas, de la más general a la más cómoda.

### 1. `SemaphoreSlim` (funciona en cualquier versión)

```csharp
public async Task<string[]> DownloadAllAsync(IEnumerable<string> urls, int maxConcurrency, CancellationToken ct)
{
    using var gate = new SemaphoreSlim(maxConcurrency);

    var tasks = urls.Select(async url =>
    {
        await gate.WaitAsync(ct);          // espera turno SIN bloquear un hilo
        try
        {
            return await _client.GetStringAsync(url, ct);
        }
        finally
        {
            gate.Release();                // libera SIEMPRE, incluso si falla
        }
    });

    return await Task.WhenAll(tasks);
}
```

Nota: aquí se **crean** todas las tareas de golpe (cada una espera en el semáforo). Con millones de elementos, eso son millones de objetos en memoria; en ese caso usa `Parallel.ForEachAsync` o un `Channel` ([[08 - Streams asíncronos y Channels]]).

### 2. `Parallel.ForEachAsync` (.NET 6+)
Ver la siguiente sección. Es la opción más simple cuando no necesitas recoger resultados en orden.

### 3. Lotes (`Chunk`, .NET 6+)

```csharp
foreach (var batch in items.Chunk(20))
{
    await Task.WhenAll(batch.Select(i => ProcessAsync(i, ct)));
}
```

Simple, pero menos eficiente: cada lote espera a su elemento **más lento** antes de empezar el siguiente.

---

## Parallel.ForEachAsync (.NET 6)

```csharp
var options = new ParallelOptions
{
    MaxDegreeOfParallelism = 8,   // por defecto: Environment.ProcessorCount
    CancellationToken = ct
};

await Parallel.ForEachAsync(orderIds, options, async (id, token) =>
{
    var order = await _api.GetOrderAsync(id, token);
    await _store.SaveAsync(order, token);
});
```

- Mantiene **como máximo N** operaciones en curso; en cuanto una termina, empieza la siguiente (mejor que los lotes).
- Consume la colección de forma perezosa (también acepta `IAsyncEnumerable<T>`).
- Si un cuerpo lanza, **deja de iniciar** nuevos elementos, espera a los que están en curso y relanza.
- No devuelve resultados: si los necesitas, guárdalos en una colección segura para hilos (`ConcurrentBag<T>`, `ConcurrentDictionary`) o usa el patrón con `SemaphoreSlim`.

> [!tip] Elige N con criterio
> Para I/O, el valor por defecto (núcleos) suele ser **bajo**: el cuello de botella es el sistema remoto, no tu CPU. Ajusta N según lo que tolera el destino (límite de conexiones del pool de BD, *rate limit* de la API) y mide.

> [!note] `Parallel.ForEach` (sin Async) es para CPU
> `Parallel.ForEach` y PLINQ son para trabajo **CPU-bound** síncrono. Pasarles una lambda async crea lambdas `async void` (porque el delegado es `Action<T>`): no esperan nada. Para async usa siempre `Parallel.ForEachAsync`.

---

## Recursos que no admiten concurrencia

No todo se puede usar desde varias tareas a la vez. Ejemplos clásicos:

| Recurso | ¿Concurrente? | Qué hacer |
|---|---|---|
| `DbContext` (EF Core) | ❌ Lanza `InvalidOperationException: A second operation was started on this context instance before a previous operation completed` | Una instancia por operación concurrente (`IDbContextFactory<T>`) o ejecución secuencial |
| `SqlConnection` | ❌ (salvo MARS, con limitaciones) | Una conexión por tarea |
| `HttpClient` | ✅ Seguro y diseñado para reutilizarse | Instancia compartida o `IHttpClientFactory` |
| `List<T>`, `Dictionary<TK,TV>` | ❌ para escrituras concurrentes | `ConcurrentDictionary`, `lock` alrededor de la escritura, o recoger resultados de `WhenAll` |
| `ServiceBusSender` | ✅ | Reutilizar |
| `Stream` | ❌ (una operación de lectura/escritura a la vez) | Serializar el acceso |

```csharp
// ✅ Concurrencia con EF Core: un contexto por tarea
public async Task<(int users, int orders)> CountsAsync(CancellationToken ct)
{
    await using var db1 = await _factory.CreateDbContextAsync(ct);
    await using var db2 = await _factory.CreateDbContextAsync(ct);
    var t1 = db1.Users.CountAsync(ct);
    var t2 = db2.Orders.CountAsync(ct);
    await Task.WhenAll(t1, t2);
    return (t1.Result, t2.Result);   // seguro: ya están completas
}
```

---

## Elegir la herramienta

| Necesito… | Herramienta |
|---|---|
| Esperar 2-5 operaciones independientes | Iniciar todas + `Task.WhenAll` |
| Procesar una colección **pequeña** (decenas) en concurrencia | `Task.WhenAll(items.Select(...))` |
| Procesar una colección **grande** con límite | `Parallel.ForEachAsync` o `SemaphoreSlim` |
| Recoger resultados de una colección grande con límite | `SemaphoreSlim` + `WhenAll`, o `Channel` |
| Quedarme con la primera respuesta | `Task.WhenAny` + cancelar el resto |
| Procesar resultados en orden de llegada | `Task.WhenEach` (.NET 9+) |
| Pipeline de varias etapas con *backpressure* | `System.Threading.Channels` o TPL Dataflow |
| Trabajo CPU-bound sobre datos en memoria | `Parallel.For/ForEach`, PLINQ |

---

## Puntos clave

- Las tareas **empiezan al llamar** al método; inicia varias y espera después para concurrencia.
- `WhenAll` espera a todas y no cancela al fallar una; `await` relanza solo la primera excepción.
- `WhenAny` devuelve la tarea ganadora; las demás **siguen ejecutándose**.
- Nunca lances concurrencia **sin límite** sobre colecciones grandes.
- `Parallel.ForEachAsync` para async; `Parallel.ForEach` solo para CPU síncrono.
- `DbContext` **no** admite operaciones concurrentes.

## Errores comunes

- `await` dentro de un bucle cuando las operaciones son independientes (lento sin necesidad).
- `Task.WhenAll` sobre 10 000 elementos sin limitar.
- Olvidar el segundo `await` tras `WhenAny`.
- `Parallel.ForEach(items, async item => ...)` → async void.
- Compartir un `DbContext` entre tareas concurrentes.

## 🎯 Para entrevistas y exámenes

- *"¿Cómo ejecutas dos llamadas independientes en paralelo?"* → Iniciar ambas sin `await`, después `await Task.WhenAll(t1, t2)`.
- *"¿Cómo limitas a 10 peticiones simultáneas?"* → `SemaphoreSlim(10)` con `WaitAsync`/`Release` en `finally`, o `Parallel.ForEachAsync` con `MaxDegreeOfParallelism = 10`.
- *"¿Qué pasa con las demás tareas tras `WhenAny`?"* → Siguen ejecutándose; hay que cancelarlas explícitamente.

## 🧠 Practica

1. Mide con `Stopwatch` la versión secuencial y la concurrente de dos `Task.Delay(500)`.
2. Descarga 200 URLs con un máximo de 10 simultáneas usando `SemaphoreSlim` y luego con `Parallel.ForEachAsync`.
3. Implementa *hedging*: pide a dos endpoints, devuelve el primero que responda **bien** (si el primero falla, espera al segundo).

## Referencias

- Microsoft Learn, *Task.WhenAll*: https://learn.microsoft.com/dotnet/api/system.threading.tasks.task.whenall
- Microsoft Learn, *Task.WhenEach* (.NET 9): https://learn.microsoft.com/dotnet/api/system.threading.tasks.task.wheneach
- Microsoft Learn, *Parallel.ForEachAsync*: https://learn.microsoft.com/dotnet/api/system.threading.tasks.parallel.foreachasync
- Microsoft Learn, *DbContext lifetime, configuration, and initialization* (sección *Avoiding DbContext threading issues*): https://learn.microsoft.com/ef/core/dbcontext-configuration/

---
⬅️ [[03 - Cómo funciona por dentro]] · ➡️ [[05 - Cancelación y timeouts]] · ⬆️ [[00 - Async en .NET - Índice]]
