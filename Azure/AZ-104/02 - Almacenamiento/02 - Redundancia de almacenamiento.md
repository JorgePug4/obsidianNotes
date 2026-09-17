---
tags: [az-104, azure, almacenamiento, redundancia, lrs, zrs, grs]
modulo: Almacenamiento
peso_examen: Alto
---

# Redundancia de almacenamiento

## ¿Qué es?

La **redundancia** define cuántas copias de los datos mantiene Azure Storage y dónde. Se elige por **cuenta** (no por contenedor ni blob). Fundamentos en [[Redundancia de almacenamiento]] (AZ-900); en AZ-104 hay que saber **qué puede cambiarse a qué**, qué exige cada escenario y cómo funciona la **conmutación por error**.

## Conceptos clave

| Opción | Copias | Dónde | Durabilidad | Protege frente a |
|---|---|---|---|---|
| **LRS** | 3 | Un centro de datos | 11 nueves | Fallo de disco/rack |
| **ZRS** | 3 | 3 zonas de disponibilidad de la región | 12 nueves | Fallo de un centro de datos/zona |
| **GRS** | 6 (3 LRS + 3 LRS) | Región primaria + región emparejada | 16 nueves | Fallo de región |
| **GZRS** | 6 (3 ZRS + 3 LRS) | Primaria en zonas + secundaria LRS | 16 nueves | Fallo de zona y de región |
| **RA-GRS / RA-GZRS** | Igual que GRS/GZRS | + **lectura** en la secundaria | 16 nueves | Igual + lectura durante incidencia |

- La replicación a la región secundaria es **asíncrona**; hay un **RPO** de minutos (últimos cambios pueden perderse en failover).
- **Regiones emparejadas**: la secundaria la decide Azure (West Europe ↔ North Europe). Algunas regiones nuevas no tienen par y no admiten GRS.
- **Endpoint secundario**: `<cuenta>-secondary.blob.core.windows.net`, solo legible con RA-*.
- **Failover administrado por Microsoft** (desastre) vs **failover iniciado por el cliente**: tú puedes iniciar el failover de la cuenta; la secundaria pasa a ser primaria y la cuenta queda en **LRS** (hay que reconfigurar GRS después). Se pierde el delta no replicado. **Failover planificado** ➕ (sin pérdida de datos, ambas regiones disponibles) está disponible para GRS/GZRS.
- Files (estándar) soporta LRS/ZRS/GRS/GZRS (Files no soporta RA-*: no hay lectura del secundario para archivos); **Premium** solo LRS/ZRS.
- **Última hora de sincronización** (Last Sync Time): propiedad que indica hasta cuándo están replicados los datos en la secundaria.

## Cómo cambiar la redundancia 🧠

- Se cambia en Configuración → Redundancia (o `az storage account update --sku`).
- Cambios **directos** permitidos: LRS ↔ GRS/RA-GRS; ZRS ↔ GZRS/RA-GZRS; GRS ↔ RA-GRS; GZRS ↔ RA-GZRS.
- Cambios que **implican zonas** (LRS → ZRS, GRS → GZRS, ZRS → LRS) requieren una **conversión** (migración iniciada por el cliente desde el portal, o solicitud de soporte) que puede tardar y tiene requisitos (no en cuentas con ciertas características, p. ej. NFSv3 o archive en algunos casos).
- Premium no puede pasar a geo.
- Blobs en nivel **Archive**: convertir a ZRS requiere rehidratar o esperar? A fecha de estas notas, las cuentas con blobs en Archive no pueden convertirse a ZRS/GZRS sin antes rehidratarlos.

```bash
az storage account update --name st001 --resource-group rg --sku Standard_RAGZRS
az storage account show --name st001 --query "{primary:primaryLocation,secondary:secondaryLocation,status:statusOfSecondary}"
az storage account failover --name st001 --resource-group rg   # failover iniciado por el cliente
```

```powershell
Set-AzStorageAccount -ResourceGroupName rg -Name st001 -SkuName Standard_RAGZRS
Get-AzStorageAccount -ResourceGroupName rg -Name st001 | Select-Object PrimaryLocation, SecondaryLocation, StatusOfSecondary
Invoke-AzStorageAccountFailover -ResourceGroupName rg -Name st001
```

## Configuración relevante para el examen

| Requisito del enunciado | Redundancia mínima |
|---|---|
| Menor coste, sin requisitos | LRS |
| Sobrevivir a la caída de un centro de datos dentro de la región | ZRS |
| Sobrevivir a la caída de la región | GRS |
| Sobrevivir a zona y región | GZRS |
| Leer los datos durante una interrupción regional sin failover | RA-GRS / RA-GZRS |
| Datos nunca deben salir de la región (residencia) | LRS o ZRS |
| Premium (block blobs / Files) | LRS o ZRS únicamente |
| Azure Files estándar con protección regional | GRS/GZRS (sin RA) |

## Ejemplo

Una app en West Europe guarda facturas. Legal exige que las facturas sobrevivan a la pérdida de la región y que se puedan **consultar** (no escribir) si West Europe cae. Solución: **RA-GRS** o **RA-GZRS**; la app lee del endpoint `-secondary` en caso de incidencia. Si además Finanzas quisiera resistencia a zona, RA-GZRS.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **LRS** | Coste mínimo | Barato, residencia local | Dev/test, datos reconstruibles, Premium |
| **ZRS** | HA regional | Sin pérdida ante fallo de zona, sincrónico | Producción en una región, requisitos de residencia |
| **GRS** | DR | Segunda región | Backups, datos críticos |
| **GZRS** | HA + DR | Lo mejor de ambos | Producción crítica |
| **RA-*** | DR con lectura | Lectura en secundaria sin failover | Apps que pueden funcionar en solo lectura |

## AZ-104 Exam Tips

- 🔥 🧠 **LRS 11 nueves, ZRS 12, GRS/GZRS 16.**
- 🔥 📌 **RA** = lectura del secundario; sin RA, el secundario no es accesible hasta el failover.
- 🧠 Tras un failover iniciado por el cliente la cuenta queda en **LRS**; hay que reactivar la geo-replicación.
- 🧠 Replicación geográfica **asíncrona** → puede haber pérdida de los últimos minutos (ver Last Sync Time).
- 🧠 Premium: **solo LRS y ZRS**. Files: **sin RA**.
- 💻 Cambiar SKU con `az storage account update --sku` y ejecutar failover con `az storage account failover`.
- ⚠️ Cambios que añaden/quitan zonas (LRS↔ZRS) son **conversiones**, no un simple cambio de SKU.

## Errores comunes

- Creer que GRS permite leer en la secundaria (solo RA-GRS).
- Elegir GZRS para una cuenta Premium.
- Confundir redundancia con copia de seguridad: la replicación también replica borrados.

## Preguntas que podrían aparecer

**1.** Tu cuenta usa GRS. La región primaria sufre una interrupción prolongada y Microsoft no ha iniciado el failover. Necesitas que las aplicaciones puedan **escribir** de nuevo lo antes posible. ¿Qué haces?
- A) Cambiar a RA-GRS · B) Iniciar un failover de cuenta administrado por el cliente · C) Esperar a Microsoft · D) Crear una cuenta nueva y AzCopy

<details><summary>Respuesta</summary>

**B.** El failover iniciado por el cliente convierte la secundaria en primaria (queda en LRS). A solo daría lectura. D perdería los datos no accesibles.
</details>

**2.** ¿Qué opción de redundancia ofrece protección frente a la caída de una zona de disponibilidad y de la región completa, con el mínimo coste?
- A) ZRS · B) GRS · C) GZRS · D) RA-GZRS

<details><summary>Respuesta</summary>

**C.** GZRS combina ZRS en la primaria con LRS en la secundaria. RA-GZRS añade lectura del secundario, que no se pide, y cuesta más.
</details>

## Relacionado

- [[01 - Cuentas de almacenamiento]]
- [[07 - Replicación de objetos (Object Replication)]]
- [[08 - Azure Backup - Recovery Services vault y Backup vault]]
- [[Redundancia de almacenamiento]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
