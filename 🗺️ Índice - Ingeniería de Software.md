---
tags: [MOC, ingenieria-de-software, arquitectura, backend]
tipo: MOC
---

# 🗺️ Índice · Ingeniería de Software

> [!info] Mapa de contenidos (MOC) de las notas técnicas de backend y arquitectura. Las notas de certificación viven en sus propios índices: [[00 - Índice - Conceptos de la nube]], [[00 - Índice - Arquitectura y cómputo]], [[AZ-900 - Autenticación y Autorización (índice)]], [[00 - Índice - Seguridad]], [[Almacenamiento en Azure (Índice)]], [[00 - Introduccion a Azure Networking]], [[00 Indice - Monitorizacion y Gestion]], [[00 - Índice - Costes y herramientas]].

## Cómo está organizado

```
🗺️ Ingeniería de Software
├── 🧱 Fundamentos de diseño
│   ├── [[Principios SOLID]]
│   ├── [[Clean Architecture]]
│   └── [[🧠 Domain-Driven Design (DDD) - Curso Completo|Domain-Driven Design]]
├── 🏗️ Sistemas distribuidos
│   ├── [[🏗️ Diseño de Microservicios]]  ← MOC propio con todos los patrones
│   ├── [[Idempotencia]]
│   └── [[Teorema CAP y BASE]]
├── 💾 Datos
│   ├── [[Bases de Datos SQL vs NoSQL]]
│   └── [[ACID en Bases de Datos]]
├── 🌐 APIs
│   ├── [[Diseño Api Rest|Diseño de APIs REST]]
│   └── [[Status Code|Códigos de estado HTTP]]
├── ⚙️ Lenguajes y runtime
│   ├── [[Task vs ValueTask|Task vs ValueTask (.NET)]]
│   └── [[Go routines|Goroutines y Channels (Go)]]
└── 🧭 Rol y carrera
    └── [[Líder Técnico (Tech Lead)]]
```

## Cómo se relacionan las notas

1. **Fundamentos → Arquitectura.** [[Principios SOLID]] (sobre todo la Inversión de Dependencias) es la base de [[Clean Architecture]], que a su vez es la forma habitual de **aislar** el modelo de dominio que propone [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]].
2. **DDD → Microservicios.** Los *Bounded Contexts* de DDD son la guía para trazar los límites de los servicios en [[🏗️ Diseño de Microservicios]].
3. **Microservicios → Datos distribuidos.** Al no compartir base de datos aparece la consistencia eventual ([[Teorema CAP y BASE]]), que se gestiona con [[Saga Pattern]], [[Transactional Outbox]], [[CQRS (Command Query Responsibility Segregation)|CQRS]] y [[Event Sourcing]]. Todos ellos dependen de la [[Idempotencia]].
4. **Microservicios → Resiliencia.** La red falla: [[Circuit Breaker]], [[Bulkhead]] y [[Retry con Backoff Exponencial]] son la respuesta, y también requieren idempotencia.
5. **Datos.** [[ACID en Bases de Datos]] explica las garantías del mundo relacional; [[Bases de Datos SQL vs NoSQL]] cuándo renunciar a parte de ellas; [[Teorema CAP y BASE]] por qué.
6. **APIs.** [[Diseño Api Rest]] y [[Status Code]] definen el contrato con el exterior; el [[API Gateway]] es donde ese contrato se expone en microservicios.
7. **Runtime.** [[Task vs ValueTask]] y [[Go routines]] cubren la concurrencia en los dos lenguajes que uso, con una comparación cruzada.
8. **Rol.** El [[Líder Técnico (Tech Lead)]] es quien debe tener criterio sobre todo lo anterior.

## Puentes con las notas de Azure (AZ-900)

| Concepto de ingeniería | Nota de Azure relacionada |
|---|---|
| Gestión de secretos en microservicios | [[Azure Key Vault]] |
| Observabilidad (logs, métricas, trazas) | [[Azure Monitor]] |
| API Gateway vs balanceador de capa 7 | [[06 - Azure Application Gateway]], [[04 - Azure Load Balancer]] |
| NoSQL y niveles de consistencia | [[Azure Cosmos DB]] |
| Consistencia eventual en replicación | [[Redundancia de almacenamiento]] |
| Autenticación / autorización, `401` vs `403` | [[Autenticación vs. Autorización]], [[Microsoft Entra ID]] |
| Zero Trust en la seguridad interna | [[Confianza cero (Zero Trust)]] |
| Gobernanza que un Tech Lead debe conocer | [[Gobernanza en Azure]], [[Azure RBAC]] |

## Rutas de estudio sugeridas

- **Preparar entrevista de backend .NET (2 semanas):** SOLID → Clean Architecture → ACID → SQL vs NoSQL → REST + Status Codes → Task vs ValueTask → Microservicios (visión general) → Circuit Breaker, Retry, Idempotencia.
- **Profundizar en microservicios:** DDD → Diseño de Microservicios → Saga → Outbox → CQRS → Event Sourcing → CAP → API Gateway → Service Discovery → resiliencia completa.
- **Rol de Tech Lead:** Líder Técnico → Clean Architecture → DDD (estratégico) → Microservicios (cuándo no) → Gobernanza en Azure.

## Convención de las notas

Cada nota técnica sigue la misma estructura para facilitar el repaso: **¿Qué es? · ¿Para qué sirve? · Conceptos relacionados · ¿Cómo funciona? · Ejemplo · Ventajas · Desventajas · Comparación · Puntos clave · Errores comunes · 🎯 Para entrevistas y exámenes · Referencias**. Las secciones que no aportan a un tema concreto se omiten.
