---
tags: [microservicios, patrones, resiliencia, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
---

# Retry con Backoff Exponencial

> [!info] Categoría: **Resiliencia**. El patrón más simple y, a la vez, el más peligroso si se aplica mal.

## ¿Qué es?

**Retry** consiste en volver a intentar una operación que falló. **Backoff exponencial** significa que la espera entre intentos **crece de forma exponencial** (1 s, 2 s, 4 s, 8 s...). Se le suele añadir **jitter** (una variación aleatoria) para que muchos clientes no reintenten al mismo tiempo.

## ¿Para qué sirve?

Muchos fallos en sistemas distribuidos son **transitorios**: un pico de red, un reinicio de pod, un *failover* de base de datos que dura medio segundo. Reintentar resuelve estos casos sin que el usuario se entere.

El backoff evita el efecto **manada atronadora (*thundering herd*)**: si 10 000 clientes reintentan a la vez cada segundo contra un servicio que acaba de reiniciarse, lo tumban de nuevo antes de que arranque.

## Conceptos relacionados

- [[Idempotencia]] → **requisito previo**: solo es seguro reintentar operaciones idempotentes.
- [[Circuit Breaker]] → cuando los reintentos no bastan, hay que dejar de llamar.
- [[Bulkhead]] → limita cuántos reintentos concurrentes pueden consumir recursos.
- [[Transactional Outbox]] → el *worker* que publica mensajes reintenta con backoff.
- [[Saga Pattern]] → las transacciones compensatorias suelen ejecutarse con reintentos.
- [[Status Code|Códigos de estado HTTP]] → qué códigos merecen reintento y cuáles no.

## ¿Cómo funciona?

```
Intento 1 ──falla──► espera 1 s (+ jitter)
Intento 2 ──falla──► espera 2 s (+ jitter)
Intento 3 ──falla──► espera 4 s (+ jitter)
Intento 4 ──falla──► se rinde → excepción / fallback / cola de mensajes muertos
```

Fórmula habitual:

```
espera = min(base × 2^intento + aleatorio(0, jitter), espera_máxima)
```

### Decidir qué se reintenta

| Situación | ¿Reintentar? | Por qué |
|---|---|---|
| Timeout, conexión rechazada | ✅ Sí | Probablemente transitorio |
| HTTP `503`, `502`, `504`, `429` | ✅ Sí (respetando `Retry-After`) | El servidor pide paciencia |
| HTTP `500` | ⚠️ Depende | Puede ser un bug determinista: reintentar no lo arregla |
| HTTP `400`, `401`, `403`, `404`, `422` | ❌ No | Error del cliente: la petición seguirá siendo inválida |
| Operación no idempotente sin protección | ❌ No | Riesgo de duplicar el efecto (dos cobros) |

## Ejemplo

### En .NET con Polly v8

```csharp
builder.Services.AddHttpClient("inventario")
    .AddResilienceHandler("inventario-pipeline", pipeline =>
    {
        pipeline.AddRetry(new HttpRetryStrategyOptions
        {
            MaxRetryAttempts = 3,
            Delay = TimeSpan.FromSeconds(1),
            BackoffType = DelayBackoffType.Exponential,
            UseJitter = true,
            // Por defecto reintenta 5xx, 408, 429 y HttpRequestException
        });
    });
```

### Reintento en un consumidor de mensajes

Cuando el reintento se agota, el mensaje no debe perderse: se envía a una **cola de mensajes muertos** (*Dead Letter Queue*, DLQ) para análisis o reproceso manual. Azure Service Bus, RabbitMQ y Kafka (con un *topic* dedicado) soportan este mecanismo.

## Ventajas

- Resuelve la mayoría de fallos transitorios de forma transparente.
- Muy fácil de implementar con librerías.
- El backoff con jitter reparte la carga y protege al servicio que se recupera.

## Desventajas / Limitaciones

- **Amplifica la carga** sobre un servicio ya degradado si no hay backoff ni Circuit Breaker.
- **Aumenta la latencia** percibida: 3 reintentos con backoff pueden sumar varios segundos.
- Reintentar operaciones **no idempotentes** produce duplicados (el error clásico: cobrar dos veces).
- **Reintentos anidados**: si A reintenta 3 veces a B, y B reintenta 3 veces a C, C recibe hasta 9 llamadas por una petición original.

## Comparación

| Estrategia | Espera entre intentos | Cuándo usarla |
|---|---|---|
| **Inmediato** | 0 | Solo para fallos ultra breves (colisión optimista en BD) |
| **Lineal** | 1 s, 2 s, 3 s | Cargas pequeñas y predecibles |
| **Exponencial** | 1 s, 2 s, 4 s, 8 s | Estándar para llamadas de red |
| **Exponencial + jitter** | 1.3 s, 2.7 s, 3.9 s... | **Recomendado** en producción con muchos clientes |

## Puntos clave

- Reintenta solo **fallos transitorios** y **operaciones idempotentes**.
- Usa **backoff exponencial con jitter** y un **máximo de intentos**.
- Respeta la cabecera `Retry-After`.
- Combínalo con **[[Circuit Breaker]]** para no insistir sobre un servicio caído.
- Lo que no se puede reintentar más va a una **DLQ**, no a la basura.

## Errores comunes

- Reintentar `4xx`.
- Reintentar sin límite ("hasta que funcione").
- Reintentar sin jitter con miles de clientes sincronizados.
- Reintentar un `POST` de pago sin clave de idempotencia.
- Poner el Retry *dentro* del Circuit Breaker, de modo que cada reintento cuenta como fallo.

## 🎯 Para entrevistas y exámenes

- *"¿Qué es el jitter y por qué importa?"* → Aleatoriedad en la espera para evitar que todos reintenten a la vez.
- *"¿Qué condición debe cumplir una operación para reintentarla con seguridad?"* → Ser idempotente.
- *"¿Qué haces con un mensaje que agotó sus reintentos?"* → Dead Letter Queue.

## Referencias

- Microsoft Learn, *Retry pattern*: https://learn.microsoft.com/azure/architecture/patterns/retry
- AWS Architecture Blog, *Exponential Backoff And Jitter*: https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- Polly, *Retry strategy*: https://www.pollydocs.org/strategies/retry

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
