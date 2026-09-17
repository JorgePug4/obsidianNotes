---
tags: [microservicios, arquitectura, MOC, dotnet]
up: "[[🗺️ Índice - Ingeniería de Software]]"
tipo: MOC
---

# 🏗️ Diseño de Microservicios

> [!info] Esta nota es el **mapa de contenidos (MOC)** de microservicios. Cada patrón tiene su propia nota; aquí está la visión general y cómo se relacionan.

## ¿Qué son los microservicios?

Un estilo arquitectónico que estructura una aplicación como un **conjunto de servicios pequeños, autónomos y desplegables de forma independiente**, cada uno responsable de una **capacidad de negocio** concreta y comunicándose por red (HTTP/gRPC o mensajes).

El término lo popularizaron James Lewis y Martin Fowler (2014). La alternativa clásica es el **monolito**: una sola unidad de despliegue con todo el código.

### Monolito vs Microservicios

| Criterio | Monolito | Microservicios |
|---|---|---|
| Despliegue | Una unidad | Independiente por servicio |
| Escalado | Todo o nada | Por servicio, según carga |
| Base de datos | Compartida | **Una por servicio** |
| Tecnología | Homogénea | Puede variar por servicio (con moderación) |
| Transacciones | ACID locales | Distribuidas → [[Saga Pattern]] |
| Complejidad | En el código | En la **operación** (red, despliegue, observabilidad) |
| Equipos | Un equipo grande o varios pisándose | Un equipo por servicio (Ley de Conway) |
| Cuándo | Producto nuevo, equipo pequeño, dominio aún poco claro | Dominio conocido, varios equipos, necesidades de escalado distintas |

> [!warning] "Monolith first"
> Martin Fowler y Sam Newman recomiendan empezar con un **monolito modular** bien estructurado (con [[🧠 Domain-Driven Design (DDD) - Curso Completo|Bounded Contexts]] claros) y extraer microservicios cuando haya una razón concreta. Un monolito mal dividido se convierte en un **monolito distribuido**: todos los costes de la red sin ninguna de las ventajas.

---

## 🗺️ Mapa de patrones

```
🏗️ Diseño de Microservicios
├── 🌐 Infraestructura y enrutamiento
│   ├── [[API Gateway]]
│   └── [[Service Discovery]]
├── 🔄 Transaccionalidad y datos
│   ├── [[Saga Pattern]]
│   ├── [[Transactional Outbox]]
│   ├── [[CQRS (Command Query Responsibility Segregation)|CQRS]]
│   └── [[Event Sourcing]]
├── 🛡️ Resiliencia
│   ├── [[Circuit Breaker]]
│   ├── [[Bulkhead]]
│   └── [[Retry con Backoff Exponencial]]
└── 🧩 Transversales
    ├── [[Idempotencia]]
    ├── [[Teorema CAP y BASE]]
    ├── [[Diseño Api Rest|Diseño de APIs REST]] · [[Status Code|Códigos HTTP]]
    └── [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] · [[Clean Architecture]]
```

**Cómo se relacionan:** DDD define *dónde* cortar (Bounded Contexts → servicios). El API Gateway y Service Discovery resuelven *cómo se encuentran y se exponen*. Saga, Outbox, CQRS y Event Sourcing resuelven *cómo mantener datos consistentes sin una base de datos compartida*. Circuit Breaker, Bulkhead y Retry resuelven *qué pasa cuando la red falla*. La Idempotencia es el pegamento que hace seguros los reintentos y la mensajería.

---

## 1. 🎯 Principios de diseño (Core)

- **Descomposición por dominio ([[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]])**: identificar *Bounded Contexts* para definir límites claros. Alternativa: descomposición por **capacidad de negocio** o por **subdominio**.
- **Autonomía total**: despliegue, escalado y evolución independientes. Un servicio debe poder desplegarse sin coordinar con otros.
- **Database-per-Service**: cada servicio es dueño de su persistencia; **nadie más accede a su base de datos directamente**. Evita el acoplamiento a nivel de datos y permite elegir el almacén adecuado ([[Bases de Datos SQL vs NoSQL]]). Coste: no hay JOINs ni transacciones entre servicios.
- **Desacoplamiento operacional**: minimizar dependencias en tiempo de ejecución; preferir comunicación asíncrona donde sea posible.
- **Ley de Conway**: la arquitectura acaba reflejando la estructura de comunicación de la organización. Alinea equipos y servicios (*Team Topologies*).
- **Diseño para el fallo**: la red **va** a fallar; cada servicio debe seguir degradándose de forma controlada.

> [!tip] Prueba del acoplamiento
> Si el cambio en un servicio requiere desplegar otros dos, tus microservicios están fuertemente acoplados. Revisa tus límites de dominio.

### Migración desde un monolito: Strangler Fig

Patrón (Fowler) para migrar gradualmente: se coloca un [[API Gateway]] delante del monolito y se van **redirigiendo rutas** a nuevos microservicios, una funcionalidad a la vez, hasta que el monolito queda "estrangulado". Evita el *big-bang rewrite*.

---

## 2. 📡 Estrategias de comunicación

| Estilo | Protocolo | Caso de uso | Trade-off |
|---|---|---|---|
| **Síncrona** | **REST** / HTTP + JSON | Exposición hacia el exterior, frontends, APIs públicas. Ver [[Diseño Api Rest]]. | Simple y universal; acoplamiento temporal (el destino debe estar disponible) |
| **Síncrona** | **gRPC** (HTTP/2 + Protobuf) | Comunicación interna de alto rendimiento (*service-to-service*), streaming | Muy eficiente y tipado; menos legible, soporte limitado en navegadores (gRPC-Web) |
| **Asíncrona** | **Mensajería** (RabbitMQ, Azure Service Bus, Kafka) | Eventos de dominio, desacoplamiento, picos de carga, procesos largos | Desacopla en tiempo y disponibilidad; añade consistencia eventual y complejidad operativa |

### Comandos vs eventos (asíncrono)

- **Comando**: *"haz esto"*, dirigido a un destinatario concreto (cola). Ej.: `CobrarTarjeta`.
- **Evento**: *"esto ocurrió"*, publicado a quien quiera escucharlo (tópico / pub-sub). Ej.: `PedidoCreado`. Es la base de la **Event-Driven Architecture** y de las [[Saga Pattern|Sagas coreografiadas]].

> [!important] Las llamadas síncronas encadenadas son el mayor riesgo
> A → B → C → D en cadena multiplica la latencia y hace que la disponibilidad total sea el **producto** de las disponibilidades (0.99 × 0.99 × 0.99 × 0.99 ≈ 0.96). Prefiere eventos o agrega en el Gateway.

---

## 3. 💾 Gestión de datos y consistencia

Sin base de datos compartida, la consistencia entre servicios pasa a ser **eventual** ([[Teorema CAP y BASE]]). Patrones:

- [[Saga Pattern]]: transacciones distribuidas como secuencia de transacciones locales con compensaciones (coreografía vs orquestación).
- [[Transactional Outbox]]: publicar eventos de forma fiable junto con el cambio en BD (evita el *dual-write problem*).
- [[CQRS (Command Query Responsibility Segregation)|CQRS]]: separar el modelo de escritura del de lectura.
- [[Event Sourcing]]: persistir la historia de eventos en lugar del estado actual.
- **API Composition**: para consultas que cruzan servicios, un componente (a menudo el Gateway o un BFF) llama a varios servicios y une los resultados en memoria. Alternativa: una **vista materializada** mantenida por eventos (CQRS).

---

## 4. 🛡️ Resiliencia y tolerancia a fallos

- [[Circuit Breaker]]: previene fallos en cascada cortando el flujo hacia servicios degradados.
- [[Bulkhead]]: aísla recursos para que una dependencia lenta no agote todo el sistema.
- [[Retry con Backoff Exponencial]]: reintentos automáticos con espera creciente y *jitter*, solo sobre operaciones [[Idempotencia|idempotentes]].
- **Timeouts**: toda llamada de red debe tener un límite. Sin timeout, ningún otro patrón funciona.
- **Health Checks**: endpoints para orquestadores:
  - `/health/live` (*liveness*): "el proceso está vivo" → si falla, Kubernetes reinicia el pod.
  - `/health/ready` (*readiness*): "puedo recibir tráfico" (BD conectada, caché caliente) → si falla, se retira del balanceo sin reiniciar.
  - En .NET: `AddHealthChecks()` + `MapHealthChecks("/health/ready", ...)`.
- **Degradación controlada** (*graceful degradation*): mostrar recomendaciones genéricas si el servicio de recomendaciones cae, en vez de romper la página.

En .NET 8+, `Microsoft.Extensions.Http.Resilience` (Polly v8) agrupa timeout, retry, circuit breaker y limitador de concurrencia en una sola pipeline.

---

## 5. 🔐 Seguridad

- **Identidad**: autenticación basada en **JWT** con estándares **OAuth 2.0 / OpenID Connect**. Un proveedor de identidad central (Microsoft Entra ID, Keycloak, Auth0, Duende IdentityServer) emite tokens; los servicios los validan. Ver también [[Autenticación vs. Autorización]].
- [[API Gateway]]: centraliza autenticación, terminación TLS, filtrado y *rate limiting*.
- **Seguridad interna**: no confiar solo en el perímetro ([[Confianza cero (Zero Trust)|Zero Trust]]). Opciones: mTLS entre servicios (Service Mesh), propagación del token, o al menos red privada.
- **Gestión de secretos**: cadenas de conexión y claves en un almacén como [[Azure Key Vault]], nunca en el código ni en variables de entorno en claro del repositorio.
- **Principio de mínimo privilegio** para las identidades de cada servicio (*managed identities* en Azure).

---

## 6. 🚀 Observabilidad y DevOps

### Los tres pilares de la observabilidad

| Pilar | Qué responde | Herramientas |
|---|---|---|
| **Logs estructurados** | ¿Qué pasó exactamente? | Serilog → Seq / Elastic / [[Azure Monitor]] (Log Analytics) |
| **Métricas** | ¿Cómo está el sistema? (latencia p99, errores/s, CPU) | Prometheus + Grafana, Application Insights |
| **Trazas distribuidas** | ¿Por dónde pasó esta petición y dónde tardó? | **OpenTelemetry** → Jaeger, Zipkin, Application Insights |

- **Correlation ID**: un identificador que viaja en las cabeceras (`traceparent` del estándar W3C Trace Context) por todos los servicios que atraviesa una petición. Sin él, depurar un fallo entre 8 servicios es imposible.
- **OpenTelemetry** es el estándar actual (neutral respecto al proveedor) para instrumentar los tres pilares; .NET lo soporta de forma nativa.

### Despliegue

- **CI/CD** con un pipeline **por servicio**, no uno global.
- **Contenedores** (Docker) y orquestación (**Kubernetes**, Azure Container Apps).
- **Despliegue progresivo**: *Canary* (un porcentaje de tráfico a la nueva versión) o *Blue-Green* (dos entornos, cambio de tráfico instantáneo).
- **Infraestructura como código (IaC)**: Terraform, Bicep, Pulumi.
- **Feature flags** para desacoplar despliegue de activación.

---

## 7. 🧪 Estrategia de testing

La pirámide de pruebas en microservicios añade un nivel clave: los **tests de contrato**.

| Nivel | Qué valida | Herramientas .NET |
|---|---|---|
| **Unit tests** | Lógica interna del servicio, sin red ni BD | xUnit, NUnit, FluentAssertions |
| **Integration tests** | El servicio con sus dependencias reales (BD, broker) en contenedores | `WebApplicationFactory`, **Testcontainers** |
| **Contract tests** | Que consumidor y proveedor de una API siguen entendiéndose, **sin desplegar ambos** | **Pact** (consumer-driven contracts) |
| **End-to-end** | Flujo completo a través de varios servicios | Playwright, Postman/Newman. Pocos y críticos: son lentos y frágiles |

> [!tip] Los tests de contrato son la respuesta a "¿y si cambio mi API y rompo a alguien?"
> El consumidor publica el contrato que espera; el proveedor lo verifica en su propio CI. Se detecta la rotura **antes** de desplegar.

---

## ⚠️ Anti-patrones frecuentes

- **Monolito distribuido**: servicios que deben desplegarse juntos.
- **Base de datos compartida** entre servicios.
- **Nanoservicios**: servicios tan pequeños que la sobrecarga de red supera el valor.
- **Llamadas síncronas en cadena** para todo.
- **Sin observabilidad**: 20 servicios y ningún *correlation ID*.
- **Empezar con microservicios** en un dominio que aún no se entiende.

## 🎯 Para entrevistas y exámenes

- *"¿Cuándo NO usarías microservicios?"* → Equipo pequeño, dominio poco claro, producto en validación, sin capacidad de operar Kubernetes/observabilidad.
- *"¿Cómo mantienes consistencia sin transacciones distribuidas?"* → Saga + Outbox + idempotencia, aceptando consistencia eventual.
- *"¿Cómo evitas fallos en cascada?"* → Timeouts + Circuit Breaker + Bulkhead + Retry con backoff.
- *"¿Qué es el problema de la doble escritura?"* → Ver [[Transactional Outbox]].
- *"¿Qué es un monolito distribuido?"* → Microservicios acoplados que deben desplegarse juntos.

## Referencias

- Chris Richardson, *Microservices Patterns* (Manning, 2018) y https://microservices.io/patterns/
- Sam Newman, *Building Microservices*, 2ª ed. (O'Reilly, 2021)
- Martin Fowler & James Lewis, *Microservices* (2014): https://martinfowler.com/articles/microservices.html
- Microsoft, *.NET Microservices: Architecture for Containerized .NET Applications* (guía gratuita): https://learn.microsoft.com/dotnet/architecture/microservices/
- Microsoft Learn, *Cloud Design Patterns*: https://learn.microsoft.com/azure/architecture/patterns/

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
