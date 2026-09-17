---
tags: [microservicios, patrones, resiliencia, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
---

# Bulkhead (Compartimentos estancos)

> [!info] Categoría: **Resiliencia**. El nombre viene de los mamparos de un barco: si una sección se inunda, las demás siguen a flote.

## ¿Qué es?

El **Bulkhead** es un patrón que **aísla los recursos** (hilos, conexiones, memoria, instancias) que usa cada dependencia o cada tipo de carga, de modo que el agotamiento de un compartimento **no arrastre al resto** del sistema.

## ¿Para qué sirve?

Imagina un servicio de pedidos que llama a dos dependencias: el servicio de inventario (rápido) y una pasarela de pagos externa (a veces lenta). Sin Bulkhead, si la pasarela empieza a tardar 30 segundos por llamada, todas las conexiones y hilos disponibles se quedan esperando a pagos y **las consultas de inventario también dejan de responder**, aunque inventario esté perfectamente sano.

Con Bulkhead, pagos tiene un máximo de, por ejemplo, 20 llamadas concurrentes. Cuando se agotan, las nuevas llamadas a pagos se rechazan de inmediato, pero inventario sigue funcionando con sus propios recursos.

## Conceptos relacionados

- [[Circuit Breaker]] → corta el flujo cuando detecta fallos; Bulkhead limita el flujo de forma preventiva.
- [[Retry con Backoff Exponencial]] → los reintentos consumen recursos; el Bulkhead pone un techo.
- [[🏗️ Diseño de Microservicios]] → sección de resiliencia.
- [[Go routines|Goroutines y Channels]] → el patrón *Worker Pool* de Go es una forma de Bulkhead: limita cuántas goroutines trabajan a la vez.

## ¿Cómo funciona?

Existen dos niveles de aplicación:

### 1. Bulkhead a nivel de código (concurrencia)

Se asigna a cada dependencia un **límite de ejecuciones concurrentes** y, opcionalmente, una **cola de espera** acotada.

```
Peticiones ──► [ Bulkhead pagos: máx 20 activas, cola 10 ] ──► Pasarela de pagos
Peticiones ──► [ Bulkhead inventario: máx 50 activas     ] ──► Servicio inventario
```

Cuando el compartimento está lleno y la cola también, la petición se rechaza con una excepción (`RateLimiterRejectedException` en Polly v8) que la aplicación debe manejar.

### 2. Bulkhead a nivel de infraestructura

- **Pools de conexiones separados** por base de datos o dependencia.
- **Instancias o pods dedicados** para cargas críticas y no críticas (ej. un *deployment* de Kubernetes para la API pública y otro para procesos batch).
- **Colas separadas** para tipos de mensaje con distinta prioridad.
- **Límites de CPU y memoria** por contenedor (*resource limits* en Kubernetes).

## Ejemplo

### En .NET con Polly v8

En Polly v8 el antiguo `BulkheadPolicy` fue reemplazado por la estrategia de **concurrency limiter**, basada en `System.Threading.RateLimiting`:

```csharp
builder.Services.AddHttpClient("pagos")
    .AddResilienceHandler("pagos-pipeline", pipeline =>
    {
        pipeline.AddConcurrencyLimiter(
            permitLimit: 20,   // máximo de llamadas simultáneas
            queueLimit: 10);   // cuántas esperan turno antes de rechazar
    });
```

Manejo del rechazo:

```csharp
try
{
    return await _pagosClient.PostAsJsonAsync("/cobros", cobro);
}
catch (RateLimiterRejectedException)
{
    // Compartimento lleno: encolar para procesar después o responder 503
    await _colaReintentos.EncolarAsync(cobro);
    throw new ServicioSaturadoException("Pagos saturado, se procesará en breve");
}
```

### En Go

Un `chan struct{}` con buffer actúa como semáforo, o directamente un *Worker Pool* de tamaño fijo (ver [[Go routines]]).

## Ventajas

- Un fallo o lentitud en una dependencia **queda contenido**.
- Permite priorizar: la funcionalidad crítica recibe recursos garantizados.
- Falla de forma **predecible** (rechazo inmediato) en lugar de degradarse lentamente.
- Facilita dimensionar: sabes cuántas llamadas concurrentes puede recibir cada dependencia.

## Desventajas / Limitaciones

- **Uso menos eficiente de recursos**: un compartimento puede estar ocioso mientras otro rechaza peticiones.
- Añade **complejidad de configuración**: hay que elegir tamaños para cada compartimento.
- Si el límite es demasiado bajo, rechazas tráfico legítimo en horas pico.

## Comparación

| Patrón | Pregunta que responde |
|---|---|
| **Bulkhead** | ¿Cuántas llamadas simultáneas permito a esta dependencia? |
| **[[Circuit Breaker]]** | ¿Debo dejar de llamar a esta dependencia porque está fallando? |
| **Rate Limiting** | ¿Cuántas peticiones por segundo acepto de este cliente? |
| **Timeout** | ¿Cuánto espero una respuesta antes de rendirme? |

> [!tip] Bulkhead y Circuit Breaker se complementan
> Bulkhead evita que una dependencia lenta consuma todo; Circuit Breaker evita seguir llamando a una dependencia rota. Un sistema resiliente usa ambos.

## Puntos clave

- Aísla recursos por dependencia o tipo de carga.
- Es **preventivo**: actúa siempre, no solo cuando hay fallos.
- En Polly v8 se implementa con `AddConcurrencyLimiter`.
- También aplica a infraestructura: pools, pods, colas, límites de contenedor.

## Errores comunes

- Un único pool de conexiones o de hilos compartido por todas las dependencias.
- Dimensionar los compartimentos "a ojo" sin medir la carga real.
- No manejar la excepción de rechazo y devolver un `500` genérico en lugar de un `503` con `Retry-After`.

## 🎯 Para entrevistas y exámenes

- *"¿De dónde viene el nombre?"* → De los mamparos de los barcos.
- *"Diferencia con Circuit Breaker"* → Bulkhead limita concurrencia de forma preventiva; Circuit Breaker reacciona a fallos.
- *"Ejemplo de Bulkhead en infraestructura"* → Pods dedicados por carga, pools de conexión separados.

## Referencias

- Microsoft Learn, *Bulkhead pattern*: https://learn.microsoft.com/azure/architecture/patterns/bulkhead
- Polly, *Rate limiter / concurrency limiter*: https://www.pollydocs.org/strategies/rate-limiter

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
