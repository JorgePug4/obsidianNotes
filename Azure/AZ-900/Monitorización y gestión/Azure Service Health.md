# Azure Service Health

## Concepto

**Azure Service Health** te informa sobre el **estado de la plataforma Azure** y cómo te afecta a ti: incidentes, mantenimientos planificados y avisos de salud relevantes para **tus** suscripciones, regiones y servicios.

Problema que resuelve: tu aplicación falla y no sabes si el error es tuyo o de Azure. Service Health te dice si Azure tiene un incidente en tu región con los servicios que usas.

Para qué se usa: saber si Azure está afectando a tus recursos, prepararte para mantenimientos y recibir alertas cuando hay problemas de plataforma.

## Características principales

Service Health tiene **tres vistas** que debes distinguir:

| Vista | Qué muestra | Personalizado para ti |
|---|---|---|
| **Azure Status** (status.azure.com) | Estado **global** de todos los servicios de Azure en todas las regiones | **No**, es público y genérico |
| **Service Health** | Incidentes, mantenimientos y avisos que afectan a **tus suscripciones y regiones** | **Sí** |
| **Resource Health** | Estado de **un recurso concreto** (esta VM está disponible / no disponible) | **Sí**, a nivel de recurso |

Tipos de eventos que reporta Service Health:
- **Incidentes de servicio** (service issues): problemas activos en Azure que te afectan.
- **Mantenimiento planificado** (planned maintenance): trabajos futuros que pueden impactar tus recursos.
- **Avisos de salud** (health advisories): cambios que requieren acción tuya (retirada de un servicio, cambio de API, cuota).
- **Avisos de seguridad** (security advisories).

Otras características:
- Permite crear **alertas de Service Health** (se apoyan en Azure Monitor) para recibir aviso cuando ocurra un incidente en tu región.
- Guarda el **historial** de incidentes y publica **informes post-incidente** (RCA).
- Es **gratuito**.

> [!warning] Service Health vs Azure Monitor
> Monitor: "**mi** VM tiene la CPU alta" (problema tuyo). Service Health: "**Azure** tiene un incidente en la región donde está mi VM" (problema de la plataforma). Si la pregunta dice "interrupción", "incidente en Azure", "mantenimiento programado por Microsoft", es Service Health.

> [!warning] Azure Status vs Service Health
> Azure Status es la página pública **global**. Service Health es la vista **personalizada** dentro del portal. Si la pregunta pide "información específica de mis suscripciones", es Service Health, no Azure Status.

## Casos de uso

- Tus VMs de West Europe no responden. Miras Service Health y ves un incidente de red en esa región: no es tu culpa, espera a la resolución.
- Microsoft va a reiniciar hosts la semana que viene: Service Health te avisa con antelación del **mantenimiento planificado**.
- Microsoft retira una versión de API que usas: llega un **aviso de salud**.
- Quieres que te avisen por email si hay un incidente en tus servicios: **alerta de Service Health**.

## Comparaciones

| Escenario | Herramienta |
|---|---|
| ¿Azure tiene un problema global ahora mismo? | Azure Status |
| ¿Hay un incidente que afecte a mis suscripciones? | Service Health |
| ¿Esta VM concreta está sana? | Resource Health |
| ¿Mi VM tiene la CPU alta? | Azure Monitor |
| ¿Cómo reduzco el coste de mis VMs? | Azure Advisor |

## Conceptos que debo memorizar

> [!important]
> - Service Health = estado de **Azure** aplicado a **tus** recursos.
> - Tres vistas: **Azure Status** (global), **Service Health** (tus suscripciones), **Resource Health** (un recurso).
> - Eventos: **incidentes, mantenimiento planificado, avisos de salud, avisos de seguridad**.
> - Permite crear **alertas** e incluye **historial** e informes post-incidente.
> - Es **gratuito**.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *interrupción de Azure, incidente, mantenimiento planificado, región afectada, estado del servicio, informe post-incidente*.
> - "Global / todas las regiones / público" → **Azure Status**.
> - "Mis suscripciones / mis regiones / personalizado" → **Service Health**.
> - "Un recurso específico" → **Resource Health**.
> - Trampa: "¿Service Health te avisa si tu aplicación tiene un bug?" **No**. Eso es Monitor / Application Insights.
> - Trampa: "¿Service Health muestra problemas de todos los clientes de Azure?" No, muestra los que **te afectan**. Azure Status muestra lo global.

## Ejemplo de pregunta de examen

**Pregunta 1.** Quieres saber si hay un mantenimiento planificado de Microsoft que pueda afectar a las máquinas virtuales de tu suscripción la próxima semana. ¿Qué herramienta usas?

- A) Azure Monitor
- B) Azure Service Health
- C) Azure Advisor
- D) Azure Policy

**Respuesta: B.** Service Health publica los mantenimientos planificados que afectan a tus recursos.
- A) Monitor no informa de las operaciones de Microsoft.
- C) Advisor da recomendaciones, no calendarios de mantenimiento.
- D) Policy no tiene relación.

**Pregunta 2.** Necesitas comprobar si una máquina virtual concreta está disponible o si Azure ha detectado un problema en ella. ¿Qué vista utilizas?

- A) Azure Status
- B) Service Health
- C) Resource Health
- D) Log Analytics

**Respuesta: C.** Resource Health muestra el estado de un recurso individual.
- A) Azure Status es global.
- B) Service Health es a nivel de suscripción y servicio, no de recurso individual.
- D) Log Analytics consulta registros, no da un estado de salud directo.

## 🧠 Resumen para el examen

1. Service Health = salud de la plataforma Azure vista desde tus suscripciones.
2. Azure Status (global) vs Service Health (tuyo) vs Resource Health (un recurso).
3. Informa de incidentes, mantenimientos planificados y avisos de salud y seguridad.
4. Permite alertas para que te avisen de incidentes.
5. Incluye historial e informes post-incidente.
6. Gratuito.
7. Monitor = tus recursos. Service Health = Azure.

---
