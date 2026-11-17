---
tags: [dotnet, csharp, async, nivel/12, testing]
up: "[[00 - Async en .NET - Índice]]"
---

# 12 - Testing de código asíncrono

> Nivel 🔴 12 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: niveles 5, 6 y 9.

## Contenido
- [[#Tests async básicos]]
- [[#Probar excepciones y cancelación]]
- [[#Dobles de prueba que devuelven Task]]
- [[#Controlar el orden con TaskCompletionSource]]
- [[#Controlar el tiempo con FakeTimeProvider]]
- [[#Probar BackgroundService]]
- [[#Probar concurrencia]]
- [[#🧠 Practica]]

---

## Tests async básicos

Los frameworks modernos (xUnit, NUnit, MSTest) soportan métodos de test `async Task`:

```csharp
public class PriceServiceTests
{
    [Fact]
    public async Task GetPriceAsync_returns_cached_value()
    {
        var sut = new PriceService(new InMemoryPriceCache(("SKU1", 10m)));

        decimal price = await sut.GetPriceAsync("SKU1", CancellationToken.None);

        Assert.Equal(10m, price);
    }
}
```

Reglas:
- El test devuelve **`async Task`**, nunca `async void` (el runner no esperaría y el test pasaría siempre).
- **Nunca `.Result` ni `.Wait()`** en tests (el analizador xUnit1031 lo avisa).
- En xUnit v3, usa `TestContext.Current.CancellationToken` para que el test se cancele si el runner aborta (el analizador xUnit1051 sugiere pasarlo).

---

## Probar excepciones y cancelación

```csharp
[Fact]
public async Task CreateAsync_throws_when_stock_is_insufficient()
{
    var ex = await Assert.ThrowsAsync<InsufficientStockException>(
        () => _sut.CreateAsync(new Order("SKU1", quantity: 999), CancellationToken.None));

    Assert.Equal("SKU1", ex.Sku);
}

[Fact]
public async Task ProcessAsync_honours_cancellation()
{
    using var cts = new CancellationTokenSource();
    cts.Cancel();

    // ThrowsAnyAsync acepta tipos derivados (TaskCanceledException deriva de OperationCanceledException)
    await Assert.ThrowsAnyAsync<OperationCanceledException>(() => _sut.ProcessAsync(cts.Token));
}
```

> [!warning] Olvidar el `await` en `ThrowsAsync`
> `Assert.ThrowsAsync(...)` sin `await` devuelve un `Task` que nadie observa: el test **pasa siempre**. Los analizadores de xUnit avisan de este error.

---

## Dobles de prueba que devuelven Task

```csharp
// NSubstitute
var repo = Substitute.For<IOrderRepository>();
repo.GetAsync(42, Arg.Any<CancellationToken>()).Returns(new Order(42));          // envuelve en Task automáticamente
repo.SaveAsync(Arg.Any<Order>(), Arg.Any<CancellationToken>())
    .Returns(Task.FromException(new DbUpdateException("conflict")));            // simular fallo

// Moq
mock.Setup(r => r.GetAsync(42, It.IsAny<CancellationToken>())).ReturnsAsync(new Order(42));
mock.Setup(r => r.SaveAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()))
    .ThrowsAsync(new DbUpdateException("conflict"));
```

Un *fake* escrito a mano suele ser más claro que un mock para servicios async con estado:

```csharp
public sealed class FakeClock : IClock { public DateTimeOffset UtcNow { get; set; } }
public sealed class InMemoryOrderRepository : IOrderRepository
{
    private readonly ConcurrentDictionary<int, Order> _orders = new();
    public Task<Order?> GetAsync(int id, CancellationToken ct) => Task.FromResult(_orders.GetValueOrDefault(id));
    public Task SaveAsync(Order o, CancellationToken ct) { _orders[o.Id] = o; return Task.CompletedTask; }
}
```

> [!note] Mocks "demasiado síncronos"
> `Task.FromResult` completa **síncronamente**, así que tu código nunca se suspende en el test. Bugs que solo aparecen cuando hay suspensión real (orden de ejecución, estado compartido modificado entre `await`s) no se detectan. Para esos casos, usa el patrón de `TaskCompletionSource` siguiente o un `await Task.Yield()` en el fake.

---

## Controlar el orden con TaskCompletionSource

Para probar "qué pasa **mientras** una operación está en curso", haz que el doble devuelva un `Task` que tú completas cuando quieras:

```csharp
[Fact]
public async Task Concurrent_callers_share_a_single_load()
{
    var loadStarted = 0;
    var tcs = new TaskCompletionSource<Product>(TaskCreationOptions.RunContinuationsAsynchronously);
    var loader = Substitute.For<IProductLoader>();
    loader.LoadAsync("p1").Returns(_ => { Interlocked.Increment(ref loadStarted); return tcs.Task; });

    var sut = new SingleFlightProductCache(loader);

    Task<Product> a = sut.GetProductAsync("p1");
    Task<Product> b = sut.GetProductAsync("p1");     // llega mientras la primera está "en vuelo"

    Assert.False(a.IsCompleted);
    tcs.SetResult(new Product("p1"));                 // ahora sí, termina la carga

    Assert.Same(await a, await b);
    Assert.Equal(1, loadStarted);                     // solo una carga real
}
```

---

## Controlar el tiempo con FakeTimeProvider

El código que depende del tiempo (`Task.Delay`, timeouts, `PeriodicTimer`, expiraciones) es difícil de probar: o el test tarda lo mismo que el retardo real, o es intermitente. Solución (.NET 8+): inyecta `TimeProvider` en el código y usa `FakeTimeProvider` (paquete `Microsoft.Extensions.TimeProvider.Testing`) en el test.

```csharp
// Código de producción
public sealed class RetryingSender(ISender inner, TimeProvider time)
{
    public async Task SendAsync(Message m, CancellationToken ct)
    {
        for (int attempt = 1; ; attempt++)
        {
            try { await inner.SendAsync(m, ct); return; }
            catch (TransientException) when (attempt < 3)
            {
                await Task.Delay(TimeSpan.FromSeconds(10 * attempt), time, ct);   // retardo con reloj inyectado
            }
        }
    }
}

// Test: se ejecuta en milisegundos, sin esperar 30 s reales
[Fact]
public async Task Retries_after_backoff()
{
    var time = new FakeTimeProvider();
    var inner = Substitute.For<ISender>();
    inner.SendAsync(default!, default).ReturnsForAnyArgs(
        Task.FromException(new TransientException()),
        Task.CompletedTask);

    var sut = new RetryingSender(inner, time);
    Task send = sut.SendAsync(new Message(), CancellationToken.None);

    Assert.False(send.IsCompleted);            // está esperando el backoff de 10 s
    time.Advance(TimeSpan.FromSeconds(10));    // "pasan" 10 s instantáneamente
    await send;

    await inner.Received(2).SendAsync(Arg.Any<Message>(), Arg.Any<CancellationToken>());
}
```

Claves:
- `Advance` dispara los temporizadores cuyo vencimiento ya llegó.
- El retardo tiene que estar **registrado antes** de llamar a `Advance` (aquí lo está porque el método corre síncronamente hasta el `await Task.Delay`). Si entre medias hay suspensiones reales, puede que necesites esperar a que el código llegue al punto de espera antes de avanzar.
- `PeriodicTimer`, `CancellationTokenSource(TimeSpan, TimeProvider)` y `WaitAsync(TimeSpan, TimeProvider)` también aceptan el `TimeProvider`.

---

## Probar BackgroundService

```csharp
[Fact]
public async Task Worker_processes_queue_and_stops_gracefully()
{
    var queue = new BackgroundTaskQueue();
    var processed = new TaskCompletionSource(TaskCreationOptions.RunContinuationsAsynchronously);
    await using var services = new ServiceCollection().AddLogging().BuildServiceProvider();
    var worker = new QueuedWorker(queue, services.GetRequiredService<IServiceScopeFactory>(),
                                  NullLogger<QueuedWorker>.Instance);

    await worker.StartAsync(CancellationToken.None);
    await queue.EnqueueAsync((_, _) => { processed.SetResult(); return ValueTask.CompletedTask; });

    await processed.Task.WaitAsync(TimeSpan.FromSeconds(5));   // timeout: el test nunca se cuelga
    await worker.StopAsync(CancellationToken.None);
}
```

Para tests de integración completos, `WebApplicationFactory<TProgram>` (ASP.NET Core) o `Host.CreateApplicationBuilder` en el test, y [[Testing con Redis y Testcontainers|Testcontainers]] para dependencias reales.

> [!tip] Pon siempre un timeout a lo que esperas en un test
> `await algo.WaitAsync(TimeSpan.FromSeconds(5))` convierte un test colgado (que bloquea el pipeline de CI) en un fallo claro.

---

## Probar concurrencia

Los errores de concurrencia son **no deterministas**: un test que pasa una vez no demuestra nada. Estrategias:

1. **Forzar el entrelazado** con `TaskCompletionSource` (como arriba): controlas exactamente qué se ejecuta antes.
2. **Repetir con mucha concurrencia** para aumentar la probabilidad de la carrera:

```csharp
[Fact]
public async Task Counter_is_thread_safe()
{
    var counter = new AsyncCounter();
    await Task.WhenAll(Enumerable.Range(0, 1_000).Select(_ => Task.Run(() => counter.IncrementAsync())));
    Assert.Equal(1_000, counter.Value);
}
```

3. **Revisar el diseño**: si un componente es difícil de probar en concurrencia, quizá comparte demasiado estado. Inmutabilidad y propietarios únicos (un consumidor de `Channel`) simplifican los tests.

---

## Puntos clave

- Tests `async Task`, nunca `async void`, nunca `.Result`.
- `await Assert.ThrowsAsync` / `ThrowsAnyAsync<OperationCanceledException>`.
- `Task.FromResult` en mocks no ejercita la suspensión; usa `TaskCompletionSource` para controlar el orden.
- `TimeProvider` + `FakeTimeProvider` para probar retardos y timeouts sin esperar.
- Pon timeouts (`WaitAsync`) a todo lo que esperes en un test.

## Errores comunes

- `Thread.Sleep`/`Task.Delay` reales en tests ("esperar a que termine").
- Olvidar `await` en `Assert.ThrowsAsync`.
- Tests que dependen de `DateTime.UtcNow` en lugar de un reloj inyectable.
- Concluir que el código es thread-safe porque un test concurrente pasó una vez.

## 🎯 Para entrevistas y exámenes

- *"¿Cómo pruebas código con `Task.Delay(30 s)`?"* → Inyectando `TimeProvider` y usando `FakeTimeProvider.Advance` en el test.
- *"¿Por qué un test `async void` es peligroso?"* → El runner no lo espera; las aserciones se ejecutan después de que el test "pasó".
- *"¿Cómo pruebas que dos llamadas concurrentes comparten una sola carga?"* → Doble que devuelve el `Task` de un `TaskCompletionSource` que completas tras lanzar ambas llamadas.

## 🧠 Practica

1. Prueba `GetWithTimeoutAsync` del [[05 - Cancelación y timeouts#Tokens enlazados|nivel 5]] con `FakeTimeProvider` (necesitarás que acepte un `TimeProvider`).
2. Escribe un test que demuestre la carrera de un contador sin sincronizar y luego pásalo a verde con `Interlocked`.
3. Prueba el `OutboxPublisher` del [[11 - Async en ASP.NET Core y servicios en segundo plano#BackgroundService|nivel 11]] con `PeriodicTimer` y `FakeTimeProvider`.

## Referencias

- Microsoft Learn, *Test with TimeProvider / FakeTimeProvider*: https://learn.microsoft.com/dotnet/standard/datetime/timeprovider-overview
- xUnit, *Analyzers* (xUnit1031, xUnit1051): https://xunit.net/xunit.analyzers/rules/
- Microsoft Learn, *Integration tests in ASP.NET Core*: https://learn.microsoft.com/aspnet/core/test/integration-tests

---
⬅️ [[11 - Async en ASP.NET Core y servicios en segundo plano]] · ➡️ [[13 - Anti-patrones, checklist y entrevistas]] · ⬆️ [[00 - Async en .NET - Índice]]
