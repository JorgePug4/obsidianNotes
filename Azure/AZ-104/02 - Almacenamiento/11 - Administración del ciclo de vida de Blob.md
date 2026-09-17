---
tags: [az-104, azure, almacenamiento, blob, lifecycle]
modulo: Almacenamiento
peso_examen: Alto
---

# Administración del ciclo de vida de Blob (Lifecycle Management)

## ¿Qué es?

La **administración del ciclo de vida** es un conjunto de **reglas** en la cuenta de almacenamiento que **mueven blobs entre niveles** (Hot → Cool → Cold → Archive) o **los eliminan** automáticamente según su antigüedad, para optimizar costes sin intervención manual.

## ¿Para qué sirve?

- Enfriar datos que dejan de usarse (logs, backups, imágenes antiguas).
- Cumplir retención: borrar a los N días.
- Limpiar versiones y snapshots antiguos.

## Conceptos clave

- Se define como **directiva JSON** con hasta **100 reglas** por cuenta 🧠.
- Cada regla tiene: nombre, `enabled`, **tipo** (`Lifecycle`), **definición** con **filtros** (`blobTypes`: blockBlob / appendBlob; `prefixMatch`: contenedor/prefijo; `blobIndexMatch`: etiquetas de índice) y **acciones** para blob base, snapshots y versiones.
- **Acciones** 🧠: `tierToCool`, `tierToCold`, `tierToArchive`, `enableAutoTierToHotFromCool`, `delete`. Para **snapshots** y **versiones**: `tierToCool/Cold/Archive` y `delete`.
- **Condiciones**: `daysAfterModificationGreaterThan` (última modificación), `daysAfterCreationGreaterThan` (creación), `daysAfterLastAccessTimeGreaterThan` (último acceso; requiere habilitar **seguimiento de tiempo de acceso**, access time tracking), `daysAfterLastTierChangeGreaterThan` (para Archive, evita cargo por eliminación anticipada).
- Solo **block blobs** y **append blobs** (append blobs: solo `delete`). GPv2, Premium block blob (solo delete, sin niveles) y Blob Storage.
- Se ejecuta **una vez al día**; los cambios en la directiva pueden tardar hasta **24 horas** en aplicarse y hasta 24 h más en ejecutarse 🧠.
- Las reglas de nivel **no** se aplican en cuentas con redundancia que no soporte el nivel destino (por ejemplo, Archive en ZRS).
- Combina bien con **versionado** (borrar versiones > 90 días) y **soft delete**.
- Rol necesario: Contributor/Storage Account Contributor (plano de control).

## Cómo funciona

```json
{
  "rules": [
    {
      "name": "enfriar-logs",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": { "blobTypes": ["blockBlob"], "prefixMatch": ["logs/app1"] },
        "actions": {
          "baseBlob": {
            "tierToCool":    { "daysAfterModificationGreaterThan": 30 },
            "tierToCold":    { "daysAfterModificationGreaterThan": 90 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 180 },
            "delete":        { "daysAfterModificationGreaterThan": 2555 }
          },
          "snapshot": { "delete": { "daysAfterCreationGreaterThan": 90 } },
          "version":  { "delete": { "daysAfterCreationGreaterThan": 90 } }
        }
      }
    }
  ]
}
```

```bash
az storage account management-policy create --account-name st001 --resource-group rg --policy @policy.json
az storage account management-policy show --account-name st001 --resource-group rg
az storage account blob-service-properties update --account-name st001 --resource-group rg --enable-last-access-tracking true
```

```powershell
$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction TierToCool -DaysAfterModificationGreaterThan 30
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToArchive -DaysAfterModificationGreaterThan 180
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction Delete -DaysAfterModificationGreaterThan 2555
$filter = New-AzStorageAccountManagementPolicyFilter -PrefixMatch "logs/app1" -BlobType blockBlob
$rule = New-AzStorageAccountManagementPolicyRule -Name enfriar-logs -Action $action -Filter $filter
Set-AzStorageAccountManagementPolicy -ResourceGroupName rg -StorageAccountName st001 -Rule $rule
```

Portal: cuenta → **Administración del ciclo de vida** → Agregar regla → ámbito (todos los blobs / filtrar por prefijo/tags) → tipos → condiciones (si-entonces) → filtros.

## Configuración relevante para el examen

| Escenario | Regla |
|---|---|
| Blobs no modificados en 30 días → Cool; 90 → Cold; 1 año → Archive | tierToCool 30 / tierToCold 90 / tierToArchive 365 sobre última modificación |
| Eliminar blobs a los 7 años | delete daysAfterModificationGreaterThan 2555 |
| Solo el contenedor `logs` | prefixMatch `logs` (formato `contenedor/prefijo`) |
| Basado en último **acceso** (lectura) | Habilitar access time tracking + daysAfterLastAccessTimeGreaterThan |
| Volver a Hot automáticamente si se vuelve a leer | enableAutoTierToHotFromCool |
| Limpiar versiones antiguas | acción sobre `version` con delete |
| Evitar cargo por eliminación anticipada al borrar desde Archive | daysAfterLastTierChangeGreaterThan ≥ 180 |

## Ejemplo

Una app guarda facturas PDF en `facturas/`. Requisito legal: conservar 10 años; en la práctica se consultan durante los 2 primeros meses. Regla: Hot al subir → Cool a los 60 días (modificación) → Archive a los 180 días → delete a los 3650 días. Además, borrar versiones antiguas a los 30 días para no pagar el versionado.

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Lifecycle management** | Automatizar niveles y borrado | Sin código, por reglas | Siempre que haya patrón temporal |
| **Cambio manual de nivel** | Casos puntuales | Inmediato | Un blob concreto |
| **AzCopy con `--block-blob-tier`** | Nivel al subir | Control en carga | Cargas masivas iniciales |
| **Azure Functions / Automation** ➕ | Lógica compleja | Cualquier condición | Reglas que lifecycle no cubre |
| **Retención de Backup** | Backups gestionados por Azure Backup | Integrado en la política | VMs, Files, discos |

## 💻 Laboratorio: regla de ciclo de vida

1. Crear contenedor `logs` y subir varios blobs.
2. Cuenta → Administración del ciclo de vida → Agregar regla `enfriar-logs`: block blobs, filtrar por `logs/`, mover a Cool si no se modifica en 1 día, eliminar a los 3 días (valores cortos para el laboratorio).
3. Ver el JSON generado en la vista de código.
4. Habilitar el seguimiento de último acceso y añadir una segunda regla por último acceso.
5. Volver en 24-48 h y comprobar los niveles (la ejecución es diaria).

## AZ-104 Exam Tips

- ⭐ Reglas **si-entonces** por **última modificación, creación o último acceso**, con acciones **tierToCool/Cold/Archive/delete**.
- 🔥 🧠 Se ejecuta **una vez al día**; la directiva tarda hasta **24 h** en surtir efecto.
- 🧠 Último acceso requiere **access time tracking**.
- 🧠 Solo **block/append blobs** (append: solo delete); Premium block blob solo delete.
- 🧠 Hasta **100 reglas**; prefijo en formato `contenedor/ruta`.
- 💻 Crear una regla en el portal y reconocer el JSON.
- ⚠️ Las reglas no rehidratan desde Archive (solo hacia Archive y delete).
- ⚠️ Archive no aplica en ZRS/GZRS.

## Errores comunes

- Esperar que la regla actúe inmediatamente.
- Usar la condición de último acceso sin habilitar el tracking.
- Poner el prefijo sin el nombre del contenedor.

## Preguntas que podrían aparecer

**1.** Creas una regla de ciclo de vida para mover a Archive los blobs no modificados en 90 días. Han pasado 2 horas y nada ha cambiado. ¿Por qué?
- A) La regla es inválida · B) Las reglas se ejecutan una vez al día y pueden tardar hasta 24 h · C) Falta el rol Blob Data Owner · D) Archive no existe en GPv2

<details><summary>Respuesta</summary>

**B.** La ejecución es diaria y los cambios de directiva tardan hasta 24 horas.
</details>

**2.** Quieres mover a Cool los blobs que no se han **leído** en 60 días. ¿Qué debes habilitar primero?
- A) Versionado · B) Change feed · C) Seguimiento del tiempo de último acceso · D) Soft delete

<details><summary>Respuesta</summary>

**C.** La condición `daysAfterLastAccessTimeGreaterThan` requiere access time tracking en la cuenta.
</details>

## Relacionado

- [[10 - Niveles de acceso de Blob (Hot, Cool, Cold, Archive)]]
- [[12 - Versionado, instantáneas y eliminación temporal de Blob]]
- [[09 - Azure Blob Storage]]
- [[00 - Índice - Almacenamiento]]
