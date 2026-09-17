---
tags: [az-104, azure, monitorizacion, kql, log-analytics]
modulo: Monitorización y mantenimiento
peso_examen: Alto
---

# Consultas KQL (Kusto Query Language)

## ¿Qué es?

**KQL** es el lenguaje de consulta de Azure Monitor Logs, Application Insights, Microsoft Sentinel y Azure Data Explorer. Es **de solo lectura**: se parte de una **tabla** y se encadenan **operadores** con el símbolo de tubería `|`.

En AZ-104 no hay que escribir consultas complejas, pero sí **leerlas e interpretarlas** y saber elegir la correcta.

## ¿Para qué sirve?

- Buscar errores, auditar cambios, medir rendimiento.
- Definir alertas de búsqueda de registros y libros.

## Estructura básica 🧠

```kusto
Tabla
| where   <filtro>            // filtrar filas
| project <columnas>          // elegir columnas
| summarize <agregación> by <campo>   // agrupar
| order by <campo> desc       // ordenar (sort by es sinónimo)
| take 10                     // limitar (limit es sinónimo)
```

## Operadores que hay que reconocer 🧠

| Operador | Qué hace | Ejemplo |
|---|---|---|
| `where` | Filtra filas | `| where TimeGenerated > ago(1h)` |
| `project` / `project-away` | Selecciona / descarta columnas | `| project Computer, CounterValue` |
| `extend` | Crea una columna calculada | `| extend GB = CounterValue/1024` |
| `summarize` | Agrega (count, avg, sum, min, max, percentile) | `| summarize avg(CounterValue) by Computer` |
| `count` | Número de filas | `| count` |
| `distinct` | Valores únicos | `| distinct Computer` |
| `top` / `take` / `limit` | N filas | `| top 5 by CounterValue desc` |
| `order by` / `sort by` | Ordena | `| order by TimeGenerated desc` |
| `render` | Visualiza | `| render timechart` |
| `join` | Une tablas | `| join kind=inner (Heartbeat) on Computer` |
| `union` | Combina tablas | `union Event, Syslog` |
| `let` | Variable | `let umbral = 80;` |
| `bin()` | Agrupa por intervalos de tiempo | `summarize count() by bin(TimeGenerated, 1h)` |
| `ago()` / `now()` / `startofday()` | Tiempo relativo | `ago(7d)` |
| `search` | Búsqueda libre (costosa) | `search "error"` |
| `parse` / `extract` | Extraer texto | |
| `make-series` | Series temporales | |

Operadores de comparación de cadenas: `==`, `=~` (sin distinguir mayúsculas), `!=`, `contains`, `has`, `startswith`, `endswith`, `matches regex`, `in`.

> [!tip] `has` vs `contains`
> `has` busca **palabras completas** y es **más eficiente**; `contains` busca subcadenas. En consultas grandes, preferir `has`.

## Consultas típicas del examen y del día a día

```kusto
// 1. ¿El agente está enviando datos?
Heartbeat
| where TimeGenerated > ago(30m)
| summarize LastHeartbeat = max(TimeGenerated) by Computer

// 2. CPU media por VM en la última hora, en intervalos de 5 minutos
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where TimeGenerated > ago(1h)
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| render timechart

// 3. Errores del registro de eventos de Windows
Event
| where EventLevelName == "Error"
| where TimeGenerated > ago(24h)
| summarize Errores = count() by Computer, Source
| order by Errores desc

// 4. Quién eliminó recursos (registro de actividad)
AzureActivity
| where OperationNameValue contains "delete"
| where ActivityStatusValue == "Success"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup, _ResourceId
| order by TimeGenerated desc

// 5. Inicios de sesión fallidos en Entra ID
SigninLogs
| where ResultType != 0
| summarize Intentos = count() by UserPrincipalName, ResultDescription
| order by Intentos desc

// 6. Errores HTTP 5xx de una Web App
AppServiceHTTPLogs
| where ScStatus >= 500
| summarize count() by bin(TimeGenerated, 1h), CsHost
| render columnchart

// 7. VMs que no han enviado latido en la última hora (posible caída)
Heartbeat
| summarize LastCall = max(TimeGenerated) by Computer
| where LastCall < ago(1h)

// 8. Espacio libre en disco por debajo del 10 %
Perf
| where CounterName == "% Free Space"
| summarize FreePct = avg(CounterValue) by Computer, InstanceName
| where FreePct < 10
```

## Dónde se ejecutan

- **Monitor → Logs** (ámbito: workspace, suscripción o recurso).
- **Recurso → Logs** (contexto de recurso, filtra automáticamente).
- **Alertas de búsqueda de registros** (log search alerts).
- **Libros (workbooks)** y paneles.
- CLI: `az monitor log-analytics query -w <workspaceId> --analytics-query "Heartbeat | take 10"`.
- PowerShell: `Invoke-AzOperationalInsightsQuery -WorkspaceId <id> -Query "..."`.

## Configuración relevante para el examen

| Necesidad | Consulta/enfoque |
|---|---|
| Comprobar conectividad del agente | `Heartbeat` |
| Rendimiento del invitado | `Perf` |
| Eventos de Windows | `Event`; Linux → `Syslog` |
| Operaciones del plano de control | `AzureActivity` |
| Logs de recurso (modo heredado) | `AzureDiagnostics` |
| Métricas exportadas | `AzureMetrics` |
| Gráfico de evolución | `| render timechart` |
| Agrupar por hora | `bin(TimeGenerated, 1h)` |
| Últimas 24 horas | `where TimeGenerated > ago(24h)` |
| Guardar para reutilizar | **Consultas guardadas** o **función** |

## Ejemplo

Tras una incidencia, el equipo ejecuta `AzureActivity | where OperationNameValue contains "Microsoft.Compute/virtualMachines/delete"` y descubre quién y cuándo eliminó la VM. Después crea una **alerta de búsqueda de registros** con esa consulta, evaluada cada 5 minutos, ligada a un grupo de acciones con correo y webhook.

## Comparaciones

| Herramienta | Uso | Cuándo |
|---|---|---|
| **Explorador de métricas** | Números y gráficos rápidos | Rendimiento en tiempo casi real |
| **Logs (KQL)** | Eventos y correlación | Diagnóstico, auditoría, alertas complejas |
| **Insights** | Vistas preconfiguradas | Diagnóstico guiado |
| **Workbooks** | Informes interactivos | Compartir análisis |
| **Sentinel** | SIEM | Seguridad avanzada |

## AZ-104 Exam Tips

- ⭐ Estructura: **Tabla | where | summarize | order by | render**.
- 🔥 🧠 `ago(1h)`, `bin(TimeGenerated, 5m)`, `summarize count() by`, `render timechart`.
- 🧠 Tablas clave: **Heartbeat, Perf, Event, Syslog, AzureActivity, AzureDiagnostics, AzureMetrics, SigninLogs**.
- 🧠 `=~` ignora mayúsculas; `has` es más eficiente que `contains`.
- 💻 Ejecutar consultas desde Monitor → Logs y crear alertas a partir de ellas.
- ⚠️ KQL es **solo lectura**: no modifica datos.

## Errores comunes

- Confundir `summarize` (agrupa) con `project` (selecciona columnas).
- Olvidar el filtro temporal y consultar todo el histórico (lento y caro).
- Buscar tablas que no existen porque falta la configuración de diagnóstico.

## Preguntas que podrían aparecer

**1.** ¿Qué consulta muestra el número de eventos de error por equipo en las últimas 24 horas?
- A) `Event | project Computer` · B) `Event | where EventLevelName == "Error" | where TimeGenerated > ago(24h) | summarize count() by Computer` · C) `Event | take 24` · D) `Perf | summarize avg(CounterValue)`

<details><summary>Respuesta</summary>

**B.** Filtra por nivel y tiempo y agrupa con summarize.
</details>

**2.** ¿Qué tabla consultas para comprobar si el agente de una máquina virtual está enviando datos?
- A) Perf · B) Event · C) Heartbeat · D) AzureActivity

<details><summary>Respuesta</summary>

**C.** Heartbeat registra el latido periódico del agente.
</details>

**3.** ¿Qué operador agrupa los resultados en intervalos de una hora?
- A) `ago(1h)` · B) `bin(TimeGenerated, 1h)` · C) `take 1h` · D) `render timechart`

<details><summary>Respuesta</summary>

**B.** `bin()` agrupa valores en intervalos; `ago()` filtra el rango temporal.
</details>

## Relacionado

- [[03 - Logs - Log Analytics y configuración de diagnóstico]]
- [[05 - Alertas, grupos de acciones y reglas de procesamiento]]
- [[06 - Azure Monitor Insights (VM, Storage, Network)]]
- [[00 - Índice - Monitorización y mantenimiento]]
