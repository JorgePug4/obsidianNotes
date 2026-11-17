---
tags: [cloud-native, antipatrones, arquitectura]
aliases: [Antipatrones cloud native, Monolito distribuido, Chatty I/O, Noisy Neighbor, Retry Storm]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 11 - Antipatrones

> [!info] Objetivo del capítulo
> Reconocer los errores de diseño más frecuentes en sistemas cloud native. Muchos aparecen **al aplicar mal un patrón**. Fuentes: Microsoft *Performance antipatterns for cloud applications*, Sam Newman, Chris Richardson y experiencia común de la industria (indicado cuando es criterio).

## Cómo leer cada antipatrón

**Síntoma** (cómo lo detectas) → **Causa** → **Solución** (patrón o práctica).

---

## A. Antipatrones de arquitectura

### 1. Monolito distribuido
- **Síntoma:** para desplegar un cambio hay que desplegar varios servicios a la vez y en orden; los servicios fallan juntos; los equipos se bloquean entre sí.
- **Causa:** límites de servicio mal trazados (por capas técnicas en vez de por dominio), BD compartida, cadenas de llamadas síncronas, librerías compartidas con lógica de negocio.
- **Solución:** rediseñar límites con [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] (bounded contexts), database per service, comunicación asíncrona, contratos versionados. A veces la solución honesta es **volver a un monolito modular**.

### 2. Base de datos compartida (*Shared Database*)
- **Síntoma:** un cambio de esquema rompe otro servicio; bloqueos y contención entre servicios; nadie sabe quién es dueño de una tabla.
- **Solución:** Database per Service; exponer datos por API o eventos; Materialized View para lecturas cruzadas. Ver [[CN 05 - Patrones de gestión de datos]].

### 3. Nanoservicios (*servicios demasiado pequeños*)
- **Síntoma:** decenas de servicios con una o dos operaciones cada uno; casi toda la latencia es red; cada funcionalidad toca 6 servicios.
- **Causa:** dividir por entidades o por funciones en lugar de por capacidades de negocio.
- **Solución:** fusionar servicios que cambian juntos y se llaman siempre juntos. Criterio útil: un servicio debería poder evolucionar con autonomía y ser propiedad de un equipo.

### 4. Cadenas síncronas largas
- **Síntoma:** la latencia y la disponibilidad empeoran con cada servicio nuevo; un servicio lento al final tumba el inicio.
- **Causa:** A→B→C→D→E por HTTP. La disponibilidad compuesta es el producto de las disponibilidades.
- **Solución:** asincronía (eventos, colas), copias locales de datos (*event-carried state transfer*), agregación en paralelo, resiliencia en cada salto.

### 5. Microservicios por moda (*résumé-driven architecture*)
- **Síntoma:** equipo de 5 personas con 20 servicios, Kubernetes, service mesh y Kafka para una app con 100 usuarios.
- **Solución:** empezar con un monolito modular y extraer cuando haya una razón concreta (escalado distinto, equipos independientes, ritmos de cambio distintos). *Esto es una recomendación compartida por Fowler, Newman y otros, no una ley: hay contextos donde empezar distribuido está justificado.*

### 6. El gateway inteligente (*Smart Gateway / ESB 2.0*)
- **Síntoma:** el API Gateway contiene reglas de negocio, transformaciones complejas y orquestación; un equipo central es cuello de botella.
- **Solución:** gateway "tonto" (routing, seguridad, límites); lógica en los servicios; BFF propiedad del equipo del frontend. Principio de Lewis y Fowler: *smart endpoints and dumb pipes*.

### 7. Librería compartida "core" con lógica de dominio
- **Síntoma:** actualizar `Empresa.Core` obliga a recompilar y redesplegar todos los servicios.
- **Solución:** compartir sólo código **técnico** y estable (logging, telemetría); duplicar deliberadamente lo pequeño de dominio; los contratos se comparten como esquemas (OpenAPI, protobuf), no como DLLs con lógica.

---

## B. Antipatrones de rendimiento (catálogo de Microsoft)

| Antipatrón | Síntoma | Solución |
|---|---|---|
| **Busy Database** | La BD ejecuta lógica pesada (procedimientos almacenados enormes, formateo) y se satura | Mover el procesamiento a la capa de aplicación, que escala horizontalmente |
| **Busy Front End** | El frontend/API hace trabajo pesado en el hilo de la petición | Mover a workers en segundo plano con colas (Async Request-Reply) |
| **Chatty I/O** | Muchas peticiones pequeñas (N+1, una llamada por elemento) | Agrupar en lotes, cargar lo necesario en una llamada |
| **Extraneous Fetching** | Traer más datos de los necesarios (`SELECT *`, filtrar en memoria) | Proyecciones, filtrar y paginar en origen |
| **Improper Instantiation** | Crear y destruir objetos pensados para reutilizarse (`new HttpClient()` por petición → agotamiento de sockets) | Singletons / `IHttpClientFactory`, reutilizar clientes de SDK |
| **Monolithic Persistence** | Un único almacén para datos con patrones de uso muy distintos (transacciones, logs, sesiones, búsqueda) | Almacén adecuado para cada tipo de dato |
| **No Caching** | Se piden repetidamente los mismos datos | Cache-Aside |
| **Noisy Neighbor** | Un tenant o una carga consume recursos compartidos y degrada a los demás | Cuotas, rate limiting por tenant, Bulkhead, stamps dedicados |
| **Retry Storm** | Tras un fallo, los reintentos multiplican la carga e impiden la recuperación | Backoff + jitter, circuit breaker, reintentar en una sola capa, retry budget |
| **Synchronous I/O** | Bloquear hilos esperando E/S (`.Result`, `.Wait()`) → *thread pool starvation* | `async`/`await` de extremo a extremo |

Fuente: https://learn.microsoft.com/azure/architecture/antipatterns/

---

## C. Antipatrones operativos y de plataforma

| Antipatrón | Por qué es un problema | Corrección |
|---|---|---|
| **Pets en Kubernetes** (cambios con `kubectl edit` en producción) | Deriva; el siguiente despliegue lo sobrescribe | GitOps; todo por Git |
| **Imágenes `:latest`** | No reproducible; rollback imposible de razonar | Tags inmutables y versionados, o digest |
| **Sin requests/limits** | Scheduling a ciegas, OOMKilled, vecinos ruidosos | Predictable Demands |
| **Liveness que comprueba dependencias** | Reinicios en cascada cuando cae la BD | Liveness sólo del proceso; dependencias en readiness |
| **Logs a ficheros dentro del contenedor** | Se pierden al morir el Pod; llenan el disco | stdout + agregación |
| **Secretos en el repositorio o en ConfigMaps** | Filtración | Key Vault + Workload Identity |
| **Un único entorno compartido para pruebas** | Equipos que se bloquean y pruebas no fiables | Entornos efímeros por PR, IaC |
| **Observabilidad "para después"** | Primer incidente serio sin datos para diagnosticar | OpenTelemetry desde el día 1 |
| **Multirregión sin probar el failover** | El DR falla el día que se necesita | Simulacros periódicos, Chaos Engineering |

---

## D. Antipatrones de mensajería

| Antipatrón | Problema | Corrección |
|---|---|---|
| Consumidor no idempotente | Duplicados → cobros dobles | Inbox / operaciones idempotentes |
| *Dual write* (BD + broker sin Outbox) | Eventos perdidos o falsos | Transactional Outbox |
| DLQ sin vigilancia | Pérdida silenciosa de datos | Alertas + runbook de reproceso |
| Eventos con el modelo interno de la BD | Acoplamiento al esquema | Eventos de dominio diseñados y versionados |
| Usar eventos para pedir respuestas ("evento-RPC") | Complejidad sin desacoplamiento real | Llamada síncrona o Request-Reply explícito |
| Cola como base de datos (consultar, buscar mensajes) | Los brokers no están hechos para eso | Persistir el estado en una BD |

---

## Señales de alerta en una revisión de diseño

Preguntas que destapan antipatrones:

1. ¿Puedo desplegar este servicio **solo**, sin coordinar con otros?
2. ¿Quién es el **único dueño** de cada tabla?
3. Si cae la dependencia X, ¿qué ve el usuario? ¿Está **probado**?
4. ¿Qué pasa si este mensaje llega **dos veces**? ¿Y **desordenado**?
5. ¿Dónde se reintenta? ¿En **cuántas capas**?
6. ¿Cómo sabré que esto está fallando **antes** que el cliente?
7. ¿Hay algún secreto que una persona tenga que copiar a mano?
8. ¿Qué problema concreto resuelve este patrón **hoy**? (Si la respuesta es "en el futuro", sospecha.)

## 🎯 Para entrevistas

- *"¿Qué es un monolito distribuido y cómo lo detectas?"* → despliegues acoplados, BD compartida, fallos conjuntos; pregunta 1 de la lista.
- *"¿Por qué `new HttpClient()` por petición es un problema?"* → agotamiento de sockets (TIME_WAIT); y un único `HttpClient` estático no respeta cambios de DNS → `IHttpClientFactory` o `SocketsHttpHandler.PooledConnectionLifetime`.
- *"¿Qué es un retry storm?"* → amplificación de carga por reintentos en varias capas; backoff + jitter + circuit breaker + una sola capa.

## Referencias

- Microsoft, *Performance antipatterns for cloud applications*: https://learn.microsoft.com/azure/architecture/antipatterns/
- James Lewis y Martin Fowler, *Microservices* (2014): https://martinfowler.com/articles/microservices.html
- Martin Fowler, *MonolithFirst*: https://martinfowler.com/bliki/MonolithFirst.html
- Sam Newman, *Building Microservices*, 2.ª ed. (O'Reilly, 2021) y *Monolith to Microservices* (2019).
- Microsoft, *Guidelines for using HttpClient*: https://learn.microsoft.com/dotnet/fundamentals/networking/http/httpclient-guidelines

---
⬅️ [[CN 10 - Observabilidad]] · ➡️ [[CN 12 - Guía de decisión y trade-offs]]
