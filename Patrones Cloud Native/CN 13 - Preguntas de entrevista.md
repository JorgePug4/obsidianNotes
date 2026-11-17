---
tags: [cloud-native, patrones, entrevista, repaso]
aliases: [Entrevista patrones cloud native, Interview cloud native patterns]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 13 - Preguntas de entrevista

> [!info] Cómo usar esta nota
> Intenta responder **en voz alta** antes de desplegar la respuesta. Una buena respuesta de nivel Senior tiene tres partes: **qué es**, **cuándo lo usaría** y **qué cuesta / cuándo no**. Las respuestas son guiones, no textos para memorizar.

Niveles: 🟢 Junior · 🔵 Mid · 🟣 Senior · 🟠 Staff/Arquitecto

---

## Fundamentos

> [!question]- 1. 🟢 ¿Qué es una aplicación cloud native? ¿Es lo mismo que "estar en la nube"?
> Una app diseñada para entornos dinámicos: débilmente acoplada, resiliente, observable, escalable horizontalmente y desplegada de forma automatizada (definición CNCF). No es lo mismo: un *lift and shift* de una VM está en la nube pero no es cloud native. Tampoco exige microservicios: un monolito modular stateless con config externa y CI/CD cumple casi todo. → [[CN 01 - Fundamentos Cloud Native]]

> [!question]- 2. 🟢 Nombra cuatro de los 12 factores y cómo se aplican en Kubernetes.
> Config (ConfigMap/Secret, variables de entorno); Processes stateless (Pods reemplazables, estado en Redis/BD); Disposability (SIGTERM, preStop, readiness); Logs a stdout (recogidos por un DaemonSet). Bonus: Admin processes como `Job` (migraciones).

> [!question]- 3. 🔵 ¿Qué son las falacias de la computación distribuida? Pon un ejemplo de patrón que corrige una.
> Ocho supuestos falsos (la red es fiable, latencia cero, ancho de banda infinito, red segura, topología fija, un administrador, transporte gratis, red homogénea). "La red es fiable" → Retry + Circuit Breaker; "la topología no cambia" → Service Discovery; "la red es segura" → mTLS.

## Contenedores y Kubernetes

> [!question]- 4. 🟢 ¿Diferencia entre Sidecar, Ambassador y Adapter?
> Los tres son contenedores auxiliares en el mismo Pod. Sidecar extiende la app (logs, proxy de mesh). Ambassador es un proxy de **salida** que simplifica el acceso a servicios externos. Adapter normaliza la interfaz de la app hacia **fuera** (exporter de Prometheus). → [[CN 02 - Patrones de contenedores y Kubernetes]]

> [!question]- 5. 🔵 Liveness vs readiness. ¿Qué pasa si la liveness comprueba la base de datos?
> Liveness reinicia el contenedor; readiness lo saca del balanceo sin reiniciar. Si la liveness comprueba la BD y esta cae, Kubernetes reinicia todos los Pods en bucle (no arregla nada, pierde capacidad y genera una avalancha de conexiones al volver la BD). Las dependencias van en readiness, y con cuidado.

> [!question]- 6. 🟣 ¿Qué es un Operator? ¿Cuándo escribirías uno?
> Controller + CRDs que codifica el conocimiento operativo de una app (backups, failover, upgrades) mediante un bucle de reconciliación. Lo escribiría para software con estado y operaciones complejas repetibles que no tenga ya un operador maduro; para una app stateless, un Helm chart basta. Coste: mantener un componente crítico que actúa automáticamente.

## Comunicación

> [!question]- 7. 🟢 ¿Qué hace un API Gateway? ¿Qué no debería hacer?
> Punto de entrada único: routing, agregación, offloading (TLS, JWT, rate limit, caché). No debería contener lógica de negocio (se convierte en monolito/ESB). Riesgos: punto único de fallo y cuello de botella organizativo. → [[API Gateway]]

> [!question]- 8. 🔵 ¿Cuándo usarías BFF en lugar de un único API Gateway?
> Cuando hay varios tipos de cliente con necesidades realmente distintas y equipos de frontend separados: cada BFF es del equipo de su frontend y evoluciona a su ritmo. También como patrón de seguridad en SPAs (tokens en el servidor, cookie HttpOnly). No lo usaría con un solo tipo de cliente.

> [!question]- 9. 🔵 API Gateway vs Service Mesh.
> Gateway: tráfico norte-sur (clientes → sistema), funciones de borde. Mesh: tráfico este-oeste (servicio ↔ servicio), mTLS, reintentos, telemetría uniforme vía proxies. Se complementan. El mesh tiene un coste operativo alto; con pocos servicios o un solo lenguaje, librerías + NetworkPolicy suelen bastar.

> [!question]- 10. 🔵 Tienes un endpoint que genera un informe en 5 minutos. ¿Cómo lo diseñas?
> Async Request-Reply: `POST` → `202 Accepted` + `Location` a un recurso de estado; el trabajo se encola y lo procesa un worker; `GET` de estado con `Retry-After`; al terminar, redirección o enlace al resultado (descarga por Valet Key si es grande). Alternativas al polling: webhook, SignalR/SSE.

> [!question]- 11. 🟣 ¿Cómo migrarías un monolito .NET Framework a .NET moderno sin parar el negocio?
> Strangler Fig: YARP como fachada delante del monolito; extraer un bounded context de bajo riesgo y alto valor; Anti-Corruption Layer para no heredar el modelo legacy; sincronizar datos durante la transición (eventos/CDC); feature flags para revertir; fecha de retirada de cada parte. Medir con trazas que el tráfico migra como se espera.

## Mensajería

> [!question]- 12. 🟢 Comando vs evento.
> Comando: "haz esto", imperativo, un destinatario que es dueño de la acción. Evento: "esto pasó", en pasado, cero o muchos interesados que el emisor no conoce. El evento acopla menos.

> [!question]- 13. 🔵 ¿Existe la entrega exactly-once?
> En general no entre sistemas independientes. Se consigue el **efecto** de una vez con at-least-once + consumidores idempotentes (Inbox, deduplicación por MessageId, operaciones idempotentes). La deduplicación del broker ayuda con reenvíos del productor, pero no sustituye la idempotencia del consumidor.

> [!question]- 14. 🔵 Queue-Based Load Leveling vs Competing Consumers.
> Load Leveling usa la cola para **proteger** a un consumidor procesando a ritmo sostenible; Competing Consumers añade instancias para **aumentar** el ritmo. Se combinan, con KEDA escalando por longitud de cola, vigilando los límites de lo que hay detrás (la BD).

> [!question]- 15. 🟣 Necesitas procesar en orden los eventos de cada pedido con 20 consumidores. ¿Cómo?
> Orden por clave, no global: Service Bus sessions con `SessionId = pedidoId`, o partition key en Kafka/Event Hubs. Paralelismo entre claves, orden dentro de cada una. Riesgos: claves calientes, un mensaje fallido bloquea su sesión; idempotencia igualmente necesaria.

> [!question]- 16. 🟣 Coreografía u orquestación para un checkout de 6 pasos con compensaciones. Justifica.
> Orquestación: el flujo queda explícito en un sitio, es más fácil gestionar compensaciones, timeouts y estados, y ver dónde está cada pedido. Coste: el orquestador debe ser persistente y HA (Durable Functions, Temporal, MassTransit sagas). Coreografía la dejaría para reacciones simples (notificar, actualizar búsqueda).

## Datos

> [!question]- 17. 🔵 ¿Por qué cada microservicio debería tener su propia base de datos? ¿Qué pierdes?
> Autonomía de despliegue y evolución, elegir el almacén adecuado, sin acoplamiento por esquema. Pierdes JOINs (→ API Composition / CQRS) y transacciones entre servicios (→ Saga), y ganas consistencia eventual y duplicación de datos.

> [!question]- 18. 🔵 ¿Qué es el problema del dual write y cómo se resuelve?
> Escribir en BD y publicar en el broker sin transacción común: puedes perder el evento o publicar uno falso. Transactional Outbox (misma transacción + relay) o CDC sobre la tabla outbox. Consumidores idempotentes porque el relay es at-least-once. → [[Transactional Outbox]]

> [!question]- 19. 🟣 Explica Saga. ¿Qué propiedad ACID no tiene y cómo lo mitigas?
> Secuencia de transacciones locales con compensaciones; coreografiada u orquestada. No tiene aislamiento (I): otros ven estados intermedios. Mitigación: estados explícitos (`PendienteDePago`), semantic locks, releer antes de actuar, ordenar pasos con la *pivot transaction* (lo no compensable al final). → [[Saga Pattern]]

> [!question]- 20. 🔵 Cache-Aside: flujo y dos problemas.
> Leer de caché → miss → leer de BD → guardar con TTL; al escribir, invalidar. Problemas: stampede (bloqueo por clave/coalescing, jitter en TTL, HybridCache), datos obsoletos (TTL, invalidación por eventos), penetración (cachear el "no existe").

> [!question]- 21. 🟣 ¿Cuándo NO usarías CQRS o Event Sourcing?
> En dominios CRUD sencillos, sin diferencias fuertes entre lectura y escritura, o cuando el equipo no puede asumir la consistencia eventual y el versionado de eventos. Event Sourcing sólo donde el historial es valor de negocio. Ambos se aplican a partes concretas, no a todo el sistema.

> [!question]- 22. 🟣 ¿Cómo elegirías una clave de sharding?
> Alta cardinalidad, distribución uniforme de carga y alineada con el patrón de acceso principal (que la mayoría de consultas vayan a un shard). Evitar claves monotónicas en rangos (hotspots). En SaaS, `tenantId`, con tratamiento especial para tenants enormes (clave jerárquica o shard dedicado).

## Resiliencia

> [!question]- 23. 🟢 Retry vs Circuit Breaker.
> Retry insiste ante fallos transitorios; Circuit Breaker deja de insistir cuando la dependencia está fallando y falla rápido, dejando que se recupere. Se combinan: el breaker dentro del retry, y el retry no debe machacar un circuito abierto. → [[Circuit Breaker]], [[Retry con Backoff Exponencial]]

> [!question]- 24. 🔵 ¿Por qué backoff exponencial con jitter?
> El backoff da tiempo a recuperarse; el jitter desincroniza a miles de clientes que fallaron a la vez, evitando oleadas sincronizadas de reintentos (*thundering herd*).

> [!question]- 25. 🟣 La home tarda 30 s porque el servicio de recomendaciones está lento. ¿Qué haces hoy y qué a medio plazo?
> Hoy: timeout corto en esa llamada, circuit breaker, fallback a recomendaciones genéricas cacheadas, bulkhead para no agotar el pool del resto. Medio plazo: cargar esa sección de forma asíncrona en el cliente, SLO por dependencia, revisar por qué es lenta con trazas.

> [!question]- 26. 🟣 Rate limiting vs throttling vs load shedding.
> Rate limiting: cuota fija por cliente (429 + Retry-After). Throttling: el sistema limita el consumo según su carga para proteger el SLA. Load shedding: descartar deliberadamente parte del tráfico bajo sobrecarga para servir bien al resto. Con varias réplicas, el rate limit global va en el gateway o en un contador compartido.

## Escalabilidad, despliegue, seguridad, observabilidad

> [!question]- 27. 🔵 ¿Qué necesita una app ASP.NET Core para escalar a varias réplicas sin problemas?
> Stateless (sesión/caché en Redis), ficheros en Blob, **claves de Data Protection compartidas**, config externa, arranque rápido, manejar SIGTERM, health checks, sin trabajos en memoria (`Task.Run`) que se pierden.

> [!question]- 28. 🔵 Blue-Green vs Canary. ¿Qué tienen en común como requisito?
> Blue-Green conmuta todo de golpe con rollback instantáneo y 2× infra temporal; Canary expone a un % creciente con métricas, menor radio de impacto. Requisito común: compatibilidad entre versiones (BD y contratos) → expand and contract.

> [!question]- 29. 🟣 ¿Cómo accede un Pod de AKS a Key Vault y a Azure SQL sin ningún secreto?
> Microsoft Entra Workload ID: ServiceAccount anotada y federada (OIDC) con una identidad gestionada de Entra; `WorkloadIdentityCredential`/`DefaultAzureCredential`; RBAC en Key Vault y usuario de Entra en SQL. Sin cadenas de conexión con contraseña.

> [!question]- 30. 🟠 Te piden 99,99 % de disponibilidad para un servicio que depende de forma síncrona de tres servicios con 99,9 %. ¿Qué respondes?
> Que es incoherente: la disponibilidad compuesta de esas dependencias es ≈ 99,7 %. Opciones: desacoplar con asincronía/colas, caché y fallbacks para que las dependencias no sean críticas en el camino, redundancia de las dependencias, o renegociar el SLO. Y preguntar qué necesita el negocio realmente (99,99 % ≈ 4 min/mes) y cuánto está dispuesto a pagar.

> [!question]- 31. 🟠 Diseña la observabilidad de un sistema nuevo de 10 servicios.
> OpenTelemetry en todos (trazas, métricas, logs) con W3C Trace Context, propagado también en mensajes; Collector con tail sampling y filtrado de PII; métricas RED por servicio y USE de recursos; SLOs por recorrido de usuario y alertas por burn rate con runbooks; dashboards por servicio y por recorrido; eventos de despliegue para correlacionar.

> [!question]- 32. 🟠 ¿Cuándo recomendarías NO usar microservicios?
> Equipo pequeño, producto nuevo con dominio aún incierto, sin capacidad de operar plataforma distribuida (CI/CD, observabilidad, on-call), o sin necesidades de escalado/despliegue independientes. Alternativa: monolito modular con límites claros, listo para extraer servicios cuando haya una razón concreta.

---

## Preguntas de diseño abiertas (para practicar)

1. Diseña un sistema de reservas de entradas para conciertos con picos de 500.000 usuarios en el minuto de apertura. *(Pistas: cola de espera virtual, Load Leveling, reserva temporal con expiración, idempotencia del pago, CDN, rate limiting, Saga para pago+asiento.)*
2. Diseña un SaaS multi-tenant de facturación con clientes en la UE y en EE. UU. *(Pistas: Deployment Stamps por región, residencia de datos, modelo de tenancy híbrido, noisy neighbor, ring deployments.)*
3. Diseña la ingesta de documentos (PDF) con OCR, traducción e indexación. *(Pistas: Valet Key para subir, Quarantine, Claim Check, Pipes and Filters, KEDA, DLQ.)*

---
⬅️ [[CN 12 - Guía de decisión y trade-offs]] · ➡️ [[CN 14 - Cheatsheet]]
