---
tags: [dotnet, csharp, async, nivel/11, aspnetcore, hosting]
up: "[[00 - Async en .NET - Índice]]"
---

# 11 - Async en ASP.NET Core y servicios en segundo plano

> Nivel 🔴 11 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: niveles 5, 6 y 8.

## Contenido
- [[#El pipeline de ASP.NET Core es async]]
- [[#Reglas para endpoints]]
- [[#HttpContext y concurrencia]]
- [[#EF Core async]]
- [[#HttpClient async]]
- [[#BackgroundService]]
- [[#Servicios scoped en segundo plano]]
- [[#Cola de trabajo en segundo plano]]
- [[#Apagado ordenado]]
- [[#🧠 Practica]]

---

## El pipeline de ASP.NET Core es async

Todo en ASP.NET Core es asíncrono de punta a punta: Kestrel lee el socket de forma async, cada middleware es un `Task InvokeAsync(HttpContext ctx)`, y tu endpoint devuelve un `Task`. Mientras tu código espera a la BD, el hilo vuelve al pool y atiende otras peticiones.

```csharp
app.Use(async (ctx, next) =>
{
    var sw = Stopwatch.StartNew();
    await next(ctx);                         // el resto del pipeline (incluido tu endpoint)
    ctx.RequestServices.GetRequiredService<ILogger<Program>>()
       .LogInformation("{Path} tardó {Ms} ms", ctx.Request.Path, sw.ElapsedMilliseconds);
});
```

> [!note] I/O síncrona deshabilitada
> Desde ASP.NET Core 3.0, `AllowSynchronousIO` es `false` en Kestrel: leer el cuerpo de la petición o escribir la respuesta de forma síncrona (`Request.Body.Read`, `Response.Body.Write`) lanza `InvalidOperationException`. Es una protección contra *thread pool starvation*. Si una librería antigua lo necesita, mejor buscar alternativa que reactivarlo.

---

## Reglas para endpoints

```csharp
// ✅ Minimal API
app.MapPost("/orders", async (CreateOrder cmd, IOrderService svc, CancellationToken ct) =>
{
    var id = await svc.CreateAsync(cmd, ct);
    return Results.Created($"/orders/{id}", new { id });
});

// ✅ Controlador
[HttpGet("{id:int}")]
public async Task<ActionResult<OrderDto>> Get(int id, CancellationToken ct)
{
    var order = await _svc.GetAsync(id, ct);
    return order is null ? NotFound() : Ok(order);
}
```

Checklist:
1. Devuelve `Task`/`Task<T>`/`ValueTask<T>`; **nunca** `async void`.
2. Acepta `CancellationToken` (se enlaza a `HttpContext.RequestAborted`) y propágalo.
3. **Nada de `.Result`/`.Wait()`** ni I/O síncrona.
4. **Nada de `Task.Run`** para "hacerlo async".
5. Si necesitas hacer algo después de responder, **no** lo lances suelto: encólalo ([[#Cola de trabajo en segundo plano]]).

---

## HttpContext y concurrencia

- `HttpContext` **no es seguro para hilos**: no lo uses desde varias tareas concurrentes de la misma petición (p. ej. dentro de un `Task.WhenAll` que escribe cabeceras).
- **No lo uses después de que la petición termine** (en un fire-and-forget): el objeto se recicla y verás datos de **otra** petición o excepciones.
- `IHttpContextAccessor` usa `AsyncLocal` por debajo; funciona a través de `await`, pero las mismas reglas aplican. Extrae los datos que necesites (usuario, *correlation id*) **antes** de pasar trabajo a segundo plano.

---

## EF Core async

```csharp
var orders = await db.Orders
    .Where(o => o.CustomerId == customerId)
    .AsNoTracking()
    .ToListAsync(ct);

db.Orders.Add(newOrder);
await db.SaveChangesAsync(ct);
```

- Usa siempre los métodos `*Async` (`ToListAsync`, `FirstOrDefaultAsync`, `SaveChangesAsync`, `ExecuteUpdateAsync`).
- **Un `DbContext` = una operación a la vez.** El `DbContext` scoped de la petición no se puede usar en paralelo; para paralelizar, `IDbContextFactory<T>` ([[04 - Composición de tareas#Recursos que no admiten concurrencia|ver nivel 4]]).
- `AsAsyncEnumerable()` para streaming ([[08 - Streams asíncronos y Channels#IAsyncEnumerable en EF Core y ASP.NET Core|nivel 8]]).
- La propia documentación de EF Core indica que `Add`/`AddRange` síncronos son los correctos en casi todos los casos; `AddAsync` solo tiene sentido con generadores de valores que acceden a la BD (como HiLo de SQL Server).

---

## HttpClient async

```csharp
builder.Services.AddHttpClient<CatalogClient>(c => c.BaseAddress = new Uri("https://catalog"))
       .AddStandardResilienceHandler();    // Microsoft.Extensions.Http.Resilience: retry, timeout, circuit breaker

public sealed class CatalogClient(HttpClient http)
{
    public Task<Product?> GetAsync(string sku, CancellationToken ct) =>
        http.GetFromJsonAsync<Product>($"products/{sku}", ct);
}
```

- `IHttpClientFactory` gestiona el pool de conexiones y la rotación de DNS. No crees `new HttpClient()` por petición.
- La resiliencia (reintentos, timeouts, *circuit breaker*) es async por naturaleza: ver [[Retry con Backoff Exponencial]] y [[Circuit Breaker]].

---

## BackgroundService

`BackgroundService` es la clase base para trabajos de larga duración dentro del host (API o Worker Service).

```csharp
public sealed class OutboxPublisher(IServiceScopeFactory scopes, ILogger<OutboxPublisher> log) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));
        try
        {
            while (await timer.WaitForNextTickAsync(stoppingToken))
            {
                try
                {
                    await PublishPendingAsync(stoppingToken);
                }
                catch (Exception ex) when (ex is not OperationCanceledException)
                {
                    // un fallo puntual NO debe matar el servicio (ni el host)
                    log.LogError(ex, "Fallo publicando el outbox; se reintenta en el siguiente tick");
                }
            }
        }
        catch (OperationCanceledException) when (stoppingToken.IsCancellationRequested)
        {
            // apagado normal
        }
    }

    private async Task PublishPendingAsync(CancellationToken ct)
    {
        await using var scope = scopes.CreateAsyncScope();
        var outbox = scope.ServiceProvider.GetRequiredService<IOutboxRepository>();
        // ...
    }
}

builder.Services.AddHostedService<OutboxPublisher>();
```

Reglas de `BackgroundService`:

| Regla | Motivo |
|---|---|
| Captura excepciones **dentro** del bucle | Desde .NET 6, una excepción no controlada en `ExecuteAsync` **detiene el host** por defecto (`BackgroundServiceExceptionBehavior.StopHost`). En .NET 5 y anteriores, el servicio moría en silencio |
| Respeta `stoppingToken` | Es la señal de apagado (SIGTERM, Ctrl+C) |
| No bloquees al principio de `ExecuteAsync` | Históricamente, la parte síncrona inicial de `ExecuteAsync` (antes del primer `await` incompleto) se ejecutaba durante el **arranque** del host y podía retrasarlo o bloquearlo. Las versiones recientes de .NET han cambiado este comportamiento para ejecutar `ExecuteAsync` en segundo plano; si apuntas a varias versiones, empezar con `await Task.Yield()` es una protección barata |
| Crea un *scope* por unidad de trabajo | El servicio es singleton; `DbContext` y otros scoped no pueden inyectarse directamente |

---

## Servicios scoped en segundo plano

Un `BackgroundService` es **singleton**. Inyectar un `DbContext` (scoped) directamente da error de validación de DI o, peor, un único contexto compartido para siempre. Crea un scope por iteración o por mensaje:

```csharp
await using var scope = _scopeFactory.CreateAsyncScope();   // CreateAsyncScope (.NET 6+) respeta IAsyncDisposable
var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
await db.SaveChangesAsync(ct);
```

---

## Cola de trabajo en segundo plano

Patrón para "responder ya y hacer el trabajo después" **dentro del proceso**, combinando `Channel<T>` + `BackgroundService`:

```csharp
public interface IBackgroundTaskQueue
{
    ValueTask EnqueueAsync(Func<IServiceProvider, CancellationToken, ValueTask> work, CancellationToken ct = default);
}

public sealed class BackgroundTaskQueue : IBackgroundTaskQueue
{
    private readonly Channel<Func<IServiceProvider, CancellationToken, ValueTask>> _channel =
        Channel.CreateBounded<Func<IServiceProvider, CancellationToken, ValueTask>>(
            new BoundedChannelOptions(1_000) { FullMode = BoundedChannelFullMode.Wait });

    public ValueTask EnqueueAsync(Func<IServiceProvider, CancellationToken, ValueTask> work, CancellationToken ct = default)
        => _channel.Writer.WriteAsync(work, ct);

    public IAsyncEnumerable<Func<IServiceProvider, CancellationToken, ValueTask>> ReadAllAsync(CancellationToken ct)
        => _channel.Reader.ReadAllAsync(ct);
}

public sealed class QueuedWorker(BackgroundTaskQueue queue, IServiceScopeFactory scopes, ILogger<QueuedWorker> log)
    : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await Parallel.ForEachAsync(queue.ReadAllAsync(stoppingToken),
            new ParallelOptions { MaxDegreeOfParallelism = 4, CancellationToken = stoppingToken },
            async (work, ct) =>
            {
                await using var scope = scopes.CreateAsyncScope();     // scope propio: no depende de la petición
                try { await work(scope.ServiceProvider, ct); }
                catch (Exception ex) when (ex is not OperationCanceledException)
                {
                    log.LogError(ex, "Fallo en trabajo en segundo plano");
                }
            });
    }
}

// Registro
builder.Services.AddSingleton<BackgroundTaskQueue>();
builder.Services.AddSingleton<IBackgroundTaskQueue>(sp => sp.GetRequiredService<BackgroundTaskQueue>());
builder.Services.AddHostedService<QueuedWorker>();

// Uso en un endpoint: copia los DATOS, no el HttpContext ni servicios scoped
app.MapPost("/signup", async (SignupRequest req, IBackgroundTaskQueue queue, CancellationToken ct) =>
{
    var email = req.Email;
    await queue.EnqueueAsync(async (sp, token) =>
        await sp.GetRequiredService<IEmailSender>().SendWelcomeAsync(email, token), ct);
    return Results.Accepted();
});
```

> [!danger] Límites de este patrón
> Es memoria del proceso: un reinicio, despliegue o caída **pierde** lo encolado. Para trabajo que no puede perderse usa una cola persistente ([[00 - Indice y ruta de estudio|Azure Service Bus]], [[Transactional Outbox]]) o una librería con almacenamiento (Hangfire, Quartz con persistencia).

---

## Apagado ordenado

Secuencia cuando el host recibe `SIGTERM` (por ejemplo, Kubernetes al borrar un pod):

1. `IHostApplicationLifetime.ApplicationStopping` se dispara; Kestrel deja de aceptar conexiones nuevas y espera las peticiones en curso.
2. Se cancela el `stoppingToken` de cada `BackgroundService`, y se llama a `StopAsync` de los hosted services.
3. El host espera como máximo `HostOptions.ShutdownTimeout` (30 s por defecto desde .NET 6; antes 5 s). Lo que no termine se abandona.

```csharp
builder.Services.Configure<HostOptions>(o => o.ShutdownTimeout = TimeSpan.FromSeconds(45));
```

En Kubernetes, `terminationGracePeriodSeconds` (30 s por defecto) debe ser **mayor** que el `ShutdownTimeout` más el retardo de desregistro del Service, o el pod recibirá `SIGKILL` a mitad del apagado. Ver [[13 - Kubernetes para .NET]].

> [!tip] .NET 8: `IHostedLifecycleService`
> Añade `StartingAsync`, `StartedAsync`, `StoppingAsync` y `StoppedAsync` para engancharse a fases concretas. Y `HostOptions.ServicesStartConcurrently`/`ServicesStopConcurrently` permiten arrancar/parar los hosted services en paralelo.

---

## Puntos clave

- ASP.NET Core es async de extremo a extremo y prohíbe I/O síncrona por defecto.
- Endpoints: `Task`, `CancellationToken`, sin bloqueos, sin `Task.Run`, sin fire-and-forget suelto.
- `HttpContext` no es thread-safe ni válido tras la petición.
- `DbContext`: una operación a la vez; usa `*Async`.
- `BackgroundService`: captura excepciones, respeta `stoppingToken`, crea scopes.
- Cola `Channel` + `BackgroundService` para trabajo diferido no crítico; cola persistente para lo crítico.
- Ajusta `ShutdownTimeout` y `terminationGracePeriodSeconds` juntos.

## Errores comunes

- Inyectar `DbContext` en un `BackgroundService`.
- Capturar `HttpContext` en un trabajo en segundo plano.
- Bucle de `BackgroundService` sin `try/catch` → el host se detiene al primer error.
- `Task.Run` en un controlador para "enviar el email después".
- Creer que lo encolado en memoria sobrevive a un despliegue.

## 🎯 Para entrevistas y exámenes

- *"¿Cómo harías trabajo en segundo plano después de responder una petición?"* → Cola con `Channel` consumida por un `BackgroundService` con scope propio; si debe ser fiable, cola persistente/outbox.
- *"¿Qué pasa si `ExecuteAsync` lanza una excepción?"* → Desde .NET 6, se detiene el host (configurable con `BackgroundServiceExceptionBehavior`).
- *"¿Cómo usas un `DbContext` en un `BackgroundService`?"* → `IServiceScopeFactory.CreateAsyncScope()` por unidad de trabajo.

## 🧠 Practica

1. Implementa la cola de trabajo de esta nota y encola 100 trabajos con `Task.Delay` aleatorio. Detén la app con Ctrl+C a mitad: ¿cuántos se completan y cuántos se pierden?
2. Escribe un `BackgroundService` que lance una excepción a los 5 s. Observa que el host se detiene; luego cambia `BackgroundServiceExceptionBehavior` a `Ignore` y compara.
3. Haz un endpoint que use el `DbContext` de la petición dentro de un `Task.Run` sin esperar. Observa el `ObjectDisposedException`.

## Referencias

- Microsoft Learn, *Background tasks with hosted services in ASP.NET Core*: https://learn.microsoft.com/aspnet/core/fundamentals/host/hosted-services
- Microsoft Learn, *Worker Services in .NET*: https://learn.microsoft.com/dotnet/core/extensions/workers
- Microsoft Learn, *Asynchronous Programming (EF Core)*: https://learn.microsoft.com/ef/core/miscellaneous/async
- Microsoft Learn, *ASP.NET Core Best Practices* (secciones sobre async y HttpContext): https://learn.microsoft.com/aspnet/core/fundamentals/best-practices
- Microsoft Learn, *Build resilient HTTP apps*: https://learn.microsoft.com/dotnet/core/resilience/http-resilience

---
⬅️ [[10 - Rendimiento y diagnóstico]] · ➡️ [[12 - Testing de código asíncrono]] · ⬆️ [[00 - Async en .NET - Índice]]
