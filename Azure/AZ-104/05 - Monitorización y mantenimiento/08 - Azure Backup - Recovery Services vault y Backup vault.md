---
tags: [az-104, azure, backup, recovery-services-vault, backup-vault]
modulo: Monitorización y mantenimiento
peso_examen: Muy alto
---

# Azure Backup: Recovery Services vault y Backup vault

## ¿Qué es?

**Azure Backup** es el servicio de copias de seguridad como servicio (BaaS). Los datos se guardan en un **almacén (vault)**, que es el recurso que define **redundancia, cifrado, políticas y control de acceso**. Existen **dos tipos de almacén** y el examen pregunta cuál usar.

## Los dos almacenes 🧠

| | **Recovery Services vault (RSV)** | **Backup vault** |
|---|---|---|
| Antigüedad | El clásico | El de nueva generación |
| **Cargas soportadas** | **VMs de Azure**, **Azure Files**, **SQL Server en VM de Azure**, **SAP HANA en VM**, **agente MARS** (archivos/carpetas on-premises), **DPM/MABS** | **Discos administrados (Azure Disk)**, **Blobs (operacional y vaulted)**, **Azure Database for PostgreSQL (incluido Flexible Server)**, **AKS**, **Azure Files (vaulted)** ➕ |
| **Azure Site Recovery** | **Sí** (también aloja ASR) | No |
| Redundancia por defecto | **GRS** (configurable a LRS o ZRS **antes** de proteger el primer elemento) | **ZRS**/GRS según carga |
| Restauración entre regiones | **Cross Region Restore** (requiere GRS + habilitarlo) | Según carga |
| Gestión | Backup center / vault | Backup center / vault |

> [!important] Regla para el examen
> **VM, Azure Files, SQL/SAP en VM, on-premises (MARS/MABS) y Site Recovery → Recovery Services vault.**
> **Discos, blobs, PostgreSQL, AKS → Backup vault.**

## Conceptos clave

- **Redundancia del almacén** 🧠: LRS, ZRS o GRS. **Solo se puede cambiar antes de proteger el primer elemento**.
- **Cross Region Restore (CRR)**: restaurar en la región emparejada; requiere **GRS** y activarlo explícitamente.
- **Eliminación temporal (soft delete)** 🧠: los datos de copia eliminados se conservan **14 días adicionales sin coste** (ahora configurable entre 14 y 180 días en el modelo *enhanced soft delete*); **activada por defecto** y, con *always-on*, no se puede desactivar.
- **Cifrado**: por defecto con claves de Microsoft; opción de **CMK** y de **inmutabilidad del almacén** (vault lock) ➕.
- **Configuración de seguridad**: **MUA (Multi-User Authorization)** con Resource Guard para operaciones críticas ➕.
- **Restricción de restauración entre suscripciones** ➕.
- **Roles RBAC** 🧠: **Backup Contributor** (todo menos borrar el vault y dar acceso), **Backup Operator** (copias y restauraciones, no políticas ni borrar), **Backup Reader** (solo lectura). También *Backup Contributor* a nivel de vault.
- **Backup center**: vista unificada de todos los vaults, trabajos, políticas y cumplimiento.
- Un vault es **regional**: protege recursos de su región (con excepciones); la VM y el vault deben estar en la **misma región** 🧠.
- **Límites**: 1000 elementos protegidos por vault de VMs (orientativo); varios vaults por suscripción.

## Cómo funciona

```bash
# Recovery Services vault
az backup vault create -g rg-bkp -n rsv-contoso -l westeurope
az backup vault backup-properties set -g rg-bkp -n rsv-contoso --backup-storage-redundancy GeoRedundant
az backup vault backup-properties set -g rg-bkp -n rsv-contoso --cross-region-restore-flag true
# Soft delete
az backup vault backup-properties set -g rg-bkp -n rsv-contoso --soft-delete-feature-state Enable
# Backup vault (nueva generación)
az dataprotection backup-vault create -g rg-bkp -v bv-contoso -l westeurope --storage-settings datastore-type=VaultStore type=ZoneRedundant
az backup vault list -o table
```

```powershell
New-AzRecoveryServicesVault -ResourceGroupName rg-bkp -Name rsv-contoso -Location westeurope
Set-AzRecoveryServicesBackupProperty -Vault $vault -BackupStorageRedundancy GeoRedundant
Set-AzRecoveryServicesVaultContext -Vault $vault
New-AzDataProtectionBackupVault -ResourceGroupName rg-bkp -VaultName bv-contoso -Location westeurope -StorageSetting ...
```

Portal: **Almacenes de Recovery Services** / **Almacenes de Backup** → Crear; Backup center → **+ Copia de seguridad**.

## Configuración relevante para el examen

| Carga de trabajo | Almacén |
|---|---|
| Máquina virtual de Azure | **Recovery Services vault** |
| Recurso compartido de Azure Files (snapshots) | **Recovery Services vault** |
| SQL Server o SAP HANA dentro de una VM de Azure | **Recovery Services vault** |
| Archivos y carpetas de un servidor on-premises (agente MARS) | **Recovery Services vault** |
| Replicación y failover de VMs a otra región (ASR) | **Recovery Services vault** |
| Disco administrado (snapshots gestionados) | **Backup vault** |
| Blobs de una cuenta de almacenamiento | **Backup vault** |
| Azure Database for PostgreSQL | **Backup vault** |
| Clúster de AKS | **Backup vault** |

| Escenario | Solución |
|---|---|
| Restaurar en la región secundaria | GRS + **Cross Region Restore** habilitado |
| Datos no deben salir de la región | Cambiar la redundancia a **LRS/ZRS antes** de proteger nada |
| Proteger frente a borrado malicioso de copias | **Soft delete** (14+ días) + **MUA** + inmutabilidad |
| Usuario que solo debe lanzar restauraciones | Rol **Backup Operator** |
| Vault y VM en regiones distintas | No es posible: crear el vault en la región de la VM |

## Ejemplo

Contoso crea `rsv-weu` en West Europe con redundancia **GRS** y **Cross Region Restore** habilitado antes de proteger nada, para poder restaurar en North Europe ante un desastre. Protege 30 VMs y 5 recursos compartidos de Azure Files. Para los **discos** de una base de datos crítica y para los **blobs** de auditoría crea además un **Backup vault**, porque esas cargas no son compatibles con el RSV.

## Comparaciones

| Mecanismo | Alcance | Recuperación | Cuándo utilizarlo |
|---|---|---|---|
| **Azure Backup (RSV)** | VMs, Files, SQL/SAP, on-premises | Puntos de restauración con retención larga | Copias de seguridad gestionadas |
| **Backup vault** | Discos, blobs, PostgreSQL, AKS | Según carga | Cargas de nueva generación |
| **Azure Site Recovery** | Replicación continua de VMs | **Failover** a otra región (RPO bajo) | Continuidad de negocio |
| **Snapshots de disco** | Un disco | Manual, puntual | Antes de un cambio concreto |
| **Redundancia de Storage (GRS)** | Datos de la cuenta | No protege de borrados | Durabilidad, no backup |
| **Soft delete / versionado** | Blobs y Files | Deshacer borrados | Complemento |

## AZ-104 Exam Tips

- 🔥 🧠 **RSV**: VMs, Azure Files, SQL/SAP en VM, MARS/MABS y **ASR**. **Backup vault**: discos, blobs, PostgreSQL, AKS.
- 🔥 🧠 La **redundancia solo se cambia antes del primer elemento protegido**; por defecto **GRS**.
- 🔥 🧠 **Soft delete** conserva las copias **14 días** adicionales sin coste (configurable hasta 180 en el modelo actual).
- 🧠 **Cross Region Restore** requiere GRS y habilitación explícita.
- 🧠 El vault debe estar en la **misma región** que la VM.
- 🧠 Roles: **Backup Contributor / Operator / Reader**.
- 💻 Crear vaults, ajustar redundancia y soft delete, usar Backup center.
- 📌 Backup (puntos de restauración, retención) vs Site Recovery (replicación y failover).

## Errores comunes

- Intentar proteger discos o blobs en un Recovery Services vault.
- Cambiar la redundancia después de proteger una VM (ya no se puede).
- Crear el vault en otra región que la VM.

## Preguntas que podrían aparecer

**1.** Necesitas proteger blobs de una cuenta de almacenamiento y discos administrados. ¿Qué tipo de almacén creas?
- A) Recovery Services vault · B) Backup vault · C) Key Vault · D) Storage account

<details><summary>Respuesta</summary>

**B.** Los blobs, discos, PostgreSQL y AKS se protegen con Backup vault.
</details>

**2.** Creas un Recovery Services vault y proteges una VM. Después intentas cambiar la redundancia a LRS y la opción está deshabilitada. ¿Por qué?
- A) Falta el rol Owner · B) La redundancia solo puede cambiarse antes de proteger el primer elemento · C) LRS no es compatible con VMs · D) Hay que deshabilitar el soft delete

<details><summary>Respuesta</summary>

**B.** Una vez hay elementos protegidos, la redundancia queda fijada.
</details>

**3.** ¿Qué servicio se configura dentro de un Recovery Services vault además de las copias de seguridad?
- A) Azure Monitor · B) Azure Site Recovery · C) Azure Policy · D) Azure Advisor

<details><summary>Respuesta</summary>

**B.** El RSV aloja también la replicación y el failover de Site Recovery.
</details>

## Relacionado

- [[09 - Directivas de copia de seguridad]]
- [[10 - Operaciones de copia de seguridad y restauración]]
- [[11 - Azure Site Recovery]]
- [[12 - Informes y alertas de copias de seguridad]]
- [[00 - Índice - Monitorización y mantenimiento]]
