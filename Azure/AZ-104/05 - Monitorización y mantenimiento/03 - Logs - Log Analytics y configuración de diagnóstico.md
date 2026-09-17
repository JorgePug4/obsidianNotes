---
tags: [az-104, azure, monitorizacion, logs, log-analytics, dcr]
modulo: Monitorización y mantenimiento
peso_examen: Muy alto
---

# Logs: Log Analytics y configuración de diagnóstico

## ¿Qué es?

Los **registros (logs)** de Azure Monitor son eventos con estructura almacenados en un **área de trabajo de Log Analytics (Log Analytics workspace)**, donde se consultan con **KQL**. Para que lleguen hace falta configurarlo: **configuración de diagnóstico** (recursos de Azure) o **Azure Monitor Agent + Data Collection Rules** (sistema operativo invitado).

## ¿Para qué sirve?

- Auditar operaciones, diagnosticar errores y correlacionar eventos.
- Alimentar alertas de búsqueda de registros, libros y Sentinel.

## Conceptos clave

### Área de trabajo de Log Analytics 🧠
- Recurso de Azure con **región** y **grupo de recursos**; los datos se guardan ahí.
- **Retención**: interactiva **30 días incluidos** (31 en algunos casos), configurable hasta **730 días**; más allá, **retención de archivo (archive)** hasta **12 años** ➕.
- **Planes de tabla** ➕: **Analytics** (consulta completa), **Basic/Auxiliary** (ingesta barata, consultas limitadas).
- **Niveles de compromiso (commitment tiers)** para reducir coste por GB.
- **Control de acceso**: *Require workspace permissions* o **contexto de recurso** (los usuarios ven los logs de los recursos sobre los que tienen permiso).
- Roles: **Log Analytics Reader** / **Log Analytics Contributor** / Monitoring Contributor.
- Un área de trabajo puede recibir datos de **varias suscripciones y regiones**; conviene consolidar.

### Configuración de diagnóstico (diagnostic settings) 🧠
- Se crea **por recurso** (o por Azure Policy a escala) e indica:
  - **Categorías de registros** (por ejemplo, `AuditEvent` en Key Vault, `StorageRead` en Storage, `AppServiceHTTPLogs`) y **AllMetrics**.
  - **Destinos**: **Log Analytics**, **Storage account**, **Event Hub**, partner.
- Se pueden crear **hasta 5 configuraciones** por recurso.
- Sin ella, **no hay logs de recurso**.
- Para el **registro de actividad**: Monitor → Registro de actividad → **Exportar configuración de diagnóstico**.

### Azure Monitor Agent (AMA) y DCR 🧠
- El **Log Analytics Agent (MMA/OMS) está retirado** (agosto de 2024) ⚠️: la respuesta correcta hoy es **AMA**.
- **Data Collection Rule (DCR)**: define **qué** se recopila (contadores de rendimiento, registros de eventos de Windows, syslog, logs de texto), **de dónde** y **a dónde** (workspace, tabla, métricas).
- **Data Collection Endpoint (DCE)** ➕: necesario en escenarios de red privada o ingesta personalizada.
- Se asocia la DCR a las VMs (o a Arc-enabled servers) por **asociación de regla**.
- **VM Insights** usa AMA + DCR y añade el **Dependency Agent** para el mapa.

### Tablas frecuentes 🧠

| Tabla | Contenido |
|---|---|
| `Heartbeat` | Latido del agente (comprobar conectividad) |
| `Perf` | Contadores de rendimiento del invitado |
| `Event` | Registro de eventos de Windows |
| `Syslog` | Syslog de Linux |
| `InsightsMetrics` | Métricas de VM Insights |
| `AzureActivity` | Registro de actividad |
| `AzureDiagnostics` | Logs de recurso (modo heredado, tabla común) |
| `AzureMetrics` | Métricas exportadas |
| `SigninLogs` / `AuditLogs` | Entra ID |
| `StorageBlobLogs`, `AppServiceHTTPLogs`, `AzureFirewallNetworkRule`… | Tablas específicas por recurso |
| `ContainerLogV2` | Contenedores |

## Cómo funciona

```bash
# Área de trabajo
az monitor log-analytics workspace create -g rg-mon -n law-contoso --retention-time 90
# Configuración de diagnóstico de un recurso a Log Analytics
az monitor diagnostic-settings create -n diag-kv --resource <keyVaultId> \
  --workspace $(az monitor log-analytics workspace show -g rg-mon -n law-contoso --query id -o tsv) \
  --logs '[{"category":"AuditEvent","enabled":true}]' --metrics '[{"category":"AllMetrics","enabled":true}]'
# A cuenta de almacenamiento con retención
az monitor diagnostic-settings create -n diag-archivo --resource <resourceId> --storage-account <storageId> \
  --logs '[{"category":"AuditEvent","enabled":true}]'
# Exportar el registro de actividad
az monitor diagnostic-settings subscription create -n diag-activity --location westeurope \
  --workspace <lawId> --logs '[{"category":"Administrative","enabled":true},{"category":"Security","enabled":true}]'
# DCR y agente
az monitor data-collection rule create -g rg-mon -n dcr-vm --rule-file dcr.json
az vm extension set -g rg-web --vm-name vm-web01 -n AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor
az monitor data-collection rule association create -g rg-mon -n assoc-vm01 --rule-id <dcrId> --resource <vmId>
```

```powershell
New-AzOperationalInsightsWorkspace -ResourceGroupName rg-mon -Name law-contoso -Location westeurope -RetentionInDays 90
Set-AzDiagnosticSetting -ResourceId $kv.Id -WorkspaceId $law.ResourceId -Enabled $true -Category AuditEvent
```

Portal: recurso → **Configuración de diagnóstico** → Agregar; Monitor → **Áreas de trabajo de Log Analytics**; Monitor → **Reglas de recopilación de datos**.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Consultar logs de un recurso con KQL | Configuración de diagnóstico → **Log Analytics** |
| Archivar logs 7 años al menor coste | Configuración de diagnóstico → **cuenta de almacenamiento** (o archive del workspace) |
| Enviar logs a Splunk/QRadar | Configuración de diagnóstico → **Event Hub** |
| Recopilar el registro de eventos de seguridad de Windows | **AMA + DCR** con el filtro de eventos |
| Recopilar syslog de Linux | **AMA + DCR** (facility y severidad) |
| Comprobar que el agente envía datos | Consulta `Heartbeat` |
| Aplicar diagnóstico a todos los recursos nuevos | **Azure Policy DeployIfNotExists** |
| Retención de 2 años en el workspace | Ajustar retención (hasta 730 días) |
| Reducir coste de ingesta | Filtrar en la DCR, planes Basic, commitment tiers |

## Ejemplo

Seguridad pide auditar accesos al Key Vault durante 2 años y alertar en tiempo real. Se crea `law-contoso` con retención de 730 días, una **configuración de diagnóstico** en el Key Vault con la categoría `AuditEvent` hacia el workspace, y otra hacia una cuenta de almacenamiento como archivo barato. Sobre la tabla `AzureDiagnostics` se crea una alerta de búsqueda de registros para accesos denegados.

## Comparaciones

| Destino | Coste | Consulta | Cuándo utilizarlo |
|---|---|---|---|
| **Log Analytics** | Medio | **KQL**, alertas, workbooks | Análisis y alertas |
| **Storage account** | **Bajo** | Descargar/consultar externamente | Archivado y cumplimiento |
| **Event Hub** | Medio | Streaming a terceros | SIEM externo |

| Agente | Estado | Uso |
|---|---|---|
| **Azure Monitor Agent (AMA)** | **Actual** | Logs y métricas del invitado con DCR |
| **Log Analytics Agent (MMA/OMS)** | **Retirado (2024)** | No usar |
| **Dependency Agent** | Complemento | Mapa de dependencias en VM Insights |
| **Diagnostics extension (WAD/LAD)** | Heredado | Métricas del invitado a Storage |

## 💻 Laboratorio: diagnóstico y agente

1. Crear `law-lab` con retención de 30 días.
2. Crear una configuración de diagnóstico en una cuenta de almacenamiento (categoría `StorageRead` del servicio blob) hacia el workspace.
3. Instalar **AMA** en una VM y crear una **DCR** que recoja `Percentage CPU` y el registro de eventos del sistema.
4. Esperar unos minutos y consultar `Heartbeat | where Computer == "vm-lab"`.
5. Comprobar `Perf` y `Event` con KQL ([[04 - Consultas KQL]]).

## AZ-104 Exam Tips

- 🔥 🧠 Sin **configuración de diagnóstico** no hay logs de recurso.
- 🔥 🧠 Tres destinos: **Log Analytics, Storage, Event Hub** (máx. 5 configuraciones por recurso).
- 🔥 🧠 El **agente actual es AMA con DCR**; el Log Analytics Agent está **retirado**.
- 🧠 Retención del workspace: 30 días incluidos, hasta **730**; archivo hasta 12 años.
- 🧠 `Heartbeat` para comprobar el agente.
- 💻 Crear workspace, configuración de diagnóstico, instalar AMA y asociar DCR.
- 📌 Registro de actividad (plano de control, 90 días) vs registros de recurso (plano de datos, requieren diagnóstico).

## Errores comunes

- Buscar tablas vacías por no haber creado la configuración de diagnóstico.
- Responder "Log Analytics Agent" en el examen actual.
- Enviar todo a Log Analytics sin filtrar y disparar el coste.

## Preguntas que podrían aparecer

**1.** Quieres recopilar el registro de eventos de seguridad de 50 VMs Windows en un área de trabajo. ¿Qué implementas?
- A) Log Analytics Agent y una solución · B) Azure Monitor Agent con una regla de recopilación de datos asociada a las VMs · C) Configuración de diagnóstico en cada VM · D) Application Insights

<details><summary>Respuesta</summary>

**B.** Los datos del sistema operativo invitado requieren AMA y DCR; el agente antiguo está retirado.
</details>

**2.** Necesitas conservar durante 7 años los registros de auditoría de un Key Vault con el menor coste posible. ¿Qué destino eliges en la configuración de diagnóstico?
- A) Log Analytics con retención de 730 días · B) Una cuenta de almacenamiento · C) Event Hub · D) Application Insights

<details><summary>Respuesta</summary>

**B.** El almacenamiento en blobs (con niveles Cool/Archive) es la opción más barata para archivado largo.
</details>

## Relacionado

- [[04 - Consultas KQL]]
- [[01 - Azure Monitor - visión general]]
- [[06 - Azure Monitor Insights (VM, Storage, Network)]]
- [[11 - Extensiones de VM y automatización]]
- [[00 - Índice - Monitorización y mantenimiento]]
