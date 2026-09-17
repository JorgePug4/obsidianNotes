---
tags: [microservicios, patrones, resiliencia, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
---

# Circuit Breaker (Cortacircuitos)

> [!info] Categoría: **Resiliencia**. Patrón popularizado por Michael Nygard en *Release It!* (2007) y por Martin Fowler.

## ¿Qué es?

El **Circuit Breaker** es un patrón que **protege a un servicio que llama a otro** (o a una base de datos, una API externa, etc.) cuando ese destino está fallando o respondiendo demasiado lento.

Funciona igual que el interruptor eléctrico de una casa: si detecta demasiados fallos, **"abre el circuito"** y deja de enviar peticiones al servicio dañado durante un tiempo, en lugar de seguir insistiendo y empeorar la situación.

## ¿Para qué sirve?

- **Evitar fallos en cascada**: si el servicio B está caído y A sigue llamándolo, A acumula hilos bloqueados esperando timeouts, se queda sin recursos y también cae. Después cae C, que dependía de A.
- **Fallar rápido (*fail fast*)**: mientras el circuito está abierto, la llamada falla en microsegundos en vez de esperar 30 segundos de timeout.
- **Dar tiempo de recuperación** al servicio degradado, que no recibe tráfico mientras se recupera.
- **Permitir respuestas alternativas (*fallback*)**: devolver datos de caché, un valor por defecto o un mensaje "intenta más tarde".

## Conceptos relacionados

- [[Retry con Backoff Exponencial]] → se combina con el Circuit Breaker, pero **nunca debe reintentar contra un circuito abierto**.
- [[Bulkhead]] → aísla recursos; el Circuit Breaker corta el flujo. Juntos forman la base de la resiliencia.
- [[🏗️ Diseño de Microservicios]] → visión general de resiliencia y tolerancia a fallos.
- [[API Gateway]] → lugar habitual para aplicar Circuit Breakers perimetrales.
- [[Idempotencia]] → necesaria para que los reintentos tras cerrar el circuito sean seguros.
- [[04 - Azure Load Balancer|Azure Load Balancer]] → los *health probes* de un balanceador cumplen una función parecida a nivel de infraestructura.

## ¿Cómo funciona?

El patrón es una **máquina de estados** con tres estados:

```
            fallos > umbral
 🟢 CERRADO ─────────────────► 🔴 ABIERTO
    ▲                              │
    │ prueba OK                    │ pasa el tiempo de espera
    │                              ▼
    └────────────────────── 🟡 MEDIO ABIERTO
              prueba falla ──────► vuelve a 🔴 ABIERTO
```

| Estado | Qué ocurre | Cuándo cambia |
|---|---|---|
| 🟢 **Cerrado** (*Closed*) | Las peticiones fluyen con normalidad. Se cuentan los fallos. | Si los fallos superan el umbral (ej. 50 % de errores en 30 s, o 5 fallos seguidos) → **Abierto**. |
| 🔴 **Abierto** (*Open*) | Toda petición falla inmediatamente con una excepción (`BrokenCircuitException` en Polly) **sin llamar al destino**. | Cuando pasa el tiempo de espera (*break duration*, ej. 30 s) → **Medio abierto**. |
| 🟡 **Medio abierto** (*Half-Open*) | Deja pasar **una o pocas peticiones de prueba**. | Si la prueba tiene éxito → **Cerrado**. Si falla → **Abierto** otra vez. |

> [!important] El estado Abierto es el corazón del patrón
> No es un error, es una **decisión deliberada** de no llamar. La aplicación debe manejar esa excepción con un *fallback*, no dejarla explotar.

### Parámetros típicos de configuración

- **Umbral de fallos**: número o porcentaje de errores que abre el circuito.
- **Ventana de muestreo**: periodo en el que se cuentan los fallos (ej. últimos 30 s).
- **Mínimo de peticiones** (*minimum throughput*): evita abrir el circuito con 1 fallo de 1 petición.
- **Duración de la apertura** (*break duration*): cuánto tiempo permanece abierto.
- **Qué se considera fallo**: excepciones, códigos HTTP 5xx, timeouts. Un `404` normalmente **no** debería contar como fallo.

## Ejemplo

### En .NET con Polly v8 (`Microsoft.Extensions.Http.Resilience`)

Desde .NET 8, la forma recomendada de aplicar resiliencia a `HttpClient` es el paquete `Microsoft.Extensions.Http.Resilience`, construido sobre Polly v8:

```csharp
builder.Services.AddHttpClient("catalogo", c =>
    {
        c.BaseAddress = new Uri("https://catalogo.internal");
    })
    .AddResilienceHandler("catalogo-pipeline", pipeline =>
    {
        pipeline.AddCircuitBreaker(new HttpCircuitBreakerStrategyOptions
        {
            FailureRatio = 0.5,                          // 50 % de fallos...
            SamplingDuration = TimeSpan.FromSeconds(30), // ...en 30 segundos
            MinimumThroughput = 10,                      // con al menos 10 peticiones
            BreakDuration = TimeSpan.FromSeconds(20)     // abierto durante 20 s
        });
        pipeline.AddTimeout(TimeSpan.FromSeconds(5));
    });
```

Y en el consumidor, manejar el circuito abierto con un *fallback*:

```csharp
try
{
    var productos = await _client.GetFromJsonAsync<List<Producto>>("/productos");
    return productos;
}
catch (BrokenCircuitException)
{
    // El circuito está abierto: devolvemos la última copia en caché
    return _cache.Get<List<Producto>>("productos") ?? [];
}
```

> [!tip] Atajo
> `AddStandardResilienceHandler()` configura de golpe una pipeline estándar: *rate limiter*, timeout total, retry, circuit breaker y timeout por intento.

### Fuera de .NET

- **Java**: Resilience4j (Hystrix de Netflix está descontinuado desde 2018).
- **Go**: `sony/gobreaker`, `afex/hystrix-go`.
- **Infraestructura**: Istio y Envoy implementan *outlier detection*, que es un Circuit Breaker a nivel de malla de servicios (*service mesh*).

## Ventajas

- Evita que un fallo local se convierta en una caída global.
- Reduce latencia en escenarios de fallo (*fail fast*).
- Protege al servicio degradado de una avalancha de reintentos.
- Da visibilidad: el cambio de estado es un evento que se puede monitorizar y alertar.

## Desventajas / Limitaciones

- **Configuración delicada**: umbrales mal ajustados abren el circuito por fallos normales, o nunca lo abren.
- **Falsos positivos**: un `4xx` contado como fallo puede abrir el circuito sin que el servicio esté caído.
- **Estado local**: cada instancia del servicio tiene su propio circuito. Con 10 réplicas puede haber 10 estados distintos (existen implementaciones distribuidas, pero añaden complejidad).
- **Necesita un fallback pensado**: sin él, sólo cambias un timeout lento por un error rápido.

## Comparación

| Patrón | Qué hace | Cuándo actúa |
|---|---|---|
| **Circuit Breaker** | Deja de llamar a un destino que falla | Después de detectar fallos repetidos |
| **[[Retry con Backoff Exponencial\|Retry]]** | Vuelve a intentar una llamada fallida | En cada fallo individual |
| **[[Bulkhead]]** | Limita cuántas llamadas concurrentes van a un destino | Siempre, de forma preventiva |
| **Timeout** | Corta una llamada que tarda demasiado | Por cada llamada |

> [!warning] Orden correcto de la pipeline
> El orden habitual, de fuera hacia dentro: **Timeout total → Retry → Circuit Breaker → Timeout por intento**. Si pones el Retry *dentro* del Circuit Breaker, los reintentos contarán como fallos y abrirán el circuito prematuramente.

## Puntos clave

- Tres estados: **Cerrado → Abierto → Medio abierto**.
- Su objetivo es **evitar fallos en cascada**, no reintentar.
- Abierto = **fallar rápido sin llamar** al destino.
- Siempre necesita un **fallback** y buena **observabilidad**.
- En .NET moderno: **Polly v8** vía `Microsoft.Extensions.Http.Resilience`.

## Errores comunes

- Contar `4xx` (errores del cliente) como fallos del servicio.
- No definir `MinimumThroughput` y abrir el circuito con la primera petición fallida del día.
- Reintentar dentro del circuito abierto (el Retry debe estar fuera).
- Usar un solo Circuit Breaker compartido para todas las dependencias: si cae una API externa, se corta también la base de datos.
- No monitorizar los cambios de estado: el circuito se abre y nadie se entera.

## 🎯 Para entrevistas y exámenes

- Pregunta típica: *"¿Qué diferencia hay entre Retry y Circuit Breaker?"* → Retry insiste; Circuit Breaker deja de insistir.
- *"¿Qué pasa en el estado Half-Open?"* → Se permiten peticiones de prueba para decidir si cerrar o reabrir.
- *"¿Por qué no basta con un timeout?"* → El timeout protege una llamada; el Circuit Breaker protege al sistema de muchas llamadas condenadas al fallo.
- Saber nombrar una librería: **Polly** (.NET), **Resilience4j** (Java).

## Referencias

- Martin Fowler, *CircuitBreaker*: https://martinfowler.com/bliki/CircuitBreaker.html
- Microsoft Learn, *Circuit Breaker pattern*: https://learn.microsoft.com/azure/architecture/patterns/circuit-breaker
- Documentación de Polly: https://www.pollydocs.org/

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
