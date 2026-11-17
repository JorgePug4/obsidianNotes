---
tags: [cloud-native, patrones, resiliencia, polly, dotnet]
aliases: [Resiliencia cloud native, Timeout pattern, Fallback, Rate Limiting, Throttling, Load Shedding, Compensating Transaction, Scheduler Agent Supervisor, Graceful degradation]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 06 - Resiliencia y tolerancia a fallos

> [!info] Objetivo del capítulo
> Que un fallo **parcial** (una instancia, una dependencia, una zona) no se convierta en un fallo **total**. Fuentes: Michael Nygard, *Release It!*; Microsoft *Cloud Design Patterns*; documentación de Polly.

## Contenido
- [[#0. Conceptos]]
- [[#1. Timeout]]
- [[#2. Retry]]
- [[#3. Circuit Breaker]]
- [[#4. Bulkhead]]
- [[#5. Fallback y degradación elegante]]
- [[#6. Rate Limiting]]
- [[#7. Throttling]]
- [[#8. Load Shedding]]
- [[#9. Compensating Transaction]]
- [[#10. Scheduler Agent Supervisor]]
- [[#11. Redundancia: zonas y regiones]]
- [[#12. Cómo combinarlos: la pipeline de resiliencia]]
- [[#13. Verificar la resiliencia: Chaos Engineering]]

---

## 0. Conceptos

| Concepto | Definición |
|---|---|
| **Fallo transitorio** | Temporal y que se resuelve solo (pérdida de paquete, `503` momentáneo, *throttling*). Reintentar tiene sentido |
| **Fallo permanente** | No se resuelve reintentando (`400`, `404`, `401`, bug). Reintentar sólo añade carga |
| **Fallo en cascada** | Un componente lento/caído agota los recursos de quienes lo llaman, que a su vez caen |
| **Fallo parcial** | Parte del sistema funciona y parte no — **el estado normal** de un sistema distribuido |
| **Alta disponibilidad** | Seguir operando ante fallos de componentes (redundancia en una región) |
| **Recuperación ante desastres (DR)** | Recuperarse de la pérdida de una región completa. Se mide con **RTO** (cuánto tardo en volver) y **RPO** (cuántos datos puedo perder) |

> [!important] Lo lento es peor que lo caído
> Un servicio caído falla rápido. Un servicio **lento** retiene hilos, conexiones y memoria de quien lo llama durante segundos. La mayoría de los fallos en cascada empiezan por **latencia**, no por errores. Por eso el primer patrón es el **timeout**.

---

## 1. Timeout

**Problema.** Sin límite de tiempo, una llamada a una dependencia lenta puede esperar indefinidamente (el `HttpClient` de .NET tiene **100 s** por defecto), acumulando recursos bloqueados.

**Solución.** Poner un **tiempo máximo** a cada operación remota y tratar su vencimiento como un fallo.

**Cómo elegir el valor:** a partir de la **latencia observada** de la dependencia (p. ej. un poco por encima del p99), no "a ojo". Y con **presupuesto de tiempo** (*deadline propagation*): si el cliente sólo espera 2 s, no tiene sentido que un servicio interno espere 10 s. gRPC propaga *deadlines* de forma nativa.

Dos timeouts en la pipeline:
- **Por intento**: limita cada llamada individual.
- **Total**: limita la operación completa, incluidos los reintentos.

Propaga siempre el `CancellationToken` en .NET: si el cliente cancela, deja de trabajar.

---

## 2. Retry

> Nota detallada: [[Retry con Backoff Exponencial]].

**Problema.** Los fallos transitorios son frecuentes en la nube; fallar al primer error degrada la experiencia sin necesidad.

**Solución.** Reintentar **sólo fallos transitorios**, un número **limitado** de veces, con **backoff exponencial** (1 s, 2 s, 4 s…) y **jitter** (aleatoriedad) para que miles de clientes no reintenten sincronizados.

**Reglas:**
- Sólo operaciones **idempotentes** (o con *idempotency key*). Un `POST /pagos` reintentado sin protección puede cobrar dos veces. Ver [[Idempotencia]].
- Respeta `Retry-After` en `429` y `503`.
- **Reintenta en una sola capa.** Si A→B→C y cada capa reintenta 3 veces, un fallo de C genera 3 × 3 × 3 = 27 llamadas (*retry storm*).
- Considera un **retry budget**: no más de un X % de las peticiones pueden ser reintentos (lo implementan Envoy/Finagle; en Polly se aproxima con circuit breaker + rate limiter).

---

## 3. Circuit Breaker

> Nota detallada: [[Circuit Breaker]].

**Problema.** Seguir llamando a una dependencia que está fallando desperdicia recursos, aumenta la latencia y le impide recuperarse.

**Solución.** Máquina de estados **Cerrado → Abierto → Medio abierto**. Al superar un umbral de fallos se abre y las llamadas **fallan inmediatamente** sin llegar al destino; tras un tiempo, deja pasar peticiones de prueba.

**Retry vs Circuit Breaker en una frase:** Retry asume que el siguiente intento puede salir bien; Circuit Breaker asume que no, y deja de intentarlo.

---

## 4. Bulkhead

> Nota detallada: [[Bulkhead]].

**Problema.** Si todas las llamadas comparten el mismo pool (hilos, conexiones, memoria), una dependencia lenta lo agota y deja sin recursos a las demás.

**Solución.** **Compartimentar** recursos, como los mamparos estancos de un barco: un pool/límite de concurrencia por dependencia, por tipo de cliente o por tipo de carga. Si un compartimento se inunda, el resto sigue a flote.

Niveles: concurrencia en el código (`ConcurrencyLimiter` de Polly), pools de conexiones separados, **instancias o node pools separados** (p. ej. un Deployment distinto para la API pública y para los procesos batch), incluso **clústeres o stamps** separados (ver [[CN 07 - Escalabilidad y rendimiento]]).

---

## 5. Fallback y degradación elegante

**Problema.** Cuando una dependencia no responde, devolver un error 500 puede ser peor que dar una respuesta parcial.

**Solución.** Definir **de antemano** qué hacer cuando algo falla:

| Estrategia | Ejemplo |
|---|---|
| Datos de caché (aunque estén algo obsoletos) | Mostrar el catálogo de hace 5 minutos |
| Valor por defecto | Recomendaciones genéricas en lugar de personalizadas |
| Funcionalidad desactivada | Ocultar la sección "reseñas" si el servicio de reseñas cae |
| Encolar para después | "Hemos recibido tu pedido, te confirmaremos en breve" |
| Error claro y rápido | `503` con `Retry-After`, en lugar de colgar 30 s |

**Diseño:** clasifica las funcionalidades en **críticas** (pagar) y **no críticas** (recomendaciones). Las no críticas nunca deberían poder tumbar a las críticas. Un fallback no debe depender de lo mismo que ha fallado, y debe estar **probado** (los fallbacks que nunca se ejecutan suelen estar rotos).

---

## 6. Rate Limiting

**Problema.** Un cliente (por error, abuso o un bucle) envía demasiadas peticiones y degrada el servicio para todos; o tu servicio llama a una API externa con límites y debe respetarlos.

**Solución.** Limitar el número de peticiones por unidad de tiempo, **por clave** (usuario, IP, API key, tenant). Al superarlo: `429 Too Many Requests` con `Retry-After`.

| Algoritmo | Cómo funciona | Característica |
|---|---|---|
| **Fixed window** | N peticiones por ventana fija (por minuto) | Simple; permite ráfagas de 2N en el cambio de ventana |
| **Sliding window** | La ventana se desplaza (por segmentos) | Más suave que la fija |
| **Token bucket** | Un cubo se rellena a ritmo constante; cada petición consume un token | Permite ráfagas controladas hasta el tamaño del cubo |
| **Concurrency** | Máximo N peticiones **simultáneas** | Protege recursos, no ritmo |

```csharp
// ASP.NET Core (.NET 7+): Microsoft.AspNetCore.RateLimiting
builder.Services.AddRateLimiter(o =>
{
    o.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
    o.AddPolicy("por-usuario", ctx =>
        RateLimitPartition.GetTokenBucketLimiter(
            partitionKey: ctx.User.Identity?.Name ?? ctx.Connection.RemoteIpAddress?.ToString() ?? "anon",
            factory: _ => new TokenBucketRateLimiterOptions
            {
                TokenLimit = 100,                              // ráfaga máxima
                TokensPerPeriod = 20,
                ReplenishmentPeriod = TimeSpan.FromSeconds(1), // 20 req/s sostenidas
                QueueLimit = 0
            }));
});
app.UseRateLimiter();
app.MapGet("/api/pedidos", ObtenerPedidos).RequireRateLimiting("por-usuario");
```

> [!warning] Con varias réplicas
> El limitador de ASP.NET Core es **por instancia**. Con 10 réplicas, el límite efectivo es 10×. Para un límite global, aplícalo en el **gateway** (API Management, Envoy, NGINX) o usa un contador compartido (Redis).

---

## 7. Throttling

**Problema.** La carga supera la capacidad y escalar no es posible o no es lo bastante rápido (el autoscaling tarda minutos).

**Solución.** Controlar el consumo de recursos **por parte del propio servicio** según su estado, para mantener el SLA de lo importante. Microsoft lo describe como alternativa o complemento al autoscaling. Estrategias:
- Rechazar peticiones de un usuario/tenant que supera su cuota (aquí se solapa con Rate Limiting).
- **Degradar** funcionalidades no esenciales.
- Priorizar clientes/operaciones críticas (ver Priority Queue en [[CN 04 - Mensajería y arquitectura orientada a eventos]]).
- Diferir trabajo a una cola (Load Leveling).

**Rate Limiting vs Throttling:** los términos se usan a menudo como sinónimos. Una distinción útil: el *rate limiting* aplica **una cuota fija por cliente**; el *throttling* **reacciona a la carga del sistema** para protegerlo.

---

## 8. Load Shedding

**Problema.** Bajo sobrecarga, intentar atender todas las peticiones hace que **todas** fallen por timeout (el servicio dedica recursos a peticiones cuyo cliente ya abandonó).

**Solución.** **Rechazar deliberadamente** parte de las peticiones de forma rápida (`503`) para poder atender bien al resto. Mejor servir al 80 % que fallar al 100 %.

Señales para empezar a descartar: longitud de la cola interna, latencia, uso de CPU, número de peticiones en curso. Descarta primero lo **menos importante** (tráfico de bots, prefetch, analítica) y lo que ya ha superado su deadline. El `ConcurrencyLimiter` con `QueueLimit` pequeño es una forma simple de load shedding.

---

## 9. Compensating Transaction

**Problema.** Una operación de varios pasos sobre recursos distintos (sin transacción común) falla a mitad. Los pasos ya completados no se pueden "deshacer" con un rollback.

**Solución.** Definir, para cada paso, una **acción compensatoria** que deshace su efecto **a nivel de negocio** (no técnico): reembolsar un cobro, cancelar una reserva, enviar un email de rectificación. Se ejecutan en orden inverso. Es el mecanismo de deshacer de una **Saga**.

**Detalles importantes:**
- La compensación **no restaura el estado exacto** anterior (el cliente pudo recibir un email; el stock pudo venderse a otro mientras tanto).
- Debe ser **idempotente** y **reintentable**: también puede fallar.
- Registra el progreso de forma persistente para saber qué compensar tras un fallo del coordinador.
- Algunas acciones **no son compensables** (enviar un SMS, una transferencia irreversible): colócalas al final del flujo (*pivot transaction*).

---

## 10. Scheduler Agent Supervisor

**Problema.** Un flujo de trabajo distribuido (varios pasos, servicios externos) puede quedarse a medias si un paso falla o un nodo cae, y nadie se entera.

**Solución.** Tres roles (patrón del catálogo de Microsoft):
- **Scheduler**: orquesta los pasos y registra el estado de cada uno en un almacén persistente.
- **Agent**: ejecuta un paso concreto (llama a un servicio), con su propio timeout y retry.
- **Supervisor**: revisa periódicamente el almacén de estado, detecta pasos atascados o fallidos (por *timeout* del paso) y ordena reintentarlos o compensarlos.

En la práctica, los **motores de workflow durables** implementan este patrón por ti: Azure Durable Functions, Temporal, Dapr Workflow, AWS Step Functions. Escribirlo a mano rara vez compensa.

---

## 11. Redundancia: zonas y regiones

| Nivel | Protege frente a | Ejemplo Azure | Coste/complejidad |
|---|---|---|---|
| Varias instancias | Fallo de un proceso o VM | `replicas: 3`, Pod anti-affinity | Bajo |
| **Zonas de disponibilidad** | Fallo de un datacenter | AKS con nodos en 3 zonas, SQL zone-redundant, almacenamiento ZRS | Medio |
| **Multirregión activo-pasivo** | Pérdida de una región | Réplica en otra región + failover (Front Door, Traffic Manager) | Alto |
| **Multirregión activo-activo** | Pérdida de una región, con latencia baja global | Patrón **Geode** ([[CN 07 - Escalabilidad y rendimiento]]), Cosmos DB multi-write | Muy alto (conflictos de datos) |

El objetivo de disponibilidad (SLO) y el RTO/RPO decididos con negocio determinan el nivel; no todo sistema necesita multirregión.

---

## 12. Cómo combinarlos: la pipeline de resiliencia

Orden recomendado por Polly / `Microsoft.Extensions.Http.Resilience`, **de fuera hacia dentro**:

```
Petición
  └─► Rate limiter (bulkhead de concurrencia)
        └─► Timeout TOTAL
              └─► Retry (backoff + jitter)
                    └─► Circuit Breaker
                          └─► Timeout POR INTENTO
                                └─► llamada HTTP
```

**Por qué en ese orden:**
- El timeout total va fuera del retry para limitar la operación completa.
- El circuit breaker va **dentro** del retry: cada intento pasa por él, y cuando se abre, el retry recibe `BrokenCircuitException` inmediatamente (configúralo para no reintentar esa excepción, o para esperar).
- El timeout por intento va dentro de todo, para que un intento colgado cuente como fallo.

```csharp
// La "pipeline estándar" ya trae este orden con valores por defecto razonables
// "https+http://" es el esquema de Microsoft.Extensions.ServiceDiscovery (requiere AddServiceDiscovery());
// sin él, usa la URL normal del Service de Kubernetes: http://catalogo
builder.Services.AddHttpClient<CatalogoClient>(c => c.BaseAddress = new("https+http://catalogo"))
    .AddStandardResilienceHandler(o =>
    {
        o.AttemptTimeout.Timeout = TimeSpan.FromSeconds(2);
        o.TotalRequestTimeout.Timeout = TimeSpan.FromSeconds(10);
        o.Retry.MaxRetryAttempts = 3;
        o.Retry.UseJitter = true;
        o.CircuitBreaker.FailureRatio = 0.5;
        o.CircuitBreaker.SamplingDuration = TimeSpan.FromSeconds(30);   // debe ser ≥ 2× AttemptTimeout
    });
```

> [!tip] Patrones de resiliencia y sus capas
> | Capa | Qué aplica |
> |---|---|
> | Cliente HTTP en el código (Polly) | Timeout, retry, circuit breaker, fallback con lógica de negocio |
> | Service mesh / sidecar | Timeout, retry, outlier detection, mTLS — sin lógica de negocio |
> | Gateway | Rate limiting global, autenticación, caché |
> | Plataforma (K8s) | Réplicas, probes, PDB, anti-affinity, zonas |
> Decide **qué capa hace qué** y documéntalo: la duplicación es la causa habitual de *retry storms*.

---

## 13. Verificar la resiliencia: Chaos Engineering

Los patrones de resiliencia **no probados** fallan cuando más se necesitan. *Chaos Engineering* (Netflix, Chaos Monkey, 2011) consiste en **inyectar fallos controlados** para verificar que el sistema se comporta como se espera:

1. Define el **estado estable** (métricas normales: latencia, tasa de éxito).
2. Formula una **hipótesis** ("si cae una zona, la tasa de éxito se mantiene > 99 %").
3. **Inyecta** el fallo (matar Pods, añadir latencia, cortar una dependencia) con un **radio de impacto** limitado.
4. **Observa** y corrige.

Herramientas: Azure Chaos Studio, Chaos Mesh, LitmusChaos, AWS Fault Injection Service; en .NET, las estrategias de **chaos** de Polly v8 (`AddChaosLatency`, `AddChaosFault`, `AddChaosOutcome`) para pruebas en desarrollo.

---

## Puntos clave

- Lo **lento** causa más cascadas que lo caído → **timeout siempre**.
- Retry: sólo transitorios, idempotente, backoff + jitter, **una sola capa**.
- Circuit Breaker deja de insistir; Bulkhead aísla; Fallback decide qué devolver.
- Rate Limiting = cuota por cliente; Throttling/Load Shedding = protegerse de la sobrecarga.
- Compensating Transaction deshace **semánticamente**; debe ser idempotente.
- Orden de la pipeline: rate limiter → timeout total → retry → circuit breaker → timeout por intento.
- Si no lo has probado con fallos reales, no sabes si funciona.

## Errores comunes

- `HttpClient` con el timeout por defecto de 100 s.
- Reintentar `POST` no idempotentes; reintentar `400`/`401`/`404`.
- Retries en cliente + mesh + gateway (*retry storm*).
- Rate limiter en memoria con muchas réplicas creyendo que es global.
- Fallbacks nunca probados.
- Circuit breaker único para todas las dependencias.

## 🎯 Para entrevistas

- *"El servicio de recomendaciones está lento y la home tarda 30 s. ¿Qué harías?"* → timeout corto, circuit breaker, fallback a recomendaciones genéricas en caché, bulkhead para que no consuma el pool del resto; a medio plazo, cargar la sección de forma asíncrona.
- *"¿Por qué jitter?"* → para desincronizar los reintentos de muchos clientes y evitar picos.
- *"¿Rate limiting vs throttling vs load shedding?"* → cuota por cliente; adaptación a la carga del sistema; descarte deliberado bajo sobrecarga.
- *"¿Qué es una transacción compensatoria y qué limitaciones tiene?"* → deshacer de negocio; no restaura exactamente, puede fallar, hay acciones no compensables.

## Referencias

- Microsoft, *Retry*, *Circuit Breaker*, *Bulkhead*, *Rate Limiting*, *Throttling*, *Compensating Transaction*, *Scheduler Agent Supervisor*: https://learn.microsoft.com/azure/architecture/patterns/
- Microsoft, *Build resilient HTTP apps* (`Microsoft.Extensions.Http.Resilience`): https://learn.microsoft.com/dotnet/core/resilience/http-resilience
- Polly: https://www.pollydocs.org/ (incluye *Chaos engineering*)
- Microsoft, *Rate limiting middleware in ASP.NET Core*: https://learn.microsoft.com/aspnet/core/performance/rate-limit
- AWS Builders' Library, *Timeouts, retries, and backoff with jitter* y *Using load shedding to avoid overload*: https://aws.amazon.com/builders-library/
- Michael Nygard, *Release It!*, 2.ª ed. (Pragmatic Bookshelf, 2018).
- *Principles of Chaos Engineering*: https://principlesofchaos.org/

---
⬅️ [[CN 05 - Patrones de gestión de datos]] · ➡️ [[CN 07 - Escalabilidad y rendimiento]]
