---
tags: [az-104, azure, almacenamiento, files, snapshots, soft-delete]
modulo: Almacenamiento
peso_examen: Medio-Alto
---

# Instantáneas y eliminación temporal en Azure Files

## ¿Qué es?

- **Instantánea (share snapshot)**: copia **de solo lectura, en un punto en el tiempo, de un recurso compartido completo**. Es incremental (solo se almacenan los cambios) y permite recuperar archivos individuales o todo el recurso.
- **Eliminación temporal (soft delete)**: conserva los **recursos compartidos eliminados** durante un período de retención para poder **restaurarlos**.

## ¿Para qué sirve?

- Deshacer borrados o modificaciones accidentales de archivos ("Versiones anteriores" en Windows).
- Recuperar un recurso compartido borrado por error.
- Base de **Azure Backup** para Files (que crea y gestiona snapshots).

## Conceptos clave

### Snapshots
- Hasta **200 snapshots** por recurso compartido 🧠.
- Se crean **manualmente** (portal, CLI, PowerShell, REST) o **automáticamente por Azure Backup** según una política.
- Ocupan solo el **delta**; se facturan por espacio diferencial.
- **No se pueden modificar**; se pueden eliminar individualmente. **Para borrar un recurso compartido hay que borrar antes sus snapshots** (o usar la opción de incluirlas).
- Acceso: portal (Instantáneas → examinar → descargar/restaurar), Windows **"Versiones anteriores"** (`\\cuenta.file.core.windows.net\share` → propiedades del archivo), UNC con `@GMT-...` o montaje de la snapshot en Linux.
- **Restaurar**: un archivo (copiar desde la snapshot) o **todo el recurso compartido** (restaurar snapshot, sobrescribe el estado actual).
- No caducan por sí solas; para limpiarlas: Azure Backup (política) o borrado manual/script.
- Azure File Sync también crea snapshots durante la sincronización inicial.

### Soft delete
- Se configura **a nivel de cuenta** para el servicio Files; retención **1-365 días** 🧠 (por defecto 7 en el portal; habilitado por defecto en cuentas nuevas).
- Un recurso compartido eliminado temporalmente **conserva sus snapshots** y se **restaura con todo**.
- Se factura durante la retención (Standard: capacidad usada; Premium: capacidad aprovisionada).
- Deshabilitar soft delete no purga lo ya eliminado; se puede **eliminar permanentemente** (purge) desde el portal.
- ⚠️ No protege archivos individuales: para eso, snapshots o Backup. Tampoco protege la cuenta de almacenamiento.

## Cómo funciona

```bash
# Snapshot
az storage share snapshot --account-name st001 --name share1 --account-key <key>
az storage share list --account-name st001 --include-snapshots --account-key <key> -o table
# Restaurar un archivo desde una snapshot (copiando)
az storage file copy start --account-name st001 --destination-share share1 --destination-path docs/plan.docx \
  --source-share share1 --source-path docs/plan.docx --source-snapshot "2026-09-01T10:00:00.0000000Z" --account-key <key>
# Soft delete (a nivel de cuenta)
az storage account file-service-properties update --account-name st001 --resource-group rg --enable-delete-retention true --delete-retention-days 14
# Listar y restaurar recursos compartidos eliminados
az storage share-rm list --storage-account st001 --resource-group rg --include-deleted
az storage share-rm restore --storage-account st001 --resource-group rg --name share1 --deleted-version <version>
```

```powershell
$share = Get-AzRmStorageShare -ResourceGroupName rg -StorageAccountName st001 -Name share1
New-AzRmStorageShare -ResourceGroupName rg -StorageAccountName st001 -Name share1 -Snapshot
Update-AzStorageFileServiceProperty -ResourceGroupName rg -StorageAccountName st001 -EnableShareDeleteRetentionPolicy $true -ShareRetentionDays 14
Get-AzRmStorageShare -ResourceGroupName rg -StorageAccountName st001 -IncludeDeleted
Restore-AzRmStorageShare -ResourceGroupName rg -StorageAccountName st001 -Name share1 -DeletedShareVersion <version>
```

Portal: recurso compartido → **Instantáneas** → Agregar / examinar / restaurar. Cuenta → Recursos compartidos de archivos → **Eliminación temporal** (retención) → "Mostrar recursos eliminados" → Restaurar.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Recuperar un archivo modificado ayer | Snapshot de ayer → "Versiones anteriores" o copiar desde la snapshot |
| Recuperar un recurso compartido eliminado hace 3 días (retención 14) | Soft delete → restaurar |
| No se puede eliminar un recurso compartido | Tiene snapshots → eliminarlas primero |
| Snapshots automáticas diarias con retención de 30 días | **Azure Backup** para Azure Files (Recovery Services vault) |
| Proteger frente a borrado de la cuenta | Lock CanNotDelete (soft delete no cubre la cuenta) |
| Backups fuera de la cuenta (protección frente a ransomware que borre la cuenta) | **Vaulted backup** de Azure Files en Backup vault ➕ |

## Ejemplo

Cada noche Azure Backup crea una snapshot de `share1` con retención de 30 días. Un usuario borra la carpeta `contratos`. El administrador abre el recurso compartido en el portal → Instantáneas → snapshot de anoche → restaura la carpeta sin afectar al resto. Semanas después, otro administrador elimina `share1` completo: como el soft delete está en 14 días, lo restaura con todas sus snapshots.

## Comparaciones

| Mecanismo | Nivel | Protege frente a | Cuándo utilizarlo |
|---|---|---|---|
| **Snapshot** | Recurso compartido (punto en el tiempo) | Cambios/borrados de archivos | Recuperación granular de archivos |
| **Soft delete** | Recurso compartido eliminado | Borrado del recurso compartido | Siempre habilitado |
| **Azure Backup (snapshot-based)** | Gestión de snapshots | Igual que snapshot + política y retención | Backups programados |
| **Azure Backup (vaulted)** ➕ | Copia en Backup vault | Borrado de cuenta, ransomware | Protección adicional fuera de la cuenta |
| **Azure File Sync** | Servidor local | Pérdida del servidor | Caché local + nube |

## AZ-104 Exam Tips

- 🔥 🧠 Hasta **200 snapshots** por recurso compartido; son **de solo lectura e incrementales**.
- 🔥 🧠 Soft delete de Files: **1-365 días**, a nivel de **cuenta**, restaura el recurso **con sus snapshots**.
- 📌 Snapshot = archivos individuales; soft delete = recurso compartido completo.
- 🧠 No se puede borrar un recurso compartido con snapshots sin borrarlas.
- 💻 Crear snapshot, restaurar archivo desde "Versiones anteriores", habilitar soft delete, restaurar recurso eliminado.
- ⚠️ Azure Backup para Files guarda las snapshots **en la misma cuenta** (modelo snapshot); para copia fuera, vaulted backup.

## Errores comunes

- Confundir soft delete de Blob con el de Files (configuraciones separadas).
- Creer que soft delete permite recuperar un archivo individual.
- Olvidar la retención de snapshots (crecen sin límite si no hay política).

## Preguntas que podrían aparecer

**1.** Un usuario sobrescribió un documento en un recurso compartido de Azure Files esta mañana. Existe una instantánea de anoche. ¿Cuál es la forma más sencilla de recuperar la versión anterior?
- A) Restaurar el recurso compartido completo · B) Abrir "Versiones anteriores" del archivo en Windows o copiar desde la instantánea en el portal · C) Restaurar desde soft delete · D) Usar AzCopy sync

<details><summary>Respuesta</summary>

**B.** Las instantáneas permiten recuperar archivos individuales; restaurar todo el recurso perdería los cambios posteriores de otros usuarios.
</details>

**2.** Intentas eliminar un recurso compartido y recibes un error. ¿Cuál es la causa más probable?
- A) Está en nivel Cool · B) Tiene instantáneas · C) Tiene soft delete habilitado · D) Es NFS

<details><summary>Respuesta</summary>

**B.** Hay que eliminar las instantáneas (o marcar la opción de incluirlas) antes de eliminar el recurso compartido.
</details>

## Relacionado

- [[13 - Azure Files]]
- [[12 - Versionado, instantáneas y eliminación temporal de Blob]]
- [[10 - Operaciones de copia de seguridad y restauración]]
- [[16 - Azure File Sync]]
- [[00 - Índice - Almacenamiento]]
