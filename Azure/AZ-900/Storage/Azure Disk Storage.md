---
tags: [az-900, azure, almacenamiento, discos]
---


## 1. Concepto

**Azure Disk Storage** ofrece discos virtuales **administrados** que se adjuntan a una máquina virtual y se comportan exactamente como un disco físico: el sistema operativo los ve como `C:`, `D:` o `/dev/sdX`, los formatea y escribe en ellos.

**Qué problema resuelve:** dar a una máquina virtual almacenamiento persistente de bloque, con el rendimiento y la durabilidad de un disco de servidor, sin comprar cabinas ni gestionar RAID.

**Para qué se usa:** disco del sistema operativo de una VM, discos de datos para bases de datos y aplicaciones, y migraciones *lift and shift* de servidores locales.

> [!important] "Administrado" significa que Azure gestiona la cuenta de almacenamiento subyacente por ti. Tú solo eliges tamaño y tipo de disco.

## 2. Características principales

- Son **discos de bloque persistentes**: los datos sobreviven al apagado y reinicio de la VM.
- Cada disco se adjunta normalmente a **una sola máquina virtual** a la vez.
- Una VM tiene: **disco de SO**, **disco temporal** (volátil, se pierde al reubicar la VM) y opcionalmente **discos de datos**.
- Técnicamente son **VHD** almacenados como page blobs, pero Azure lo administra.
- **Cifrado en reposo automático** (Azure Storage Service Encryption).
- Admiten **instantáneas (snapshots)** y copias de seguridad con Azure Backup.

### Tipos de disco, de más rápido a más barato

| Tipo | Medio | Uso típico en AZ-900 |
|---|---|---|
| **Ultra Disk** | SSD | Cargas extremas: SAP HANA, bases de datos de máxima exigencia |
| **Premium SSD** (y v2) | SSD | Producción y cargas críticas |
| **Standard SSD** | SSD | Servidores web, desarrollo y pruebas con uso ligero |
| **Standard HDD** | HDD | Backup, datos con acceso poco frecuente, lo más barato |

> [!warning] Confusión frecuente
> El **disco temporal** de una VM no es almacenamiento persistente. Nunca guardes datos importantes ahí: se borran si la VM se reubica en otro host. En el examen, "almacenamiento persistente para una VM" siempre apunta a **discos administrados**, no al disco temporal.

## 3. Casos de uso

- Una VM Windows con SQL Server usa un **Premium SSD** para los archivos de base de datos.
- Un servidor de desarrollo usa **Standard SSD** para abaratar.
- Un servidor de archivos heredado migrado tal cual desde el centro de datos usa discos de datos adicionales.
- Se toma una **instantánea** del disco antes de aplicar un parche, para poder revertir.

## 4. Comparaciones

| | **Disk Storage** | **Blob Storage** | **Azure Files** |
|---|---|---|---|
| Tipo de almacenamiento | Bloque | Objeto | Archivos compartidos |
| Quién lo usa | Una máquina virtual | Apps por HTTP | Varias VM y equipos a la vez |
| Se ve como | Unidad del sistema operativo | URL | Unidad de red mapeada |
| ¿Se comparte fácil? | No | Sí, por URL | Sí, ese es su propósito |
| Elígelo cuando | Necesitas el disco de una VM | Sirves archivos a una app | Varios equipos leen y escriben la misma carpeta |

**Standard HDD vs Standard SSD vs Premium SSD vs Ultra:** el criterio del examen siempre es el mismo. Más rendimiento y menor latencia → más caro. Pruebas y datos fríos → HDD. Producción crítica → Premium o Ultra.

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - Disk Storage = almacenamiento de **bloque** para **máquinas virtuales**.
> - Un disco se adjunta normalmente a **una sola VM**.
> - Tipos ordenados por rendimiento: **Ultra > Premium SSD > Standard SSD > Standard HDD**.
> - El **disco temporal es volátil**, los discos administrados son persistentes.
> - Los discos administrados **no** usan niveles Hot/Cool/Archive.
> - Los discos administrados **no** viven dentro de una cuenta de almacenamiento que tú gestionas.

## 6. Tips para AZ-900

> [!tip] Palabras clave
> - "máquina virtual", "disco del sistema operativo", "VHD", "IOPS" → **Disk Storage**
> - "SAP HANA", "latencia submilisegundo", "carga de trabajo más exigente" → **Ultra Disk**
> - "entorno de pruebas", "coste mínimo", "acceso poco frecuente" → **Standard HDD**
> - "compartido entre varias máquinas virtuales" → **Azure Files**, no Disk

> [!warning] Trampas habituales
> - Una pregunta puede describir "archivos a los que acceden varias VM al mismo tiempo". Suena a disco, pero la respuesta es [[Azure Files]].
> - Ofrecer "nivel Archive" como opción para discos: no existe, los niveles de acceso son exclusivos de [[Azure Blob Storage]].
> - Mezclar **Azure Backup** (copias de seguridad) con **instantánea de disco** (punto en el tiempo de un disco concreto).

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** Necesitas almacenamiento persistente para el sistema operativo de una máquina virtual de Azure. ¿Qué servicio utilizas?

- A) Azure Blob Storage
- B) Azure Disk Storage
- C) Azure Files
- D) Azure Table Storage

**Respuesta correcta: B.** Los discos administrados proporcionan el almacenamiento de bloque persistente que necesita el sistema operativo de una VM. A es almacenamiento de objetos por HTTP. C es un recurso compartido de red, no puede ser el disco de arranque. D guarda datos NoSQL tabulares.

---

**Pregunta 2.** Una empresa ejecuta una base de datos de producción con requisitos altos de rendimiento y baja latencia. ¿Qué tipo de disco es el más adecuado?

- A) Standard HDD
- B) Standard SSD
- C) Premium SSD
- D) Archive Storage

**Respuesta correcta: C.** Premium SSD es la opción recomendada para cargas de producción con rendimiento y latencia exigentes (Ultra se reserva para casos extremos como SAP HANA). A y B no ofrecen rendimiento suficiente para una base de datos de producción crítica. D ni siquiera es un tipo de disco.

---

**Pregunta 3.** Dos máquinas virtuales necesitan leer y escribir simultáneamente en la misma carpeta de documentos. ¿Qué solución debes recomendar?

- A) Adjuntar el mismo disco administrado a ambas VM
- B) Crear un recurso compartido en Azure Files y montarlo en ambas
- C) Usar Azure Queue Storage
- D) Usar el disco temporal de cada VM

**Respuesta correcta: B.** Azure Files está diseñado precisamente para acceso simultáneo mediante SMB desde varios equipos. A no es el escenario estándar de disco administrado. C es mensajería, no almacenamiento de archivos. D es volátil y local a cada VM.

## 🧠 Resumen para el examen

1. Disk Storage = **discos administrados de bloque** para máquinas virtuales.
2. Tipos: **Ultra, Premium SSD, Standard SSD, Standard HDD**, de más caro y rápido a más barato.
3. Un disco administrado se asocia normalmente a **una sola VM**.
4. El **disco temporal se pierde**; los discos administrados **persisten**.
5. Se cifran en reposo automáticamente y admiten **instantáneas**.
6. Si el escenario habla de **compartir** entre varias máquinas, la respuesta es Azure Files.
7. Los discos **no tienen niveles Hot/Cool/Archive**.
