---
tags: [cloud-native, patrones, observabilidad, opentelemetry, sre]
aliases: [Observabilidad cloud native, Distributed Tracing, Correlation ID, Health Endpoint Monitoring, SLO, SLI, RED, USE, OpenTelemetry]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 10 - Observabilidad

> [!info] Objetivo del capítulo
> Poder responder **"¿qué está pasando y por qué?"** en un sistema de decenas de servicios efímeros, sin entrar en ninguna máquina.
> Relacionado: [[10 - Observabilidad]] (Kubernetes), [[Azure Monitor]].

## Contenido
- [[#1. Monitorización vs observabilidad]]
- [[#2. Las señales: logs, métricas, trazas (y más)]]
- [[#3. Log Aggregation y logs estructurados]]
- [[#4. Correlation ID]]
- [[#5. Distributed Tracing]]
- [[#6. Métricas: RED, USE y Golden Signals]]
- [[#7. Health Endpoint Monitoring]]
- [[#8. SLI, SLO, SLA y error budget]]
- [[#9. Alertas que sirven]]
- [[#10. OpenTelemetry en .NET]]

---

## 1. Monitorización vs observabilidad

| | Monitorización | Observabilidad |
|---|---|---|
| Pregunta | "¿Está funcionando X?" (preguntas **conocidas**) | "¿Por qué pasa esto?" (preguntas **nuevas**) |
| Ejemplo | Alerta si la CPU > 90 % | "¿Por qué los pedidos de clientes de México con cupón tardan 4 s desde el martes?" |
| Requiere | Dashboards y alertas predefinidos | Telemetría rica, con contexto y alta cardinalidad, correlacionable |

La observabilidad no sustituye a la monitorización: la incluye. El término viene de la teoría de control (Kálmán, 1960): un sistema es observable si su estado interno puede deducirse de sus salidas.

---

## 2. Las señales: logs, métricas, trazas (y más)

| Señal | Qué es | Bueno para | Coste |
|---|---|---|---|
| **Logs** | Eventos discretos con texto/campos | Detalle de **qué** pasó en un punto concreto | Alto en volumen |
| **Métricas** | Valores numéricos agregados en el tiempo | Tendencias, alertas, dashboards (**cuánto**) | Bajo, pero cuidado con la **cardinalidad** de las etiquetas |
| **Trazas** | El recorrido de **una** petición por todos los servicios (spans) | **Dónde** se va el tiempo o dónde falla | Medio; se usa **muestreo** |
| **Perfiles** (*profiling*) continuos | Uso de CPU/memoria por función | **Por qué** el código es lento | Señal más reciente en OpenTelemetry |
| **Eventos** de plataforma | Despliegues, escalados, reinicios de Pods | Correlacionar cambios con incidentes | Bajo |

> [!important] El valor está en correlacionarlas
> Una alerta de métrica (tasa de error ↑) → trazas de las peticiones fallidas (qué servicio) → logs de ese span (qué excepción). Para eso, todas las señales deben compartir **trace_id** y atributos comunes (servicio, versión, entorno).

---

## 3. Log Aggregation y logs estructurados

**Problema.** Los contenedores son efímeros: cuando un Pod muere, sus logs locales desaparecen. Y buscar en logs de 200 instancias por separado es imposible.

**Solución.**
1. La app escribe a **`stdout`/`stderr`** (factor XI), en **formato estructurado** (JSON).
2. Un agente en cada nodo (DaemonSet: Fluent Bit, OpenTelemetry Collector, Azure Monitor Agent) los recoge.
3. Se envían a un **almacén central** con búsqueda (Azure Log Analytics, Elasticsearch/OpenSearch, Loki, Datadog).

```csharp
// Log estructurado: los parámetros son campos consultables, no texto concatenado
_logger.LogInformation("Pedido {PedidoId} confirmado para {ClienteId} por {Total}", pedido.Id, pedido.ClienteId, pedido.Total);
// ❌ _logger.LogInformation($"Pedido {pedido.Id} confirmado...");  → pierde la estructura y genera una plantilla distinta por valor
```

**Buenas prácticas:** niveles coherentes (Error = requiere acción); incluir `trace_id`, servicio y versión; **nunca** registrar secretos, tokens ni datos personales sin enmascarar; muestrear o limitar los logs de *debug* en producción (el coste de ingestión es real).

---

## 4. Correlation ID

**Problema.** Una petición de usuario genera llamadas a 8 servicios y 3 mensajes. Sin un identificador común, no puedes reconstruir qué logs pertenecen a esa petición.

**Solución.** Generar un **identificador único** en la entrada del sistema y **propagarlo** en cada llamada HTTP, gRPC y mensaje; incluirlo en todos los logs.

**Hoy, el estándar es W3C Trace Context**: la cabecera `traceparent` (`00-<trace-id>-<span-id>-<flags>`), que propagan automáticamente ASP.NET Core, `HttpClient` y los SDK de Azure. El `trace-id` **es** el correlation ID. En mensajería, se copia en las propiedades del mensaje (Service Bus `Diagnostic-Id`/`traceparent`).

```
Cliente ─► Gateway (genera trace-id 4bf9…) ─► Pedidos (traceparent: 00-4bf9…-a1…) ─► Service Bus (traceparent en el mensaje) ─► Facturación
          Todos los logs y spans llevan trace_id=4bf9…
```

Inventar una cabecera propia `X-Correlation-ID` sólo tiene sentido si necesitas interoperar con sistemas que no soportan W3C Trace Context.

---

## 5. Distributed Tracing

**Problema.** "La API tarda 3 segundos": ¿en qué servicio?, ¿en qué consulta?, ¿es la red?

**Solución.** Cada operación genera un **span** (nombre, inicio, duración, atributos, estado) con referencia a su padre; todos los spans de una petición forman una **traza** que se visualiza como un diagrama de cascada.

```
GET /checkout                      ████████████████████████████████  3.0 s
├─ Pedidos: POST /pedidos           ███████████████████████████       2.6 s
│  ├─ SQL: INSERT pedidos            █                                0.04 s
│  ├─ HTTP: Inventario /reservar      ████████████████████            2.1 s  ← aquí
│  │   └─ SQL: SELECT stock …          ███████████████████            2.0 s  ← consulta sin índice
│  └─ ServiceBus: send                                    █           0.03 s
```

**Muestreo:** guardar el 100 % de las trazas es caro. **Head sampling** (decidir al inicio, p. ej. 10 %) es barato pero puede perder las trazas interesantes; **tail sampling** (decidir al final, en el Collector: "guarda todas las que tengan error o duren > 1 s") es más útil pero requiere más infraestructura.

Backends: Azure Monitor Application Insights, Jaeger, Grafana Tempo, Zipkin, Honeycomb, Datadog.

---

## 6. Métricas: RED, USE y Golden Signals

Tres métodos para decidir **qué medir**:

| Método | Para qué | Métricas |
|---|---|---|
| **RED** (Tom Wilkie) | **Servicios** (peticiones) | **R**ate (peticiones/s), **E**rrors (fallidas/s), **D**uration (distribución de latencia) |
| **USE** (Brendan Gregg) | **Recursos** (CPU, disco, red, pools) | **U**tilization, **S**aturation (cola/espera), **E**rrors |
| **Four Golden Signals** (Google SRE book) | Sistemas de cara al usuario | Latency, Traffic, Errors, **Saturation** |

**Claves:**
- La latencia se mide con **histogramas** y **percentiles** (p50, p95, p99). La media oculta los problemas: con p50 = 100 ms y p99 = 5 s, la media puede parecer aceptable mientras 1 de cada 100 usuarios espera 5 s.
- Separa la latencia de las peticiones **exitosas** de la de las **fallidas** (un error rápido puede bajar la media).
- **Cardinalidad**: no uses como etiqueta de métrica valores ilimitados (ID de usuario, URL con IDs); explota el almacenamiento. Eso va en trazas y logs.
- Para colas: longitud **y** antigüedad del mensaje más viejo.

---

## 7. Health Endpoint Monitoring

**Problema.** Necesitas saber, desde fuera, si una aplicación y sus dependencias funcionan, antes de que lo noten los usuarios.

**Solución.** Patrón del catálogo de Microsoft: la app expone **endpoints de salud** que comprueban su estado y el de sus dependencias, y herramientas externas los consultan periódicamente.

Tres consumidores distintos, con necesidades distintas:

| Consumidor | Endpoint | Comprueba |
|---|---|---|
| Kubernetes liveness | `/health/live` | Sólo el proceso (ver [[CN 02 - Patrones de contenedores y Kubernetes#5. Health Probe\|Health Probe]]) |
| Kubernetes readiness / balanceador | `/health/ready` | Proceso + dependencias imprescindibles |
| Monitorización sintética (desde varias regiones) | Un flujo de negocio real (*synthetic transaction*) | Que el usuario puede, por ejemplo, iniciar sesión y ver su pedido |

**Seguridad:** no expongas detalles internos (versiones, cadenas de conexión) en endpoints públicos; protege los detallados o publícalos sólo internamente. **Rendimiento:** cachea el resultado unos segundos si las comprobaciones son costosas.

Herramientas de monitorización sintética: Application Insights *availability tests*, Grafana k6 / Synthetic Monitoring, Pingdom, Checkly.

---

## 8. SLI, SLO, SLA y error budget

Conceptos de *Site Reliability Engineering* (Google):

| Término | Qué es | Ejemplo |
|---|---|---|
| **SLI** (*Indicator*) | Una métrica que refleja la experiencia del usuario | % de peticiones `GET /pedidos` con respuesta correcta en < 300 ms |
| **SLO** (*Objective*) | El objetivo interno para ese SLI en una ventana | 99,9 % en 30 días |
| **SLA** (*Agreement*) | Compromiso **contractual** con consecuencias (penalizaciones) | 99,5 % o devolvemos el 10 % de la factura |
| **Error budget** | `1 − SLO`: cuánto fallo puedes permitirte | 0,1 % de 30 días ≈ **43 minutos** |

**Para qué sirve el error budget:** convierte la fiabilidad en una decisión de negocio. Si queda presupuesto, se puede desplegar más rápido y experimentar; si se agota, se prioriza la fiabilidad sobre funcionalidades nuevas.

| Disponibilidad | Indisponibilidad permitida al mes (30 días) | Al año |
|---|---|---|
| 99 % | ~7,2 h | ~3,65 días |
| 99,9 % | ~43 min | ~8,8 h |
| 99,95 % | ~22 min | ~4,4 h |
| 99,99 % | ~4,3 min | ~53 min |

> [!warning] El SLA de tu sistema está limitado por los de sus dependencias
> Si dependes de forma síncrona de tres servicios con SLA 99,9 %, tu disponibilidad teórica no supera ≈ 99,7 %. El SLO debe ser **más exigente** que el SLA (margen) y **alcanzable** con tus dependencias.

---

## 9. Alertas que sirven

- **Alerta sobre síntomas** que sufre el usuario (SLO en riesgo, tasa de error, latencia), no sobre causas (CPU al 80 % no es un problema si los usuarios están bien).
- **Alertas por tasa de consumo del error budget** (*burn rate*, multiventana): "a este ritmo agotamos el presupuesto del mes en 2 días" → avisar; "en 1 hora" → despertar a alguien. Es el método recomendado en el *SRE Workbook* de Google.
- Cada alerta debe ser **accionable** y tener un **runbook** enlazado.
- Las alertas que se ignoran habitualmente se eliminan o se arreglan: la **fatiga de alertas** hace que se ignoren también las importantes.

---

## 10. OpenTelemetry en .NET

**OpenTelemetry** (CNCF, fusión de OpenTracing y OpenCensus en 2019) es el estándar abierto para generar y exportar trazas, métricas y logs, independiente del proveedor. .NET lo soporta de forma nativa: `System.Diagnostics.Activity` es un span, `System.Diagnostics.Metrics.Meter` es una métrica.

```csharp
builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r.AddService("api-pedidos", serviceVersion: "1.4.2"))
    .WithTracing(t => t
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddSource("Pedidos")                     // spans propios
        .AddSource("Azure.*"))                    // SDKs de Azure (Service Bus, Storage…)
    .WithMetrics(m => m
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddRuntimeInstrumentation()
        .AddMeter("Pedidos"))
    .WithLogging()
    .UseOtlpExporter();      // envía por OTLP al Collector / backend (OTEL_EXPORTER_OTLP_ENDPOINT)
// Para Azure Monitor: paquete Azure.Monitor.OpenTelemetry.AspNetCore → .UseAzureMonitor()

// Span y métrica propios (campos de una clase de servicio)
static readonly ActivitySource Fuente = new("Pedidos");
static readonly Counter<long> PedidosCreados = new Meter("Pedidos").CreateCounter<long>("pedidos.creados");

using var activity = Fuente.StartActivity("ConfirmarPedido");
activity?.SetTag("pedido.id", pedido.Id);
PedidosCreados.Add(1, new KeyValuePair<string, object?>("canal", pedido.Canal));   // etiqueta de baja cardinalidad
```

**El OpenTelemetry Collector** (normalmente como DaemonSet o *gateway*) recibe la telemetría, la procesa (muestreo *tail*, filtrado de datos sensibles, enriquecimiento con metadatos de Kubernetes) y la exporta a uno o varios backends. Desacopla la app del proveedor.

> [!tip] .NET Aspire
> Las plantillas de .NET Aspire (*ServiceDefaults*) configuran OpenTelemetry, health checks, resiliencia y service discovery con valores por defecto razonables, y su dashboard local muestra trazas, métricas y logs durante el desarrollo.

---

## Puntos clave

- Monitorización responde preguntas conocidas; observabilidad, preguntas nuevas.
- Logs **estructurados** a stdout + agregación central.
- **W3C Trace Context** (`traceparent`) = correlation ID estándar; propágalo también en mensajes.
- Trazas distribuidas para saber **dónde** se va el tiempo; muestreo *tail* para quedarte con lo interesante.
- **RED** para servicios, **USE** para recursos; percentiles, no medias; cuidado con la cardinalidad.
- **SLO + error budget** convierten la fiabilidad en decisión de negocio; alerta por *burn rate*.
- **OpenTelemetry** como estándar; el Collector desacopla del proveedor.

## Errores comunes

- Interpolar strings en los logs en vez de plantillas estructuradas.
- Registrar tokens, contraseñas o datos personales.
- IDs de usuario como etiqueta de métricas.
- Alertar por CPU en vez de por experiencia de usuario.
- Perder la traza al pasar por una cola (no propagar `traceparent` en el mensaje).
- SLO del 99,99 % con dependencias de 99,9 %.

## 🎯 Para entrevistas

- *"¿Diferencia entre logs, métricas y trazas?"* → qué pasó / cuánto / dónde; y cómo se correlacionan con trace_id.
- *"¿Cómo seguirías una petición que pasa por 6 servicios y una cola?"* → OpenTelemetry, W3C Trace Context propagado en HTTP y en propiedades del mensaje, backend de trazas.
- *"¿Qué es un error budget?"* → 1 − SLO; regula velocidad de cambios vs fiabilidad.
- *"¿Por qué percentiles?"* → la media oculta la cola de latencia que sufren usuarios reales.
- *"RED vs USE"* → servicios vs recursos.

## Referencias

- Microsoft, *Health Endpoint Monitoring*: https://learn.microsoft.com/azure/architecture/patterns/health-endpoint-monitoring
- OpenTelemetry: https://opentelemetry.io/ · .NET: https://opentelemetry.io/docs/languages/dotnet/
- W3C, *Trace Context*: https://www.w3.org/TR/trace-context/
- Google, *Site Reliability Engineering* y *The Site Reliability Workbook* (gratuitos): https://sre.google/books/
- Brendan Gregg, *The USE Method*: https://www.brendangregg.com/usemethod.html
- Tom Wilkie, *The RED Method* (Grafana Labs, 2018).
- Microsoft, *.NET observability with OpenTelemetry*: https://learn.microsoft.com/dotnet/core/diagnostics/observability-with-otel

---
⬅️ [[CN 09 - Seguridad]] · ➡️ [[CN 11 - Antipatrones]]
