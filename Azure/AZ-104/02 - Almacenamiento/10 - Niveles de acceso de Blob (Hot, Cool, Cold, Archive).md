---
tags: [az-104, azure, almacenamiento, blob, niveles-acceso, archive]
modulo: Almacenamiento
peso_examen: Alto
---

# Niveles de acceso de Blob (Hot, Cool, Cold, Archive)

## ¿Qué es?

Los **niveles de acceso (access tiers)** ajustan el precio de los **block blobs** a la frecuencia de uso: cuanto menos accedes, más barato almacenar y más caro leer. Fundamentos en [[Azure Archive Storage y niveles de acceso]] (AZ-900). En AZ-104: reglas de cambio de nivel, **rehidratación**, **cargos por eliminación anticipada** y automatización con ciclo de vida.

## Conceptos clave

| Nivel | Estado | Coste almacenamiento | Coste acceso | Retención mínima 🧠 | Latencia | Uso |
|---|---|---|---|---|---|---|
| **Hot** | Online | Más alto | Más bajo | Ninguna | ms | Datos activos |
| **Cool** | Online | Menor | Mayor | **30 días** | ms | Acceso infrecuente, ≥ 30 días |
| **Cold** | Online | Aún menor | Aún mayor | **90 días** | ms | Acceso raro, ≥ 90 días |
| **Archive** | **Offline** | Mínimo | Máximo + rehidratación | **180 días** | **horas** | Retención larga, casi nunca se lee |

- El nivel se define a nivel de **cuenta** (predeterminado: Hot, Cool o Cold; **Archive no** puede ser predeterminado) y **por blob** (lo sobrescribe).
- Solo **block blobs** en cuentas **GPv2 / Blob Storage / Premium block blob (sin niveles)**. Premium no tiene niveles.
- **Retención mínima / cargo por eliminación anticipada**: si borras o mueves a un nivel más caliente antes de 30/90/180 días, se cobra el resto del período.
- **Archive** 🧠: el blob **no se puede leer ni modificar** hasta **rehidratarlo**. Se pueden leer sus metadatos y propiedades. Dos formas de rehidratar:
  - **Cambiar el nivel** (Set Blob Tier) a Hot/Cool/Cold: el blob sale de Archive.
  - **Copiar** (Copy Blob) a un nuevo blob en nivel online: el original sigue en Archive (útil para conservar).
  - **Prioridad de rehidratación**: **Standard** (hasta **15 horas**) o **High** (menos de **1 hora** para objetos < 10 GB, más caro).
- Cambiar entre niveles online (Hot ↔ Cool ↔ Cold) es inmediato. Cool/Cold → Hot se cobra como lectura.
- Redundancia y Archive: Archive no está disponible en **ZRS/GZRS/RA-GZRS**? A fecha de estas notas Archive soporta LRS, GRS y RA-GRS (no las variantes con zonas). Si la cuenta es ZRS, no se puede archivar. ⚠️ Verificar en Learn si tu región lo cambia.
- Se puede definir el nivel al subir (`--tier`), con AzCopy (`--block-blob-tier`), con **ciclo de vida** ([[11 - Administración del ciclo de vida de Blob]]) o manualmente.

## Cómo funciona

```bash
az storage account update --name st001 --resource-group rg --access-tier Cool   # predeterminado de la cuenta
az storage blob set-tier --account-name st001 --container-name backups --name db.bak --tier Archive --auth-mode login
az storage blob set-tier --account-name st001 --container-name backups --name db.bak --tier Hot --rehydrate-priority High --auth-mode login
az storage blob show --account-name st001 --container-name backups --name db.bak --query "properties.{tier:blobTier,status:rehydrationStatus}" --auth-mode login
# Rehidratar copiando a otro blob
az storage blob copy start --account-name st001 --destination-container backups --destination-blob db-restore.bak --source-container backups --source-blob db.bak --tier Hot --rehydrate-priority Standard --auth-mode login
```

```powershell
$blob = Get-AzStorageBlob -Container backups -Blob db.bak -Context $ctx
$blob.BlobClient.SetAccessTier("Hot", $null, "High")   # rehidratación de alta prioridad
```

## Configuración relevante para el examen

| Enunciado | Nivel |
|---|---|
| Acceso frecuente, sin restricciones | Hot |
| Se lee pocas veces al mes, se conserva ≥ 30 días, acceso inmediato | Cool |
| Se lee un par de veces al año, ≥ 90 días, acceso inmediato | Cold |
| Se conserva años, "puede tardar horas en recuperarse", mínimo coste | Archive |
| Recuperar un blob de Archive en menos de 1 hora | Rehidratar con prioridad **High** |
| Recuperar de Archive sin perder la copia archivada | **Copiar** a un blob nuevo en nivel online |
| Mover automáticamente a Cool tras 30 días sin modificar | Regla de ciclo de vida |

## Ejemplo

Copias de seguridad diarias de 200 GB deben conservarse 7 años; se leen casi nunca, pero cuando se necesitan hay margen de un día. Solución: subir directamente en Cool (retención mínima 30 días) y una regla de ciclo de vida que las mueva a **Archive** a los 30 días y las elimine a los 7 años. Si un auditor pide una copia, rehidratación **Standard** (hasta 15 h) es suficiente y más barata.

## Comparaciones

| Nivel | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Hot** | Activo | Lectura barata | Datos de trabajo |
| **Cool** | Infrecuente | Almacenamiento barato, online | Backups recientes, datos de 1-3 meses |
| **Cold** | Muy infrecuente | Más barato que Cool, online | Datos de 3-12 meses, acceso raro pero inmediato |
| **Archive** | Retención | Mínimo coste | Cumplimiento, años sin acceso |

## AZ-104 Exam Tips

- 🔥 🧠 Retención mínima: **Cool 30 / Cold 90 / Archive 180 días**.
- 🔥 🧠 Rehidratación: **Standard ≤ 15 h**, **High < 1 h** (< 10 GB).
- 🧠 Archive es **offline**: no se puede leer; solo metadatos. **No** puede ser el nivel predeterminado de la cuenta.
- 🧠 Archive **no está disponible en cuentas ZRS/GZRS** ni en Premium.
- 💻 `az storage blob set-tier` con `--rehydrate-priority`.
- 📌 Set Blob Tier (cambia el blob) vs Copy Blob (conserva el archivado).
- ⚠️ Cambiar de Archive a Hot antes de 180 días genera cargo por eliminación anticipada.

## Errores comunes

- Elegir Archive cuando el enunciado exige acceso inmediato.
- Creer que se puede descargar directamente un blob archivado.
- Configurar Archive como nivel predeterminado de la cuenta.

## Preguntas que podrían aparecer

**1.** Un blob de 5 GB está en Archive y un auditor lo necesita en menos de una hora. ¿Qué haces?
- A) Descargarlo directamente · B) Cambiar el nivel a Hot con prioridad de rehidratación High · C) Cambiar el nivel a Cool con prioridad Standard · D) Copiarlo con AzCopy

<details><summary>Respuesta</summary>

**B.** La rehidratación de alta prioridad completa en menos de una hora para blobs menores de 10 GB.
</details>

**2.** Subes datos a Cool y los borras a los 10 días. ¿Qué ocurre?
- A) Nada · B) Se cobra el resto de los 30 días de retención mínima · C) Los datos pasan a Archive · D) Se bloquea la eliminación

<details><summary>Respuesta</summary>

**B.** Cool tiene 30 días de retención mínima; el borrado anticipado genera cargo proporcional.
</details>

## Relacionado

- [[09 - Azure Blob Storage]]
- [[11 - Administración del ciclo de vida de Blob]]
- [[02 - Redundancia de almacenamiento]]
- [[Azure Archive Storage y niveles de acceso]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
