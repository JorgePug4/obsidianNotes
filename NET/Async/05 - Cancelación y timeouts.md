---
tags: [dotnet, csharp, async, nivel/5, cancelacion]
up: "[[00 - Async en .NET - Índice]]"
---

# 05 - Cancelación y timeouts

> Nivel 🟡 5 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[04 - Composición de tareas]].

## Contenido
- [[#El modelo cooperativo]]
- [[#CancellationTokenSource y CancellationToken]]
- [[#Propagar el token]]
- [[#Observar la cancelación en tu código]]
- [[#Timeouts]]
- [[#Tokens enlazados]]
- [[#WaitAsync (.NET 6)]]
- [[#Excepciones de cancelación]]
- [[#Cancelación en ASP.NET Core y en el host]]
- [[#TimeProvider (.NET 8)]]
- [[#Cuándo NO cancelar]]
- [[#🧠 Practica]]

---

## El modelo cooperativo

En .NET **no se puede matar una tarea desde fuera** (`Thread.Abort` ya no existe en .NET Core). La cancelación es **cooperativa**:

1. Alguien **solicita** la cancelación (`cts.Cancel()`).
2. El código que trabaja **comprueba** el token o se lo pasa a APIs que lo comprueban.
3. Ese código **decide** detenerse limpiamente, normalmente lanzando `OperationCanceledException`.

Analogía: el token es una bandera roja que alguien levanta. Si el corredor no mira la bandera, sigue corriendo.

---

## CancellationTokenSource y CancellationToken

| Tipo | Rol | Quién lo tiene |
|---|---|---|
| `CancellationTokenSource` (CTS) | **Emisor**: puede cancelar | Quien controla la operación |
| `CancellationToken` (struct) | **Receptor**: solo puede observar | Todo el código que trabaja |

```csharp
using var cts = new CancellationTokenSource();
Task work = ProcessAsync(cts.Token);

// … más tarde, p. ej. el usuario pulsa "Cancelar"
cts.Cancel();          // en .NET 8+: await cts.CancelAsync(); (ejecuta los callbacks registrados de forma asíncrona)

try { await work; }
catch (OperationCanceledException) { Console.WriteLine("Cancelado"); }
```

- `CancellationToken.None` (o `default`) = un token que nunca se cancela.
- **Dispón el CTS** (`using`): si tiene un temporizador (`CancelAfter`) o registros, libera recursos.
- Separar emisor y receptor es intencionado: quien recibe un token **no puede** cancelar a los demás.

---

## Propagar el token

Regla: **todo método async de I/O acepta un `CancellationToken` como último parámetro y lo pasa hacia abajo**.

```csharp
public async Task<Invoice> CreateInvoiceAsync(int orderId, CancellationToken ct = default)
{
    var order = await _db.Orders.FirstAsync(o => o.Id == orderId, ct);   // ✅ lo pasa a EF Core
    var rate  = await _http.GetFromJsonAsync<Rate>("rates/eur", ct);     // ✅ lo pasa a HttpClient
    var pdf   = await _pdf.RenderAsync(order, rate!, ct);                // ✅ lo pasa a tu propio servicio
    return new Invoice(order, pdf);
}
```

Un token que no se pasa es una cancelación que **no ocurre**. Los analizadores ayudan: **CA2016** ("Forward the CancellationToken parameter to methods that take one") avisa cuando olvidas propagarlo.

---

## Observar la cancelación en tu código

Para bucles de CPU o trabajo propio que no llama a APIs con token:

```csharp
public async Task ProcessFilesAsync(IEnumerable<string> paths, CancellationToken ct)
{
    foreach (var path in paths)
    {
        ct.ThrowIfCancellationRequested();     // punto de comprobación barato
        await ProcessFileAsync(path, ct);
    }
}

// Variante sin excepción (cuando cancelar es un resultado normal, p. ej. devolver lo procesado hasta ahora)
if (ct.IsCancellationRequested) return partialResult;
```

### `Register`: reaccionar a la cancelación
Para APIs antiguas que no aceptan token pero tienen un método para abortar:

```csharp
using CancellationTokenRegistration reg = ct.Register(() => legacyClient.Abort());
await legacyClient.DoWorkAsync();
```

El `using` sobre el registro es importante: si no, el callback queda vivo tanto como el token (fuga de memoria si el token es de larga duración).

---

## Timeouts

Un timeout es simplemente una cancelación **programada en el tiempo**:

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
// o: cts.CancelAfter(TimeSpan.FromSeconds(5));
var data = await _http.GetStringAsync(url, cts.Token);
```

> [!note] `HttpClient.Timeout`
> `HttpClient` tiene su propio `Timeout` (100 s por defecto). Al expirar lanza `TaskCanceledException` cuyo `InnerException` es un `TimeoutException` (.NET 5+), lo que permite distinguirlo de una cancelación del llamador. En producción se suele configurar el timeout con **Polly / Microsoft.Extensions.Http.Resilience** (timeout por intento + timeout total), ver [[Retry con Backoff Exponencial]].

---

## Tokens enlazados

Necesitas cancelar si **el llamador cancela O pasa el timeout**:

```csharp
public async Task<string> GetWithTimeoutAsync(string url, CancellationToken ct)
{
    using var linked = CancellationTokenSource.CreateLinkedTokenSource(ct);
    linked.CancelAfter(TimeSpan.FromSeconds(3));

    try
    {
        return await _http.GetStringAsync(url, linked.Token);
    }
    catch (OperationCanceledException) when (!ct.IsCancellationRequested)
    {
        // El llamador NO canceló → fue nuestro timeout
        throw new TimeoutException($"{url} no respondió en 3 s");
    }
}
```

El filtro `when` es la forma idiomática de **distinguir timeout de cancelación del llamador**. Dispón siempre el CTS enlazado: se registra en el token padre y, si no se libera, crea una fuga mientras el padre viva.

---

## WaitAsync (.NET 6)

`WaitAsync` añade un timeout o token a **cualquier tarea**, incluso si la operación no acepta token:

```csharp
Task<Result> op = legacy.ComputeAsync();                       // no acepta CancellationToken
Result r = await op.WaitAsync(TimeSpan.FromSeconds(10), ct);   // TimeoutException o OperationCanceledException
```

> [!warning] `WaitAsync` deja de **esperar**, no **cancela**
> La operación subyacente sigue ejecutándose en segundo plano. Úsalo cuando no hay alternativa; si la API acepta token, pásaselo. Lo mismo aplica al antiguo patrón `Task.WhenAny(op, Task.Delay(timeout))`.

---

## Excepciones de cancelación

- `OperationCanceledException` (OCE): la excepción base de cancelación.
- `TaskCanceledException`: deriva de OCE; la lanzan `Task.Delay`, `HttpClient` y otras.
- **Captura siempre `OperationCanceledException`** (cubre ambas).
- `ThrowIfCancellationRequested()` lanza OCE con el token asociado (`ex.CancellationToken`).
- Si un método async lanza OCE con el mismo token con el que fue cancelado, el `Task` termina en estado **`Canceled`** (no `Faulted`).

```csharp
try
{
    await DoWorkAsync(ct);
}
catch (OperationCanceledException) when (ct.IsCancellationRequested)
{
    _logger.LogInformation("Operación cancelada por el llamador"); // no es un error: no lo registres como Error
}
```

> [!tip] No conviertas la cancelación en un error
> Una petición HTTP abortada por el cliente o un apagado del servicio no es un fallo de tu sistema. Regístralo como `Information`/`Debug`, no como `Error`, o tus alertas se llenarán de ruido.

---

## Cancelación en ASP.NET Core y en el host

### Peticiones HTTP
ASP.NET Core enlaza automáticamente un parámetro `CancellationToken` a `HttpContext.RequestAborted` (se cancela si el cliente cierra la conexión):

```csharp
// Minimal API
app.MapGet("/reports/{id}", async (int id, IReportService svc, CancellationToken ct) =>
    Results.Ok(await svc.BuildAsync(id, ct)));

// Controlador
[HttpGet("{id}")]
public async Task<IActionResult> Get(int id, CancellationToken ct) =>
    Ok(await _svc.BuildAsync(id, ct));
```

### Servicios en segundo plano
`BackgroundService.ExecuteAsync(CancellationToken stoppingToken)` recibe un token que se cancela cuando el host se detiene (p. ej. `SIGTERM` en Kubernetes). Detalles en [[11 - Async en ASP.NET Core y servicios en segundo plano]].

---

## TimeProvider (.NET 8)

`TimeProvider` es una abstracción del tiempo que permite **controlar el reloj en tests**. Las APIs de tiempo aceptan uno:

```csharp
public sealed class TokenRefresher(TimeProvider time)
{
    public async Task WaitForExpiryAsync(TimeSpan ttl, CancellationToken ct)
    {
        await Task.Delay(ttl, time, ct);                          // .NET 8+
        using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5), time); // CTS con reloj inyectado
        // ...
    }
}
// Producción: services.AddSingleton(TimeProvider.System);
// Tests: FakeTimeProvider (paquete Microsoft.Extensions.TimeProvider.Testing) → ver nivel 12
```

---

## Cuándo NO cancelar

La cancelación a mitad de una operación con efectos secundarios puede dejar el sistema **inconsistente**. Define un **punto de no retorno**:

```csharp
public async Task PlaceOrderAsync(Order order, CancellationToken ct)
{
    await ValidateAsync(order, ct);               // cancelable: aún no hay efectos
    await _payments.ChargeAsync(order, ct);       // cobrado…

    // A partir de aquí NO cancelar: cobrar sin guardar el pedido es peor que tardar un poco más.
    await _db.SaveOrderAsync(order, CancellationToken.None);
    await _outbox.EnqueueAsync(new OrderPlaced(order.Id), CancellationToken.None);
}
```

Mejor aún: diseña la operación para que sea **idempotente** y reintentable ([[Idempotencia]], [[Transactional Outbox]]).

---

## Puntos clave

- La cancelación es **cooperativa**: solicitar ≠ detener.
- CTS emite, `CancellationToken` observa. **Dispón** los CTS.
- **Propaga** el token a todas las APIs que lo acepten (CA2016).
- Timeout = CTS con `CancelAfter`; combina con el token del llamador vía `CreateLinkedTokenSource`.
- `WaitAsync` deja de esperar pero **no detiene** la operación.
- Captura `OperationCanceledException`; usa `when (ct.IsCancellationRequested)` para distinguir.
- No canceles tras el punto de no retorno.

## Errores comunes

- Aceptar `CancellationToken` y no pasarlo a nada.
- Capturar `Exception` y registrar la cancelación como error.
- No disponer `CreateLinkedTokenSource` (fuga de registros).
- Creer que `cts.Cancel()` detiene inmediatamente una tarea.
- Usar `Task.Delay(timeout)` + `WhenAny` sin cancelar la operación perdedora.

## 🎯 Para entrevistas y exámenes

- *"¿Cómo cancelas una tarea en .NET?"* → Cooperativamente: `CancellationTokenSource.Cancel()` y el código observa el token (lo pasa a APIs o llama a `ThrowIfCancellationRequested`).
- *"¿Cómo implementas un timeout que también respete la cancelación del llamador?"* → `CreateLinkedTokenSource(ct)` + `CancelAfter(timeout)`; filtro `when` para distinguir.
- *"¿Diferencia entre `OperationCanceledException` y `TaskCanceledException`?"* → La segunda deriva de la primera; captura la base.

## 🧠 Practica

1. Escribe un bucle que procese 100 elementos con `Task.Delay(100, ct)` y cancélalo a los 2 s. Comprueba el estado final del `Task` (`IsCanceled`).
2. Implementa `GetWithTimeoutAsync` y verifica que lanza `TimeoutException` por timeout y `OperationCanceledException` si cancela el llamador.
3. En una Minimal API, añade un endpoint lento con `Task.Delay(10_000, ct)` y cancela la petición desde el navegador o con `curl --max-time 1`. Registra lo que ocurre.

## Referencias

- Microsoft Learn, *Cancellation in managed threads*: https://learn.microsoft.com/dotnet/standard/threading/cancellation-in-managed-threads
- Microsoft Learn, *Task.WaitAsync*: https://learn.microsoft.com/dotnet/api/system.threading.tasks.task.waitasync
- Microsoft Learn, *TimeProvider*: https://learn.microsoft.com/dotnet/standard/datetime/timeprovider-overview
- Microsoft Learn, *CA2016*: https://learn.microsoft.com/dotnet/fundamentals/code-analysis/quality-rules/ca2016

---
⬅️ [[04 - Composición de tareas]] · ➡️ [[06 - Manejo de excepciones]] · ⬆️ [[00 - Async en .NET - Índice]]
