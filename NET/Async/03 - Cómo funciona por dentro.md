---
tags: [dotnet, csharp, async, nivel/3, internals]
up: "[[00 - Async en .NET - Índice]]"
---

# 03 - Cómo funciona por dentro

> Nivel 🟡 3 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Requisito: [[02 - async y await - lo básico]].

## Contenido
- [[#La idea: el compilador reescribe tu método]]
- [[#La máquina de estados generada]]
- [[#El awaiter pattern]]
- [[#Dónde se ejecuta la continuación]]
- [[#ExecutionContext vs SynchronizationContext]]
- [[#Hacer tu propio awaitable]]
- [[#Coste de un método async]]
- [[#Runtime async: hacia dónde va .NET]]
- [[#🧠 Practica]]

---

## La idea: el compilador reescribe tu método

El CLR no "entiende" `await`. Es **azúcar sintáctico**: el compilador de C# convierte tu método en:

1. Un **stub** con la misma firma que crea la máquina de estados, la arranca y devuelve su `Task`.
2. Un **tipo máquina de estados** (`struct` en Release, `class` en Debug) que implementa `IAsyncStateMachine`. Tus variables locales pasan a ser **campos**, y el cuerpo se divide en trozos separados por cada `await`.

Analogía: un libro con marcapáginas. Cada `await` es un punto donde puedes cerrar el libro (suspender), guardar la página (`state`) y lo que recordabas (campos), y más tarde abrirlo y seguir leyendo exactamente ahí.

---

## La máquina de estados generada

Código original:

```csharp
public async Task<int> SumPagesAsync(HttpClient client)
{
    string a = await client.GetStringAsync("https://a.example");
    string b = await client.GetStringAsync("https://b.example");
    return a.Length + b.Length;
}
```

Versión simplificada de lo que genera el compilador (puedes verlo real con [SharpLab](https://sharplab.io) eligiendo "C#" como salida):

```csharp
public Task<int> SumPagesAsync(HttpClient client)
{
    var sm = new SumPagesStateMachine
    {
        client = client,
        builder = AsyncTaskMethodBuilder<int>.Create(),
        state = -1                    // -1 = "ejecutando / no iniciado"
    };
    sm.builder.Start(ref sm);         // ejecuta MoveNext() de forma síncrona
    return sm.builder.Task;
}

struct SumPagesStateMachine : IAsyncStateMachine
{
    public int state;
    public AsyncTaskMethodBuilder<int> builder;
    public HttpClient client;
    private string a;                                  // las variables locales son campos
    private TaskAwaiter<string> awaiter;

    public void MoveNext()
    {
        int result;
        try
        {
            if (state == 0) goto Resume0;
            if (state == 1) goto Resume1;

            // ---- trozo 1: hasta el primer await ----
            awaiter = client.GetStringAsync("https://a.example").GetAwaiter();
            if (!awaiter.IsCompleted)
            {
                state = 0;
                builder.AwaitUnsafeOnCompleted(ref awaiter, ref this); // registra MoveNext como continuación
                return;                                                // ← aquí el método "devuelve el control"
            }
        Resume0:
            state = -1;
            a = awaiter.GetResult();                                   // resultado o relanza excepción

            // ---- trozo 2: hasta el segundo await ----
            awaiter = client.GetStringAsync("https://b.example").GetAwaiter();
            if (!awaiter.IsCompleted)
            {
                state = 1;
                builder.AwaitUnsafeOnCompleted(ref awaiter, ref this);
                return;
            }
        Resume1:
            state = -1;
            string b = awaiter.GetResult();

            // ---- trozo 3: final ----
            result = a.Length + b.Length;
        }
        catch (Exception ex)
        {
            state = -2;                    // -2 = terminado
            builder.SetException(ex);      // la excepción va DENTRO del Task
            return;
        }
        state = -2;
        builder.SetResult(result);         // completa el Task<int>
    }

    public void SetStateMachine(IAsyncStateMachine sm) => builder.SetStateMachine(sm);
}
```

Lo que debes entender de este código:

| Pieza | Papel |
|---|---|
| `state` | En qué `await` quedó el método. `-1` ejecutando, `0..n` suspendido en el await n, `-2` terminado. |
| Campos | Las variables locales que deben **sobrevivir** a un `await`. |
| `MoveNext()` | Se llama una vez al principio y una vez **por cada reanudación**. |
| `IsCompleted` | **Camino rápido**: si la tarea ya terminó, no hay suspensión ni asignación. |
| `AwaitUnsafeOnCompleted` | En la **primera** suspensión, el builder copia la máquina de estados (struct) al **heap** (una "caja") para que sobreviva, y registra `MoveNext` como continuación. |
| `SetException` | Por esto una excepción en un método async **no se lanza al llamar**, sino al hacer `await` del Task. |

> [!note] Consecuencia práctica: validación de argumentos
> Como todo el cuerpo está dentro del `try`, incluso `ArgumentNullException.ThrowIfNull(x)` al principio se guarda en el `Task` y se observa en el `await`, no en la llamada. Normalmente es lo deseable. Si quieres fallo inmediato ("*eager validation*"), separa en un método no async que valida y una función local async que hace el trabajo.

---

## El awaiter pattern

`await` funciona con **cualquier tipo** que cumpla un patrón (*duck typing*), no solo con `Task`:

```text
await x  requiere:
  x.GetAwaiter()            → devuelve un "awaiter" (método de instancia o de extensión)
  awaiter.IsCompleted       → bool
  awaiter.OnCompleted(Action)    (interfaz INotifyCompletion)
     o UnsafeOnCompleted(Action) (ICriticalNotifyCompletion, no captura ExecutionContext)
  awaiter.GetResult()       → void o T; lanza si hubo error
```

Por eso puedes hacer `await` de `Task`, `ValueTask`, `Task.Yield()`, `ConfiguredTaskAwaitable` (el resultado de `ConfigureAwait(false)`) o tus propios tipos.

---

## Dónde se ejecuta la continuación

Cuando un `await` sobre un `Task` se reanuda, ¿en qué hilo continúa? El `TaskAwaiter` decide así:

```text
¿Había un SynchronizationContext (no el por defecto) al hacer await, y no usaste ConfigureAwait(false)?
 ├── Sí → continuación se PUBLICA (Post) en ese contexto
 │        (WinForms/WPF/MAUI: el hilo de UI; ASP.NET clásico: el contexto de la petición)
 └── No → ¿TaskScheduler.Current es distinto del por defecto?
          ├── Sí → se encola en ese scheduler
          └── No → se ejecuta en el thread pool
                   (a menudo de forma síncrona en el hilo que completó la tarea, si es seguro)
```

- **ASP.NET Core y apps de consola no tienen `SynchronizationContext`**: la continuación va al pool, a cualquier hilo.
- La ejecución "inline" en el hilo que completa la tarea es una optimización. Tiene una consecuencia importante con `TaskCompletionSource`: si no usas `TaskCreationOptions.RunContinuationsAsynchronously`, quien llama a `SetResult` puede acabar **ejecutando el código de quien esperaba** antes de que `SetResult` retorne ([[09 - Sincronización y primitivas avanzadas#TaskCompletionSource|ver nivel 9]]).

Se profundiza en [[07 - SynchronizationContext, ConfigureAwait y deadlocks]].

---

## ExecutionContext vs SynchronizationContext

Dos "contextos" distintos que la gente confunde:

| | `ExecutionContext` | `SynchronizationContext` |
|---|---|---|
| **Qué es** | "Bolsa" de datos ambientales que viaja con el flujo lógico | Abstracción de **dónde** ejecutar código |
| **Contiene** | Valores de `AsyncLocal<T>`, contexto de seguridad, cultura (`CultureInfo.CurrentCulture`) | Un `Post`/`Send` que manda trabajo a un hilo o cola concreta |
| **Viaja a través de `await`** | **Siempre** (no se puede desactivar con `ConfigureAwait`) | Solo si existe y no usas `ConfigureAwait(false)` |
| **Ejemplos** | `Activity.Current` (trazas), `IHttpContextAccessor`, scopes de logging | Hilo de UI, contexto de petición de ASP.NET clásico |

Regla mental: `ExecutionContext` = **qué** datos me acompañan; `SynchronizationContext` = **dónde** me ejecuto.

---

## Hacer tu propio awaitable

Ejemplo didáctico: poder escribir `await TimeSpan.FromSeconds(1)`.

```csharp
public static class TimeSpanAwaiterExtensions
{
    public static TaskAwaiter GetAwaiter(this TimeSpan delay) =>
        Task.Delay(delay).GetAwaiter();
}

await TimeSpan.FromMilliseconds(500); // compila gracias al patrón GetAwaiter
```

En la práctica casi nunca necesitas crear awaiters propios; es útil saberlo para entender `ConfigureAwait`, `Task.Yield` y `ValueTask`.

### `Task.Yield()`
`await Task.Yield()` devuelve un awaitable cuyo `IsCompleted` es **siempre `false`**: fuerza a que el resto del método se ejecute como continuación (en el contexto actual o en el pool). Útil para:
- Asegurar que un método async **no corra síncronamente** al inicio (p. ej., en un bucle infinito de un `BackgroundService` que no tiene ningún `await` real al principio).
- En UI, ceder el hilo para que se procesen otros mensajes (aunque no garantiza que se repinte antes).

En .NET 8+ existe también `ConfigureAwaitOptions.ForceYielding` para lo mismo sobre un `Task`.

---

## Coste de un método async

| Situación | Coste aproximado |
|---|---|
| El método completa **síncronamente** (sin suspender) y devuelve `Task` | Se ejecuta la máquina de estados (struct en pila). Para `Task` sin resultado y algunos `Task<T>` comunes (`bool`, enteros pequeños, `null`) el runtime reutiliza tareas cacheadas; para otros `Task<T>` se asigna un `Task`. |
| El método **se suspende** al menos una vez | La máquina de estados se copia al heap en una caja que hace de `Task` (una asignación principal en .NET Core moderno), más lo que asigne la operación subyacente. |
| Muchos `await` encadenados en profundidad | Cada nivel tiene su máquina de estados: llamadas async muy anidadas tienen un coste acumulado. |

Para casi todo el código de aplicación este coste es **despreciable** frente al I/O que espera. Optimizarlo es tema de [[10 - Rendimiento y diagnóstico]] y [[Task vs ValueTask]].

### ¿Eliminar `async`/`await` ("elidir")?

```csharp
// Con async: una máquina de estados más
public async Task<User> GetAsync(int id) => await _repo.GetAsync(id);

// Sin async: devuelve el Task directamente
public Task<User> GetAsync(int id) => _repo.GetAsync(id);
```

Elidir ahorra un poco, pero **cambia el comportamiento** en algunos casos:
- Dentro de `using`/`try`: el recurso se libera **antes** de que la tarea termine. ❌
- Las excepciones síncronas (validación) se lanzan **en la llamada**, no en el `await`.
- El método no aparece en la pila de llamadas de la excepción (depuración más difícil).

Regla razonable: **elide solo en pasarelas triviales de una línea sin `using`/`try`**; en caso de duda, mantén `async`/`await`.

---

## Runtime async: hacia dónde va .NET

> [!info] Estado a finales de 2026 — verifica antes de usarlo
> El equipo de .NET está trabajando en **"runtime async"**: que el propio runtime (no el compilador de C#) implemente la suspensión y reanudación de métodos async, para reducir asignaciones y el coste de las máquinas de estados y mejorar las pilas de llamadas. Se introdujo como característica **experimental/preview en .NET 10**. Es un cambio de implementación: **el modelo de programación (`async`/`await`, `Task`) no cambia**, así que todo lo de este curso sigue aplicando. Consulta las notas de la versión de .NET que uses antes de activarlo.

---

## Puntos clave

- `async`/`await` es una transformación del **compilador** a una máquina de estados.
- Cada `await` es un punto de suspensión; el estado y las variables locales viven en campos.
- Si la tarea ya terminó, **no hay suspensión** (camino rápido).
- Las excepciones se guardan en el `Task`; se observan en el `await`.
- `ExecutionContext` siempre fluye; `SynchronizationContext` solo si existe y no usas `ConfigureAwait(false)`.
- Cualquier tipo con `GetAwaiter()` es *awaitable*.

## Errores comunes

- Creer que la continuación siempre vuelve al **mismo hilo**: solo pasa si hay un `SynchronizationContext` de un solo hilo (UI).
- Elidir `async` dentro de un `using` y liberar el recurso antes de tiempo.
- Usar `[ThreadStatic]` o `ThreadLocal<T>` para datos que deben sobrevivir a un `await` (usa `AsyncLocal<T>`).

## 🎯 Para entrevistas y exámenes

- *"¿Qué genera el compilador para un método async?"* → Una máquina de estados que implementa `IAsyncStateMachine`, con `MoveNext`, un campo `state`, las variables locales como campos y un *builder* que crea el `Task`.
- *"¿Qué necesita un tipo para poder hacer `await`?"* → Un `GetAwaiter()` que devuelva un objeto con `IsCompleted`, `OnCompleted` y `GetResult`.
- *"¿Diferencia entre `ExecutionContext` y `SynchronizationContext`?"* → El primero transporta datos ambientales (siempre fluye); el segundo decide dónde se ejecuta la continuación.

## 🧠 Practica

1. Pega `SumPagesAsync` en SharpLab (salida C#, modo Release) y localiza `MoveNext`, `<>1__state` y `AwaitUnsafeOnCompleted`.
2. Escribe un método async que valide un argumento nulo. Llama al método **sin** `await` dentro de un `try/catch`. ¿Se captura? (No: la excepción está en el `Task`.)
3. Implementa `await TimeSpan` con el método de extensión de esta nota.

## Referencias

- Stephen Toub, *How Async/Await Really Works in C#*: https://devblogs.microsoft.com/dotnet/how-async-await-really-works/
- Stephen Toub, *ExecutionContext vs SynchronizationContext*: https://devblogs.microsoft.com/dotnet/executioncontext-vs-synchronizationcontext/
- C# specification, *Await expressions*: https://learn.microsoft.com/dotnet/csharp/language-reference/language-specification/expressions#1298-await-expressions
- SharpLab: https://sharplab.io

---
⬅️ [[02 - async y await - lo básico]] · ➡️ [[04 - Composición de tareas]] · ⬆️ [[00 - Async en .NET - Índice]]
