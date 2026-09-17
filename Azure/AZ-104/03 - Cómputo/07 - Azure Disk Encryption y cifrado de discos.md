---
tags: [az-104, azure, computo, vm, cifrado, ade, key-vault]
modulo: Cómputo
peso_examen: Alto
---

# Azure Disk Encryption y cifrado de discos

## ¿Qué es?

Azure ofrece **cuatro mecanismos** para cifrar los discos de una VM. El temario nombra explícitamente **Azure Disk Encryption (ADE)**, pero el examen pregunta por las diferencias entre todos:

| Mecanismo | Dónde cifra | Clave | Cubre disco temporal y caché |
|---|---|---|---|
| **SSE con PMK** (por defecto) | Plataforma de almacenamiento | Microsoft | No (solo discos administrados) |
| **SSE con CMK** | Plataforma de almacenamiento | Tuya, en Key Vault (disk encryption set) | No |
| **Encryption at host** | En el **host** antes de salir a almacenamiento | PMK o CMK | **Sí** (temporal, caché) |
| **Azure Disk Encryption (ADE)** | **Dentro del SO invitado** (BitLocker / dm-crypt) | Tuya, en Key Vault | **Sí** (incluido disco temporal) |
| **Confidential disk encryption** ➕ | Ligada al vTPM de la VM | Clave vinculada a la VM | Disco de SO en VMs confidenciales |

## ¿Para qué sirve?

- Cumplir normativas que exigen cifrado con claves propias y control del cliente.
- Proteger el disco temporal y las cachés (no cubiertas por SSE).
- Demostrar cifrado "en el invitado" cuando el auditor lo exige.

## Conceptos clave

### SSE (Storage Service Encryption) para discos
- **Siempre activo**, sin coste, AES-256, transparente.
- Con **CMK** se usa un **Disk Encryption Set (DES)**: recurso que enlaza el disco con una clave de Key Vault mediante **identidad administrada**. El Key Vault necesita **soft delete** y **purge protection**.
- Un DES tiene ámbito de región; los discos deben estar en la misma región.
- Se puede aplicar a discos de SO y de datos, snapshots e imágenes.

### Encryption at host
- Se habilita en la **VM** (no en el disco): `--encryption-at-host`. Requiere **registrar la característica** en la suscripción (`EncryptionAtHost`) 🧠.
- Cifra también **disco temporal, caché de disco y flujo al almacenamiento**.
- No todos los tamaños lo soportan; la VM debe estar **desasignada** para habilitarlo.
- **No compatible con ADE** en la misma VM.

### Azure Disk Encryption (ADE)
- Usa **BitLocker** (Windows) o **dm-crypt** (Linux) **dentro del SO**.
- Requiere un **Key Vault** en la **misma región y suscripción** que la VM, con la directiva de acceso **"Azure Disk Encryption for volume encryption"** habilitada (o los roles RBAC equivalentes) 🧠.
- Puede usar una **KEK** (key encryption key) opcional para envolver la clave (BEK).
- Cifra disco de **SO**, discos de **datos** y el **disco temporal**.
- **No soportado** en 🧠: VMs de la serie **Basic**, **A-series**, VMs con menos de la memoria mínima (2 GB para Linux con disco de datos, 8 GB recomendado para SO), **Ultra Disk**, **Premium SSD v2**, **discos compartidos**, **Trusted Launch con algunas configuraciones**, y no se combina con SSE+CMK ni con encryption at host.
- ⚠️ **ADE está anunciado para retirada el 15 de septiembre de 2028**; Microsoft recomienda migrar a encryption at host / CMK. Sigue estando en el temario actual.
- Requiere que la VM esté **en ejecución** y que el agente de Azure funcione.
- Para VMs con ADE, los **snapshots y backups** deben incluir la configuración de cifrado; el Key Vault no se puede borrar.

## Cómo funciona

```bash
# 1) Key Vault preparado para ADE
az keyvault create --name kv-ade --resource-group rg-web --location westeurope --enabled-for-disk-encryption true
# 2) Habilitar ADE en la VM
az vm encryption enable --resource-group rg-web --name vm-web01 --disk-encryption-keyvault kv-ade --volume-type All
az vm encryption show --resource-group rg-web --name vm-web01 -o table
az vm encryption disable --resource-group rg-web --name vm-web01 --volume-type All

# Encryption at host
az feature register --namespace Microsoft.Compute --name EncryptionAtHost
az vm update --resource-group rg-web --name vm-web01 --set securityProfile.encryptionAtHost=true   # VM desasignada

# SSE con CMK (disk encryption set)
az disk-encryption-set create --resource-group rg-web --name des01 --key-url <keyUrl> --source-vault kv-ade
az disk update --resource-group rg-web --name disk-data01 --disk-encryption-set des01 --encryption-type EncryptionAtRestWithCustomerKey
```

```powershell
$kv = Get-AzKeyVault -VaultName kv-ade -ResourceGroupName rg-web
Set-AzVMDiskEncryptionExtension -ResourceGroupName rg-web -VMName vm-web01 -DiskEncryptionKeyVaultUrl $kv.VaultUri -DiskEncryptionKeyVaultId $kv.ResourceId -VolumeType All
Get-AzVmDiskEncryptionStatus -ResourceGroupName rg-web -VMName vm-web01
Disable-AzVMDiskEncryption -ResourceGroupName rg-web -VMName vm-web01
```

Portal: VM → **Discos** → **Cifrado adicional** / **Configuración de cifrado**: elegir tipo (PMK/CMK), disk encryption set, o habilitar ADE desde "Cifrado de disco".

## Configuración relevante para el examen

| Requisito del enunciado | Mecanismo |
|---|---|
| "Cifrado en reposo" sin más detalles | Ya está: **SSE con PMK** |
| "Con nuestras propias claves en Key Vault" | **SSE + CMK** (disk encryption set) |
| "También el disco temporal y las cachés" | **Encryption at host** (o ADE) |
| "BitLocker / dm-crypt dentro del sistema operativo" | **ADE** |
| "Key Vault debe estar en la misma región que la VM" | **ADE** |
| VM de serie A o Basic | **ADE no soportado** |
| Ultra Disk o Premium SSD v2 | **ADE no soportado** → usar CMK/encryption at host |
| "Sin coste adicional y sin configuración" | SSE con PMK |

## Ejemplo

Un cliente del sector salud exige que los discos de sus VMs estén cifrados con claves que ellos controlan y roten, incluyendo el disco temporal. Sus VMs son `Standard_D4s_v5` con Premium SSD. Solución recomendada: **encryption at host** con **CMK** (disk encryption set), evitando ADE por su retirada anunciada. Si el auditor exigiera explícitamente BitLocker en el invitado, se usaría ADE con Key Vault en la misma región.

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **SSE PMK** | Base | Gratis, transparente, siempre activo | Por defecto |
| **SSE CMK** | Control de claves | Rotación y revocación propias | Cumplimiento de claves |
| **Encryption at host** | Cifrado extremo a extremo desde el host | Cubre temporal y caché, sin agente | Recomendado hoy sobre ADE |
| **ADE** | Cifrado en el invitado | BitLocker/dm-crypt visibles en el SO | Requisito explícito de cifrado en el invitado |
| **Confidential** | VMs confidenciales | Clave ligada al vTPM | Datos altamente sensibles |

## 💻 Laboratorio: cifrado de discos

1. Crear un Key Vault con soft delete, purge protection y **enabled-for-disk-encryption**.
2. Habilitar ADE en una VM de prueba con `az vm encryption enable --volume-type All` y verificar el estado.
3. Comprobar en el SO (Windows: `manage-bde -status`) que el volumen está cifrado.
4. Deshabilitar ADE, desasignar la VM y habilitar **encryption at host** (registrando antes la característica).
5. Crear un disk encryption set y cambiar un disco de datos a CMK.

## AZ-104 Exam Tips

- 🔥 🧠 **ADE = BitLocker/dm-crypt en el invitado**; **Key Vault en la misma región y suscripción**, con "enabled for disk encryption".
- 🔥 🧠 ADE **no soportado** en series **Basic y A**, ni en **Ultra / Premium SSD v2**, ni junto a encryption at host o SSE+CMK en el mismo disco.
- 🧠 **Encryption at host** cubre disco temporal y caché; requiere **registrar la característica** y VM desasignada.
- 🧠 SSE con CMK usa un **Disk Encryption Set** + identidad administrada.
- 💻 `az vm encryption enable/show/disable`, `Set-AzVMDiskEncryptionExtension`.
- 📌 SSE (plataforma) vs ADE (invitado) vs encryption at host (host).
- ⚠️ Borrar el Key Vault o la clave deja la VM **sin arrancar**.

## Errores comunes

- Crear el Key Vault en otra región que la VM para ADE.
- Intentar ADE en una VM `Standard_A2_v2` o con Ultra Disk.
- Combinar ADE y encryption at host en la misma VM.

## Preguntas que podrían aparecer

**1.** Habilitas Azure Disk Encryption en una VM y falla. El Key Vault existe en otra región. ¿Qué debes hacer?
- A) Habilitar purge protection · B) Crear un Key Vault en la misma región y suscripción que la VM · C) Cambiar el tamaño de la VM · D) Usar Ultra Disk

<details><summary>Respuesta</summary>

**B.** ADE exige que el Key Vault esté en la misma región y suscripción que la VM, con la opción de cifrado de disco habilitada.
</details>

**2.** Un requisito exige cifrar también el disco temporal y la caché del host, sin instalar agentes en el sistema operativo. ¿Qué configuras?
- A) SSE con PMK · B) Azure Disk Encryption · C) Encryption at host · D) Acceso anónimo deshabilitado

<details><summary>Respuesta</summary>

**C.** Encryption at host cifra los datos en el host, incluidos disco temporal y cachés, sin agente en el invitado.
</details>

## Relacionado

- [[06 - Discos administrados]]
- [[04 - Máquinas virtuales - creación y configuración]]
- [[06 - Cifrado de cuentas de almacenamiento]]
- [[Azure Key Vault]] (AZ-900)
- [[00 - Índice - Cómputo]]
