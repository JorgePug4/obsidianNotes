---
tags: [az-104, azure, backup, restauracion, mars]
modulo: Monitorización y mantenimiento
peso_examen: Muy alto
---

# Operaciones de copia de seguridad y restauración

## ¿Qué es?

El día a día de Azure Backup: **habilitar la protección**, lanzar copias **bajo demanda**, **restaurar** (VM completa, discos, archivos individuales, recursos compartidos, bases de datos) y **supervisar los trabajos**.

## Conceptos clave

### Habilitar la protección 🧠
- Desde la **VM → Copia de seguridad**, desde el **vault → +Copia de seguridad** o desde **Backup center**.
- Se instala automáticamente la **extensión VMSnapshot** (Windows/Linux) en la VM.
- Requisitos: VM en la **misma región** que el vault; el agente de Azure funcionando; para copias **coherentes con la aplicación**, VSS en Windows y scripts pre/post en Linux.
- Tipos de coherencia 🧠: **coherente con la aplicación** (mejor), **coherente con el archivo**, **coherente con el bloqueo (crash)** (VM apagada).

### Restaurar una VM de Azure 🧠

| Opción | Qué hace | Cuándo |
|---|---|---|
| **Crear una VM nueva** | Despliega una VM completa desde el punto de restauración | Recuperación rápida, nombre nuevo |
| **Reemplazar los discos existentes** | Sustituye los discos de la VM original | Conservar la VM, su nombre e IP |
| **Restaurar discos** | Crea los discos y una plantilla; tú creas la VM | Control total, cambios de tamaño/red |
| **Restauración de archivos individuales (Item-level recovery)** | Monta el punto de restauración como unidad mediante un **script** descargable y se copian archivos | Recuperar unos pocos archivos |
| **Cross Region Restore** | Restaurar en la región secundaria | Desastre regional (requiere GRS + CRR) |
| **Cross Subscription Restore** ➕ | Restaurar en otra suscripción | Reorganizaciones |

- Se necesita una **cuenta de almacenamiento de staging** en la misma región para algunas restauraciones.
- La restauración **no borra** el punto de restauración.
- **Restauración de archivos**: el script monta los discos (iSCSI) en un equipo con el mismo SO; al terminar hay que **desmontar** desde el portal.

### Azure Files
- Restauración **completa** del recurso compartido o de **archivos y carpetas** individuales, en la ubicación original o alternativa, con opción de sobrescribir o crear copia.

### SQL Server en VM / SAP HANA
- Restauración a un **punto en el tiempo** (con logs), a la misma instancia u otra, o como archivos.

### Agente MARS (on-premises) 🧠
- Instala el **Microsoft Azure Recovery Services agent** en el servidor; protege **archivos, carpetas y estado del sistema** (no VMs completas).
- Requiere **credenciales del vault** y una **frase de contraseña** para el cifrado que el cliente debe custodiar (si se pierde, no hay recuperación).
- Hasta **3 copias al día**.
- **MABS/DPM** para cargas más complejas.

### Supervisión
- **Trabajos de copia de seguridad** en el vault (estado, duración, errores) y en **Backup center**.
- Reintentos automáticos ante fallos transitorios.

## Cómo funciona

```bash
# Habilitar protección
az backup protection enable-for-vm -g rg-bkp -v rsv-contoso --vm $(az vm show -g rg-web -n vm-web01 --query id -o tsv) --policy-name DefaultPolicy
# Copia bajo demanda
az backup protection backup-now -g rg-bkp -v rsv-contoso -c <container> -i <item> --retain-until 20-10-2026
# Listar puntos de recuperación
az backup recoverypoint list -g rg-bkp -v rsv-contoso -c <container> -i <item> -o table
# Restaurar discos
az backup restore restore-disks -g rg-bkp -v rsv-contoso -c <container> -i <item> \
  --storage-account ststaging --rp-name <recoveryPointName> --target-resource-group rg-restore
# Restauración de archivos (genera el script de montaje)
az backup restore files mount-rp -g rg-bkp -v rsv-contoso -c <container> -i <item> --rp-name <rp>
az backup restore files unmount-rp -g rg-bkp -v rsv-contoso -c <container> -i <item> --rp-name <rp>
# Trabajos
az backup job list -g rg-bkp -v rsv-contoso -o table
# Detener protección
az backup protection disable -g rg-bkp -v rsv-contoso -c <container> -i <item> --retain-recovery-points true
```

```powershell
Set-AzRecoveryServicesVaultContext -Vault $vault
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName rg-web -Name vm-web01 -Policy $pol
$item = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -Name vm-web01
Backup-AzRecoveryServicesBackupItem -Item $item
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $item
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName ststaging -StorageAccountResourceGroupName rg-bkp
Get-AzRecoveryServicesBackupJob
```

## Configuración relevante para el examen

| Escenario | Opción de restauración |
|---|---|
| La VM no arranca tras una actualización y debe conservar su nombre e IP | **Reemplazar los discos existentes** |
| Recuperar un servidor completo con otro nombre para pruebas | **Crear una VM nueva** |
| Recuperar 3 archivos borrados de una VM | **Restauración de archivos individuales** (script de montaje) |
| Restaurar en la región secundaria tras un desastre | **Cross Region Restore** (GRS + CRR) |
| Necesito cambiar el tamaño o la red al restaurar | **Restaurar discos** y crear la VM manualmente |
| Copia extra antes de un cambio importante | **Copia de seguridad ahora** con retención propia |
| Proteger archivos de un servidor físico on-premises | **Agente MARS** en un Recovery Services vault |
| Se perdió la frase de contraseña de MARS | Los datos **no se pueden recuperar** |
| Comprobar por qué falló la copia de anoche | **Trabajos de copia de seguridad** del vault |

## Ejemplo

Un ransomware cifra una VM de ficheros. El administrador abre el vault, selecciona el punto de restauración de hace 3 días y elige **Reemplazar los discos existentes** para conservar la identidad y la IP de la VM. Para dos documentos concretos de otra VM usa la **restauración de archivos individuales**, monta el punto como unidad, copia los archivos y desmonta.

## Comparaciones

| Método | Velocidad | Conserva identidad | Cuándo |
|---|---|---|---|
| **Crear VM nueva** | Media | No | Pruebas, recuperación paralela |
| **Reemplazar discos** | Media | **Sí** | Recuperar la VM original |
| **Restaurar discos** | Media | Manual | Control total |
| **Archivos individuales** | **Rápida** | Sí | Pocos archivos |
| **Snapshot de disco** | Rápida | Manual | Antes de un cambio puntual |
| **Site Recovery failover** | **Muy rápida (RPO bajo)** | Sí (en destino) | Desastre regional |

## 💻 Laboratorio: copia y restauración de VM

1. Crear el vault `rsv-lab` en la región de la VM y habilitar la copia con la política por defecto.
2. Lanzar **Copia de seguridad ahora** y seguir el trabajo hasta que termine.
3. Borrar un archivo dentro de la VM y usar **Restauración de archivos** para recuperarlo (montar y desmontar).
4. Probar **Restaurar discos** a un grupo de recursos nuevo con una cuenta de staging.
5. Detener la protección **conservando los datos** y comprobar que los puntos siguen disponibles.

## AZ-104 Exam Tips

- ⭐ Tres formas de restaurar una VM: **crear nueva**, **reemplazar discos**, **restaurar discos**; y **archivos individuales**.
- 🔥 🧠 **Reemplazar discos** conserva nombre, IP y configuración de la VM original.
- 🔥 🧠 La **restauración de archivos** usa un **script de montaje** y hay que **desmontar** al terminar.
- 🧠 La VM y el vault deben estar en la **misma región**; la restauración necesita una cuenta de **staging**.
- 🧠 **MARS** protege archivos, carpetas y estado del sistema, **no VMs**; la frase de contraseña es irrecuperable.
- 🧠 Coherencia: **aplicación > archivo > bloqueo**.
- 💻 `az backup protection enable-for-vm`, `backup-now`, `restore restore-disks`, `restore files mount-rp`.
- ⚠️ Cross Region Restore exige **GRS** y habilitación previa.

## Errores comunes

- Intentar restaurar en otra región sin CRR.
- Olvidar desmontar el punto de restauración tras recuperar archivos.
- Usar MARS esperando copias de VMs completas.

## Preguntas que podrían aparecer

**1.** Una VM crítica ha quedado corrupta y debe recuperarse conservando su nombre, su IP privada y sus etiquetas. ¿Qué opción de restauración eliges?
- A) Crear una VM nueva · B) Reemplazar los discos existentes · C) Restaurar discos y crear otra VM · D) Restauración de archivos

<details><summary>Respuesta</summary>

**B.** Reemplazar los discos mantiene el recurso de VM original con su identidad de red y configuración.
</details>

**2.** Un usuario ha borrado una carpeta dentro de una VM protegida. ¿Cuál es la forma más rápida de recuperarla?
- A) Restaurar la VM completa · B) Restauración de archivos individuales montando el punto de restauración · C) Cross Region Restore · D) Crear un snapshot

<details><summary>Respuesta</summary>

**B.** La recuperación a nivel de elemento monta el punto de restauración y permite copiar solo lo necesario.
</details>

**3.** ¿Qué componente se instala en la máquina virtual al habilitar Azure Backup?
- A) Azure Monitor Agent · B) La extensión VMSnapshot · C) El agente MARS · D) Dependency Agent

<details><summary>Respuesta</summary>

**B.** La extensión de instantánea coordina las copias coherentes con la aplicación.
</details>

## Relacionado

- [[08 - Azure Backup - Recovery Services vault y Backup vault]]
- [[09 - Directivas de copia de seguridad]]
- [[11 - Azure Site Recovery]]
- [[12 - Informes y alertas de copias de seguridad]]
- [[00 - Índice - Monitorización y mantenimiento]]
