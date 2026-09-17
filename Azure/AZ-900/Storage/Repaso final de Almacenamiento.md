---
tags: [az-900, azure, almacenamiento, repaso]
---

# 🎯 Repaso final de Almacenamiento

Nota de consolidación de la Sección 7. Úsala el día antes del examen.

## 1. Los conceptos más importantes

1. **La cuenta de almacenamiento** es el contenedor lógico con nombre único global que agrupa Blob, Files, Queue y Table.
2. **Blob** guarda objetos no estructurados y es el único servicio con **niveles de acceso**.
3. **Files** es un recurso compartido montable por **SMB/NFS**; **Disk** es el disco de una **VM**.
4. Los **niveles de acceso** ajustan el coste a la frecuencia de uso: Hot, Cool, Cold, Archive.
5. La **redundancia** define cuántas copias y dónde: LRS, ZRS, GRS, GZRS y variantes RA.
6. **Rendimiento (Estándar/Premium)**, **nivel de acceso** y **redundancia** son tres decisiones independientes.
7. Para mover datos: **AzCopy** (CLI), **Storage Explorer** (GUI), **File Sync** (sincronización), **Data Box** (físico), **Azure Migrate** (cargas de trabajo).
8. **Cosmos DB** es la base de datos NoSQL global de Azure, no un servicio de almacenamiento de archivos.
9. El **cifrado en reposo** está siempre activo y es gratuito.
10. **Redundancia no es copia de seguridad.**

## 2. Tabla de servicios y su propósito

| Servicio               | Tipo de dato             | Cómo se accede                                            | Propósito principal                               |
| ---------------------- | ------------------------ | --------------------------------------------------------- | ------------------------------------------------- |
| [[Azure Blob Storage]] | Objetos no estructurados | HTTP/HTTPS, REST, SDK                                     | Imágenes, vídeo, backups, data lake, web estática |
| [[Azure Files]]        | Archivos compartidos     | SMB / NFS, unidad montada                                 | Sustituir servidores de archivos, rutas UNC       |
| [[Azure Disk Storage]] | Bloque                   | Adjunto a una VM                                          | Disco de SO y de datos de máquinas virtuales      |
| **Queue Storage**      | Mensajes                 | REST                                                      | Desacoplar componentes de una aplicación          |
| **Table Storage**      | NoSQL clave-valor        | REST                                                      | Datos tabulares sencillos al mínimo coste         |
| [[Azure Cosmos DB]]    | NoSQL multimodelo        | API (NoSQL, Mongo, Cassandra, Gremlin, Table, PostgreSQL) | Aplicaciones globales de baja latencia            |
| **Azure SQL Database** | Relacional               | T-SQL                                                     | Datos relacionales administrados                  |

## 3. Diferencias que más fácilmente se confunden

| Par confuso | Cómo distinguirlo |
|---|---|
| **Blob vs Files** | Por **HTTP/URL** → Blob. **Montado como unidad (SMB)** → Files |
| **Files vs Disk** | **Varias VM a la vez** → Files. **Una sola VM, disco del SO** → Disk |
| **Cool vs Cold** | Ambos en línea. Cold es **más barato de guardar, más caro de leer**, retención de **90** días frente a 30 |
| **Cold vs Archive** | Cold está **en línea**; Archive está **sin conexión** y exige **rehidratación de horas** |
| **GRS vs RA-GRS** | Solo **RA-GRS** permite **leer el secundario** sin conmutación por error |
| **ZRS vs GRS** | ZRS = **3 zonas, misma región**. GRS = **dos regiones** |
| **GZRS** | ZRS en primaria + **LRS** en secundaria. La "Z" no viaja |
| **Nivel de acceso vs redundancia vs rendimiento** | Coste por uso / copias y ubicación / HDD frente a SSD |
| **AzCopy vs Storage Explorer** | **CLI** frente a **interfaz gráfica** |
| **File Sync vs Data Box vs Azure Migrate** | Sincronización continua / dispositivo físico puntual / migración de cargas de trabajo |
| **Cosmos DB vs SQL Database** | **NoSQL global** frente a **relacional** |
| **Table Storage vs Cosmos DB** | Table es barato y regional; Cosmos es global, rápido y caro |
| **Redundancia vs Backup** | La redundancia replica también los borrados. Para deshacer errores: **soft delete, versionado o Azure Backup** |

## 4. Diez tips de examen

> [!tip] Estrategia
> 1. Localiza primero el **protocolo o la forma de acceso** del enunciado: HTTP → Blob; SMB/unidad → Files; disco de VM → Disk.
> 2. Cuando pidan **el menor coste** y no exijan acceso rápido, piensa en **Archive**. Si exigen acceso inmediato, Archive se descarta de golpe.
> 3. "Los datos no pueden salir de la región" **elimina cualquier opción con G** (GRS, GZRS, RA-GRS, RA-GZRS).
> 4. "Leer durante la interrupción" siempre significa **RA-GRS o RA-GZRS**.
> 5. **LRS 11 nueves; GRS y GZRS 16 nueves.** Si ves "cinco nueves", hablan de disponibilidad, no de durabilidad.
> 6. "Automatizar el ahorro de costes conforme envejecen los datos" → **administración del ciclo de vida** de Blob.
> 7. "Red insuficiente" o "cientos de TB" → **Azure Data Box**, nunca AzCopy.
> 8. Si la palabra **script** o **línea de comandos** aparece, la respuesta es **AzCopy**.
> 9. **Premium nunca es la opción más barata** y **nunca tiene redundancia geográfica**.
> 10. Si el enunciado menciona **relaciones, claves foráneas o T-SQL**, la respuesta no es Cosmos DB.

## 5. Diez preguntas de repaso tipo AZ-900

**1.** ¿Qué servicio deberías usar para almacenar copias de seguridad que se conservarán siete años y casi nunca se consultarán, con el mínimo coste posible?

- A) Blob en nivel Hot · B) Blob en nivel Archive · C) Azure Files · D) Premium SSD

<details><summary>Respuesta</summary>

**B.** Archive es el nivel más barato de almacenamiento y el escenario no exige acceso inmediato. A es el más caro de almacenar, C no tiene niveles de acceso y D es la opción más cara de todas.
</details>

---

**2.** Una empresa necesita que sus datos sobrevivan a la pérdida completa de una región de Azure. ¿Qué redundancia es la mínima que cumple el requisito?

- A) LRS · B) ZRS · C) GRS · D) Premium LRS

<details><summary>Respuesta</summary>

**C.** GRS replica en una segunda región emparejada. A protege solo dentro de un centro de datos y B solo dentro de la región. D sigue siendo local.
</details>

---

**3.** ¿Qué tipo de blob se usa para registros que solo se escriben añadiendo información al final?

- A) Block blob · B) Append blob · C) Page blob · D) Archive blob

<details><summary>Respuesta</summary>

**B.** Los append blobs están optimizados para operaciones de anexión, típicas de logs. A es para archivos completos, C para discos y D no existe como tipo de blob.
</details>

---

**4.** Necesitas montar una carpeta compartida en varias máquinas virtuales Windows y también en portátiles locales. ¿Qué servicio eliges?

- A) Blob Storage · B) Disk Storage · C) Azure Files · D) Queue Storage

<details><summary>Respuesta</summary>

**C.** Azure Files se monta por SMB desde la nube y desde local simultáneamente. A no se monta como unidad, B se adjunta a una sola VM y D es mensajería.
</details>

---

**5.** ¿Cuál de estas afirmaciones sobre GRS es verdadera?

- A) Permite leer los datos del secundario en cualquier momento · B) Replica de forma síncrona a la región secundaria · C) La región secundaria usa LRS · D) Guarda 3 copias en total

<details><summary>Respuesta</summary>

**C.** En la región secundaria Azure siempre usa LRS. A corresponde a RA-GRS, B es falso porque la replicación geográfica es asíncrona y D es incorrecto: son 6 copias.
</details>

---

**6.** Un administrador quiere subir manualmente unos archivos a una cuenta de almacenamiento usando una interfaz gráfica desde su Mac. ¿Qué herramienta usa?

- A) AzCopy · B) Azure Storage Explorer · C) Azure Data Box · D) Azure Migrate

<details><summary>Respuesta</summary>

**B.** Storage Explorer es una aplicación de escritorio con interfaz gráfica multiplataforma. A es de línea de comandos, C es un dispositivo físico y D migra cargas de trabajo.
</details>

---

**7.** ¿Qué elemento se configura a nivel de **cuenta de almacenamiento** y afecta a todos sus servicios?

- A) El tipo de blob · B) La opción de redundancia · C) El nivel Archive · D) El tamaño del disco

<details><summary>Respuesta</summary>

**B.** La redundancia se define para toda la cuenta. A se decide por blob, C solo se aplica a blobs individuales y D pertenece a los discos administrados, que son un recurso aparte.
</details>

---

**8.** Una aplicación global necesita una base de datos NoSQL con escrituras en varias regiones y latencia mínima. ¿Qué servicio cumple?

- A) Azure SQL Database · B) Azure Table Storage · C) Azure Cosmos DB · D) Azure Blob Storage

<details><summary>Respuesta</summary>

**C.** Cosmos DB ofrece distribución global con escrituras multirregión y milisegundos de latencia. A es relacional, B es regional y sin garantías de latencia, D no es una base de datos.
</details>

---

**9.** Una empresa debe transferir 90 TB a Azure desde una ubicación con conectividad muy limitada. ¿Qué solución recomiendas?

- A) AzCopy · B) Azure File Sync · C) Azure Data Box · D) Storage Explorer

<details><summary>Respuesta</summary>

**C.** Data Box permite trasladar decenas de terabytes físicamente. A, B y D dependen de la red, que es justamente la limitación descrita.
</details>

---

**10.** Necesitas el coste de almacenamiento más bajo posible para datos consultados un par de veces al año, pero con recuperación **inmediata**. ¿Qué nivel eliges?

- A) Hot · B) Cool · C) Cold · D) Archive

<details><summary>Respuesta</summary>

**C.** Cold es un nivel en línea más barato que Cool, pensado para acceso muy poco frecuente con recuperación instantánea. A y B cuestan más almacenar y D no permite acceso inmediato.
</details>

## 🧠 Última pasada antes del examen

> [!important] Lo que no puede fallarte
> - Blob = objetos · Files = SMB/NFS · Disk = VM · Queue = mensajes · Table = clave-valor.
> - Hot / Cool (30 d) / Cold (90 d) / Archive (180 d, sin conexión, horas).
> - LRS 3 copias · ZRS 3 zonas · GRS 6 copias en 2 regiones · GZRS = ZRS + LRS · RA = lectura del secundario.
> - AzCopy CLI · Storage Explorer GUI · File Sync sincroniza · Data Box físico · Azure Migrate cargas de trabajo.
> - Premium = SSD, solo LRS y ZRS, sin niveles de acceso.
> - Cosmos DB = NoSQL global multimodelo con SLA de hasta 99,999%.
> - Cifrado en reposo siempre activo; redundancia no es backup.

Volver al índice: [[Almacenamiento en Azure (Índice)]]
