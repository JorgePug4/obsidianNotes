---
tags: [az-900, azure, almacenamiento, blob]
---

# Azure Blob Storage

Clases 77 a 81 del curso (visión general, creación, blob en acción, sitio web estático).

## 1. Concepto

**Blob** viene de *Binary Large Object*. Es el almacenamiento de **objetos** de Azure: guarda archivos de cualquier tipo y tamaño sin imponerles una estructura.

**Qué problema resuelve:** guardar cantidades masivas de datos **no estructurados** (fotos, vídeos, PDF, backups, logs, datasets) de forma barata, con capacidad prácticamente ilimitada y accesibles desde cualquier punto del mundo por HTTP.

**Para qué se usa:** servir imágenes y vídeo a una web o app, guardar copias de seguridad, alojar un sitio web estático, y como base de un *data lake* para análisis.

### Jerarquía

```
Cuenta de almacenamiento
└── Contenedor (container)
    └── Blob (archivo)
```

Solo hay **un nivel de contenedores**: no existen contenedores dentro de contenedores. Lo que parecen "carpetas" son en realidad prefijos en el nombre del blob (`facturas/2026/ene.pdf` es un único nombre de blob).

## 2. Características principales

### Tipos de blob

| Tipo | Para qué sirve | Ejemplo |
|---|---|---|
| **Block blob** | Archivos y objetos en general. **El más habitual** | Imágenes, vídeos, documentos, backups |
| **Append blob** | Escritura al final del archivo | Logs, registros de auditoría |
| **Page blob** | Lecturas/escrituras aleatorias, hasta 8 TB | Discos duros virtuales (VHD) de las VM |

### Otros puntos clave

- Capacidad prácticamente **ilimitada**; pagas por lo que usas.
- Admite **niveles de acceso** para optimizar coste: Hot, Cool, Cold y Archive (ver [[Azure Archive Storage y niveles de acceso]]).
- **Administración del ciclo de vida**: reglas automáticas que mueven blobs a niveles más baratos o los eliminan pasados X días. Es la respuesta correcta cuando la pregunta dice "reducir costes automáticamente".
- **Versionado, instantáneas y eliminación temporal (soft delete)** para protegerse de borrados accidentales.
- **Nivel de acceso público del contenedor**: privado (por defecto), blob (lectura anónima de los archivos) o contenedor (lectura anónima incluida la lista de archivos).
- **Azure Data Lake Storage Gen2** es Blob Storage con espacio de nombres jerárquico activado, orientado a big data. *Para AZ-900 basta con saber que existe y que se apoya en Blob.*

### Sitio web estático (clase 81)

Blob Storage puede servir un sitio web estático (HTML, CSS, JS, imágenes) sin ningún servidor. Al activarlo, Azure crea un contenedor especial llamado **`$web`** y te da un endpoint público.

- Sirve **solo contenido estático**. Nada de PHP, .NET, Node ni bases de datos del lado servidor.
- Si necesitas backend, dominio propio con SSL gratuito o CI/CD desde GitHub, lo correcto es **Azure Static Web Apps** o **Azure App Service**.

> [!warning] Confusión frecuente
> Un contenedor **no es una carpeta** y tampoco tiene nada que ver con los contenedores de Docker ni con Azure Container Instances. En el examen, "contenedor" dentro del contexto de almacenamiento siempre significa el agrupador de blobs.

## 3. Casos de uso

- Una app móvil sube las fotos de perfil de los usuarios a un contenedor de Blob.
- Una empresa guarda 10 años de backups de base de datos moviéndolos automáticamente a Archive al mes de creados.
- Una plataforma de streaming sirve vídeo directamente desde Blob con Azure CDN delante.
- Un equipo de datos deja ficheros CSV crudos en Blob para procesarlos después con herramientas de análisis.
- Una landing page de campaña se publica como sitio web estático en el contenedor `$web`.

## 4. Comparaciones

| | **Blob Storage** | **Azure Files** | **Disk Storage** |
|---|---|---|---|
| Tipo | Objetos | Recurso compartido de archivos | Bloques (disco) |
| Acceso | HTTP/HTTPS, REST, SDK | SMB / NFS, montaje como unidad | Solo adjunto a una VM |
| Estructura | Plana (contenedor → blob) | Carpetas y subcarpetas reales | Sistema de archivos del SO |
| Compartido | Muchos clientes por HTTP | Muchas VM y equipos a la vez | Normalmente **una sola VM** |
| Elígelo si | Sirves archivos a apps o web | Reemplazas un servidor de archivos | Necesitas el disco de una VM |

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - Blob = almacenamiento de **objetos** para datos **no estructurados**.
> - Jerarquía: **cuenta → contenedor → blob**. Un solo nivel de contenedores.
> - Tres tipos: **block** (archivos), **append** (logs), **page** (discos/VHD).
> - Contenedor especial **`$web`** para el sitio web estático.
> - La **administración del ciclo de vida** automatiza el cambio de nivel y el borrado.
> - Blob soporta niveles de acceso; **Files y Disk no**.

## 6. Tips para AZ-900

> [!tip] Palabras clave
> - "no estructurado", "objetos", "imágenes, vídeos, documentos" → **Blob**
> - "registros que solo se añaden al final", "logs" → **append blob**
> - "disco duro virtual", "VHD" → **page blob**
> - "reducir costes automáticamente con el tiempo" → **administración del ciclo de vida**
> - "sitio web sin servidor, solo HTML y CSS" → **sitio web estático en Blob** (o Static Web Apps)

> [!warning] Trampas habituales
> - Si el enunciado menciona **SMB, unidad de red o ruta UNC**, la respuesta es [[Azure Files]] aunque hablen de "archivos". Blob no se monta como unidad de forma nativa.
> - Si menciona **base de datos, código del servidor o API**, el sitio web estático de Blob queda descartado.
> - "Datos estructurados con relaciones y consultas SQL" nunca es Blob: eso es Azure SQL Database.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** Tu empresa necesita almacenar millones de imágenes y vídeos que consumirá una aplicación móvil mediante HTTPS. ¿Qué servicio es el más adecuado?

- A) Azure Files
- B) Azure Blob Storage
- C) Azure Table Storage
- D) Azure Disk Storage

**Respuesta correcta: B.** Imagen y vídeo son datos no estructurados accedidos por HTTP, el caso de uso central de Blob. A implicaría montar un recurso SMB, innecesario aquí. C guarda datos tabulares clave-valor, no binarios grandes. D requiere estar adjunto a una máquina virtual.

---

**Pregunta 2.** ¿Qué tipo de blob se utiliza para almacenar los discos duros virtuales (VHD) de las máquinas virtuales?

- A) Block blob
- B) Append blob
- C) Page blob
- D) Archive blob

**Respuesta correcta: C.** Los page blobs están optimizados para lecturas y escrituras aleatorias, que es lo que hace un disco. A es para archivos completos. B solo permite añadir al final, útil para logs. D no existe como tipo de blob: Archive es un **nivel de acceso**, no un tipo.

---

**Pregunta 3.** Una empresa quiere publicar una página de marketing compuesta únicamente por HTML, CSS, JavaScript e imágenes, con el mínimo coste y sin administrar servidores. ¿Qué solución cumple el requisito?

- A) Desplegar una máquina virtual con IIS
- B) Habilitar el sitio web estático en Azure Blob Storage
- C) Usar Azure Queue Storage
- D) Crear un recurso compartido en Azure Files

**Respuesta correcta: B.** Blob puede servir contenido estático desde el contenedor `$web` sin servidor y con coste mínimo. A obliga a administrar un servidor, justo lo que se quiere evitar. C es mensajería. D no publica contenido en Internet por HTTP.

## 🧠 Resumen para el examen

1. Blob almacena **objetos no estructurados**: imágenes, vídeo, backups, logs, datasets.
2. Jerarquía **cuenta → contenedor → blob**, sin contenedores anidados.
3. Tipos: **block** (lo normal), **append** (logs), **page** (VHD).
4. Admite **niveles de acceso** y **reglas de ciclo de vida** para ahorrar costes.
5. Puede servir un **sitio web estático** desde el contenedor `$web`, sin backend.
6. Se accede por **HTTP/HTTPS y REST**, no se monta como unidad de red.
7. Es la base de **Azure Data Lake Storage Gen2** (detalle menor en AZ-900).
8. Protección de datos: **soft delete, instantáneas y versionado**.
