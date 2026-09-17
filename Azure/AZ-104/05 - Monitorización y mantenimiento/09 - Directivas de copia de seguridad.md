---
tags: [az-104, azure, backup, politicas, retencion]
modulo: Monitorización y mantenimiento
peso_examen: Muy alto
---

# Directivas de copia de seguridad

## ¿Qué es?

Una **directiva (política) de copia de seguridad** define **cuándo** se hace la copia (frecuencia y horario) y **cuánto tiempo** se conservan los puntos de recuperación (retención diaria, semanal, mensual y anual). Se crea en el almacén y se aplica a uno o varios elementos protegidos.

## Conceptos clave

### Directivas de VM de Azure: Standard vs Enhanced 🧠

| | **Standard** | **Enhanced** |
|---|---|---|
| Frecuencia | **Diaria** (una vez al día) | **Diaria** o **cada 4, 6, 8 o 12 horas** (hasta múltiples copias al día) |
| Retención de instantánea (instant restore) | **1 a 5 días** (por defecto **2**) | **1 a 30 días** (por defecto **7**) |
| Soporte de VM | VMs estándar | **Trusted Launch VMs**, **Ultra Disk**, **Premium SSD v2**, instantáneas coherentes con varios discos |
| Disponibilidad | General | Recomendada para cargas nuevas |
| Cambio | No se puede pasar de Standard a Enhanced en un elemento ya protegido ⚠️ | — |

### Elementos de una política 🧠
- **Frecuencia y hora** (zona horaria configurable).
- **Retención**:
  - **Diaria**: N días.
  - **Semanal**: qué día, N semanas.
  - **Mensual**: día concreto o "primer/último" día de la semana, N meses.
  - **Anual**: mes y día, N años.
  - Máximo **9999 días** (~27 años) para VMs; **hasta 99 años** con retención a largo plazo en algunos escenarios.
- **Instant restore (snapshot tier)**: la instantánea se guarda junto a la VM para restauraciones **rápidas** durante N días antes de transferirse al almacén.
- Por defecto en el portal: **copia diaria, retención de 30 días** y snapshot 2 días (Standard).

### Otras cargas
- **Azure Files**: copias **diarias** (Standard) o **cada hora** con política mejorada; retención diaria/semanal/mensual/anual; modelo **snapshot** (en la cuenta) y **vaulted** ➕ (en el vault, protege ante borrado de la cuenta).
- **SQL Server en VM**: **completa** (diaria/semanal), **diferencial** y **log** (cada 15 min a 24 h), con retención independiente; permite **restauración a un punto en el tiempo**.
- **SAP HANA**: completa, diferencial, incremental y logs.
- **Blobs**: **operacional** (en la propia cuenta, basado en soft delete/versionado, retención hasta 360 días) y **vaulted** (copia en el vault) ➕.
- **Discos**: snapshots incrementales gestionados, frecuencia por horas y retención en días.
- **MARS (archivos on-premises)**: hasta 3 copias al día, retención hasta 9999 días.

### Gestión
- Una política se puede **modificar** y afecta a todos los elementos asociados (los cambios de retención se aplican a partir de ese momento).
- **Cambiar de política** un elemento protegido: posible entre políticas del mismo tipo.
- **Detener la protección**: *conservando los datos* (se mantienen los puntos hasta su retención) o *eliminando los datos* (entra en soft delete).

## Cómo funciona

```bash
# Ver políticas
az backup policy list -g rg-bkp -v rsv-contoso -o table
# Crear/actualizar desde JSON
az backup policy create -g rg-bkp -v rsv-contoso -n pol-diaria --backup-management-type AzureIaasVM --policy @policy.json
# Aplicar al proteger una VM
az backup protection enable-for-vm -g rg-bkp -v rsv-contoso --vm vm-web01 --policy-name pol-diaria
# Cambiar de política
az backup item set-policy -g rg-bkp -v rsv-contoso -c <containerName> -n <itemName> -p pol-4h
```

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -VaultId $vault.ID
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy" -VaultId $vault.ID
$pol.SchedulePolicy.ScheduleRunTimes[0] = (Get-Date -Hour 2 -Minute 0 -Second 0).ToUniversalTime()
$pol.RetentionPolicy.DailySchedule.DurationCountInDays = 60
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol -VaultId $vault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name vm-web01 -ResourceGroupName rg-web -VaultId $vault.ID
```

Portal: vault → **Directivas de copia de seguridad** → Agregar / editar; o durante "Copia de seguridad" de la VM.

## Configuración relevante para el examen

| Requisito | Configuración |
|---|---|
| Copias cada 4 horas de una VM | Directiva **Enhanced** |
| Conservar las copias 7 años | Retención **anual** de 7 años (política con retención larga) |
| Restauración muy rápida en las últimas 2 semanas | Instant restore de **Enhanced** (hasta 30 días) |
| Proteger una VM con Trusted Launch | Directiva **Enhanced** |
| Copias cada hora de un recurso compartido de Azure Files | Política **mejorada** de Azure Files |
| Restauración a un punto en el tiempo de SQL en VM | Política con **copias de log** cada 15 min |
| Dejar de pagar por una VM sin perder las copias | **Detener protección conservando los datos** |
| Cambiar de Standard a Enhanced en una VM protegida | **No es posible**: proteger de nuevo con la política Enhanced |

## Ejemplo

Producción exige RPO de 4 horas y retención de 7 años para las VMs de facturación. Se crea una directiva **Enhanced** con copias cada 4 horas, retención diaria de 30 días, semanal de 12 semanas, mensual de 24 meses y anual de 7 años, con **instant restore de 7 días**. Para las VMs de desarrollo se usa una política **Standard** diaria con 7 días de retención y snapshot de 2 días.

## Comparaciones

| Política | Frecuencia | Instant restore | Cuándo utilizarla |
|---|---|---|---|
| **Standard (VM)** | Diaria | 1-5 días (2 por defecto) | Cargas normales |
| **Enhanced (VM)** | Hasta cada 4 h | 1-30 días (7 por defecto) | RPO bajo, Trusted Launch, Ultra/Premium v2 |
| **Azure Files (estándar)** | Diaria | Snapshots en la cuenta | Recursos compartidos |
| **Azure Files (vaulted)** ➕ | Diaria | Copia en el vault | Protección ante borrado de la cuenta |
| **SQL en VM** | Full + diferencial + log | — | Punto en el tiempo |

## AZ-104 Exam Tips

- 🔥 🧠 **Enhanced** permite **múltiples copias al día (cada 4 h)** e instant restore de **hasta 30 días**; **Standard** es diaria con 1-5 días.
- 🔥 🧠 El **instant restore** por defecto es **2 días** (Standard) y **7 días** (Enhanced).
- 🧠 Retención diaria/semanal/mensual/anual; hasta **9999 días**.
- 🧠 **No se puede migrar** una VM protegida de Standard a Enhanced.
- 🧠 Detener protección **conservando** o **eliminando** los datos.
- 💻 Crear y editar políticas en el portal, `az backup policy`, `Set-AzRecoveryServicesBackupProtectionPolicy`.
- ⚠️ Cambiar la retención afecta a los puntos futuros y, según el caso, a la caducidad de los existentes.

## Errores comunes

- Elegir Standard cuando el enunciado pide RPO de horas.
- Confundir retención de instantánea (instant restore) con retención en el almacén.
- Pensar que detener la protección borra siempre los datos.

## Preguntas que podrían aparecer

**1.** Necesitas copias de seguridad de una VM cada 4 horas. ¿Qué tipo de directiva configuras?
- A) Standard · B) Enhanced · C) MARS · D) Operacional

<details><summary>Respuesta</summary>

**B.** Solo la directiva Enhanced admite varias copias al día.
</details>

**2.** ¿Cuál es la retención máxima de la instantánea (instant restore) en una directiva Standard de VM?
- A) 2 días · B) 5 días · C) 7 días · D) 30 días

<details><summary>Respuesta</summary>

**B.** Standard admite de 1 a 5 días; Enhanced llega a 30.
</details>

**3.** Quieres dejar de hacer copias de una VM que se va a apagar, pero conservar los puntos de restauración existentes. ¿Qué haces?
- A) Eliminar el vault · B) Detener la protección conservando los datos de copia · C) Cambiar la política · D) Eliminar la VM

<details><summary>Respuesta</summary>

**B.** Esa opción detiene las copias futuras y mantiene los puntos hasta que expire su retención.
</details>

## Relacionado

- [[08 - Azure Backup - Recovery Services vault y Backup vault]]
- [[10 - Operaciones de copia de seguridad y restauración]]
- [[12 - Informes y alertas de copias de seguridad]]
- [[00 - Índice - Monitorización y mantenimiento]]
