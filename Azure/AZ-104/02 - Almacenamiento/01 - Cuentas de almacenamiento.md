---
tags: [az-104, azure, almacenamiento, storage-account]
modulo: Almacenamiento
peso_examen: Alto
---

# Cuentas de almacenamiento (Storage Accounts)

## ¿Qué es?

La **cuenta de almacenamiento** es el espacio de nombres único que agrupa los servicios de datos de Azure Storage: **Blob** (objetos, incluido Data Lake Gen2), **Files** (SMB/NFS), **Queue** (mensajes) y **Table** (NoSQL). Fundamentos en [[Azure Storage - Cuentas de almacenamiento]] (AZ-900). Aquí: decisiones de creación que el examen pregunta.

## ¿Para qué sirve?

- Guardar archivos, backups, imágenes, logs, recursos compartidos y colas.
- Servir de destino de diagnósticos, exportaciones de coste, boot diagnostics, flujos de red.

## Conceptos clave

- **Nombre**: 3-24 caracteres, **minúsculas y números**, **único global** (forma parte del endpoint `https://<cuenta>.blob.core.windows.net`).
- **Tipos de cuenta** 🧠:

| Tipo | Rendimiento | Servicios | Redundancias | Cuándo |
|---|---|---|---|---|
| **Standard general-purpose v2 (GPv2)** | Estándar (HDD) | Blob, Files, Queue, Table, Data Lake Gen2 | LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS | Por defecto para casi todo |
| **Premium block blobs** | Premium (SSD) | Solo block/append blobs | LRS, ZRS | Alta tasa de transacciones, baja latencia |
| **Premium file shares (FileStorage)** | Premium (SSD) | Solo Files (SMB y **NFS**) | LRS, ZRS | Recursos compartidos exigentes, NFS |
| **Premium page blobs** | Premium (SSD) | Solo page blobs | LRS | Discos no administrados (legado) |

- **GPv1** y **Blob Storage (legacy)**: obsoletos; se pueden **actualizar a GPv2** (irreversible).
- **Endpoints**: por servicio (`blob`, `file`, `queue`, `table`, `dfs`, `web` para sitio estático). Opción de **endpoints con DNS de zona de Azure** ➕ para superar el límite de cuentas por suscripción.
- **Límites**: 250 cuentas por región y suscripción (por defecto; ampliable a 500), 5 PiB por cuenta (ampliable), 20 000 req/s por cuenta (estándar).
- Configuración al crear: suscripción, RG, nombre, **región**, **rendimiento (Standard/Premium)**, **redundancia**, y en pestañas avanzadas: **transferencia segura (HTTPS) obligatoria**, **permitir acceso anónimo a blobs**, **permitir acceso por clave**, **versión mínima de TLS (1.2)**, **espacio de nombres jerárquico (Data Lake Gen2)**, **nivel de acceso predeterminado (Hot/Cool/Cold)**, **SFTP**, **NFSv3**, **grandes recursos compartidos**, red (público / VNet seleccionadas / privado), protección de datos (soft delete, versionado, change feed, point-in-time restore), cifrado (CMK, cifrado de infraestructura).
- Cosas que **no se pueden cambiar** después: nombre, región, tipo de cuenta (salvo GPv1→GPv2), **cifrado de infraestructura**, **espacio de nombres jerárquico** (hoy se puede habilitar posteriormente en GPv2 con un proceso de actualización, pero no deshabilitar), NFSv3.
- Cosas que **sí se cambian**: redundancia (con restricciones, ver [[02 - Redundancia de almacenamiento]]), nivel de acceso predeterminado, red, claves, TLS mínimo.
- **Cifrado en reposo** siempre activo (SSE, AES-256). Ver [[06 - Cifrado de cuentas de almacenamiento]].

## Cómo funciona (creación)

```bash
az storage account create \
  --name stcontoso001 --resource-group rg-storage --location westeurope \
  --sku Standard_GZRS --kind StorageV2 --access-tier Hot \
  --min-tls-version TLS1_2 --https-only true --allow-blob-public-access false
az storage account show --name stcontoso001 --query "{sku:sku.name,kind:kind,tier:accessTier}"
az storage account update --name stcontoso001 --sku Standard_GRS   # cambiar redundancia
```

```powershell
New-AzStorageAccount -ResourceGroupName rg-storage -Name stcontoso001 -Location westeurope `
  -SkuName Standard_GZRS -Kind StorageV2 -AccessTier Hot -MinimumTlsVersion TLS1_2 -EnableHttpsTrafficOnly $true
Set-AzStorageAccount -ResourceGroupName rg-storage -Name stcontoso001 -SkuName Standard_GRS
```

SKUs en CLI/PowerShell: `Standard_LRS`, `Standard_ZRS`, `Standard_GRS`, `Standard_RAGRS`, `Standard_GZRS`, `Standard_RAGZRS`, `Premium_LRS`, `Premium_ZRS`. Kinds: `StorageV2`, `BlockBlobStorage`, `FileStorage`, `Storage` (v1), `BlobStorage`.

## Componentes

| Componente | Nota para el examen |
|---|---|
| Servicios de datos | Blob, Files, Queue, Table (Disk NO está dentro) |
| Claves de acceso (2) | Rotación; ver [[05 - Claves de acceso y autorización con Microsoft Entra ID]] |
| Redes | Firewall, service endpoints, private endpoints ([[03 - Firewalls y redes virtuales de Azure Storage]]) |
| Cifrado | MMK / CMK, infraestructura ([[06 - Cifrado de cuentas de almacenamiento]]) |
| Protección de datos | Soft delete, versionado, snapshots ([[12 - Versionado, instantáneas y eliminación temporal de Blob]]) |
| Replicación | Redundancia ([[02 - Redundancia de almacenamiento]]) y object replication ([[07 - Replicación de objetos (Object Replication)]]) |
| Sitio web estático | Contenedor `$web`, endpoint `web` |
| Diagnóstico/Insights | Storage insights ([[06 - Azure Monitor Insights (VM, Storage, Network)]]) |

## Configuración relevante para el examen

- "Necesito **NFS** para Files" → cuenta **Premium FileStorage**.
- "Necesito **Data Lake / análisis big data**" → GPv2 con **espacio de nombres jerárquico**.
- "Máximo rendimiento para blobs pequeños con muchas transacciones" → **Premium block blobs**.
- "Redundancia geográfica" → solo **Standard GPv2** (Premium solo LRS/ZRS).
- "Impedir acceso con clave; solo Entra ID" → deshabilitar **Allow storage account key access**.
- "Solo HTTPS" → **Secure transfer required** (rompe SMB sin cifrado? No: SMB 3.x con cifrado sigue funcionando; rompe acceso HTTP y clientes SMB antiguos).
- "Impedir contenedores públicos" → **Allow Blob anonymous access = Disabled** a nivel de cuenta.
- **Actualizar GPv1 → GPv2**: Configuración → Actualizar (sin tiempo de inactividad; puede cambiar la facturación).

## Ejemplo

Contoso necesita una cuenta para copias de seguridad de larga duración que deba sobrevivir a la pérdida de una región, con lectura desde la secundaria durante incidencias, y otra cuenta para un recurso compartido NFS de baja latencia. Solución: cuenta 1 = **Standard GPv2 con RA-GZRS** y nivel predeterminado Cool; cuenta 2 = **Premium FileStorage con ZRS** y NFS habilitado.

## Comparaciones

| Tipo de cuenta | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Standard GPv2** | Uso general | Todos los servicios, todas las redundancias, niveles de acceso | Casi siempre |
| **Premium block blobs** | Blobs con muchas operaciones | Latencia de un dígito de ms | Streaming, IoT, cargas interactivas |
| **Premium FileStorage** | Recursos compartidos exigentes | IOPS altas, NFS 4.1 | Bases de datos, HPC, Linux |
| **Premium page blobs** | Discos VHD sin administrar | Legado | Evitar; usar discos administrados |

## 💻 Laboratorio: crear y configurar una cuenta

1. Crear `st<iniciales>lab01` Standard GPv2, ZRS, Hot, TLS 1.2, HTTPS obligatorio, sin acceso anónimo.
2. Revisar los endpoints en "Puntos de conexión" y copiar la URL de blob.
3. Cambiar la redundancia a GRS y observar el estado "Replicación en curso".
4. Habilitar soft delete de blobs (7 días) y versionado en "Protección de datos".
5. Con CLI: `az storage account show --name ... --query primaryEndpoints`.

## AZ-104 Exam Tips

- ⭐ **GPv2** es la respuesta por defecto salvo que pidan Premium (latencia) o NFS (FileStorage).
- 🔥 🧠 **Premium = solo LRS/ZRS**; nada de GRS/GZRS.
- 🧠 Nombre 3-24, minúsculas+números, único global.
- 🧠 No se cambia: nombre, región, tipo (excepto v1→v2), cifrado de infraestructura.
- 💻 `az storage account create/update`, `New-AzStorageAccount`, `Set-AzStorageAccount`.
- 📌 Discos administrados **no** viven en la cuenta.
- ⚠️ "Secure transfer required" obliga HTTPS y SMB cifrado.

## Errores comunes

- Elegir Premium con GRS (no existe).
- Crear una cuenta FileStorage y esperar usar blobs en ella.
- Olvidar que el nivel de acceso de cuenta solo aplica a blobs (no a Files).

## Preguntas que podrían aparecer

**1.** Necesitas una cuenta de almacenamiento que ofrezca recursos compartidos NFS 4.1 para máquinas virtuales Linux. ¿Qué creas?
- A) Standard GPv2 con LRS · B) Premium FileStorage · C) Premium block blobs · D) Standard GPv2 con espacio de nombres jerárquico

<details><summary>Respuesta</summary>

**B.** NFS para Azure Files solo está disponible en cuentas Premium FileStorage.
</details>

**2.** ¿Cuál de estas propiedades NO puede modificarse tras crear una cuenta de almacenamiento?
- A) Redundancia · B) Nivel de acceso predeterminado · C) Región · D) Versión mínima de TLS

<details><summary>Respuesta</summary>

**C.** La región (y el nombre) son fijos; para cambiarla hay que crear otra cuenta y copiar los datos.
</details>

**3.** Necesitas almacenamiento de blobs con la latencia más baja posible para una aplicación con miles de operaciones pequeñas por segundo y solo tolera pérdida de datos si falla toda la región. ¿Qué eliges?
- A) Standard GPv2 GRS · B) Premium block blobs ZRS · C) Premium FileStorage LRS · D) Standard GPv2 Hot LRS

<details><summary>Respuesta</summary>

**B.** Premium block blobs da la latencia requerida; ZRS protege frente a fallo de zona (la única opción geográfica no existe en Premium).
</details>

## Relacionado

- [[02 - Redundancia de almacenamiento]]
- [[03 - Firewalls y redes virtuales de Azure Storage]]
- [[06 - Cifrado de cuentas de almacenamiento]]
- [[09 - Azure Blob Storage]]
- [[13 - Azure Files]]
- [[Azure Storage - Cuentas de almacenamiento]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
