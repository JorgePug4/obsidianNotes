---
tags: [az-900, azure, almacenamiento, premium]
---

# Almacenamiento Premium

Clases 92 y 93 del curso. Tema breve: en AZ-900 solo se pregunta la diferencia entre Estándar y Premium.

## 1. Concepto

El **rendimiento** de una cuenta de almacenamiento se elige al crearla y tiene dos opciones:

- **Estándar**: respaldado por discos duros magnéticos (**HDD**). Coste más bajo, latencia mayor.
- **Premium**: respaldado por **SSD**. Latencia de milisegundos de un dígito, mayor rendimiento y coste más alto.

**Qué problema resuelve:** aplicaciones que no toleran latencia (bases de datos transaccionales, analítica en tiempo real, cargas con mucha E/S).

**Detalle económico importante:** en Premium pagas por la **capacidad aprovisionada**, no solo por lo que consumes. Reservas rendimiento y lo pagas aunque no lo uses.

## 2. Características principales

Existen tres tipos de cuenta Premium, cada uno especializado:

| Tipo de cuenta Premium | Para qué | Servicio |
|---|---|---|
| **Blobs en bloque Premium** | Tasas de transacción altas y objetos pequeños | Blob |
| **Recursos compartidos de archivos Premium** | Archivos con E/S intensa, SMB/NFS | Files |
| **Blobs en páginas Premium** | Discos y VHD | Page blobs |

Puntos que conviene saber:

- Premium **no admite los niveles de acceso** Hot/Cool/Cold/Archive en cuentas de blobs en bloque. La optimización por niveles pertenece al mundo Estándar.
- Redundancia disponible en Premium: **LRS y ZRS**. No hay opciones geográficas (GRS/GZRS).
- El **rendimiento no se puede cambiar** de Estándar a Premium en una cuenta existente: hay que crear otra y migrar los datos.
- En discos administrados el equivalente es la gama **Premium SSD / Ultra Disk** frente a **Standard SSD / Standard HDD**. Ver [[Azure Disk Storage]].

> [!warning] Confusión frecuente
> **Rendimiento (Estándar/Premium) ≠ nivel de acceso (Hot/Cool/Cold/Archive) ≠ redundancia (LRS/ZRS/GRS/GZRS).** Son tres decisiones separadas del mismo formulario de creación de la cuenta. El examen las mezcla a propósito.

## 3. Casos de uso

- Una base de datos transaccional en una VM con **Premium SSD**.
- Una aplicación de análisis en tiempo real que lee millones de objetos pequeños, sobre **blobs en bloque Premium**.
- Una aplicación de diseño gráfico cuyos usuarios abren ficheros enormes desde un **recurso compartido de archivos Premium**.
- Un entorno de desarrollo o un repositorio de backups: **Estándar**, más barato, suficiente.

## 4. Comparaciones

| | **Estándar** | **Premium** |
|---|---|---|
| Medio | HDD | **SSD** |
| Latencia | Mayor | **Milisegundos de un dígito** |
| Coste | Bajo | Alto |
| Modelo de pago | Por consumo | Por **capacidad aprovisionada** |
| Niveles de acceso | **Sí** (Hot/Cool/Cold/Archive) | No en blobs en bloque |
| Redundancia | LRS, ZRS, GRS, GZRS y variantes RA | **Solo LRS y ZRS** |
| Elígelo si | Datos generales, backups, desarrollo | Necesitas rendimiento y latencia bajos |

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - **Premium = SSD**, **Estándar = HDD**.
> - Premium se factura por **capacidad aprovisionada**.
> - Premium **no** ofrece redundancia geográfica: **solo LRS y ZRS**.
> - Premium en blobs en bloque **no** usa niveles de acceso.
> - El rendimiento se decide **al crear la cuenta** y no se cambia después.

## 6. Tips para AZ-900

> [!tip] Palabras clave
> - "baja latencia", "alto rendimiento", "IOPS", "transacciones por segundo", "tiempo real" → **Premium**
> - "coste mínimo", "acceso poco frecuente", "pruebas" → **Estándar**
> - "SSD" → **Premium**; "HDD" → **Estándar**

> [!warning] Trampas habituales
> - Una pregunta pide "el menor coste" y ofrece Premium como respuesta tentadora por sonar mejor. Premium nunca es la opción de menor coste.
> - Combinar "Premium" con "replicación geográfica GRS": no existe esa combinación.
> - Confundir "Premium SSD" (tipo de **disco** de VM) con "cuenta de almacenamiento Premium" (rendimiento de la **cuenta**). Son conceptos paralelos pero distintos.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** ¿Cuál es la diferencia principal entre una cuenta de almacenamiento estándar y una premium?

- A) La premium replica los datos en más regiones
- B) La premium usa unidades SSD y ofrece menor latencia
- C) La estándar no admite cifrado en reposo
- D) La premium incluye copias de seguridad automáticas

**Respuesta correcta: B.** Premium se apoya en SSD para dar menor latencia y mayor rendimiento. A es falso: Premium está limitado a LRS y ZRS. C es falso, el cifrado en reposo es universal y gratuito. D no forma parte del rendimiento de la cuenta.

---

**Pregunta 2.** Una empresa necesita una cuenta de almacenamiento premium replicada en una segunda región geográfica. ¿Es posible?

- A) Sí, usando GRS
- B) Sí, usando RA-GZRS
- C) No, las cuentas premium solo admiten LRS y ZRS
- D) Sí, pero solo con blobs en páginas

**Respuesta correcta: C.** Las cuentas premium no ofrecen opciones de redundancia geográfica. A, B y D describen combinaciones que no están disponibles para Premium.

---

**Pregunta 3.** Una aplicación de análisis en tiempo real requiere latencia de milisegundos de un dígito al leer objetos pequeños. ¿Qué configuración eliges?

- A) Cuenta estándar con nivel Cool
- B) Cuenta estándar con nivel Archive
- C) Cuenta premium de blobs en bloque
- D) Cuenta estándar con redundancia GZRS

**Respuesta correcta: C.** Los blobs en bloque premium están diseñados para tasas de transacción altas y latencia muy baja. A y B optimizan coste, no latencia, y Archive ni siquiera está en línea. D mejora la resiliencia, pero no el rendimiento.

## 🧠 Resumen para el examen

1. **Estándar = HDD**, más barato; **Premium = SSD**, más rápido y más caro.
2. Premium se paga por **capacidad aprovisionada**.
3. Tres sabores Premium: **blobs en bloque, recursos compartidos de archivos y blobs en páginas**.
4. Premium admite **solo LRS y ZRS**.
5. Premium en blobs en bloque **no usa niveles de acceso**.
6. El rendimiento **se fija al crear la cuenta**.
7. Si la pregunta prioriza **coste**, la respuesta nunca es Premium; si prioriza **latencia**, casi siempre lo es.
