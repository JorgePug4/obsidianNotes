---
tags: [dotnet, csharp, async, nivel/8, streams, channels]
up: "[[00 - Async en .NET - Índice]]"
---

# 08 - Streams asíncronos y Channels

> Nivel 🟠 8 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[04 - Composición de tareas]], [[05 - Cancelación y timeouts]].

## Contenido
- [[#El problema: muchos valores que llegan con el tiempo]]
- [[#IAsyncEnumerable y await foreach]]
- [[#Cancelación en streams asíncronos]]
- [[#LINQ sobre IAsyncEnumerable]]
- [[#IAsyncEnumerable en EF Core y ASP.NET Core]]
- [[#System.Threading.Channels]]
- [[#Channels acotados y backpressure]]
- [[#Pipeline de varias etapas]]
- [[#Channels vs alternativas]]
- [[#🧠 Practica]]

---

## El problema: muchos valores que llegan con el tiempo

- `Task<T>` → **un** valor en el futuro.
- `IEnumerable<T>` → **muchos** valores, pero obtener cada uno es **síncrono**.
- `Task<List<T>>` → muchos valores, pero esperas a **todos** antes de procesar el primero (memoria y latencia).
- `IAsyncEnumerable<T>` → **muchos** valores, cada uno obtenido **de forma asíncrona**. Procesas a medida que llegan.

Ejemplos: leer un fichero de 5 GB línea a línea, paginar una API, leer filas de una consulta enorme, consumir eventos.

---

## IAsyncEnumerable y await foreach

Disponible desde C# 8 / .NET Core 3.0. Un **iterador asíncrono** combina `async` y `yield return`:

```csharp
public async IAsyncEnumerable<Customer> GetAllCustomersAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    string? nextPage = "customers?page=1";
    while (nextPage is not null)
    {
        var page = await _http.GetFromJsonAsync<Page<Customer>>(nextPage, ct);
        foreach (var c in page!.Items)
            yield return c;                 // el consumidor recibe cada cliente sin esperar a todas las páginas
        nextPage = page.Next;
    }
}

// Consumo
await foreach (var customer in GetAllCustomersAsync(ct))
{
    await SyncAsync(customer, ct);
}
```

Características:
- **Perezoso (*pull*)**: no se pide la página 2 hasta que el consumidor termina con la página 1. El consumidor marca el ritmo → *backpressure* natural.
- Memoria constante: solo una página a la vez.
- `await foreach` llama a `DisposeAsync` del enumerador al terminar (incluso con `break` o excepción), lo que ejecuta los `finally` del iterador.

---

## Cancelación en streams asíncronos

Dos formas de pasar un token, que se combinan gracias a `[EnumeratorCancellation]`:

```csharp
// 1. Como argumento del método
await foreach (var c in GetAllCustomersAsync(ct)) { ... }

// 2. Con WithCancellation (útil cuando recibes un IAsyncEnumerable<T> ya creado)
IAsyncEnumerable<Customer> stream = GetAllCustomersAsync();
await foreach (var c in stream.WithCancellation(ct)) { ... }
```

Sin el atributo `[EnumeratorCancellation]`, el token de `WithCancellation` **no llega** a tu iterador (el compilador avisa con CS8425).

`ConfigureAwait(false)` también se aplica al stream: `await foreach (var x in stream.ConfigureAwait(false))`.

---

## LINQ sobre IAsyncEnumerable

Los operadores LINQ (`Where`, `Select`, `Take`, `ToListAsync`…) para `IAsyncEnumerable<T>` venían históricamente del paquete **`System.Linq.Async`** (proyecto dotnet/reactive). A partir de **.NET 10** la BCL incluye `System.Linq.AsyncEnumerable` con estos operadores; si apuntas a versiones anteriores, usa el paquete.

```csharp
var vip = await GetAllCustomersAsync(ct)
    .Where(c => c.TotalSpent > 10_000)
    .Take(100)
    .ToListAsync(ct);
```

---

## IAsyncEnumerable en EF Core y ASP.NET Core

```csharp
// EF Core: streaming de filas sin cargar todo en memoria
await foreach (var order in db.Orders.Where(o => o.Year == 2026).AsAsyncEnumerable().WithCancellation(ct))
{
    await exporter.WriteAsync(order, ct);
}

// ASP.NET Core (.NET 6+): System.Text.Json serializa el array a medida que se generan los elementos
app.MapGet("/orders/export", (AppDbContext db) =>
    db.Orders.AsNoTracking().AsAsyncEnumerable());
```

> [!warning] Conexión abierta durante todo el stream
> Mientras recorres `AsAsyncEnumerable()`, la conexión a la BD sigue abierta y el `DbContext` está ocupado: no puedes lanzar otra consulta con el mismo contexto dentro del bucle. Si el procesamiento de cada fila es lento, considera leer por lotes.

---

## System.Threading.Channels

Un `Channel<T>` es una **cola asíncrona segura para hilos** que conecta **productores** y **consumidores**. Es la herramienta moderna para patrones productor-consumidor en .NET (incluida en la BCL desde .NET Core 3.0).

```text
 Productores ──► [ Writer ] ═══ cola ═══ [ Reader ] ──► Consumidores
                WriteAsync                ReadAllAsync
                Complete()                (termina cuando se completa y se vacía)
```

```csharp
var channel = Channel.CreateUnbounded<Order>();

// Productor
var producer = Task.Run(async () =>
{
    foreach (var order in await LoadOrdersAsync(ct))
        await channel.Writer.WriteAsync(order, ct);
    channel.Writer.Complete();                     // ⚠️ imprescindible: si no, el consumidor espera para siempre
});

// Consumidor
var consumer = Task.Run(async () =>
{
    await foreach (var order in channel.Reader.ReadAllAsync(ct))
        await ProcessAsync(order, ct);
});

await Task.WhenAll(producer, consumer);
```

Para propagar un error del productor al consumidor: `channel.Writer.Complete(exception)`. El consumidor recibirá esa excepción al leer.

---

## Channels acotados y backpressure

Un canal **sin límite** (`CreateUnbounded`) crece sin control si el productor es más rápido que el consumidor → `OutOfMemoryException` en producción. En la mayoría de los casos usa un canal **acotado**:

```csharp
var channel = Channel.CreateBounded<WorkItem>(new BoundedChannelOptions(capacity: 1_000)
{
    FullMode = BoundedChannelFullMode.Wait,   // el productor ESPERA (asíncronamente) si está lleno
    SingleReader = false,                     // true permite optimizaciones si solo hay un consumidor
    SingleWriter = false
});
```

| `FullMode` | Cuando el canal está lleno… | Uso típico |
|---|---|---|
| `Wait` (por defecto) | `WriteAsync` espera hasta que haya hueco | Trabajo que no puede perderse → **backpressure** |
| `DropOldest` | Descarta el elemento más antiguo | Telemetría, "último estado" |
| `DropNewest` | Descarta el elemento más reciente de la cola | Casos poco comunes |
| `DropWrite` | Descarta el que intentas escribir | Muestreo bajo carga |

Con `Drop*`, `TryWrite` y `WriteAsync` devuelven éxito aunque se descarte algo; desde .NET 6 puedes pasar un callback `itemDropped` a `CreateBounded` para registrarlo.

### Varios consumidores en paralelo

```csharp
var consumers = Enumerable.Range(0, 4).Select(_ => Task.Run(async () =>
{
    await foreach (var item in channel.Reader.ReadAllAsync(ct))
        await HandleAsync(item, ct);
})).ToArray();

await Task.WhenAll(consumers);   // terminan cuando el writer se completa y la cola se vacía
```

Cada elemento lo recibe **un solo** consumidor (no es pub/sub).

---

## Pipeline de varias etapas

```csharp
// descargar → parsear → guardar, cada etapa con su propio grado de concurrencia
var urls    = Channel.CreateBounded<string>(100);
var pages   = Channel.CreateBounded<string>(50);
var records = Channel.CreateBounded<Record>(500);

Task stage1 = RunStageAsync(urls.Reader, pages.Writer, workers: 8,
    async (url, ct) => await http.GetStringAsync(url, ct), ct);

Task stage2 = RunStageAsync(pages.Reader, records.Writer, workers: Environment.ProcessorCount,
    (html, ct) => ValueTask.FromResult(Parser.Parse(html)), ct);   // etapa CPU-bound

Task stage3 = Task.Run(async () =>
{
    var batch = new List<Record>(200);
    await foreach (var record in records.Reader.ReadAllAsync(ct))
    {
        batch.Add(record);
        if (batch.Count == 200) { await repo.BulkInsertAsync(batch, ct); batch.Clear(); }
    }
    if (batch.Count > 0) await repo.BulkInsertAsync(batch, ct);   // último lote parcial
}, ct);

foreach (var u in seedUrls) await urls.Writer.WriteAsync(u, ct);
urls.Writer.Complete();
await Task.WhenAll(stage1, stage2, stage3);

static async Task RunStageAsync<TIn, TOut>(
    ChannelReader<TIn> input, ChannelWriter<TOut> output, int workers,
    Func<TIn, CancellationToken, ValueTask<TOut>> transform, CancellationToken ct)
{
    try
    {
        await Parallel.ForEachAsync(input.ReadAllAsync(ct),
            new ParallelOptions { MaxDegreeOfParallelism = workers, CancellationToken = ct },
            async (item, token) => await output.WriteAsync(await transform(item, token), token));
        output.Complete();
    }
    catch (Exception ex)
    {
        output.Complete(ex);       // propaga el fallo a la siguiente etapa
        throw;
    }
}
```

Los canales acotados entre etapas hacen que, si la BD es lenta, la etapa 3 frene a la 2 y esta a la 1: **la memoria queda acotada** y nadie se desborda.

> [!note] Sobre el ejemplo
> `stage2` con `transform` síncrono funciona, pero si `Parse` es pesado conviene que la etapa use `Task.Run` o que `Parallel.ForEachAsync` reparta el trabajo (ya lo hace sobre el pool). El objetivo del ejemplo es mostrar la estructura.

---

## Channels vs alternativas

| Herramienta | Async | Backpressure | Cuándo |
|---|---|---|---|
| `Channel<T>` | ✅ | ✅ (acotado) | Productor-consumidor en proceso. **Opción por defecto** |
| `BlockingCollection<T>` | ❌ (bloquea hilos) | ✅ | Código síncrono heredado; evitar en código async |
| `ConcurrentQueue<T>` | — (no espera) | ❌ | Estructura de datos, no coordina espera |
| TPL Dataflow (`System.Threading.Tasks.Dataflow`) | ✅ | ✅ | Grafos complejos con *linking*, *batching*, *broadcast* integrados |
| `IObservable<T>` / Rx | Push | Limitado | Flujos de eventos con operadores temporales (throttle, buffer por tiempo) |
| Cola externa (Service Bus, RabbitMQ, Redis Streams) | ✅ | ✅ | Trabajo que **debe sobrevivir** a un reinicio o repartirse entre procesos. Ver [[00 - Indice y ruta de estudio\|Azure Service Bus]] |

> [!warning] Un `Channel` vive en memoria
> Si el proceso se reinicia (despliegue, OOM, `SIGKILL`), **lo que esté en el canal se pierde**. Para trabajo crítico, usa una cola persistente.

---

## Puntos clave

- `IAsyncEnumerable<T>` + `await foreach` = muchos valores asíncronos, procesados a medida que llegan, con memoria constante.
- Usa `[EnumeratorCancellation]` en el parámetro `CancellationToken` del iterador.
- `Channel<T>` = cola asíncrona productor-consumidor. **Llama siempre a `Writer.Complete()`**.
- Prefiere canales **acotados** con `FullMode.Wait` para tener backpressure.
- Los canales son en memoria: no garantizan durabilidad.

## Errores comunes

- Devolver `Task<List<T>>` con millones de elementos en lugar de streaming.
- Olvidar `Writer.Complete()` → consumidores colgados para siempre.
- `CreateUnbounded` con un productor más rápido que el consumidor.
- Usar `BlockingCollection` en código async.
- Tratar un `Channel` como cola duradera.

## 🎯 Para entrevistas y exámenes

- *"¿Diferencia entre `Task<IEnumerable<T>>` e `IAsyncEnumerable<T>`?"* → El primero espera a tener todos los elementos; el segundo entrega cada uno en cuanto está disponible.
- *"¿Cómo implementas productor-consumidor en .NET moderno?"* → `System.Threading.Channels`, acotado, con `WriteAsync`/`ReadAllAsync` y `Complete()`.
- *"¿Qué es backpressure?"* → Que el consumidor lento frene al productor (canal acotado con `Wait`), para que la memoria no crezca sin límite.

## 🧠 Practica

1. Escribe un iterador async que lea un fichero de texto grande línea a línea con `StreamReader.ReadLineAsync` y cuente palabras.
2. Implementa un productor que genere 10 000 números y 3 consumidores que los sumen, con un canal acotado de 100. Verifica la suma total.
3. Cambia `FullMode` a `DropOldest` con capacidad 10 y un consumidor lento. ¿Cuántos elementos se pierden?

## Referencias

- Microsoft Learn, *Generate and consume async streams*: https://learn.microsoft.com/dotnet/csharp/asynchronous-programming/generate-consume-asynchronous-stream
- Microsoft Learn, *System.Threading.Channels library*: https://learn.microsoft.com/dotnet/core/extensions/channels
- Stephen Toub, *An Introduction to System.Threading.Channels*: https://devblogs.microsoft.com/dotnet/an-introduction-to-system-threading-channels/

---
⬅️ [[07 - SynchronizationContext, ConfigureAwait y deadlocks]] · ➡️ [[09 - Sincronización y primitivas avanzadas]] · ⬆️ [[00 - Async en .NET - Índice]]
