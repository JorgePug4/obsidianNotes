---
tags: [az-104, azure, almacenamiento, azcopy, storage-explorer, herramientas]
modulo: Almacenamiento
peso_examen: Medio-Alto
---

# Azure Storage Explorer y AzCopy

## ¿Qué es?

- **Azure Storage Explorer**: aplicación de escritorio **gráfica** (Windows, macOS, Linux) para explorar y administrar blobs, archivos, colas, tablas, discos administrados y Data Lake, en Azure y en el emulador local (Azurite). Usa AzCopy por debajo para transferencias.
- **AzCopy**: utilidad de **línea de comandos** de alto rendimiento para copiar datos hacia/desde/entre cuentas de almacenamiento (Blob, Files, Data Lake) y desde AWS S3 / Google Cloud Storage.

Fundamentos en [[Movimiento y migración de datos]] (AZ-900). En AZ-104: **cómo autenticarse**, **qué comando** usar y **cuándo** cada herramienta.

## ¿Para qué sirve?

- Subir/descargar/mover archivos y carpetas completas.
- Sincronizar un directorio local con un contenedor.
- Copiar entre cuentas sin pasar por tu máquina (server-to-server).
- Administrar metadatos, niveles, SAS, snapshots desde una GUI.

## Conceptos clave

- **Autenticación de AzCopy** 🧠:
  - `azcopy login` → **Microsoft Entra ID** (requiere rol de datos, p. ej. Storage Blob Data Contributor). Válido para Blob y Data Lake; para **Files** vía Entra hace falta habilitar OAuth (nuevo) o usar SAS.
  - **SAS** en la URL (funciona para Blob y Files).
  - Identidad administrada (`azcopy login --identity`) en VMs.
  - **Clave de cuenta**: AzCopy v10 **no** usa la clave directamente (solo para generar SAS).
- **Comandos** 🧠: `copy` (cp), `sync`, `remove`, `list`, `make`, `jobs list/resume`, `bench`.
- `copy` es **server-to-server** cuando origen y destino son URLs de Azure (no pasa por el cliente).
- `sync`: replica cambios (por fecha de modificación); opción `--delete-destination` para borrar en destino lo que no está en origen. Solo Blob ↔ local, Blob ↔ Blob, Files ↔ local.
- Flags útiles: `--recursive`, `--include-pattern`, `--exclude-pattern`, `--block-blob-tier`, `--overwrite`, `--from-to`, `--preserve-smb-permissions` (Files), `--put-md5`, `--cap-mbps`.
- **Jobs**: cada operación crea un job reanudable (`azcopy jobs resume <id>`), útil ante interrupciones.
- **Storage Explorer**: conecta con cuenta de Entra, SAS, clave, cadena de conexión o emulador. Permite gestionar **directivas de acceso**, **generar SAS**, cambiar **nivel de acceso**, ver **snapshots** y **versiones**, **soft-deleted** items, **subir VHD a disco administrado**.

## Cómo funciona

```bash
# Autenticación con Entra ID
azcopy login --tenant-id <tenantId>
# Subir una carpeta a un contenedor
azcopy copy "C:\datos" "https://st001.blob.core.windows.net/docs" --recursive
# Descargar
azcopy copy "https://st001.blob.core.windows.net/docs/informe.pdf?<SAS>" "C:\descargas\informe.pdf"
# Copiar entre cuentas (server-to-server)
azcopy copy "https://stsrc.blob.core.windows.net/docs?<SAS>" "https://stdst.blob.core.windows.net/docs?<SAS>" --recursive
# Sincronizar (solo cambios) y borrar en destino lo que no exista en origen
azcopy sync "C:\web" "https://st001.blob.core.windows.net/\$web?<SAS>" --recursive --delete-destination=true
# Subir a Azure Files
azcopy copy "C:\compartido" "https://st001.file.core.windows.net/share1?<SAS>" --recursive --preserve-smb-permissions=true
# Copiar desde AWS S3
azcopy copy "https://s3.amazonaws.com/bucket/" "https://st001.blob.core.windows.net/s3?<SAS>" --recursive
# Establecer nivel al subir
azcopy copy "backup.tar" "https://st001.blob.core.windows.net/backups?<SAS>" --block-blob-tier Archive
azcopy jobs list
azcopy jobs resume <jobId>
```

Equivalentes en Azure CLI (menos rendimiento, útiles en scripts): `az storage blob upload-batch`, `az storage blob download-batch`, `az storage blob copy start`, `az storage file upload`.

PowerShell: `Set-AzStorageBlobContent`, `Get-AzStorageBlobContent`, `Start-AzStorageBlobCopy`, `Set-AzStorageFileContent`.

## Componentes / configuración relevante para el examen

| Necesidad | Herramienta |
|---|---|
| Subir 5 TB con scripts, máximo rendimiento, reanudable | **AzCopy copy** |
| Mantener sincronizado un directorio local con un contenedor cada noche | **AzCopy sync** programado |
| Copiar entre dos cuentas sin descargar | **AzCopy copy** URL→URL (server-to-server) |
| Administrar snapshots, SAS y niveles con interfaz gráfica desde macOS | **Storage Explorer** |
| Migrar desde S3 | **AzCopy** (o Data Factory) |
| Cientos de TB sin ancho de banda | **Azure Data Box** |
| Sincronizar servidores de archivos on-premises con Azure Files | **Azure File Sync** ([[16 - Azure File Sync]]) |
| Ver blobs eliminados (soft delete) o versiones | **Storage Explorer** (mostrar eliminados) o portal |

Permisos: AzCopy con Entra ID necesita **Storage Blob Data Contributor** (o Reader para solo descargar); con SAS, permisos `rwl` (`c` para crear, `d` para sync con delete).

## Ejemplo

Una empresa migra un servidor web de 800 GB a un contenedor `$web` (sitio estático). Se genera una SAS de escritura para el contenedor y se ejecuta `azcopy copy` con `--recursive`; el job se interrumpe por red y se reanuda con `azcopy jobs resume`. Luego se programa `azcopy sync` diario para publicar solo cambios.

## Comparaciones

| Herramienta | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **AzCopy** | CLI de transferencia masiva | Rendimiento, reanudable, sync, S3/GCS | Scripts, migraciones grandes |
| **Storage Explorer** | GUI multiplataforma | Visual, gestiona SAS/directivas/snapshots | Administración diaria |
| **Azure CLI / PowerShell** | Scripts generales | Integrado en automatización | Operaciones puntuales en scripts |
| **Portal** | Web | Sin instalar | Subidas pequeñas, configuración |
| **Data Box** | Dispositivo físico | Sin red | 40 TB a PB |
| **File Sync** | Sincronización continua | Caché local, niveles | Servidores de archivos |

## 💻 Laboratorio: AzCopy

1. Instalar AzCopy y ejecutar `azcopy login`.
2. Asignarte **Storage Blob Data Contributor** en una cuenta y subir una carpeta con `azcopy copy --recursive`.
3. Modificar dos archivos locales, borrar uno, y ejecutar `azcopy sync ... --delete-destination=true`; verificar el resultado.
4. Generar una SAS de contenedor y copiar a otra cuenta con `azcopy copy URL URL`.
5. Abrir Storage Explorer, conectar con Entra ID y cambiar el nivel de un blob a Cool.

## AZ-104 Exam Tips

- ⭐ "Línea de comandos", "script", "máximo rendimiento", "reanudar" → **AzCopy**. "Interfaz gráfica" → **Storage Explorer**.
- 🔥 🧠 `azcopy copy` (copiar) vs `azcopy sync` (solo cambios, con opción de borrar en destino).
- 🧠 AzCopy se autentica con **Entra ID (`azcopy login`)** o **SAS**; no con la clave.
- 🧠 Copia **cuenta a cuenta** sin pasar por el cliente.
- 💻 Reconocer la sintaxis `azcopy copy "<origen>" "<destino>?<SAS>" --recursive`.
- ⚠️ `sync` no soporta Files ↔ Files ni page blobs; usa `copy`.
- ⚠️ Para Entra ID con AzCopy hace falta un **rol de datos** (Contributor no basta).

## Errores comunes

- Usar AzCopy con Contributor de suscripción y recibir 403 (falta rol de datos).
- Olvidar `--recursive` al copiar carpetas.
- Confundir AzCopy con Azure File Sync (sincronización continua de servidores).

## Preguntas que podrían aparecer

**1.** Debes copiar 2 TB de blobs de la cuenta A a la cuenta B, en regiones distintas, sin que los datos pasen por tu equipo y pudiendo reanudar si se interrumpe. ¿Qué usas?
- A) Storage Explorer arrastrando carpetas · B) `azcopy copy` con URLs de origen y destino · C) Portal · D) `az storage blob upload-batch`

<details><summary>Respuesta</summary>

**B.** AzCopy realiza copias servidor a servidor y guarda jobs reanudables.
</details>

**2.** Ejecutas `azcopy login` y al copiar a un contenedor recibes "403 AuthorizationPermissionMismatch". Tienes el rol Contributor en la suscripción. ¿Qué haces?
- A) Asignarte Owner · B) Asignarte Storage Blob Data Contributor en la cuenta · C) Regenerar claves · D) Usar `--recursive`

<details><summary>Respuesta</summary>

**B.** AzCopy con Entra ID necesita permisos de plano de datos.
</details>

## Relacionado

- [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]]
- [[05 - Claves de acceso y autorización con Microsoft Entra ID]]
- [[16 - Azure File Sync]]
- [[Movimiento y migración de datos]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
