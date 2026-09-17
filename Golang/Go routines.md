---
tags: [golang, concurrencia, tiempo-real, backend]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [Goroutines, Goroutines y Channels, Concurrencia en Go]
---

# Goroutines y Channels para Aplicaciones en Tiempo Real

## Índice

1. Por qué Go para tiempo real
2. Goroutines: lo básico
3. Channels: lo básico
4. `select`: el corazón de la concurrencia reactiva
5. El paquete `sync`: WaitGroup, Mutex, Once
6. `context`: cancelación y timeouts
7. Patrones de concurrencia
8. Caso completo: servidor de chat/broadcast en tiempo real
9. Errores comunes y cómo evitarlos
10. Checklist mental para diseñar sistemas concurrentes
11. Puntos clave, errores comunes y preguntas de repaso

## Conceptos relacionados

- [[Task vs ValueTask|Task vs ValueTask en .NET]] → el otro modelo de asincronía que uso a diario; comparar `async`/`await` (cooperativo sobre el *thread pool*) con goroutines (hilos verdes M:N) ayuda a entender ambos.
- [[Bulkhead]] → el *Worker Pool* de la sección 7.1 es un Bulkhead: limita la concurrencia hacia un recurso.
- [[Retry con Backoff Exponencial]] → el ejercicio de reconexión con backoff de la sección final.
- [[Circuit Breaker]] → en Go se implementa con `sony/gobreaker`; complementa los timeouts con `context`.
- [[🏗️ Diseño de Microservicios]] → health checks, *graceful shutdown* con `context` y observabilidad aplican igual a servicios en Go.
- [[Diseño Api Rest|Diseño de APIs REST]] → los servidores WebSocket/SSE de esta nota conviven con la API REST del mismo servicio.

> [!info] Alcance de la nota
> Está orientada a **aplicaciones en tiempo real** (chats, dashboards, notificaciones). Los fundamentos (goroutines, channels, `select`, `sync`, `context`) aplican a cualquier programa en Go.

---

## 1. Por qué Go para tiempo real

Go fue diseñado con la concurrencia como ciudadano de primera clase. Para sistemas en tiempo real (chats, notificaciones push, dashboards live, trading, IoT, streaming) necesitas manejar **miles de conexiones simultáneas**, cada una esperando eventos de forma independiente, sin bloquear a las demás.

La filosofía de Go, resumida en una frase famosa:

> "No te comuniques compartiendo memoria; comparte memoria comunicándote."

Es decir: en lugar de que múltiples hilos toqueteen la misma variable protegida por locks (el modelo clásico de C/Java), en Go prefieres que las goroutines se pasen datos entre sí a través de **channels**. Esto no elimina la necesidad de `sync` en todos los casos, pero cambia el default mental.

---

## 2. Goroutines: lo básico

Una goroutine es una función que corre de forma concurrente, gestionada por el runtime de Go (no por el sistema operativo directamente). Son extremadamente baratas: arrancan con ~2KB de stack (vs ~1-8MB de un thread de OS) y el runtime las multiplexa sobre un pool de threads reales (modelo M:N).

```go
func main() {
    go decirHola() // se lanza y el main sigue sin esperar

    time.Sleep(100 * time.Millisecond) // truco feo solo para el ejemplo
}

func decirHola() {
    fmt.Println("hola desde una goroutine")
}
```

**Puntos clave:**

- `go f()` no bloquea; la ejecución continúa inmediatamente en la goroutine actual.
- Si `main()` termina, **todas** las goroutines mueren, hayan terminado o no. Por eso el `time.Sleep` de arriba es una mala práctica — la usamos solo para ilustrar el problema. La solución real es sincronización explícita (`sync.WaitGroup`, channels o `context`).
- Puedes lanzar cientos de miles de goroutines sin problema; es común en servidores que manejan una goroutine por conexión.

### Closures y la trampa clásica del loop

```go
// MAL (antes de Go 1.22)
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i) // captura la variable i, no su valor en ese momento
    }()
}
```

Antes de Go 1.22, todas las goroutines podían imprimir `3` porque compartían la misma variable `i`. Desde Go 1.22, cada iteración del `for` crea una nueva variable `i`, así que este patrón ya es seguro. Aun así, en código que debe correr en versiones anteriores o por claridad, es común pasar el valor explícitamente:

```go
for i := 0; i < 3; i++ {
    go func(n int) {
        fmt.Println(n)
    }(i)
}
```

---

## 3. Channels: lo básico

Un channel es un conducto tipado para pasar valores entre goroutines, con sincronización incluida.

```go
ch := make(chan int)       // channel sin buffer
ch2 := make(chan int, 10)  // channel con buffer de tamaño 10
```

### Sin buffer (síncrono)

Un envío `ch <- valor` **bloquea** hasta que otra goroutine hace `<-ch`. Es una cita a ciegas: ambos lados se sincronizan en ese instante exacto.

```go
func main() {
    ch := make(chan string)

    go func() {
        ch <- "listo" // bloquea hasta que alguien reciba
    }()

    msg := <-ch // bloquea hasta que alguien envíe
    fmt.Println(msg)
}
```

### Con buffer (asíncrono hasta cierto punto)

Un envío a un channel con buffer solo bloquea si el buffer está lleno. Es útil cuando quieres desacoplar productor y consumidor, o absorber ráfagas de eventos (muy común en tiempo real: picos de mensajes).

```go
ch := make(chan int, 3)
ch <- 1 // no bloquea
ch <- 2 // no bloquea
ch <- 3 // no bloquea
ch <- 4 // BLOQUEA: el buffer está lleno
```

### Cerrar channels

```go
close(ch)
```

- Cerrar indica "no voy a enviar nada más".
- Recibir de un channel cerrado devuelve inmediatamente el valor zero, y puedes detectarlo:

```go
valor, ok := <-ch
if !ok {
    fmt.Println("channel cerrado")
}
```

- `for valor := range ch` itera hasta que el channel se cierra — patrón muy común para consumir un stream de eventos.
- **Regla de oro:** solo el emisor debe cerrar el channel, nunca el receptor. Enviar a un channel cerrado causa panic.

### Direccionalidad (buena práctica en firmas de función)

```go
func productor(salida chan<- int) { salida <- 1 }   // solo puede enviar
func consumidor(entrada <-chan int) { <-entrada }    // solo puede recibir
```

Esto documenta la intención y el compilador te protege de usarlo mal.

---

## 4. `select`: el corazón de la concurrencia reactiva

`select` es como un `switch` pero para operaciones de channel. Espera a que **cualquiera** de varios casos esté listo, y si varios lo están, elige uno al azar. Es la pieza clave para sistemas en tiempo real, porque te permite reaccionar al primer evento que llegue de varias fuentes.

```go
select {
case msg := <-canalMensajes:
    fmt.Println("nuevo mensaje:", msg)
case <-canalDesconexion:
    fmt.Println("cliente desconectado")
    return
case <-time.After(30 * time.Second):
    fmt.Println("timeout: no hubo actividad en 30s")
}
```

### `select` no bloqueante con `default`

```go
select {
case msg := <-ch:
    procesar(msg)
default:
    // no hay nada listo, seguir sin bloquear
}
```

Útil para "revisar si hay algo nuevo" sin detener el loop principal — común en game loops o procesamiento de eventos donde no puedes darte el lujo de esperar.

### `select` en loop infinito (patrón típico de un "actor")

Este es probablemente el patrón más importante para tiempo real: una goroutine que vive indefinidamente reaccionando a eventos de distintas fuentes.

```go
func (c *Cliente) escuchar() {
    for {
        select {
        case msg := <-c.entrante:
            c.enviarAlSocket(msg)
        case <-c.contextoCierre.Done():
            return // salida limpia
        case <-time.After(60 * time.Second):
            c.enviarPing()
        }
    }
}
```

> [!warning] `time.After` dentro de un `for`/`select`
> Cada iteración crea un temporizador nuevo. Antes de **Go 1.23**, los temporizadores no seleccionados no se liberaban hasta expirar, lo que en loops muy activos generaba consumo de memoria. Desde Go 1.23 el recolector los libera, pero en código que deba correr en versiones anteriores, o para reiniciar el temporizador solo cuando toca, usa `time.NewTimer` + `Reset`, o un `time.Ticker` como en el ejemplo del `Hub` (sección 8).

---

## 5. El paquete `sync`: WaitGroup, Mutex, Once (cuándo SÍ usar locks)

Los channels no siempre son la herramienta correcta. Para proteger estado compartido simple (un contador, un mapa de sesiones activas), un `sync.Mutex` suele ser más simple y eficiente que un channel.

### WaitGroup: esperar a que N goroutines terminen

```go
var wg sync.WaitGroup

for _, cliente := range clientes {
    wg.Add(1)
    go func(c *Cliente) {
        defer wg.Done()
        c.enviarNotificacion(payload)
    }(cliente)
}

wg.Wait() // bloquea hasta que todas terminen
```

Patrón clásico para hacer "broadcast" a N conexiones en paralelo y esperar a que todas reciban el mensaje.

### Mutex: proteger estado compartido

```go
type Hub struct {
    mu       sync.Mutex
    clientes map[string]*Cliente
}

func (h *Hub) Agregar(c *Cliente) {
    h.mu.Lock()
    defer h.mu.Unlock()
    h.clientes[c.ID] = c
}

func (h *Hub) Contar() int {
    h.mu.Lock()
    defer h.mu.Unlock()
    return len(h.clientes)
}
```

**Regla práctica:** si el dato es compartido y de vida corta en una operación (leer/escribir un mapa, incrementar un contador), usa `Mutex`. Si estás modelando un _flujo_ de eventos o mensajes entre componentes independientes, usa `channels`.

### sync.Once: inicialización única

```go
var once sync.Once
var conexionDB *sql.DB

func obtenerDB() *sql.DB {
    once.Do(func() {
        conexionDB = conectar()
    })
    return conexionDB
}
```

### sync.RWMutex: muchas lecturas, pocas escrituras

Cuando el mapa de clientes se **lee** mucho más de lo que se **escribe** (ej. buscar a quién enviar un mensaje), `RWMutex` permite lecturas concurrentes y solo bloquea en exclusiva al escribir:

```go
func (h *Hub) Obtener(id string) (*Cliente, bool) {
    h.mu.RLock()         // varios lectores a la vez
    defer h.mu.RUnlock()
    c, ok := h.clientes[id]
    return c, ok
}
```

### errgroup: WaitGroup con propagación de errores y cancelación

El paquete `golang.org/x/sync/errgroup` es un `WaitGroup` que además **devuelve el primer error** y **cancela el contexto** de las demás goroutines cuando una falla. Es lo que se usa en la práctica para "lanza N tareas y falla rápido si alguna falla":

```go
g, ctx := errgroup.WithContext(ctx)
for _, url := range urls {
    g.Go(func() error {
        return descargar(ctx, url) // si una falla, ctx se cancela para las demás
    })
}
if err := g.Wait(); err != nil {
    log.Println("alguna descarga falló:", err)
}
```

---

## 6. `context`: cancelación y timeouts

En sistemas en tiempo real, necesitas poder decir "esta operación ya no importa, cancélala" — por ejemplo, cuando un cliente se desconecta o expira un timeout. `context.Context` es el estándar de Go para propagar cancelación a través de goroutines.

```go
func manejarConexion(ctx context.Context, conn net.Conn) {
    ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()

    canalDatos := make(chan []byte)
    go leerDelSocket(conn, canalDatos)

    select {
    case datos := <-canalDatos:
        procesar(datos)
    case <-ctx.Done():
        fmt.Println("cancelado:", ctx.Err())
        conn.Close()
    }
}
```

- `context.WithCancel(parent)` — cancelación manual.
- `context.WithTimeout(parent, d)` — cancelación automática tras `d`.
- `context.WithDeadline(parent, t)` — cancelación en un instante específico.
- Siempre llama `cancel()` (usualmente con `defer`) para liberar recursos, aunque la operación termine bien.

En un servidor real, el `ctx` normalmente nace en la conexión HTTP/WebSocket y se propaga a cada goroutine hija que trabaje para esa conexión — así cuando el cliente se va, toda la cadena se cancela en cascada.

---

## 7. Patrones de concurrencia

### 7.1 Worker Pool

Útil cuando tienes muchas tareas (ej. procesar eventos entrantes) y quieres limitar cuántas goroutines corren a la vez, evitando saturar CPU/memoria/conexiones a la DB.

```go
func iniciarWorkerPool(n int, trabajos <-chan Tarea, resultados chan<- Resultado) {
    var wg sync.WaitGroup
    for i := 0; i < n; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for t := range trabajos { // consume hasta que 'trabajos' se cierre
                resultados <- procesar(t)
            }
        }(i)
    }
    go func() {
        wg.Wait()
        close(resultados)
    }()
}
```

### 7.2 Fan-out / Fan-in

**Fan-out**: distribuir trabajo entre varias goroutines. **Fan-in**: combinar varios channels en uno solo.

```go
func fanIn(canales ...<-chan Evento) <-chan Evento {
    salida := make(chan Evento)
    var wg sync.WaitGroup

    for _, c := range canales {
        wg.Add(1)
        go func(c <-chan Evento) {
            defer wg.Done()
            for e := range c {
                salida <- e
            }
        }(c)
    }

    go func() {
        wg.Wait()
        close(salida)
    }()

    return salida
}
```

Muy usado para consolidar streams de múltiples fuentes (ej. varios sensores IoT, varios servicios de terceros) en un solo canal de eventos.

### 7.3 Pipeline

Encadenar etapas, cada una una goroutine que transforma datos y los pasa a la siguiente.

```go
func generar(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func duplicar(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * 2
        }
    }()
    return out
}

// uso: for v := range duplicar(generar(1, 2, 3)) { fmt.Println(v) }
```

### 7.4 Pub/Sub (broadcast a múltiples suscriptores)

El patrón central de cualquier chat, dashboard live, o sistema de notificaciones.

```go
type Broker struct {
    mu          sync.Mutex
    suscriptores map[chan Evento]bool
}

func NuevoBroker() *Broker {
    return &Broker{suscriptores: make(map[chan Evento]bool)}
}

func (b *Broker) Suscribir() chan Evento {
    ch := make(chan Evento, 16) // buffer para no bloquear al publicar
    b.mu.Lock()
    b.suscriptores[ch] = true
    b.mu.Unlock()
    return ch
}

func (b *Broker) Desuscribir(ch chan Evento) {
    b.mu.Lock()
    delete(b.suscriptores, ch)
    close(ch)
    b.mu.Unlock()
}

func (b *Broker) Publicar(e Evento) {
    b.mu.Lock()
    defer b.mu.Unlock()
    for ch := range b.suscriptores {
        select {
        case ch <- e:
        default:
            // suscriptor lento: descartamos el evento en vez de bloquear a todos
        }
    }
}
```

Nota el `select` con `default` en `Publicar`: es una decisión de diseño crítica en tiempo real. Si un cliente es lento (su channel está lleno), **no puedes dejar que bloquee el broadcast para todos los demás**. Aquí eliges descartar el evento para ese cliente — otras estrategias son desconectarlo, o usar un buffer más grande.

### 7.5 Rate limiting con `time.Ticker`

```go
limitador := time.NewTicker(100 * time.Millisecond)
defer limitador.Stop()

for evento := range eventosEntrantes {
    <-limitador.C // espera el siguiente "tick" antes de procesar
    procesar(evento)
}
```

---

## 8. Caso completo: servidor de chat en tiempo real

Este ejemplo junta todo: un `Hub` central con goroutine propia, channels para registro/desregistro/broadcast, y una goroutine por cliente para lectura y escritura (patrón estándar en servidores WebSocket con `gorilla/websocket` o `nhooyr.io/websocket`).

```go
package main

import (
    "context"
    "log"
    "time"
)

type Mensaje struct {
    De      string
    Texto   string
}

type Cliente struct {
    ID      string
    enviar  chan Mensaje // buffer para no bloquear al Hub
}

type Hub struct {
    registrar    chan *Cliente
    desregistrar chan *Cliente
    broadcast    chan Mensaje
    clientes     map[*Cliente]bool
}

func NuevoHub() *Hub {
    return &Hub{
        registrar:    make(chan *Cliente),
        desregistrar: make(chan *Cliente),
        broadcast:    make(chan Mensaje, 256),
        clientes:     make(map[*Cliente]bool),
    }
}

// Run es el ÚNICO lugar que toca el mapa 'clientes'.
// Al centralizar el acceso en una goroutine, evitamos necesitar un Mutex.
func (h *Hub) Run(ctx context.Context) {
    for {
        select {
        case c := <-h.registrar:
            h.clientes[c] = true
            log.Printf("cliente conectado: %s (total: %d)", c.ID, len(h.clientes))

        case c := <-h.desregistrar:
            if _, ok := h.clientes[c]; ok {
                delete(h.clientes, c)
                close(c.enviar)
            }

        case msg := <-h.broadcast:
            for c := range h.clientes {
                select {
                case c.enviar <- msg:
                default:
                    // cliente lento: lo desconectamos en vez de bloquear el Hub
                    delete(h.clientes, c)
                    close(c.enviar)
                }
            }

        case <-ctx.Done():
            log.Println("hub cerrado")
            return
        }
    }
}

// escribirACliente es la goroutine dedicada a enviar mensajes a UN cliente.
// Separar lectura/escritura en goroutines distintas por conexión es el
// patrón estándar en servidores WebSocket.
func escribirACliente(c *Cliente) {
    ticker := time.NewTicker(30 * time.Second) // heartbeat/ping
    defer ticker.Stop()

    for {
        select {
        case msg, ok := <-c.enviar:
            if !ok {
                return // el Hub cerró el channel: cliente desconectado
            }
            enviarPorSocket(c, msg)

        case <-ticker.C:
            enviarPing(c)
        }
    }
}

func enviarPorSocket(c *Cliente, m Mensaje) { /* escribir al websocket real */ }
func enviarPing(c *Cliente)                 { /* frame de ping */ }
```

**Por qué este diseño es idiomático:**

- El `Hub.Run` corre en **una sola goroutine**, así que el mapa `clientes` nunca se accede concurrentemente — cero necesidad de Mutex ahí.
- Cada cliente tiene su propio channel con buffer (`enviar`), así un cliente lento no frena a los demás.
- El `select` con `default` al hacer broadcast implementa "descarta o desconecta" en vez de bloquear.
- `context` permite apagar todo el sistema de forma ordenada (ej. en un graceful shutdown del servidor).

---

## 9. Errores comunes y cómo evitarlos

|Error|Síntoma|Solución|
|---|---|---|
|**Goroutine leak**|Una goroutine bloqueada para siempre esperando un channel que nadie va a llenar/cerrar|Siempre ten una vía de salida (`ctx.Done()`, `close`, timeout)|
|**Deadlock**|`fatal error: all goroutines are asleep`|Revisa que cada `<-ch` tenga un `ch <-` correspondiente en otra goroutine|
|**Enviar a channel cerrado**|panic: `send on closed channel`|Solo el emisor cierra; nunca cierres desde el receptor|
|**Cerrar un channel dos veces**|panic: `close of closed channel`|Usa `sync.Once` si varias rutas de código podrían cerrar el mismo channel|
|**Data race**|Bugs intermitentes, valores corruptos|Corre con `go run -race`; protege estado compartido con Mutex o pásalo por channel|
|**Bloquear el broadcast por un cliente lento**|Todo el sistema se congela cuando un cliente no lee rápido|`select` con `default`, o desconectar clientes lentos|
|**Usar channels donde un Mutex era más simple**|Código difícil de leer para algo trivial|Si es solo "proteger una variable", usa Mutex|
|**Capturar variable de loop mal (pre Go 1.22)**|Todas las goroutines ven el mismo valor final|Pasa la variable como parámetro a la goroutine, o actualiza a Go 1.22+|

**Herramienta indispensable: el race detector.**

```bash
go run -race main.go
go test -race ./...
```

Ejecuta tu código con `-race` durante desarrollo y en CI. Detecta accesos concurrentes no sincronizados que de otra forma solo aparecerían como bugs esporádicos en producción, justo el tipo de bug más caro de encontrar en sistemas de tiempo real.

---

## 10. Checklist mental para diseñar sistemas concurrentes

Cuando diseñes un componente en tiempo real, pregúntate:

1. **¿Qué goroutines viven y por cuánto tiempo?** (por conexión, por request, para siempre)
2. **¿Cómo se cancelan limpiamente?** (usa `context` desde el día uno)
3. **¿Este dato es un flujo de eventos o estado compartido?** → flujo = channel, estado = Mutex
4. **¿Qué pasa si un consumidor es lento?** → define la estrategia: buffer, descartar, o desconectar
5. **¿Quién cierra cada channel, y una sola vez?**
6. **¿Corriste con `-race`?**

---

### Para seguir practicando

- Implementa un **rate limiter distribuido** con `time.Ticker` + buffer channel (token bucket).
- Construye un **worker pool con backpressure** que rechace tareas si la cola está llena.
- Agrega **reconexión automática** al cliente del chat de arriba usando `context.WithTimeout` y retry con backoff exponencial.