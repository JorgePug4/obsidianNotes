---
tags: [az-900, azure, almacenamiento, storage-account]
---

# Azure Storage · Cuentas de almacenamiento

Clases 76 y 78-79 del curso (visión general + demos de creación).

## 1. Concepto

Una **cuenta de almacenamiento (Storage Account)** es el contenedor lógico donde viven tus datos en Azure. No es un disco ni un servidor: es un espacio de nombres único en Internet bajo el que Azure te ofrece varios servicios de datos gestionados.

**Qué problema resuelve:** guardar datos de forma duradera, segura y escalable sin comprar, mantener ni dimensionar hardware. Tú no gestionas discos, réplicas ni backups de infraestructura; Azure lo hace.

**Para qué se usa:** almacenar archivos, imágenes, backups, logs, recursos compartidos de red, colas de mensajes y tablas NoSQL simples, con acceso desde cualquier lugar por HTTPS.

## 2. Características principales

- **Nombre único global**: entre 3 y 24 caracteres, solo **minúsculas y números**. Forma parte de la URL pública.
- **Endpoints por servicio**, uno por cada tipo de dato:
  - Blob → `https://<cuenta>.blob.core.windows.net`
  - Files → `https://<cuenta>.file.core.windows.net`
  - Queue → `https://<cuenta>.queue.core.windows.net`
  - Table → `https://<cuenta>.table.core.windows.net`
- **Cuatro servicios de datos** dentro de la cuenta:

| Servicio | Tipo de dato | Uso típico |
|---|---|---|
| **Blob** | Objetos no estructurados | Imágenes, vídeo, backups, data lake |
| **Files** | Recursos compartidos de archivos | Unidad de red compartida (SMB/NFS) |
| **Queue** | Mensajes | Comunicación asíncrona entre componentes |
| **Table** | NoSQL clave-valor | Datos semiestructurados sencillos y baratos |

- **Cifrado en reposo automático** con Storage Service Encryption (AES de 256 bits). No se puede desactivar y no cuesta extra.
- **Cifrado en tránsito**: HTTPS obligatorio por defecto.
- Al crear la cuenta eliges: suscripción, grupo de recursos, **nombre**, **región**, **rendimiento** (Estándar o Premium, ver [[Almacenamiento Premium]]) y **redundancia** (ver [[Redundancia de almacenamiento]]).
- **Tipo de cuenta recomendado: Uso general v2 (General Purpose v2)**. Soporta todos los servicios y todos los niveles de acceso.

### Formas de dar acceso a los datos

| Mecanismo | Qué es | Cuándo se usa |
|---|---|---|
| **Claves de acceso** | Dos claves maestras con control total | Acceso administrativo, apps internas. Poco recomendable |
| **Firma de acceso compartido (SAS)** | Token con permisos y caducidad limitados | Dar acceso temporal a un cliente o socio |
| **Microsoft Entra ID + RBAC** | Identidad y roles | Opción recomendada por Microsoft |
| **Acceso anónimo** | Contenedor público de solo lectura | Web estática, imágenes públicas |

> [!warning] Confusión frecuente
> **Azure Disk Storage NO vive dentro de una cuenta de almacenamiento.** Los discos administrados son un recurso independiente de Azure. Muchos cursos los listan junto a Blob, Files, Queue y Table, pero solo esos cuatro son *servicios de datos de la cuenta de almacenamiento*.

## 3. Casos de uso

- Una tienda online guarda las fotos de producto en **Blob** y sirve la web estática desde ahí.
- Una empresa mueve su servidor de archivos a **Azure Files** y monta la unidad `Z:` desde los portátiles.
- Una app de pedidos mete cada pedido en una **Queue** para que el servicio de facturación lo procese cuando pueda.
- Un dispositivo IoT escribe telemetría barata en **Table Storage**.

## 4. Comparaciones

| Servicio de la cuenta | Estructura | Acceso | Elígelo cuando |
|---|---|---|---|
| **Blob** | Plana: cuenta → contenedor → blob | HTTP/HTTPS, REST, SDK | Guardas archivos u objetos que consume una app o la web |
| **Files** | Jerárquica: carpetas y subcarpetas | SMB / NFS (montaje como unidad) | Necesitas una unidad de red compartida entre varios equipos o VM |
| **Queue** | Cola de mensajes | REST | Desacoplas componentes de una aplicación |
| **Table** | Filas con clave de partición y de fila | REST | Necesitas NoSQL muy barato y sencillo, sin esquema |

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - Nombre de cuenta: **único a nivel mundial**, 3-24 caracteres, minúsculas y números.
> - Servicios de datos de una cuenta: **Blob, Files, Queue, Table**.
> - Tipo de cuenta por defecto y recomendado: **Uso general v2**.
> - El cifrado en reposo está **siempre activo** y es gratuito.
> - La redundancia se configura **a nivel de cuenta**, no de archivo.
> - El nivel de acceso (Hot/Cool/Cold/Archive) aplica solo a **Blob**, no a Files ni a discos.

## 6. Tips para AZ-900

> [!tip] Palabras clave que delatan la respuesta
> - "no estructurado", "objetos", "imágenes/vídeo", "acceso por HTTP" → **Blob**
> - "unidad de red", "SMB", "compartido entre varias VM", "ruta UNC \\\\servidor\\carpeta" → **Files**
> - "mensajes", "asíncrono", "desacoplar" → **Queue**
> - "clave-valor", "NoSQL barato y simple" → **Table**
> - "disco de la máquina virtual", "VHD" → **Disk**

> [!warning] Trampas habituales
> - Preguntar si puedes cambiar el **nombre** o la **región** de una cuenta ya creada: no, hay que crear otra y migrar.
> - Confundir *tipo de cuenta* (uso general v2) con *nivel de acceso* (Hot/Cool) y con *redundancia* (LRS/GRS). Son tres decisiones distintas del mismo formulario de creación.
> - Dar por hecho que Table Storage es lo mismo que [[Azure Cosmos DB]]. Cosmos DB es un servicio aparte, global y con SLA mucho más fuerte.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** ¿Cuál de los siguientes NO es un servicio de datos incluido en una cuenta de almacenamiento de Azure?

- A) Azure Blob Storage
- B) Azure Files
- C) Azure Managed Disks
- D) Azure Queue Storage

**Respuesta correcta: C.** Los discos administrados son un recurso independiente que se asocia a una máquina virtual. A, B y D sí forman parte de los servicios de datos de la cuenta, junto con Table Storage.

---

**Pregunta 2.** Una empresa necesita que los datos almacenados en Azure Storage estén cifrados en reposo. ¿Qué debe hacer?

- A) Activar manualmente el cifrado en la configuración de la cuenta
- B) Nada, el cifrado en reposo está habilitado de forma predeterminada
- C) Comprar la referencia Premium de la cuenta
- D) Implementar Azure Disk Encryption en la cuenta de almacenamiento

**Respuesta correcta: B.** Azure cifra todos los datos en reposo automáticamente con AES de 256 bits, sin coste ni configuración. A es falso porque no hay nada que activar. C mezcla rendimiento con seguridad. D aplica a discos de VM, no a cuentas de almacenamiento.

---

**Pregunta 3.** Necesitas almacenar 5 TB de vídeos de formación a los que accederán usuarios desde un portal web mediante HTTPS. ¿Qué servicio eliges?

- A) Azure Files
- B) Azure Blob Storage
- C) Azure Queue Storage
- D) Azure Disk Storage

**Respuesta correcta: B.** Vídeo es contenido no estructurado consumido por HTTP, el escenario clásico de Blob. A serviría para montar una unidad de red, no para servir contenido web. C es para mensajería. D solo se puede adjuntar a una máquina virtual.

## 🧠 Resumen para el examen

1. La cuenta de almacenamiento es el contenedor lógico de los datos y tiene **nombre único global**.
2. Contiene cuatro servicios: **Blob, Files, Queue, Table**.
3. **Los discos administrados no están dentro de la cuenta de almacenamiento.**
4. El tipo recomendado es **Uso general v2**.
5. Cifrado en reposo: **siempre activo, gratis, AES 256**.
6. Redundancia y región se eligen al crear la cuenta y **no cambian la región después**.
7. Acceso: claves, **SAS**, Entra ID + RBAC o acceso anónimo.
8. Endpoint distinto por servicio (`blob.`, `file.`, `queue.`, `table.core.windows.net`).
