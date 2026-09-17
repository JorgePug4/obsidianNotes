---
tags: [az-104, azure, almacenamiento, blob, contenedores]
modulo: Almacenamiento
peso_examen: Alto
---

# Azure Blob Storage

## ¿Qué es?

**Blob Storage** es el servicio de **objetos** de Azure: almacena cualquier archivo (texto, binario, imágenes, backups, logs) accesible por **HTTPS/REST**, organizado en **contenedores** dentro de la cuenta. Fundamentos en [[Azure Blob Storage]] (AZ-900). En AZ-104: crear y configurar contenedores, niveles de acceso, tipos de blob y opciones de protección.

## ¿Para qué sirve?

- Servir contenido a aplicaciones y sitios web estáticos.
- Almacenar copias de seguridad, archivos de log, datos para análisis (Data Lake Gen2).
- Streaming de audio/vídeo.

## Conceptos clave

- **Jerarquía**: cuenta → contenedor → blob. Es **plana**: las "carpetas" son prefijos en el nombre (`2026/enero/foto.jpg`). Con **espacio de nombres jerárquico** (Data Lake Gen2) las carpetas son reales.
- **Tipos de blob** 🧠:
  - **Block blob**: archivos en general; hasta ~190 TiB; se sube por bloques.
  - **Append blob**: optimizado para **anexar** (logs); no se modifica lo escrito.
  - **Page blob**: acceso aleatorio en páginas de 512 bytes; **discos VHD**; hasta 8 TiB.
- **Nivel de acceso público del contenedor** 🧠: **Private** (sin acceso anónimo), **Blob** (lectura anónima de blobs, no listar), **Container** (lectura anónima y listado). Solo si la cuenta permite acceso anónimo (`Allow Blob anonymous access`, deshabilitado por defecto en cuentas nuevas).
- **Niveles de acceso** (Hot/Cool/Cold/Archive): ver [[10 - Niveles de acceso de Blob (Hot, Cool, Cold, Archive)]].
- **Metadatos y propiedades**: pares clave-valor por blob/contenedor; **índice de blobs (blob index tags)** para consultas.
- **Sitio web estático**: contenedor `$web`, documento de índice y de error, endpoint `https://<cuenta>.z6.web.core.windows.net`; combinar con Azure CDN/Front Door para dominio personalizado con HTTPS.
- **Protección**: soft delete (blobs y contenedores), versionado, snapshots, change feed, point-in-time restore, **inmutabilidad (WORM)**: retención basada en tiempo y retención legal (legal hold), a nivel de contenedor o de versión.
- **Concesión (lease)**: bloqueo de escritura/eliminación de un blob (infinito o 15-60 s).
- **Data Lake Storage Gen2**: GPv2 + espacio de nombres jerárquico; ACL POSIX; endpoint `dfs`.
- **Change feed**: registro ordenado de cambios (necesario para object replication).

## Cómo funciona

```bash
az storage container create --account-name st001 --name docs --public-access off --auth-mode login
az storage blob upload --account-name st001 --container-name docs --name informe.pdf --file ./informe.pdf --tier Cool --auth-mode login
az storage blob list --account-name st001 --container-name docs -o table --auth-mode login
az storage blob set-tier --account-name st001 --container-name docs --name informe.pdf --tier Archive
az storage blob delete --account-name st001 --container-name docs --name informe.pdf
az storage blob undelete ...   # si soft delete está activo
az storage container set-permission --account-name st001 --name public --public-access blob
az storage blob service-properties update --account-name st001 --static-website --index-document index.html --404-document 404.html
```

```powershell
$ctx = New-AzStorageContext -StorageAccountName st001 -UseConnectedAccount
New-AzStorageContainer -Name docs -Permission Off -Context $ctx
Set-AzStorageBlobContent -Container docs -File .\informe.pdf -Blob informe.pdf -StandardBlobTier Cool -Context $ctx
Get-AzStorageBlob -Container docs -Context $ctx
$blob = Get-AzStorageBlob -Container docs -Blob informe.pdf -Context $ctx
$blob.BlobClient.SetAccessTier("Archive")
Enable-AzStorageStaticWebsite -Context $ctx -IndexDocument index.html -ErrorDocument404Path 404.html
```

## Componentes / configuración de un contenedor

| Configuración | Opciones | Nota |
|---|---|---|
| Nombre | 3-63 caracteres, minúsculas, números y guiones | Único en la cuenta |
| Nivel de acceso público | Private / Blob / Container | Requiere acceso anónimo permitido en la cuenta |
| Directivas de acceso | Hasta 5 stored access policies | [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]] |
| Inmutabilidad | Time-based retention / Legal hold | WORM; "locked" es irreversible |
| Metadatos | Clave-valor | |
| Soft delete de contenedor | 1-365 días | A nivel de cuenta |
| Cifrado | Encryption scope | Por contenedor |

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Servir imágenes públicas sin autenticación | Cuenta con acceso anónimo permitido + contenedor con nivel **Blob** |
| Nadie debe poder listar el contenido pero sí leer con URL directa | Nivel **Blob** (no Container) |
| Alojar una SPA/HTML estático | **Sitio web estático** (`$web`) |
| Logs que solo se anexan | **Append blob** |
| Guardar un VHD | **Page blob** (o disco administrado) |
| Cumplimiento: nadie puede borrar ni modificar durante 7 años | **Inmutabilidad**: retención basada en tiempo (bloqueada) |
| Retención mientras dure un litigio, sin fecha | **Legal hold** |
| Recuperar un blob borrado hace 3 días | **Soft delete** (si estaba activo con retención ≥ 3 días) |
| Recuperar la versión anterior tras sobrescribir | **Versionado** o **snapshot** |

## Ejemplo

Contoso publica su web corporativa estática desde `$web`. Necesita además que los PDFs legales queden inmutables durante 5 años: se crea el contenedor `legal` con **directiva de inmutabilidad basada en tiempo de 1825 días** y se **bloquea** tras validar. Ningún administrador, ni siquiera Owner, podrá borrarlos antes.

## Comparaciones

| Tipo de blob | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Block blob** | Archivos y objetos | Grandes tamaños, niveles de acceso | Casi todo |
| **Append blob** | Logs, auditoría | Anexar eficiente | Escrituras solo al final |
| **Page blob** | Discos, acceso aleatorio | Lectura/escritura por páginas | VHD, bases de datos propias |

| Nivel de acceso del contenedor | Uso | Cuándo utilizarlo |
|---|---|---|
| **Private** | Solo autenticado | Por defecto, datos internos |
| **Blob** | Lectura anónima por URL | Contenido público sin listado |
| **Container** | Lectura y listado anónimos | Rara vez; contenido totalmente público |

## AZ-104 Exam Tips

- ⭐ **Block** (archivos), **Append** (logs), **Page** (discos).
- 🔥 🧠 Acceso anónimo: **Private / Blob / Container**; y la cuenta debe permitir acceso anónimo.
- 🧠 Sitio estático: contenedor **`$web`**.
- 🧠 Inmutabilidad: **time-based retention** (fecha) vs **legal hold** (sin fecha); una directiva **bloqueada** no se puede acortar ni eliminar.
- 💻 Crear contenedor, subir blob, cambiar nivel, habilitar sitio estático; `--auth-mode login`.
- 📌 Blob (HTTP, objetos) vs Files (SMB/NFS, montaje) vs Disk (VM).
- ⚠️ El nivel de acceso de la **cuenta** (Hot/Cool) es el predeterminado; cada blob puede tener el suyo.

## Errores comunes

- Configurar el contenedor como Blob y olvidar habilitar el acceso anónimo en la cuenta (sigue siendo privado).
- Bloquear una directiva de inmutabilidad de prueba (irreversible).
- Usar page blobs para archivos normales.

## Preguntas que podrían aparecer

**1.** Necesitas que los usuarios anónimos puedan descargar archivos de un contenedor mediante su URL, pero no puedan enumerar el contenido. ¿Qué nivel de acceso configuras?
- A) Private · B) Blob · C) Container · D) Public

<details><summary>Respuesta</summary>

**B.** El nivel Blob permite lectura anónima de blobs individuales sin listar el contenedor.
</details>

**2.** Una aplicación escribe registros de auditoría añadiendo líneas al final de un archivo varias veces por segundo. ¿Qué tipo de blob es el más adecuado?
- A) Block blob · B) Page blob · C) Append blob · D) Archive blob

<details><summary>Respuesta</summary>

**C.** Los append blobs están optimizados para operaciones de anexado.
</details>

**3.** Por normativa, ciertos documentos no pueden modificarse ni borrarse durante 10 años, y ningún administrador debe poder saltarse la regla. ¿Qué configuras?
- A) Lock CanNotDelete · B) Directiva de inmutabilidad basada en tiempo en estado bloqueado · C) Soft delete de 3650 días · D) RBAC Reader para todos

<details><summary>Respuesta</summary>

**B.** Solo la retención basada en tiempo bloqueada garantiza WORM sin posibilidad de anulación. El lock protege la cuenta, no los blobs individuales, y lo puede quitar un Owner.
</details>

## Relacionado

- [[10 - Niveles de acceso de Blob (Hot, Cool, Cold, Archive)]]
- [[11 - Administración del ciclo de vida de Blob]]
- [[12 - Versionado, instantáneas y eliminación temporal de Blob]]
- [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]]
- [[Azure Blob Storage]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
