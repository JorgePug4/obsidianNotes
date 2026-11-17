---
tags: [dotnet, csharp, async, nivel/1, fundamentos]
up: "[[00 - Async en .NET - Índice]]"
---

# 01 - Fundamentos: concurrencia, paralelismo y asincronía

> Nivel 🟢 1 del [[00 - Async en .NET - Índice#📈 Roadmap|roadmap]]. Sin requisitos previos más allá de C# básico.

## Contenido
- [[#El problema que resuelve async]]
- [[#Concurrencia, paralelismo y asincronía]]
- [[#Procesos, hilos y el thread pool]]
- [[#I/O-bound vs CPU-bound]]
- [[#There is no thread]]
- [[#Por qué async escala en un servidor]]
- [[#Cuándo async NO ayuda]]
- [[#🧠 Practica]]

---

## El problema que resuelve async

### Analogía: el restaurante
Un camarero (hilo) atiende mesas (peticiones).

- **Síncrono / bloqueante:** el camarero toma el pedido de la mesa 1, va a la cocina y **se queda de pie esperando** a que el plato esté listo. Mientras tanto, nadie atiende las mesas 2, 3 y 4. Para atender más mesas necesitas **más camareros** (más hilos), que cuestan dinero (memoria) y se estorban entre sí (cambios de contexto).
- **Asíncrono:** el camarero toma el pedido, lo deja en cocina con una nota "avísame cuando esté" y **vuelve a atender otras mesas**. Cuando la cocina avisa, **cualquier camarero libre** lleva el plato.

La cocina es la base de datos, el disco o la red. El camarero esperando de pie es un hilo bloqueado en `Read()`. **Async no hace que la cocina cocine más rápido; hace que los camareros no pierdan el tiempo esperando.**

### Lo mismo en código

```csharp
// ❌ Síncrono: el hilo queda bloqueado durante toda la llamada HTTP
public string GetPage(string url)
{
    using var client = new HttpClient();
    return client.GetStringAsync(url).Result; // bloquea el hilo (además es sync-over-async, ver nivel 7)
}

// ✅ Asíncrono: el hilo se libera mientras se espera la respuesta
public async Task<string> GetPageAsync(string url, HttpClient client)
{
    return await client.GetStringAsync(url);
}
```

En la versión asíncrona, cuando la ejecución llega a `await` y la respuesta aún no ha llegado, el método **devuelve el control** a quien lo llamó. El hilo vuelve al *pool* y puede atender otra cosa. Cuando llegan los bytes por la red, el runtime **reanuda** el método (posiblemente en otro hilo).

---

## Concurrencia, paralelismo y asincronía

Son tres conceptos distintos que a menudo se confunden:

| Concepto | Definición | Ejemplo | Requiere varios núcleos |
|---|---|---|---|
| **Concurrencia** | Varias tareas **en progreso** en el mismo periodo de tiempo (pueden intercalarse) | Un servidor que atiende 1 000 peticiones "a la vez" | No |
| **Paralelismo** | Varias tareas **ejecutándose literalmente al mismo tiempo** en núcleos distintos | Procesar 8 imágenes en 8 núcleos | Sí |
| **Asincronía** | Iniciar una operación y **no bloquear** mientras termina; continuar cuando avise | Pedir datos a la BD y liberar el hilo | No |

- La asincronía es una **forma** de lograr concurrencia sin necesitar un hilo por tarea.
- El paralelismo es otra forma de concurrencia, orientada a **usar más CPU**.
- Puedes tener asincronía con un solo hilo (JavaScript en el navegador lo hace siempre).

> [!tip] Frase para entrevistas
> "La concurrencia trata de **gestionar** muchas cosas a la vez; el paralelismo de **hacer** muchas cosas a la vez. `async`/`await` es una herramienta de concurrencia para operaciones de I/O; `Parallel` y PLINQ son herramientas de paralelismo para CPU."

---

## Procesos, hilos y el thread pool

### Hilo (thread)
Unidad de ejecución que el sistema operativo planifica sobre un núcleo. Cada hilo tiene su **pila** (en Windows se reserva 1 MB por defecto para hilos de .NET; en Linux depende de `ulimit`/configuración, típicamente 1,5-8 MB reservados, aunque solo se consume la memoria que se toca). Crear y destruir hilos es caro, y tener miles provoca muchos **cambios de contexto** (*context switches*).

### Thread pool
.NET mantiene un **conjunto de hilos reutilizables** (`ThreadPool`). En lugar de crear un hilo por trabajo, encolas el trabajo y un hilo del pool lo ejecuta.

```text
           ┌──────────────── Thread Pool ────────────────┐
cola  ───► │ [trabajo][trabajo][trabajo]                  │
global     │   ▼          ▼          ▼                    │
           │ hilo 1    hilo 2    hilo 3   ... (crece/encoge)│
           └──────────────────────────────────────────────┘
```

Detalles relevantes:
- Cada hilo del pool tiene además una **cola local** (*work-stealing*): los hilos ociosos roban trabajo de las colas de otros.
- El pool empieza con un mínimo de hilos igual al número de núcleos (`ThreadPool.GetMinThreads`) y **crea más gradualmente** cuando detecta que el trabajo no avanza (algoritmo *hill climbing* + inyección por starvation). Crecer es **lento** a propósito: esto es la raíz del *thread pool starvation* ([[10 - Rendimiento y diagnóstico#Thread pool starvation|nivel 10]]).
- ASP.NET Core, `Task.Run`, las continuaciones de `await` (sin contexto) y los *timers* usan el pool.

### Completación de I/O
Las operaciones de red y disco asíncronas no usan un hilo mientras esperan: el SO notifica cuando terminan (IOCP en Windows, `epoll` en Linux, `kqueue` en macOS). .NET recibe esa notificación y **encola la continuación** en el thread pool.

---

## I/O-bound vs CPU-bound

Esta distinción decide **qué herramienta usar**. Es la pregunta más importante del curso.

| | I/O-bound | CPU-bound |
|---|---|---|
| **Dónde se gasta el tiempo** | Esperando a algo externo: red, disco, BD, otro servicio | Calculando: el procesador está ocupado |
| **Ejemplos** | `HttpClient`, EF Core, leer un fichero, Service Bus, Redis | Comprimir, cifrar, redimensionar imágenes, cálculos numéricos, parsear un JSON enorme |
| **Herramienta** | `async`/`await` con APIs asíncronas reales | `Task.Run`, `Parallel.For`, PLINQ (en apps de escritorio o trabajos batch) |
| **¿Libera hilos?** | Sí, durante la espera | No; el trabajo **necesita** un hilo |
| **Beneficio** | Escalabilidad (más peticiones con los mismos hilos) | Velocidad (usar varios núcleos) o *responsividad* (no congelar la UI) |

```csharp
// I/O-bound: await directo sobre la API asíncrona
var user = await db.Users.FirstOrDefaultAsync(u => u.Id == id, ct);

// CPU-bound en una app de escritorio: mover el cálculo fuera del hilo de UI
var hash = await Task.Run(() => ComputeExpensiveHash(bigFile));

// CPU-bound en un servidor: normalmente se ejecuta directamente, SIN Task.Run
var report = BuildReport(data); // ya estás en un hilo del pool; Task.Run solo añade overhead
```

> [!warning] Error frecuente
> Envolver código síncrono en `Task.Run` dentro de ASP.NET Core "para hacerlo async" **no mejora la escalabilidad**: liberas el hilo de la petición pero ocupas otro hilo del mismo pool. Solo añades coste. Se detalla en [[02 - async y await - lo básico#Task.Run: cuándo sí y cuándo no|Task.Run: cuándo sí y cuándo no]].

---

## There is no thread

Idea central (título del famoso artículo de Stephen Cleary): **durante una operación de I/O asíncrona pura no hay ningún hilo esperando**.

Lo que pasa al hacer `await stream.ReadAsync(buffer)` sobre un socket:

1. Tu código llama a `ReadAsync`. .NET pide al SO una lectura asíncrona y le pasa el buffer.
2. El SO registra la petición en el driver de red. **Tu método devuelve un `Task` incompleto.**
3. `await` ve que no está completo: registra "lo que queda del método" como **continuación** y **retorna**. El hilo queda libre.
4. … milisegundos después llegan paquetes. La tarjeta de red genera una interrupción, el driver copia los datos, el SO marca la operación como completada.
5. El runtime recibe la notificación (un hilo de I/O la recoge brevemente) y **encola la continuación**.
6. Un hilo del pool ejecuta el resto de tu método.

Entre 3 y 5, ningún hilo de tu proceso está dedicado a esa lectura. Por eso un servidor puede tener **decenas de miles de operaciones en vuelo** con unas pocas decenas de hilos.

---

## Por qué async escala en un servidor

Supón una API que recibe 1 000 peticiones/s y cada una espera 200 ms a la base de datos.

- **Síncrona:** cada petición ocupa un hilo 200 ms → necesitas ~200 hilos **solo esperando** (Ley de Little: concurrencia = tasa × tiempo = 1 000 × 0,2). El pool tarda en crecer; durante los picos las peticiones hacen cola y la latencia se dispara.
- **Asíncrona:** el hilo solo se ocupa durante el poco tiempo de CPU (parsear, serializar). Con unos pocos hilos atiendes la misma carga.

Async **no reduce la latencia de una petición individual** (incluso añade unos microsegundos de overhead). Lo que mejora es el **throughput** y la **estabilidad bajo carga**.

---

## Cuándo async NO ayuda

- **Trabajo puramente de CPU** en un servidor: async no aporta; el hilo está trabajando, no esperando.
- **Operaciones que se completan en nanosegundos** (leer de un diccionario en memoria): marcarlas `async` solo añade overhead.
- **Aplicaciones de baja concurrencia** (una herramienta de consola que hace una cosa): funciona igual, aunque sigue siendo buena práctica usar APIs async si existen.
- **Cuando la API subyacente no es asíncrona de verdad**: envolverla en `Task.Run` simula asincronía pero sigue ocupando un hilo ("fake async").

> [!note] Asincronía en la UI
> En WinForms/WPF/MAUI el objetivo de async es distinto: **no bloquear el hilo de UI** para que la ventana siga respondiendo. Aquí `Task.Run` para trabajo de CPU **sí** tiene sentido.

---

## Puntos clave

- Async = **no bloquear mientras se espera**. No es lo mismo que paralelismo.
- **I/O-bound → `await`**; **CPU-bound → `Task.Run`/`Parallel`** (y en servidores, casi siempre ejecución directa).
- Durante I/O asíncrono **no hay hilo esperando**.
- En servidores, async mejora **throughput y resiliencia ante picos**, no la latencia individual.
- El thread pool crece **despacio**: bloquear hilos es peligroso.

## Errores comunes

- Creer que `async` hace el código "más rápido" o "en paralelo".
- Usar `Task.Run` en ASP.NET Core para "convertir" código síncrono en async.
- Pensar que cada `await` usa un hilo nuevo.

## 🎯 Para entrevistas y exámenes

- *"¿Diferencia entre concurrencia y paralelismo?"* → Concurrencia: varias tareas en progreso intercaladas; paralelismo: ejecución simultánea en varios núcleos.
- *"¿Por qué async mejora la escalabilidad de una API?"* → Porque libera hilos del pool mientras se espera I/O; con menos hilos se atienden más peticiones y se evita la starvation.
- *"¿Async hace que mi petición sea más rápida?"* → No; la latencia individual es igual o ligeramente peor. Mejora el throughput.

## 🧠 Practica

1. Clasifica como I/O-bound o CPU-bound: llamar a una API REST, calcular un SHA-256 de 2 GB, consultar SQL Server, generar un PDF, enviar un mensaje a Service Bus, ordenar 10 millones de enteros.
2. Una API recibe 500 req/s y cada una espera 400 ms a otro servicio. ¿Cuántos hilos bloqueados necesitaría la versión síncrona? (Respuesta: 500 × 0,4 = 200.)
3. Ejecuta `ThreadPool.GetMinThreads(out var w, out var io)` y `Environment.ProcessorCount` en tu máquina y compara.

## Referencias

- Stephen Cleary, *There Is No Thread*: https://blog.stephencleary.com/2013/11/there-is-no-thread.html
- Microsoft Learn, *The managed thread pool*: https://learn.microsoft.com/dotnet/standard/threading/the-managed-thread-pool
- Microsoft Learn, *Asynchronous programming scenarios*: https://learn.microsoft.com/dotnet/csharp/asynchronous-programming/async-scenarios

---
➡️ Siguiente: [[02 - async y await - lo básico]] · ⬆️ [[00 - Async en .NET - Índice]]
