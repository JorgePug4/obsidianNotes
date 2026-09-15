---
tags: [az-900, azure, almacenamiento, migracion, azcopy, databox]
---

# Movimiento y migración de datos

Clases 90 y 91 del curso (movimiento de datos y opciones de migración adicionales).

## 1. Concepto

Azure ofrece herramientas para llevar datos **hacia** la nube, **desde** la nube y **entre** cuentas de almacenamiento. Se dividen en dos familias:

- **En línea (por red)**: los datos viajan por Internet o por una conexión privada. Rápido de empezar, limitado por el ancho de banda.
- **Sin conexión (físicas)**: Microsoft te envía un dispositivo, copias los datos y lo devuelves. La opción sensata cuando hay decenas de terabytes o la conexión es mala.

**Qué problema resuelve:** subir 200 TB por una línea de 100 Mbps tardaría meses. Con un dispositivo físico, una semana.

## 2. Herramientas que debes conocer

### Movimiento de datos (en línea)

| Herramienta | Qué es | Cuándo se usa |
|---|---|---|
| **AzCopy** | Utilidad de **línea de comandos** | Copias y sincronizaciones automatizables, scripts, volúmenes medianos |
| **Azure Storage Explorer** | Aplicación de **escritorio con interfaz gráfica** (Windows, macOS, Linux) | Explorar, subir y descargar a mano, sin comandos. Usa AzCopy por debajo |
| **Azure File Sync** | Sincroniza un **servidor Windows local** con [[Azure Files]] | Mantener el servidor local y centralizar los datos en Azure |

### Migración (grandes volúmenes)

| Opción | Tipo | Capacidad aproximada | Escenario |
|---|---|---|---|
| **Azure Data Box** | Dispositivo físico robusto | **80 TB** utilizables | Migraciones grandes con red limitada; permite importar y exportar |
| **Azure Data Box Disk** | Hasta 5 SSD por pedido | **40 TB** por pedido | Volúmenes medianos; **solo importación** |
| **Azure Migrate** | Servicio de valoración y migración | Servidores, VM, bases de datos y apps | Migrar cargas de trabajo completas, no solo archivos |

> [!tip] Regla práctica del examen
> Si el enunciado menciona **ancho de banda limitado, conexión lenta, decenas o cientos de TB o una ubicación remota**, la respuesta correcta casi siempre es **Azure Data Box**.
> Si menciona **scripts, automatización o línea de comandos**, es **AzCopy**.
> Si menciona **interfaz gráfica** para gestionar el almacenamiento a mano, es **Azure Storage Explorer**.

> [!warning] Cambios recientes
> **Azure Data Box Heavy (~1 PB) ha sido retirado** y ya no se puede pedir. Además, los antiguos trabajos del servicio **Azure Import/Export ahora se gestionan dentro del recurso Azure Data Box**. Muchos cursos y simuladores siguen listando Data Box Heavy e Import/Export como opciones separadas. En el examen, si aparecen, siguen siendo respuestas plausibles según el material oficial; en la realidad, la familia vigente es **Data Box y Data Box Disk**.

## 3. Casos de uso

- Un equipo de datos copia 2 TB de un bucket a una cuenta de Azure con **AzCopy** dentro de un script nocturno.
- Un administrador revisa y sube un par de ficheros con **Storage Explorer** desde su portátil.
- Una empresa con 150 TB de vídeo en un centro de datos rural pide un **Data Box**, lo llena y lo devuelve por mensajería.
- Una compañía evalúa sus 200 servidores locales con **Azure Migrate** antes de mover nada.
- Una sucursal conserva su servidor de archivos y lo enlaza con **Azure File Sync**.

## 4. Comparaciones

| | **AzCopy** | **Storage Explorer** | **Data Box** | **Azure Migrate** |
|---|---|---|---|---|
| Interfaz | Línea de comandos | Gráfica | Dispositivo físico | Portal |
| Vía | Red | Red | Mensajería | Red |
| Volumen típico | GB a pocos TB | GB | Decenas de TB | Servidores completos |
| Automatizable | **Sí** | No | No | Parcial |
| Elígelo si | Scripts y transferencias repetidas | Trabajo manual puntual | La red no da abasto | Migras VM, apps o bases de datos |

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - **AzCopy: línea de comandos.** **Storage Explorer: interfaz gráfica.**
> - **Azure File Sync** sincroniza servidor local ↔ Azure Files, con **cloud tiering**.
> - **Data Box: 80 TB**, dispositivo físico, importación y exportación.
> - **Data Box Disk: 40 TB** en SSD, **solo importación**.
> - **Azure Migrate** migra **cargas de trabajo** (servidores, bases de datos, apps), no simples archivos.
> - Los datos de Data Box viajan **cifrados**.

## 6. Tips para AZ-900

> [!tip] Palabras clave
> - "línea de comandos", "script", "automatizar" → **AzCopy**
> - "interfaz gráfica", "explorar visualmente", "arrastrar y soltar" → **Storage Explorer**
> - "ancho de banda limitado", "petabytes", "sin conectividad" → **Azure Data Box**
> - "evaluar y migrar servidores locales a Azure" → **Azure Migrate**
> - "mantener el servidor de archivos local sincronizado" → **Azure File Sync**

> [!warning] Trampas habituales
> - Ofrecer **Azure Migrate** para copiar archivos sueltos. Migrate está pensado para **cargas de trabajo completas**, no para mover un directorio.
> - Ofrecer **AzCopy** cuando el enunciado insiste en que la red es insuficiente. Ahí toca Data Box.
> - Confundir **Azure File Sync** (sincronización continua) con **Azure Backup** (copias de seguridad) o con una migración puntual.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** Una empresa necesita transferir 120 TB desde un centro de datos con una conexión a Internet muy limitada. ¿Qué solución debe usar?

- A) AzCopy
- B) Azure Storage Explorer
- C) Azure Data Box
- D) Azure File Sync

**Respuesta correcta: C.** Data Box es un dispositivo físico pensado exactamente para volúmenes grandes con conectividad limitada. A y B dependen de la red, que aquí es el cuello de botella. D sincroniza archivos de forma continua, no realiza una migración masiva inicial.

---

**Pregunta 2.** Un administrador quiere automatizar mediante un script la copia diaria de archivos a una cuenta de almacenamiento. ¿Qué herramienta es la adecuada?

- A) Azure Storage Explorer
- B) AzCopy
- C) Azure Data Box Disk
- D) Azure Migrate

**Respuesta correcta: B.** AzCopy es una utilidad de línea de comandos, por lo que se puede incluir en scripts y tareas programadas. A es una aplicación gráfica sin capacidad de scripting. C es un dispositivo físico para migraciones puntuales. D migra cargas de trabajo, no ficheros diarios.

---

**Pregunta 3.** ¿Qué servicio ayuda a evaluar los servidores locales y a planificar su traslado a Azure?

- A) Azure Migrate
- B) AzCopy
- C) Azure Data Box
- D) Azure Storage Explorer

**Respuesta correcta: A.** Azure Migrate proporciona valoración, dimensionamiento y ejecución de la migración de servidores, bases de datos y aplicaciones. B y D solo mueven datos. C transporta datos físicamente, sin evaluar nada.

## 🧠 Resumen para el examen

1. **AzCopy = CLI**; **Storage Explorer = GUI**. Esta pareja aparece muy a menudo.
2. **Azure File Sync** mantiene sincronizado un servidor Windows local con Azure Files.
3. **Data Box (80 TB)** para migraciones masivas con red insuficiente; importa y exporta.
4. **Data Box Disk (40 TB en SSD)**, solo importación.
5. **Azure Migrate** para mover **servidores, VM, bases de datos y aplicaciones**.
6. Regla mental: **red buena → en línea; red mala o muchos TB → dispositivo físico**.
7. Todo el tránsito y el contenido de los dispositivos va **cifrado**.
