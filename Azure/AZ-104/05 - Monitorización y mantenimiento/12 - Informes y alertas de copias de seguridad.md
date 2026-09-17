---
tags: [az-104, azure, backup, informes, alertas, backup-center]
modulo: Monitorización y mantenimiento
peso_examen: Alto
---

# Informes y alertas de copias de seguridad

## ¿Qué es?

Los mecanismos para **supervisar** Azure Backup: ver el estado de los trabajos, recibir **alertas** cuando fallan, y generar **informes** de uso, cumplimiento y optimización con **Backup Reports** sobre Log Analytics.

Es un objetivo explícito: "configurar e interpretar informes y alertas de copias de seguridad".

## Conceptos clave

### Alertas 🧠

| Tipo | Origen | Comportamiento |
|---|---|---|
| **Alertas integradas (built-in / classic)** | El propio vault | Se generan ante fallos de copia/restauración, detención de protección con eliminación de datos, etc. Notificación por correo configurable en **Propiedades del vault → Alertas de supervisión** |
| **Alertas de Azure Monitor (recomendadas)** | Métricas del vault y **Log Analytics** | Reglas de alerta estándar con **grupos de acciones** (correo, SMS, webhook, runbook, ITSM) |
| **Alertas por consulta (log search)** | Tabla del workspace | Máxima flexibilidad (por ejemplo, trabajos fallidos por política) |

- Las **alertas integradas** solo notifican por correo y agrupan por hora; las de **Azure Monitor** permiten cualquier acción y severidad.
- Métricas del vault disponibles para alertas: **Backup Health Events**, **Restore Health Events**, trabajos con error.
- Escenarios típicos de alerta: **copia fallida**, **restauración fallida**, **protección detenida con datos eliminados**, **soft delete activado sobre un elemento**.

### Informes (Backup Reports) 🧠
- Se basan en un **área de trabajo de Log Analytics**: hay que crear una **configuración de diagnóstico** en el vault que envíe las categorías de Azure Backup (`AddonAzureBackupJobs`, `AddonAzureBackupPolicy`, `AddonAzureBackupStorage`, `AddonAzureBackupProtectedInstance`, `CoreAzureBackup`) al workspace.
- **Backup Reports** (libro incluido) muestra: **trabajos** (éxito/fallo), **almacenamiento consumido**, **instancias protegidas**, **políticas**, **optimización** (elementos con retención excesiva o inactivos) y **cumplimiento**.
- Los datos tardan ~24 horas en aparecer tras habilitar el diagnóstico.
- Permite informes **entre vaults, suscripciones y regiones** si todos envían al mismo workspace.

### Backup center 🧠
- Consola unificada: inventario de elementos protegidos, **trabajos**, **directivas**, **alertas activas**, **cumplimiento de copias** (Azure Policy) y acciones (configurar copia, restaurar).
- Funciona con **Recovery Services vaults** y **Backup vaults**.
- Integra **Azure Policy** para exigir que las VMs tengan copia de seguridad (por ejemplo, la directiva integrada "Configurar copia de seguridad en VMs…").

### Supervisión de trabajos
- Vault → **Trabajos de copia de seguridad** (últimos 30 días en el vault; más en Log Analytics).
- Estados: *Completed*, *Failed*, *In progress*, *Completed with warnings*.
- Causas frecuentes de fallo: agente de VM no responde, VM apagada, extensión bloqueada, problemas de red, límite de instantáneas, disco no soportado.

## Cómo funciona

```bash
# Enviar los datos del vault a Log Analytics (para informes)
az monitor diagnostic-settings create -n diag-bkp --resource <vaultId> --workspace <lawId> \
  --logs '[{"category":"AddonAzureBackupJobs","enabled":true},{"category":"AddonAzureBackupPolicy","enabled":true},{"category":"AddonAzureBackupStorage","enabled":true},{"category":"AddonAzureBackupProtectedInstance","enabled":true},{"category":"CoreAzureBackup","enabled":true}]'
# Alerta de Azure Monitor sobre trabajos fallidos (consulta)
az monitor scheduled-query create -g rg-mon -n backup-fallidos --scopes <lawId> \
  --condition "count 'AddonAzureBackupJobs | where JobStatus == \"Failed\"' > 0" \
  --evaluation-frequency 1h --window-size 1h --severity 2 --action-groups <agId>
# Trabajos
az backup job list -g rg-bkp -v rsv-contoso --status Failed -o table
```

```powershell
Get-AzRecoveryServicesBackupJob -Status Failed -VaultId $vault.ID
Set-AzRecoveryServicesVaultProperty -VaultId $vault.ID -SoftDeleteFeatureState Enable
```

Consulta KQL típica:

```kusto
AddonAzureBackupJobs
| where JobStatus == "Failed"
| where TimeGenerated > ago(7d)
| summarize Fallos = count() by BackupItemUniqueId, JobFailureCode
| order by Fallos desc
```

Portal: vault → **Trabajos de copia de seguridad**, **Alertas de copia de seguridad**, **Propiedades → Alertas de supervisión**; **Backup center → Informes / Alertas / Trabajos**.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Recibir un correo cuando falle una copia | **Alertas integradas** del vault o alerta de Azure Monitor |
| Ejecutar un runbook cuando falle una restauración | Alerta de **Azure Monitor** con grupo de acciones |
| Informe de almacenamiento consumido por vault | **Backup Reports** (requiere diagnóstico a Log Analytics) |
| Ver todos los vaults de la organización en un sitio | **Backup center** |
| Exigir que todas las VMs tengan copia de seguridad | **Azure Policy** (visible en Backup center → Cumplimiento) |
| Analizar fallos de los últimos 90 días | Log Analytics (`AddonAzureBackupJobs`) |
| Los informes están vacíos | Falta la configuración de diagnóstico o no han pasado 24 h |

## Ejemplo

Operaciones envía los datos de sus tres vaults a `law-contoso`, habilita **Backup Reports** y detecta que el 12 % de los trabajos de un vault fallan por VMs apagadas. Crea una alerta de **búsqueda de registros** que avisa al grupo de acciones `ag-ops` si hay más de 3 fallos en una hora, y una directiva de Azure Policy que audita las VMs sin copia de seguridad.

## Comparaciones

| Mecanismo | Alcance | Acciones | Cuándo utilizarlo |
|---|---|---|---|
| **Alertas integradas del vault** | Un vault | Correo | Configuración rápida |
| **Alertas de Azure Monitor (métrica)** | Vault/suscripción | Grupos de acciones completos | Producción |
| **Alertas de búsqueda de registros** | Workspace | Grupos de acciones | Condiciones complejas |
| **Backup Reports** | Multi-vault | Informe interactivo | Gobernanza y optimización |
| **Backup center** | Toda la organización | Gestión y cumplimiento | Vista diaria |

## AZ-104 Exam Tips

- 🔥 🧠 **Backup Reports requiere enviar los datos del vault a Log Analytics** con una configuración de diagnóstico; tarda ~24 h.
- 🔥 🧠 Las **alertas integradas** solo envían correo; para SMS, webhooks o runbooks hay que usar **Azure Monitor + grupo de acciones**.
- 🧠 **Backup center** unifica vaults, trabajos, políticas, alertas y cumplimiento.
- 🧠 La tabla de trabajos en Log Analytics es `AddonAzureBackupJobs`.
- 💻 Configurar diagnóstico del vault, revisar trabajos, crear alertas.
- 📌 Trabajos (histórico del vault, 30 días) vs informes (Log Analytics, largo plazo).

## Errores comunes

- Esperar informes sin haber configurado el diagnóstico.
- Confiar solo en las alertas por correo integradas cuando se requiere automatización.
- Buscar los trabajos de hace 6 meses en el vault en lugar de en Log Analytics.

## Preguntas que podrían aparecer

**1.** Quieres un informe del almacenamiento consumido y del éxito de los trabajos de varios almacenes durante los últimos 90 días. ¿Qué configuras?
- A) Alertas integradas · B) Configuración de diagnóstico de los vaults hacia un área de trabajo de Log Analytics y Backup Reports · C) Backup center sin más · D) Application Insights

<details><summary>Respuesta</summary>

**B.** Backup Reports se alimenta de los datos enviados a Log Analytics.
</details>

**2.** Necesitas que un runbook se ejecute automáticamente cuando falle una copia de seguridad. ¿Qué usas?
- A) Alertas integradas del vault · B) Una alerta de Azure Monitor asociada a un grupo de acciones con el runbook · C) Backup Reports · D) Soft delete

<details><summary>Respuesta</summary>

**B.** Solo las alertas de Azure Monitor permiten acciones como runbooks, webhooks o Logic Apps.
</details>

## Relacionado

- [[08 - Azure Backup - Recovery Services vault y Backup vault]]
- [[10 - Operaciones de copia de seguridad y restauración]]
- [[05 - Alertas, grupos de acciones y reglas de procesamiento]]
- [[03 - Logs - Log Analytics y configuración de diagnóstico]]
- [[00 - Índice - Monitorización y mantenimiento]]
