---
tags: [dotnet, csharp, async, nivel/7, deadlocks]
up: "[[00 - Async en .NET - Índice]]"
---

# 07 - SynchronizationContext, ConfigureAwait y deadlocks

> Nivel 🟠 7 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[03 - Cómo funciona por dentro]].

## Contenido
- [[#Qué es un SynchronizationContext]]
- [[#Qué contexto hay en cada tipo de aplicación]]
- [[#El deadlock clásico]]
- [[#ConfigureAwait(false)]]
- [[#ConfigureAwaitOptions (.NET 8)]]
- [[#¿Debo usar ConfigureAwait(false)?]]
- [[#Sync-over-async y async-over-sync]]
- [[#Si de verdad necesitas llamar async desde código síncrono]]
- [[#🧠 Practica]]

---

## Qué es un SynchronizationContext

Una clase con un método clave, `Post(callback, state)`, que significa "**ejecuta esto donde corresponda**". Cada entorno define qué es "donde corresponda":

- **WinForms / WPF / MAUI**: en el **hilo de UI** (el único que puede tocar controles).
- **ASP.NET clásico (.NET Framework)**: en el contexto de la petición, **un hilo a la vez**, con `HttpContext.Current` disponible.
- **Por defecto** (`SynchronizationContext` base): en el thread pool.

`await` captura `SynchronizationContext.Current` y, al reanudar, hace `Post` de la continuación ahí. Por eso en una app de UI puedes hacer:

```csharp
private async void OnLoadClicked(object? s, EventArgs e)
{
    var data = await _api.GetDataAsync();  // se espera sin bloquear la UI
    grid.DataSource = data;                // ✅ estamos de vuelta en el hilo de UI
}
```

---

## Qué contexto hay en cada tipo de aplicación

| Tipo de app | `SynchronizationContext.Current` | ¿Riesgo de deadlock con `.Result`? |
|---|---|---|
| WinForms, WPF, MAUI, Avalonia | De UI (un solo hilo) | **Sí** |
| ASP.NET (.NET Framework, `System.Web`) | `AspNetSynchronizationContext` | **Sí** |
| ASP.NET Core | **Ninguno** (`null`) | No hay deadlock por contexto, **pero sí *thread pool starvation*** |
| Consola, Worker Service | Ninguno | No por contexto; sí starvation |
| xUnit v2 | Uno propio (`AsyncTestSyncContext`) que rastrea `async void` | Normalmente no, pero los tests no deberían bloquear; el analizador xUnit1031 lo avisa |
| Blazor WebAssembly | Un solo hilo (navegador) | Bloquear es imposible/peligroso |

---

## El deadlock clásico

```csharp
// WinForms / WPF / ASP.NET clásico
public string GetData()
{
    return GetDataAsync().Result;          // (1) bloquea el hilo de UI esperando
}

public async Task<string> GetDataAsync()
{
    var s = await _http.GetStringAsync(url); // (2) captura el contexto de UI
    return s.ToUpper();                       // (3) necesita el hilo de UI para continuar
}
```

Secuencia:
1. El hilo de UI llama a `GetDataAsync()`, que inicia la petición y devuelve un `Task` incompleto.
2. El hilo de UI se **bloquea** en `.Result`.
3. La respuesta HTTP llega. La continuación (`return s.ToUpper()`) se **publica en el contexto de UI**.
4. El hilo de UI está bloqueado esperando a que la tarea termine… y la tarea no puede terminar porque necesita el hilo de UI. **Deadlock.** La app se congela para siempre.

```text
  Hilo de UI ──► .Result ──► (esperando a la tarea) ─┐
                                                      │  se esperan mutuamente
  Tarea ──► continuación ──► (esperando al hilo de UI)┘
```

**Solución correcta: no bloquear.** Haz el llamador `async` y usa `await` ("async all the way").

---

## ConfigureAwait(false)

`await task.ConfigureAwait(false)` le dice al awaiter: **"no captures el contexto; continúa donde sea (thread pool)"**.

```csharp
public async Task<string> GetDataAsync()
{
    var s = await _http.GetStringAsync(url).ConfigureAwait(false);
    return s.ToUpper();    // se ejecuta en un hilo del pool, no necesita el hilo de UI
}
```

Efectos:
- **Rendimiento**: evita el `Post` al contexto (relevante en UI, irrelevante en ASP.NET Core, que no tiene contexto).
- **Mitiga** el deadlock clásico si **todos** los `await` de la cadena lo usan… pero es frágil: basta con un `await` sin él (en tu código o en una dependencia) para que vuelva el deadlock. **No es la solución al deadlock; la solución es no bloquear.**
- Tras `ConfigureAwait(false)` **no** estás en el hilo de UI: no toques controles ni `HttpContext.Current` (ASP.NET clásico).

> [!note] Lo que `ConfigureAwait(false)` NO hace
> - No afecta al `ExecutionContext`: `AsyncLocal<T>`, `Activity.Current` y la cultura **siguen fluyendo**.
> - Si la tarea ya está completa, no hay suspensión: el código **sigue en el mismo hilo y contexto**. Por eso no basta con ponerlo solo en el primer `await` "para salir del contexto".

---

## ConfigureAwaitOptions (.NET 8)

Sobrecarga `Task.ConfigureAwait(ConfigureAwaitOptions)` con opciones combinables:

| Opción | Efecto |
|---|---|
| `None` | Equivale a `ConfigureAwait(false)` |
| `ContinueOnCapturedContext` | Equivale a `ConfigureAwait(true)` (el comportamiento por defecto) |
| `SuppressThrowing` | Espera sin lanzar si la tarea falla o se cancela (solo `Task`, no `Task<T>`) |
| `ForceYielding` | Fuerza la suspensión aunque la tarea ya esté completa (como `Task.Yield()`) |

```csharp
await cts.CancelAsync();
await backgroundTask.ConfigureAwait(ConfigureAwaitOptions.SuppressThrowing); // esperar a que termine, sin importar cómo
```

---

## ¿Debo usar ConfigureAwait(false)?

| Tipo de código | Recomendación |
|---|---|
| **Librería** de propósito general (NuGet, código compartido que puede usarse desde UI) | ✅ Sí, en **todos** los `await`. Puedes ayudarte con el analizador **CA2007** o con el paquete `ConfigureAwait.Fody` |
| **Aplicación** ASP.NET Core / Worker / consola | ❌ No hace falta: no hay contexto. Añade ruido. (Hay equipos que lo ponen por consistencia; es una decisión de estilo, no de corrección) |
| **Aplicación** de UI, código que toca controles tras el `await` | ❌ No: necesitas volver al hilo de UI |
| **Aplicación** de UI, servicios internos que no tocan UI | ⚠️ Opcional; mejora un poco el rendimiento |
| ASP.NET clásico que usa `HttpContext.Current` tras el `await` | ❌ No |

---

## Sync-over-async y async-over-sync

| Anti-patrón | Qué es | Problema |
|---|---|---|
| **Sync-over-async** | Bloquear sobre código async: `.Result`, `.Wait()`, `.GetAwaiter().GetResult()` | Deadlock (con contexto) o **thread pool starvation** (sin contexto): el hilo bloqueado no hace nada útil y el pool necesita otro para completar la tarea |
| **Async-over-sync** | Exponer `FooAsync()` que solo hace `Task.Run(() => Foo())` | "Fake async": consume un hilo igual; engaña al consumidor sobre la escalabilidad |

En ASP.NET Core, el sync-over-async no produce el deadlock de UI, pero bajo carga produce algo igual de grave: **starvation**. Cada petición bloquea un hilo esperando una tarea que necesita **otro** hilo del mismo pool para terminar. El pool crece despacio; la latencia se dispara y el servicio parece "colgado" con CPU baja. Ver [[10 - Rendimiento y diagnóstico#Thread pool starvation|nivel 10]].

---

## Si de verdad necesitas llamar async desde código síncrono

A veces no hay alternativa: una interfaz síncrona de un framework que no controlas, un constructor, un `Dispose()` síncrono. Opciones, por orden de preferencia:

1. **Cambia el diseño**: busca la variante async de la interfaz (`IAsyncDisposable`, `IHostedService.StartAsync`, *factories* async en lugar de constructores async).
2. **Usa la API síncrona real** si la dependencia la ofrece (p. ej. `File.ReadAllText` en vez de `ReadAllTextAsync().Result`).
3. **Bloquea en un hilo sin contexto** y acéptalo conscientemente:

```csharp
// Evita el deadlock de contexto porque la tarea arranca en el pool (sin contexto de UI),
// pero SIGUE bloqueando un hilo: no lo uses en rutas calientes de un servidor.
var result = Task.Run(() => GetDataAsync()).GetAwaiter().GetResult();
```

> [!warning] Constructores async
> No existen constructores async. Patrones: método *factory* estático `static async Task<Foo> CreateAsync()`, inicialización perezosa con `Lazy<Task<T>>` ([[09 - Sincronización y primitivas avanzadas#Inicialización perezosa async|nivel 9]]), o inicializar en `IHostedService.StartAsync`.

---

## Puntos clave

- `SynchronizationContext` decide **dónde** continúa el código tras `await`.
- UI y ASP.NET clásico tienen contexto de un solo hilo → `.Result` = **deadlock**.
- ASP.NET Core y consola **no** tienen contexto → `.Result` = **starvation** bajo carga.
- `ConfigureAwait(false)`: obligatorio en librerías, innecesario en apps ASP.NET Core, perjudicial en código de UI que toca controles.
- La solución a ambos problemas es la misma: **async all the way**.

## Errores comunes

- "Arreglar" un deadlock añadiendo `ConfigureAwait(false)` en un solo sitio.
- Creer que en ASP.NET Core bloquear con `.Result` es seguro "porque no hay deadlock".
- Tocar controles de UI después de `ConfigureAwait(false)`.
- Exponer métodos "fake async" en una librería.

## 🎯 Para entrevistas y exámenes

- *"Explica el deadlock de `.Result` en WPF."* → El hilo de UI se bloquea esperando la tarea; la continuación de la tarea necesita el hilo de UI (contexto capturado) → espera circular.
- *"¿Por qué no ocurre en ASP.NET Core?"* → Porque no hay `SynchronizationContext`; la continuación va a cualquier hilo del pool. Pero bloquear causa starvation.
- *"¿Cuándo usarías `ConfigureAwait(false)`?"* → En código de librería, para no depender del contexto del llamador y evitar el `Post` innecesario.

## 🧠 Practica

1. En una app WinForms o WPF, reproduce el deadlock con `.Result`. Arréglalo haciendo el handler `async void` con `await`.
2. En una app de consola (sin contexto), demuestra que `.Result` **no** produce deadlock. Después instala un `SynchronizationContext` de un solo hilo (p. ej. con `Nito.AsyncEx.AsyncContext`) y repite.
3. Revisa una librería tuya y activa CA2007 en `.editorconfig`: `dotnet_diagnostic.CA2007.severity = warning`.

## Referencias

- Stephen Toub, *ConfigureAwait FAQ*: https://devblogs.microsoft.com/dotnet/configureawait-faq/
- Stephen Cleary, *Don't Block on Async Code*: https://blog.stephencleary.com/2012/07/dont-block-on-async-code.html
- Stephen Cleary, *ASP.NET Core SynchronizationContext*: https://blog.stephencleary.com/2017/03/aspnetcore-synchronization-context.html
- Microsoft Learn, *ConfigureAwaitOptions*: https://learn.microsoft.com/dotnet/api/system.threading.tasks.configureawaitoptions

---
⬅️ [[06 - Manejo de excepciones]] · ➡️ [[08 - Streams asíncronos y Channels]] · ⬆️ [[00 - Async en .NET - Índice]]
