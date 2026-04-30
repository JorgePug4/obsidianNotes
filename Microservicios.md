# Guía de Arquitectura: Microservicios

## 1. Principios de Diseño
La arquitectura de microservicios se fundamenta en la creación de componentes autónomos y especializados:

*   **Descomposición por dominio:** Basado en *Domain-Driven Design* (DDD) para identificar contextos delimitados (*Bounded Contexts*).
*   **Despliegue independiente:** Capacidad de actualizar un servicio sin afectar la disponibilidad del resto del sistema.
*   **Base de datos aislada:** Cada microservicio gestiona su propia persistencia (Database-per-Service) para evitar el acoplamiento de datos.
*   **Desacoplamiento y autonomía:** Minimiza las dependencias directas, permitiendo que cada equipo elija el stack tecnológico más adecuado.

---

## 2. Estrategias de Comunicación
El flujo de información se define según la necesidad de latencia y visibilidad:

| Protocolo | Ámbito de Uso | Ventajas Clave |
| :--- | :--- | :--- |
| **HTTP/REST** | Servicios externos (APIs) | Universal, fácil de probar y compatible con navegadores. |
| **gRPC** | Comunicación interna (E-E) | Alto rendimiento, serialización binaria y baja latencia. |
| **Mensajería** | Eventos Asíncronos | Desacoplamiento total mediante Brokers (RabbitMQ/Kafka). |

---

## 3. Resiliencia y Estabilidad
Para garantizar que el sistema sea tolerante a fallos, se implementan patrones de diseño críticos:

*   **Reintentos (Retries):** Reintento automático de peticiones fallidas con espera exponencial.
*   **Circuit Breaker:** Corta el flujo hacia un servicio degradado para evitar el colapso en cascada.
*   **Timeouts:** Definición de tiempos máximos de espera para liberar recursos rápidamente.
*   **Health Checks:** Endpoints (`/health`) para monitorear el estado de salud por parte del orquestador.

---

## 4. Escalabilidad y Rendimiento
Estrategias para absorber el incremento de demanda:

*   **Escalabilidad Vertical (Scale Up):** Aumento de recursos (**CPU, RAM**) en los nodos existentes. Útil para tareas intensivas de procesamiento.
*   **Escalabilidad Horizontal (Scale Out):** Despliegue de **más instancias** (réplicas). Es el estándar para manejar incrementos en el tráfico de red.

---

## 5. Seguridad
*   **Autenticación y Autorización:** Implementación de **JWT (JSON Web Tokens)** para transmitir identidad de forma segura entre servicios.
*   **API Gateway:** Centralización de políticas de seguridad, filtrado y *rate limiting*.

---

## 6. Observabilidad y DevOps
Fundamentales para la gestión del ciclo de vida y diagnóstico:

*   **CI/CD:** Automatización total desde el commit hasta el despliegue en producción.
*   **Monitoreo y APM:** Seguimiento detallado del performance con herramientas como **Azure App Insights** o **Dynatrace**.
*   **Trazado Distribuido:** Capacidad de seguir una petición a través de múltiples microservicios.