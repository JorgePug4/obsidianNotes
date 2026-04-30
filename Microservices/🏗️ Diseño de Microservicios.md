
## 1. 🎯 Principios de Diseño (Core)
* **Descomposición por Dominio (DDD):** Identificación de *Bounded Contexts* para definir límites claros.
* **Autonomía Total:** Despliegue, escalado y evolución independiente.
* **Database-per-Service:** Evita el acoplamiento a nivel de datos; cada servicio es dueño de su persistencia.
* **Desacoplamiento Operacional:** Minimizar dependencias en tiempo de ejecución.

> 💡 **Pro-Tip:** Si el cambio en un servicio requiere desplegar otros dos, tus microservicios están "fuertemente acoplados". Revisa tus límites de dominio.

---

## 2. 📡 Estrategias de Comunicación
| Estilo | Protocolo | Caso de Uso |
| :--- | :--- | :--- |
| **Síncrona** | `gRPC` | Comunicación interna de alto rendimiento (Service-to-Service). |
| **Síncrona** | `REST` | Exposición de servicios hacia el exterior o Frontends. |
| **Asíncrona** | `Broker` | Eventos para desacoplamiento (RabbitMQ, Service Bus, Kafka). |

---

## 3. 💾 Gestión de Datos y Consistencia
* [[Saga Pattern]]: Manejo de transacciones distribuidas (Coreografía vs Orquestación). 
* [[CQRS (Command Query Responsibility Segregation)]]:Segregación de responsabilidades de Lectura y Escritura.
* **Event Sourcing:** Persistencia basada en un log de eventos inmutables.
* [[Transactional Outbox]]


---

## 4. 🛡️ Resiliencia y Tolerancia a Fallos
* [[Circuit Breaker]]:Previene fallos en cascada cortando el flujo a servicios degradados.
* **Bulkhead:** Aislamiento de recursos para evitar el agotamiento total del sistema.
* **Retries con Backoff:** Reintentos automáticos con espera incremental.
* **Health Checks:** Endpoints `/live` y `/ready` para orquestadores (Kubernetes).

---

## 5. 🔐 Seguridad
* **Identity:** Autenticación basada en **JWT** y estándares **OAuth2/OIDC**.
* [[API Gateway]]:Centralización de seguridad, filtrado y *Rate Limiting*.
* **Secret Management:** Uso de Key Vaults para manejar credenciales de forma segura.

---

## 6. 🚀 Observabilidad y DevOps
* **Distributed Tracing:** Rastreo de peticiones de extremo a extremo (App Insights / Jaeger).
* **Logging Estructurado:** Centralización de logs para diagnóstico rápido.
* **CI/CD:** Pipelines automatizados con despliegue progresivo (Canary / Blue-Green).
* **IaC:** Infraestructura definida por código (Terraform / Pulumi).

---

## 🧪 Estrategia de Testing
1.  **Unit Tests:** Validación de lógica interna.
2.  **Contract Tests:** Garantizar compatibilidad de APIs entre servicios.
3.  **Integration Tests:** Pruebas con dependencias externas reales.


test