# Azure Monitor

## Concepto

**Azure Monitor** es la plataforma **unificada de monitorización** de Azure. Recopila **métricas** (números: CPU, latencia) y **registros** (logs: eventos, texto) de todos tus recursos, aplicaciones y del propio Azure, y te permite **analizar, visualizar y alertar** sobre ellos.

Problema que resuelve: sin monitorización descubres los problemas cuando el cliente se queja. Monitor te avisa antes y te da los datos para investigar.

Para qué se usa: ver el estado de tus recursos, diagnosticar fallos de rendimiento, recibir alertas automáticas y crear paneles.

## Características principales

Azure Monitor es el "paraguas" que agrupa varios componentes. Para AZ-900 debes reconocer estos:

| Componente | Qué es | Frase clave |
|---|---|---|
| **Métricas** (Metrics) | Datos numéricos en series temporales, casi en tiempo real | "CPU al 90 %", "latencia" |
| **Registros** (Logs) | Eventos y datos de texto almacenados en un área de trabajo | "qué pasó y cuándo" |
| **Log Analytics** | Herramienta para **consultar** los registros con lenguaje **KQL** (Kusto Query Language) | "escribir consultas sobre logs" |
| **Application Insights** | Monitorización de **aplicaciones** (APM): rendimiento, errores, uso, dependencias | "mi aplicación web va lenta" |
| **Alertas** (Alerts) | Notificación o acción automática cuando se cumple una condición | "avísame si la CPU supera 80 %" |
| **Grupos de acciones** (Action Groups) | Conjunto de destinatarios y acciones para una alerta (email, SMS, webhook, Function, Logic App) | "a quién y cómo avisar" |
| **Paneles / Workbooks** | Visualización | "dashboard" |
| **VM Insights, Container Insights** | Vistas prediseñadas por tipo de recurso | "insights" |

Flujo mental: **Recopilar → Almacenar → Analizar (Log Analytics) → Visualizar → Alertar**.

Sobre las **alertas**:
- Una alerta = **señal** (métrica o log) + **condición** (umbral) + **grupo de acciones** (qué hacer).
- Puede notificar (email, SMS, push, voz) o **actuar** (ejecutar un runbook, Azure Function, Logic App, escalado automático).

Sobre **Log Analytics**:
- Los logs se guardan en un **área de trabajo de Log Analytics** (Log Analytics workspace).
- Se consultan con **KQL**. Para AZ-900 basta con saber que existe y para qué sirve, no hay que escribir consultas.

Sobre **Application Insights**:
- Se centra en la **aplicación**, no en la infraestructura: tiempos de respuesta, excepciones, páginas más visitadas, mapa de dependencias.
- Se integra con el código mediante un SDK o con instrumentación automática.

> [!warning] Métricas vs Registros
> **Métrica** = número ligero, rápido, ideal para alertas en tiempo casi real (CPU %, peticiones/segundo). **Registro** = detalle textual, más pesado, ideal para investigar (errores, auditoría). Si la pregunta habla de "consultar", "investigar" o "KQL", es Logs/Log Analytics. Si habla de "casi en tiempo real" o "porcentaje", es Métricas.

> [!warning] Azure Monitor vs Service Health vs Advisor
> Esta es la confusión número uno del examen. Ver la tabla en la sección de comparaciones.

## Casos de uso

- Una VM se queda sin memoria cada noche: revisar **métricas** y crear una **alerta**.
- La aplicación web devuelve errores 500 y no sabes por qué: **Application Insights**.
- Auditoría pide saber quién reinició una VM el martes: consulta en **Log Analytics** sobre el registro de actividad.
- Quieres que se ejecute un script cuando el disco pase del 90 %: alerta con **grupo de acciones** que lanza una Azure Function.

## Comparaciones

| | Azure Monitor | Azure Service Health | Azure Advisor |
|---|---|---|---|
| Vigila | **Tus recursos y aplicaciones** | **La plataforma Azure** (incidentes, mantenimientos) | **Tus recursos** para dar **recomendaciones** |
| Pregunta que responde | "¿Cómo está mi VM / mi app?" | "¿Azure tiene un problema en mi región?" | "¿Cómo mejoro coste, seguridad, rendimiento?" |
| Datos | Métricas, logs, alertas | Incidentes, mantenimientos, avisos | Recomendaciones en 5 categorías |
| Ejemplo | CPU al 95 % | Storage caído en West Europe | "Apaga esta VM infrautilizada" |

| | Log Analytics | Application Insights |
|---|---|---|
| Foco | Registros de **cualquier origen** (VMs, red, Azure, on-premises) | Rendimiento y uso de **aplicaciones** |
| Cómo se usa | Consultas **KQL** sobre el área de trabajo | SDK o instrumentación en la app |
| Pregunta típica | "Consultar los logs de 200 servidores" | "Detectar excepciones y latencia en mi web" |

## Conceptos que debo memorizar

> [!important]
> - Azure Monitor recopila **métricas y registros** de recursos, aplicaciones y Azure.
> - **Log Analytics** = consultar registros con **KQL** en un **área de trabajo**.
> - **Application Insights** = monitorización de **aplicaciones** (APM).
> - **Alerta** = señal + condición + **grupo de acciones**.
> - Los grupos de acciones pueden **notificar** o **ejecutar acciones automáticas**.
> - Monitor vigila **tus** recursos; Service Health vigila **Azure**.

## Tips para AZ-900

> [!tip]
> - "Consultar registros", "KQL", "área de trabajo" → **Log Analytics**.
> - "Rendimiento de la aplicación", "excepciones", "telemetría de la app" → **Application Insights**.
> - "Notificar cuando", "umbral", "enviar SMS" → **Alertas de Azure Monitor**.
> - "Recomendaciones de coste o seguridad" → **Advisor**, no Monitor.
> - "Interrupción del servicio de Azure", "mantenimiento planificado" → **Service Health**, no Monitor.
> - Trampa: "¿Application Insights forma parte de Azure Monitor?" **Sí**. Log Analytics también.
> - Trampa: "¿Puede una alerta ejecutar una acción, no solo avisar?" **Sí**, mediante grupos de acciones.

## Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas recibir un SMS cuando el uso de CPU de una máquina virtual supere el 85 % durante 5 minutos. ¿Qué debes usar?

- A) Azure Service Health
- B) Una alerta de Azure Monitor con un grupo de acciones
- C) Azure Advisor
- D) Azure Policy

**Respuesta: B.** Las alertas de Monitor evalúan métricas y el grupo de acciones envía el SMS.
- A) Service Health informa de incidentes de Azure, no de tu CPU.
- C) Advisor recomienda, no alerta en tiempo real.
- D) Policy evalúa cumplimiento de configuración.

**Pregunta 2.** Tu equipo de desarrollo quiere ver qué páginas de la aplicación web son más lentas y qué excepciones se producen en el código. ¿Qué componente de Azure Monitor deben usar?

- A) Log Analytics
- B) Application Insights
- C) Azure Service Health
- D) Grupos de administración

**Respuesta: B.** Application Insights es la herramienta de monitorización de aplicaciones.
- A) Log Analytics consulta logs genéricos; podría servir, pero la respuesta específica para rendimiento de aplicaciones es Application Insights.
- C) Service Health no ve dentro de tu app.
- D) Nada que ver con monitorización.

**Pregunta 3.** ¿Qué lenguaje se utiliza para escribir consultas en Log Analytics?

- A) SQL
- B) PowerShell
- C) KQL (Kusto Query Language)
- D) JSON

**Respuesta: C.**
- A) SQL se usa en bases de datos, no en Log Analytics.
- B) PowerShell es un lenguaje de administración, no de consulta de logs.
- D) JSON es un formato de datos, no un lenguaje de consulta.

## 🧠 Resumen para el examen

1. Azure Monitor = plataforma única de monitorización (métricas + registros).
2. Métricas = números en tiempo casi real. Registros = eventos detallados.
3. Log Analytics = consultar registros con KQL en un área de trabajo.
4. Application Insights = monitorización de aplicaciones (APM).
5. Alerta = señal + condición + grupo de acciones.
6. Grupos de acciones: email, SMS, push, voz, webhook, Function, Logic App, runbook.
7. Monitor vigila tus recursos; Service Health vigila Azure; Advisor recomienda.
8. Log Analytics y Application Insights forman parte de Azure Monitor.

---
