---
tags: [az-104, azure, almacenamiento, blob, versioning, snapshots, soft-delete]
modulo: Almacenamiento
peso_examen: Alto
---

# Versionado, instantáneas y eliminación temporal de Blob

## ¿Qué es?

Tres mecanismos de **protección de datos** de Blob Storage frente a sobrescrituras y borrados accidentales:

- **Versionado (blob versioning)**: Azure guarda **automáticamente** una versión anterior cada vez que un blob se **modifica o elimina**.
- **Instantáneas (snapshots)**: copia de solo lectura de un blob en un momento dado, creada **manualmente** (o por una app).
- **Eliminación temporal (soft delete)**: los blobs y **contenedores** eliminados se conservan durante un período de retención y se pueden **restaurar**.

Complementos: **change feed** (registro de cambios), **point-in-time restore** (restaurar contenedores a un instante), **inmutabilidad** ([[09 - Azure Blob Storage]]).

## ¿Para qué sirve?

- Deshacer una sobrescritura (versionado o snapshot).
- Recuperar un blob o un contenedor borrado (soft delete).
- Cumplir requisitos de auditoría de cambios (change feed).

## Conceptos clave

### Versionado
- Se habilita a nivel de **cuenta**. Cada escritura crea una **versión** identificada por `versionId` (marca de tiempo).
- La **versión actual** es el blob "base"; al eliminar el blob, la versión actual pasa a ser una versión anterior (el blob "desaparece" pero las versiones siguen).
- Restaurar = **copiar** una versión anterior sobre la actual (o promoverla).
- Coste: cada versión ocupa espacio → combinar con **ciclo de vida** para borrar versiones antiguas.
- Requisito para **object replication** y para **point-in-time restore**.
- Con versionado, el soft delete protege versiones? Con versionado activo, al borrar un blob no hace falta soft delete para conservarlo (queda como versión), pero el soft delete sí protege las **versiones** borradas explícitamente.

### Snapshots
- Se crean a demanda (`az storage blob snapshot`, Storage Explorer, portal).
- Un blob puede tener muchas snapshots; se identifican por `snapshot=<timestamp>`.
- Para borrar un blob con snapshots hay que borrar las snapshots primero (o "delete with snapshots").
- Se cobran por los bloques diferentes respecto al blob base.
- Restaurar = promover una snapshot (copiarla sobre el blob base).

### Soft delete
- **Blobs**: retención **1-365 días** 🧠 (por defecto 7 en el portal). Cubre blobs, versiones y snapshots eliminados o sobrescritos (si no hay versionado). Restaurar con **Undelete**.
- **Contenedores**: retención 1-365 días; restaurar el contenedor completo (no se puede restaurar si ya existe otro con el mismo nombre).
- Los elementos eliminados temporalmente **siguen ocupando espacio** y se facturan.
- Deshabilitar el soft delete no borra los elementos ya eliminados temporalmente.
- Soft delete de blobs **no** protege frente a la eliminación de la **cuenta de almacenamiento** (para eso: lock o recuperación de cuenta dentro de 14 días vía soporte/portal).

### Point-in-time restore ➕
- Restaura **block blobs** de uno o varios contenedores a un estado anterior. Requiere **versionado, change feed y soft delete** habilitados. Retención hasta 365 días (menor que la de soft delete).

## Cómo funciona

```bash
# Habilitar protección
az storage account blob-service-properties update --account-name st001 --resource-group rg \
  --enable-versioning true --enable-delete-retention true --delete-retention-days 14 \
  --enable-container-delete-retention true --container-delete-retention-days 14 --enable-change-feed true
# Snapshots
az storage blob snapshot --account-name st001 --container-name docs --name informe.pdf --auth-mode login
az storage blob list --account-name st001 --container-name docs --include s --auth-mode login   # s=snapshots, v=versions, d=deleted
# Restaurar blob eliminado
az storage blob undelete --account-name st001 --container-name docs --name informe.pdf --auth-mode login
# Restaurar contenedor eliminado
az storage container restore --account-name st001 --name docs --deleted-version <version>
# Restaurar una versión anterior (copiar sobre la actual)
az storage blob copy start --account-name st001 --destination-container docs --destination-blob informe.pdf \
  --source-uri "https://st001.blob.core.windows.net/docs/informe.pdf?versionId=2026-09-01T10:00:00.0000000Z" --auth-mode login
```

```powershell
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName rg -StorageAccountName st001 -RetentionDays 14
Enable-AzStorageContainerDeleteRetentionPolicy -ResourceGroupName rg -StorageAccountName st001 -RetentionDays 14
Update-AzStorageBlobServiceProperty -ResourceGroupName rg -StorageAccountName st001 -IsVersioningEnabled $true
Get-AzStorageBlob -Container docs -IncludeDeleted -Context $ctx | Restore-AzStorageBlob ... # o (Get-AzStorageBlob ...).BlobBaseClient.Undelete()
```

Portal: cuenta → **Protección de datos** (versionado, soft delete de blobs y contenedores, change feed, point-in-time restore). Blob → **Versiones** / **Instantáneas**. Contenedor → "Mostrar blobs eliminados".

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Un usuario sobrescribió un archivo y hay que volver a la versión de ayer | **Versionado** (restaurar versión) o snapshot si existía |
| Un blob se borró hace 5 días; retención de soft delete = 7 | **Undelete** |
| Se borró un contenedor entero | **Soft delete de contenedores** → restaurar |
| Restaurar 100 contenedores al estado de hace 2 horas | **Point-in-time restore** |
| Requisito previo para object replication | **Versionado** (+ change feed en origen) |
| Reducir coste del versionado | Regla de ciclo de vida que borre versiones > N días |
| Proteger frente a borrado de la cuenta | **Lock** CanNotDelete |

## Ejemplo

Contoso habilita versionado y soft delete de blobs (30 días) y de contenedores (30 días). Un script defectuoso sobrescribe 5000 imágenes y borra el contenedor `thumbs`. Recuperación: restaurar el contenedor desde "contenedores eliminados" y, para las imágenes sobrescritas, promover la versión anterior (con un script que copia `versionId` anterior sobre la actual). El ciclo de vida elimina las versiones antiguas a los 60 días para contener el coste.

## Comparaciones

| Mecanismo | Automático | Protege frente a | Ventajas | Cuándo utilizarlo |
|---|---|---|---|---|
| **Versionado** | Sí, cada escritura | Sobrescritura y borrado | Sin intervención; base para replication/PITR | Siempre en datos importantes |
| **Snapshot** | No, manual | Cambios desde un punto elegido | Control del momento | Antes de operaciones arriesgadas |
| **Soft delete (blob)** | Sí | Borrado de blobs/versiones/snapshots | Undelete simple | Siempre (por defecto en cuentas nuevas) |
| **Soft delete (contenedor)** | Sí | Borrado de contenedores | Restaurar todo el contenedor | Siempre |
| **Point-in-time restore** | Sí (necesita los anteriores) | Cambios masivos | Restaurar a un instante | Cargas con muchos cambios |
| **Inmutabilidad** | Configurable | Cualquier modificación/borrado | WORM | Cumplimiento |
| **Azure Backup para Blobs** ➕ | Sí | Todo lo anterior + gestión central | Backup vault, operacional/vaulted | Gobernanza de backups |

## AZ-104 Exam Tips

- 🔥 📌 **Versionado = automático por escritura**; **snapshot = manual**; **soft delete = papelera con retención**.
- 🧠 Soft delete: **1-365 días**, para **blobs** y para **contenedores** (dos configuraciones distintas).
- 🧠 Soft delete **no** protege la cuenta ni Azure Files (Files tiene su propio soft delete, ver [[15 - Instantáneas y eliminación temporal en Azure Files]]).
- 🧠 Point-in-time restore requiere **versionado + change feed + soft delete**.
- 💻 Habilitar en "Protección de datos", listar con `--include d/v/s`, `undelete`, `container restore`.
- ⚠️ Los datos eliminados temporalmente **se facturan** hasta que expira la retención.
- ⚠️ Un blob con snapshots no se puede borrar sin borrar (o incluir) las snapshots.

## Errores comunes

- Confundir soft delete de blobs con el de contenedores (son opciones separadas).
- Habilitar versionado sin ciclo de vida y sorprenderse por el coste.
- Creer que el versionado protege frente al borrado de la cuenta.

## Preguntas que podrían aparecer

**1.** Un administrador eliminó un contenedor con 10 000 blobs hace 2 días. La cuenta tiene soft delete de blobs (14 días) pero no de contenedores. ¿Se puede recuperar el contenedor con su contenido?
- A) Sí, con Undelete de cada blob · B) No, sin soft delete de contenedores el contenedor y su contenido no son recuperables por este medio · C) Sí, con point-in-time restore · D) Sí, desde el ciclo de vida

<details><summary>Respuesta</summary>

**B.** El soft delete de blobs no cubre la eliminación del contenedor completo; hace falta el soft delete de contenedores (o una copia de seguridad).
</details>

**2.** Quieres que cada vez que se modifique un blob quede una copia de la versión anterior automáticamente. ¿Qué habilitas?
- A) Snapshots · B) Blob versioning · C) Soft delete · D) Change feed

<details><summary>Respuesta</summary>

**B.** El versionado crea versiones automáticamente en cada escritura; los snapshots son manuales.
</details>

## Relacionado

- [[09 - Azure Blob Storage]]
- [[11 - Administración del ciclo de vida de Blob]]
- [[07 - Replicación de objetos (Object Replication)]]
- [[15 - Instantáneas y eliminación temporal en Azure Files]]
- [[00 - Índice - Almacenamiento]]
