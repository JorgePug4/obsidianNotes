---
tags: [meta, informe]
---

# 📋 Informe de mejora de notas · 2026-09-17

> [!info] Resumen ejecutivo
> Se reescribieron **16 notas** de ingeniería de software, se crearon **8 notas nuevas** (7 de contenido + 1 índice) y se corrigieron **2 enlaces rotos** en las notas de Azure. Las notas de Azure AZ-900 ya seguían una estructura consistente y no se modificaron más. Las carpetas `Bet-el/` (personal) y `Survivor/` (datos de proyecto y credenciales) no se tocaron. Punto de entrada nuevo: [[🗺️ Índice - Ingeniería de Software]].

## 1. Estructura común aplicada

Todas las notas técnicas siguen ahora la misma plantilla: frontmatter (`tags`, `up`, `aliases`) · ¿Qué es? · ¿Para qué sirve? · Conceptos relacionados · ¿Cómo funciona? · Ejemplo (.NET cuando aplica) · Ventajas · Desventajas · Comparación · Puntos clave · Errores comunes · 🎯 Para entrevistas y exámenes · Referencias · enlace de vuelta al índice. Se usan callouts de Obsidian (`[!info]`, `[!tip]`, `[!warning]`, `[!important]`).

## 2. Notas nuevas creadas

| Nota | Por qué hacía falta |
|---|---|
| [[🗺️ Índice - Ingeniería de Software]] | No existía un punto de entrada para las notas no-Azure; conecta todo y tiende puentes con AZ-900 |
| [[Event Sourcing]] | Se mencionaba en Diseño de Microservicios sin nota propia; imprescindible para entender CQRS y Outbox |
| [[Bulkhead]] | Mencionado en resiliencia sin nota |
| [[Retry con Backoff Exponencial]] | Mencionado en resiliencia sin nota; concepto clave para entrevistas |
| [[Idempotencia]] | Concepto transversal que aparecía implícito en REST, Saga, Outbox y Retry sin explicarse nunca |
| [[Teorema CAP y BASE]] | SQL vs NoSQL citaba "BASE" sin definirlo; explica la consistencia eventual de microservicios |
| [[Principios SOLID]] | Citados en Tech Lead y Clean Architecture sin nota |
| Esta nota (informe) | Puede borrarse una vez revisada |

## 3. Cambios por nota

### Microservicios

- **[[🏗️ Diseño de Microservicios]]**: convertida en MOC. Añadido: definición, comparación monolito vs microservicios, "monolith first" y monolito distribuido, Ley de Conway, Strangler Fig, comandos vs eventos, el problema de la disponibilidad multiplicativa en llamadas encadenadas, liveness vs readiness, tres pilares de observabilidad con OpenTelemetry y Correlation ID, tabla de testing con contract testing (Pact, Testcontainers), anti-patrones, referencias. Enlaces nuevos: todas las notas de patrones, Idempotencia, CAP, REST, Status Code, DDD, Clean Architecture, Azure Key Vault, Azure Monitor, Confianza cero, Autenticación vs. Autorización.
- **[[Circuit Breaker]]** (era un párrafo): estados con diagrama y tabla, parámetros de configuración, ejemplo con Polly v8 / `Microsoft.Extensions.Http.Resilience` y fallback, orden correcto de la pipeline, comparación con Retry/Bulkhead/Timeout.
- **[[Saga Pattern]]** (era un párrafo): flujo con compensaciones, transacciones compensables/pivote/reintentables, coreografía vs orquestación con pros/contras, ejemplo MassTransit, anomalías por falta de aislamiento y contramedidas, comparación con ACID y 2PC.
- **[[Transactional Outbox]]** (era un párrafo): explicación del dual-write con tabla de fallos, polling vs CDC (Debezium), garantía at-least-once e Inbox, ejemplo EF Core + BackgroundService, librerías (MassTransit, NServiceBus, CAP, Wolverine).
- **[[CQRS (Command Query Responsibility Segregation)|CQRS]]** (era un párrafo): **corrección**: la nota original daba a entender que CQRS implica bases de datos separadas ("Commands → SQL, Queries → Redis/Elastic"); se aclara que la separación de almacenes es opcional (nivel 2) y que CQRS no es Event Sourcing. Añadido CQS de Meyer, ejemplo con MediatR (con aviso sobre su cambio de licencia en 2025), consistencia eventual.
- **[[API Gateway]]** (era un párrafo): tabla de responsabilidades, flujo paso a paso, BFF, ejemplo YARP, comparación LB L4 / L7 / API Gateway / Service Mesh, norte-sur vs este-oeste. Enlaces a Azure Application Gateway, Load Balancer y Key Vault.
- **[[Service Discovery]]** (era un párrafo): auto-registro vs registro por terceros, client-side vs server-side, cómo lo hace Kubernetes, health checks, ejemplo `Microsoft.Extensions.ServiceDiscovery`, tabla de herramientas.

### Arquitectura

- **[[Clean Architecture]]**: la nota original estaba **truncada** (terminaba en un bloque de código sin cerrar). Completada: Regla de Dependencia explicada con inversión de dependencias, DTOs en fronteras, estructura de solución .NET con ejemplo de código de las 4 capas, comparación con N-Tier, Hexagonal, Onion y Vertical Slice, anti-patrón del dominio anémico.
- **[[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]]**: se conservó íntegro el contenido original. Añadido: distinción estratégico/táctico al inicio, aviso Bounded Context ≠ subdominio, patrones de Context Mapping que faltaban (Partnership, Conformist, Separate Ways, Big Ball of Mud), Factories y Módulos, Domain Service vs Application Service, "el Agregado es el límite de la transacción" y referencia por ID, ejemplos en C# (Value Object con `record`, Agregado), Event Storming, tabla de anti-patrones, preguntas de repaso.
- **[[Principios SOLID]]** (nueva): un ejemplo de C# por principio, aviso DIP ≠ inyección de dependencias, comparación con DRY/KISS/YAGNI.
- **[[Idempotencia]]** (nueva): tabla de métodos HTTP, patrón `Idempotency-Key` con código y condición de carrera, Inbox pattern, idempotencia por diseño.

### Bases de datos

- **[[Bases de Datos SQL vs NoSQL]]**: **correcciones**: "NoSQL = consistencia BASE" y "SQL escala solo vertical" eran afirmaciones desactualizadas; se matizan (MongoDB con transacciones multi-documento desde 4.0, SQL distribuido, PostgreSQL JSONB). Añadido: 7 familias de NoSQL (incluidas búsqueda, series temporales y vectorial), ejemplo de documento desnormalizado, ejemplo de decisión por servicio (persistencia políglota), errores comunes.
- **[[ACID en Bases de Datos]]**: **corrección técnica**: el ejemplo de aislamiento afirmaba que "con aislamiento el usuario B recibe 'sin stock'"; en el nivel por defecto (Read Committed) eso **no** está garantizado. Se explica la actualización perdida y sus soluciones (escritura condicional, `FOR UPDATE`, concurrencia optimista con `RowVersion`). Añadido: anomalías de concurrencia, tabla de niveles de aislamiento con valores por defecto por motor, MVCC, WAL para durabilidad, ejemplo EF Core, ACID vs BASE, ACID en distribuido.
- **[[Teorema CAP y BASE]]** (nueva): lectura correcta del teorema (P no es opcional), CP vs AP con ejemplos, niveles de consistencia de Cosmos DB, PACELC, BASE, aplicación a microservicios.

### API REST

- **[[Status Code]]**: **corrección**: `422` ya no es "Unprocessable Entity" de WebDAV; está en RFC 9110 como "Unprocessable Content". Añadidos códigos importantes que faltaban: `202`, `302/307`, `405`, `409`, `410`, `412`, `415`, `429`, `501`, `502`, `504`. Añadido Problem Details (RFC 9457) con ejemplo ASP.NET Core, mapa de códigos para un CRUD, mnemotecnia ampliada. Se conservó la analogía de la hamburguesa.
- **[[Diseño Api Rest]]**: la sección "Anatomía de una petición" estaba **corrupta** (restos de un diagrama mal pegado: `Cliente -. Servidor">`); reescrita con un diagrama de texto. Añadido: las 6 restricciones de Fielding, reglas de nombrado en tabla, PUT vs PATCH (JSON Patch / Merge Patch), versionado, paginación (offset y cursor), ETags y concurrencia optimista, seguridad, OpenAPI, Modelo de Madurez de Richardson y HATEOAS, comparación con GraphQL/gRPC/WebSockets, ejemplo Minimal API.

### .NET y Go

- **[[Task vs ValueTask]]**: eliminados los artefactos `[cite: 1, 2]` (más de 15 apariciones) y el texto suelto "Cambio por device". Añadido: caché de `Task` completados del compilador, `PoolingAsyncValueTaskMethodBuilder` (.NET 6), por qué `FindAsync` devuelve `ValueTask`, `Preserve()`, versión sin `async`, tabla comparativa ampliada, cuándo usarlo en interfaces públicas.
- **[[Go routines]]**: contenido original conservado (ya era excelente). Añadido: frontmatter, conceptos relacionados, `sync.RWMutex`, `errgroup`, aviso sobre `time.After` en bucles (comportamiento pre y post Go 1.23), comparación Go vs .NET, puntos clave, preguntas de repaso, referencias. Correcciones menores: "quando" → "cuando"; la lista de mecanismos de sincronización decía "goroutines, WaitGroup, channels" (una goroutine no sincroniza) → "WaitGroup, channels o context".

### Rol

- **[[Líder Técnico (Tech Lead)]]**: la nota no tenía formato Markdown (títulos y tabla en texto plano). Reformateada. Añadido: reparto del tiempo, ADRs, C4, cómo dar feedback en revisiones, comparación ampliada de roles (Staff Engineer, Senior), anti-patrones del rol, preguntas de entrevista, referencias (Fournier, Larson, Team Topologies).

### Azure (solo enlaces)

- `00 - Índice - Seguridad`: `[[Zero Trust]]` no existía → `[[Confianza cero (Zero Trust)|Zero Trust]]`.
- `Repaso final de Almacenamiento`: `[[AZ-900 - Almacenamiento (Índice)]]` no existía → `[[Almacenamiento en Azure (Índice)]]`.

## 4. Puntos que conviene verificar

Información que puede variar con el tiempo o que conviene contrastar con la fuente oficial antes de un examen o entrevista:

- **Licencia de MediatR**: el aviso sobre el cambio a licencia comercial (2025) refleja el anuncio de su autor; revisar las condiciones vigentes antes de adoptarlo.
- **KurrentDB**: EventStoreDB se renombró a Kurrent en 2025; los enlaces y nombres de paquete pueden seguir cambiando.
- **`Microsoft.Extensions.ServiceDiscovery`** y **`Microsoft.Extensions.Http.Resilience`**: las APIs se describen según .NET 8/9; comprobar cambios en versiones posteriores.
- **Nivel de aislamiento por defecto**: Read Committed en SQL Server, PostgreSQL y Oracle; Repeatable Read en MySQL/InnoDB. Correcto a fecha de hoy, pero es configurable por servidor.
- **Comportamiento de `time.After` en Go 1.23**: descrito según las notas de la versión; verificar si trabajas con una versión distinta.
- **Modelo de licencia y nombre de los borradores IETF** (`Idempotency-Key`): sigue siendo *draft*; puede cambiar de nombre al publicarse como RFC.
- Los porcentajes de reparto del tiempo del Tech Lead son orientativos y varían por empresa.

## 5. Posibles siguientes pasos

- Notas candidatas a existir (mencionadas pero sin nota propia): **OAuth 2.0 / OpenID Connect y JWT**, **gRPC**, **Kubernetes (conceptos básicos)**, **OpenTelemetry / Observabilidad**, **Monolito modular**, **Strangler Fig**, **Testcontainers y tests de contrato**, **Concurrencia en .NET (`System.Threading.Channels`)**.
- Revisar la nota `Survivor/NoSQL/Comandos Redis` : contiene un dato aparentemente cruzado (`team:14` Indianapolis Colts con alias `JAC` y logo `jax`; `team:15` Jacksonville Jaguars con alias `IND` y logo `ind`). No se modificó por ser datos de proyecto, pero parece un error.
- `Survivor/Credentials.md` está versionado en el repositorio; si contiene secretos reales, conviene sacarlo del control de versiones.
