---
tags: [dotnet, csharp, async, nivel/2, fundamentos]
up: "[[00 - Async en .NET - Índice]]"
---

# 02 - async y await: lo básico

> Nivel 🟢 2 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[01 - Fundamentos - concurrencia, paralelismo y asincronía]].

## Contenido
- [[#Task y Task<T>: la promesa de un resultado]]
- [[#La palabra clave async]]
- [[#La palabra clave await]]
- [[#Flujo de ejecución paso a paso]]
- [[#Tipos de retorno]]
- [[#async void: el peligroso]]
- [[#Task.Run: cuándo sí y cuándo no]]
- [[#Convenciones y reglas básicas]]
- [[#async Main, await using y lambdas async]]
- [[#🧠 Practica]]

---

## Task y Task<T>: la promesa de un resultado

Un `Task` representa **una operación que puede no haber terminado todavía**. Es como el ticket del guardarropa: no es el abrigo, pero te permite recogerlo cuando esté.

- `Task` → operación sin resultado (equivale a `void`).
- `Task<T>` → operación que producirá un `T`.

Un `Task` tiene un **estado** (`TaskStatus`): `Created`, `WaitingForActivation`, `WaitingToRun`, `Running`, `WaitingForChildrenToComplete`, y tres estados finales:

| Estado final | Significado | Propiedad |
|---|---|---|
| `RanToCompletion` | Terminó bien | `IsCompletedSuccessfully == true` |
| `Faulted` | Lanzó una excepción | `IsFaulted == true`, `Exception` no nulo |
| `Canceled` | Se canceló | `IsCanceled == true` |

`IsCompleted` es `true` en **cualquiera** de los tres estados finales.

> [!note] Dos tipos de Task
> - **Promise tasks** (la gran mayoría en código async): representan una operación que **no ejecuta código propio**, como una lectura de red. No hay hilo detrás.
> - **Delegate tasks**: ejecutan un delegado en un hilo (`Task.Run(() => ...)`).
> Saber esto evita la idea errónea de que "un Task es un hilo".

---

## La palabra clave async

`async` en la firma de un método hace **dos cosas** y nada más:

1. Permite usar `await` dentro del método.
2. Le pide al compilador que transforme el método en una **máquina de estados** ([[03 - Cómo funciona por dentro|nivel 3]]) que construye y devuelve el `Task` automáticamente: lo que hagas `return` se convierte en el resultado del `Task<T>`, y lo que lances (`throw`) se guarda **dentro** del `Task`.

```csharp
public async Task<int> GetLengthAsync(string url, HttpClient client)
{
    string content = await client.GetStringAsync(url);
    return content.Length; // devuelves int; el compilador lo envuelve en Task<int>
}
```

> [!warning] `async` no hace nada asíncrono por sí solo
> Un método `async` sin ningún `await` se ejecuta **completamente de forma síncrona** y el compilador emite el aviso CS1998. No crea hilos ni "lo manda al fondo".

---

## La palabra clave await

`await tarea` significa: **"si la tarea ya terminó, sigue; si no, suspende este método, devuelve el control a quien me llamó y reanúdame cuando termine"**.

Al reanudar, `await`:
- Devuelve el **resultado** si era `Task<T>`.
- **Relanza la excepción** si la tarea falló (la primera, desempaquetada; no un `AggregateException`).
- Lanza `OperationCanceledException` (concretamente `TaskCanceledException`) si se canceló.

```csharp
try
{
    var data = await client.GetStringAsync(url); // si falla, la HttpRequestException se lanza AQUÍ
}
catch (HttpRequestException ex)
{
    logger.LogWarning(ex, "Falló la descarga");
}
```

Esta es la gran ventaja frente a callbacks: el código asíncrono **se lee como síncrono**, con `try/catch`, `using`, bucles y `return` normales.

---

## Flujo de ejecución paso a paso

```csharp
static async Task Main()
{
    Console.WriteLine("1. Main empieza");
    Task<string> t = DownloadAsync();          // llama al método
    Console.WriteLine("3. Main sigue mientras se descarga");
    string result = await t;                   // ahora sí esperamos
    Console.WriteLine($"5. Resultado: {result.Length}");
}

static async Task<string> DownloadAsync()
{
    Console.WriteLine("2. DownloadAsync empieza (síncrono hasta el primer await)");
    using var client = new HttpClient();
    string s = await client.GetStringAsync("https://example.com"); // aquí se suspende
    Console.WriteLine("4. Descarga terminada, DownloadAsync se reanuda");
    return s;
}
```

Salida: `1 → 2 → 3 → 4 → 5`.

Puntos importantes:
- **Un método async se ejecuta síncronamente hasta el primer `await` de una tarea incompleta.** Por eso "2" se imprime antes que "3".
- Si la tarea ya estaba completa (por ejemplo, un dato en caché), `await` **no suspende**: sigue de inmediato, sin cambio de hilo.
- Separar la llamada (`DownloadAsync()`) del `await` permite **iniciar varias operaciones a la vez** y esperarlas después ([[04 - Composición de tareas]]).

---

## Tipos de retorno

| Tipo | Cuándo usarlo |
|---|---|
| `Task` | Método async sin resultado. **El caso por defecto.** |
| `Task<T>` | Método async con resultado. |
| `ValueTask` / `ValueTask<T>` | Hot paths que suelen completarse síncronamente, tras medir. Ver [[Task vs ValueTask]]. |
| `IAsyncEnumerable<T>` | Secuencia de valores que llegan de forma asíncrona. Ver [[08 - Streams asíncronos y Channels]]. |
| `void` | **Solo** manejadores de eventos (`button.Click += async (s, e) => ...`). |
| Cualquier tipo con *AsyncMethodBuilder* | Avanzado (librerías). |

---

## async void: el peligroso

```csharp
// ❌ NUNCA en código normal
public async void SaveAsync() { await repo.SaveAsync(); }
```

Problemas:
1. **No se puede esperar**: quien lo llama no sabe cuándo termina.
2. **Las excepciones no se pueden capturar** con `try/catch` desde fuera: se relanzan en el `SynchronizationContext` (en UI) o directamente en el thread pool, lo que en .NET Core **termina el proceso**.
3. **No se puede probar** fácilmente.

Única excepción legítima: **manejadores de eventos**, cuya firma exige `void`. En ese caso, envuelve el cuerpo en `try/catch`:

```csharp
private async void OnSaveClicked(object? sender, EventArgs e)
{
    try
    {
        await _viewModel.SaveAsync();
    }
    catch (Exception ex)
    {
        ShowError(ex); // si no capturas aquí, la app puede cerrarse
    }
}
```

> [!danger] La trampa de las lambdas
> `Action` devuelve `void`, así que `list.ForEach(async x => await ProcessAsync(x))` crea lambdas **async void**: no espera nada y las excepciones tumban el proceso. Usa `foreach` con `await` o `Task.WhenAll`.

---

## Task.Run: cuándo sí y cuándo no

`Task.Run(func)` encola `func` en el thread pool y devuelve un `Task` que representa su ejecución.

| Situación | ¿`Task.Run`? | Motivo |
|---|---|---|
| App de escritorio/MAUI, cálculo pesado al pulsar un botón | ✅ Sí | Saca el trabajo del hilo de UI |
| Consola/batch: quieres paralelizar trabajo de CPU | ✅ Sí (o `Parallel`) | Usa más núcleos |
| ASP.NET Core: envolver código síncrono "para hacerlo async" | ❌ No | Cambias un hilo del pool por otro y añades overhead |
| ASP.NET Core: envolver una llamada que ya es async (`Task.Run(() => db.SaveAsync())`) | ❌ No | Inútil, solo overhead |
| Implementar una librería: exponer `FooAsync` que internamente hace `Task.Run(Foo)` | ❌ No ("fake async") | Engaña al consumidor; que él decida si usar `Task.Run` |
| Necesitas una API async pero solo existe la versión síncrona (p. ej. una librería antigua), en una app de escritorio | ⚠️ Aceptable | Mejor que bloquear la UI; documenta que no es async real |

```csharp
// ✅ UI: mantener la ventana responsiva
private async void OnCompressClicked(object? s, EventArgs e)
{
    progress.Visible = true;
    byte[] zipped = await Task.Run(() => Compress(_bigBuffer)); // CPU-bound fuera del hilo de UI
    progress.Visible = false;                                    // de vuelta en el hilo de UI
}
```

> [!note] `Task.Run` vs `Task.Factory.StartNew`
> `Task.Run` es la forma recomendada. `Task.Factory.StartNew` es de bajo nivel: con lambdas async devuelve `Task<Task>` (hay que llamar a `.Unwrap()`) y usa `TaskScheduler.Current` por defecto, lo que puede sorprender. Úsalo solo si necesitas `TaskCreationOptions.LongRunning` (hilo dedicado para un bucle **síncrono** de larga duración).

---

## Convenciones y reglas básicas

1. **Sufijo `Async`** en métodos que devuelven `Task`/`ValueTask`: `GetUserAsync`. (Excepción común: acciones de controladores y handlers donde el framework fija el nombre.)
2. **"Async all the way"**: si llamas a algo async, tu método también debería ser async, hasta arriba del todo. Mezclar con `.Result`/`.Wait()` provoca bloqueos y deadlocks ([[07 - SynchronizationContext, ConfigureAwait y deadlocks|nivel 7]]).
3. **Acepta un `CancellationToken`** como último parámetro en métodos de I/O ([[05 - Cancelación y timeouts|nivel 5]]).
4. **No devuelvas `null` desde un método que devuelve `Task`**: `await null` lanza `NullReferenceException`. Devuelve `Task.CompletedTask` o `Task.FromResult(x)`.
5. **Siempre espera las tareas que creas**, salvo que diseñes conscientemente un *fire-and-forget* ([[06 - Manejo de excepciones#Fire-and-forget seguro|ver cómo]]).

```csharp
// Implementar una interfaz async con un resultado ya disponible, sin async
public Task<int> GetCountAsync() => Task.FromResult(_items.Count);
public Task ResetAsync() { _items.Clear(); return Task.CompletedTask; }
```

---

## async Main, await using y lambdas async

```csharp
// async Main (C# 7.1+). Con top-level statements (C# 9+) basta con usar await directamente.
await using var conn = new SqlConnection(cs);   // IAsyncDisposable (C# 8): DisposeAsync al salir
await conn.OpenAsync();

Func<int, Task<int>> doubleAsync = async x =>
{
    await Task.Delay(10);
    return x * 2;
};
Console.WriteLine(await doubleAsync(21)); // 42
```

- `await using` llama a `DisposeAsync()` (útil para conexiones, streams, `ServiceBusClient`, etc.).
- Las lambdas async son **`Func<..., Task>`** cuando el tipo destino lo permite; asegúrate de que el destino no sea un `Action` (async void).

---

## Puntos clave

- `Task` = promesa de resultado; **no es un hilo**.
- `async` habilita `await` y genera la máquina de estados; **no** crea asincronía por sí mismo.
- El método corre síncrono **hasta el primer `await` incompleto**.
- `await` desempaqueta el resultado y **relanza la excepción original**.
- `async void` **solo** en manejadores de eventos, con `try/catch`.
- `Task.Run` para CPU en UI/consola; **no** en ASP.NET Core para "hacer async" código síncrono.

## Errores comunes

- Olvidar el `await` (aviso CS4014): la tarea se ejecuta "suelta" y las excepciones se pierden.
- `list.ForEach(async x => ...)` → async void.
- Devolver `null` en un método que devuelve `Task`.
- Marcar `async` un método sin ningún `await`.

## 🎯 Para entrevistas y exámenes

- *"¿Qué hace exactamente `async`?"* → Permite `await` y hace que el compilador genere una máquina de estados que construye el `Task`. No crea hilos.
- *"¿Por qué evitar `async void`?"* → No se puede esperar, las excepciones no se capturan y pueden matar el proceso. Solo para event handlers.
- *"¿Cuándo usarías `Task.Run`?"* → Para trabajo CPU-bound que no debe bloquear la UI o para paralelizar; no para envolver I/O ni en servidores para "hacer async".

## 🧠 Practica

1. Escribe el ejemplo de "Flujo de ejecución paso a paso" e imprime `Environment.CurrentManagedThreadId` en cada línea. ¿Cambia el hilo después del `await`? (En consola, normalmente sí.)
2. Convierte `list.ForEach(async x => await SaveAsync(x))` a una versión correcta secuencial y otra concurrente.
3. Implementa `ICache.GetAsync(string key)` devolviendo `Task.FromResult` cuando el valor está en memoria.

## Referencias

- Microsoft Learn, *async (C# reference)*: https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/async
- Microsoft Learn, *await operator*: https://learn.microsoft.com/dotnet/csharp/language-reference/operators/await
- Stephen Cleary, *Async and Await* (introducción): https://blog.stephencleary.com/2012/02/async-and-await.html
- Stephen Toub, *Task.Run vs Task.Factory.StartNew*: https://devblogs.microsoft.com/dotnet/task-run-vs-task-factory-startnew/

---
⬅️ [[01 - Fundamentos - concurrencia, paralelismo y asincronía]] · ➡️ [[03 - Cómo funciona por dentro]] · ⬆️ [[00 - Async en .NET - Índice]]
