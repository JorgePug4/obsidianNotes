---
tags: [az-900, azure, almacenamiento, redundancia, lrs, zrs, grs, gzrs]
---

# Redundancia de almacenamiento

Clases 88 y 89 del curso. Junto con los niveles de acceso, es el tema estrella de esta sección en el examen.

## 1. Concepto

La **redundancia** define **cuántas copias** de tus datos guarda Azure y **dónde** las coloca. Se configura a nivel de **cuenta de almacenamiento** y se aplica a todos los servicios de esa cuenta.

**Qué problema resuelve:** proteger los datos ante fallos de hardware, caída de un centro de datos completo o desastre regional. Azure replica siempre, como mínimo tres veces; tú decides el alcance de esa protección.

**La idea para memorizar:** cada opción es un seguro contra un **tamaño de desastre** distinto. Decide el tamaño del desastre y la opción se elige sola.

| Tamaño del desastre | Opción mínima |
|---|---|
| Falla un disco, un servidor o un rack | **LRS** |
| Se cae un centro de datos o una zona entera | **ZRS** |
| Se cae la región completa | **GRS** |
| Se cae una zona *y* quieres cubrir la región | **GZRS** |

## 2. Las opciones, una por una

| Opción | Copias | Dónde | Durabilidad anual | Lectura del secundario |
|---|---|---|---|---|
| **LRS** (locally redundant) | 3 | Un solo centro de datos en la región primaria | **11 nueves** | No aplica |
| **ZRS** (zone-redundant) | 3 | **3 zonas de disponibilidad** de la región primaria | 12 nueves | No aplica |
| **GRS** (geo-redundant) | 6 | LRS en primaria + LRS en región **secundaria emparejada** | **16 nueves** | **No**, salvo conmutación por error |
| **GZRS** (geo-zone-redundant) | 6 | **ZRS** en primaria + LRS en secundaria | 16 nueves | No, salvo conmutación por error |
| **RA-GRS** | 6 | Igual que GRS | 16 nueves | **Sí, lectura siempre** |
| **RA-GZRS** | 6 | Igual que GZRS | 16 nueves | **Sí, lectura siempre** |

Detalles que se preguntan:

- **La replicación a la región secundaria es asíncrona.** Puede haber una pequeña pérdida de datos si la primaria cae de golpe.
- **En la región secundaria los datos se guardan siempre con LRS**, elijas GRS o GZRS. La "Z" nunca viaja.
- **RA** significa *Read-Access*: acceso de **solo lectura** al secundario, en cualquier momento, sin esperar a la conmutación por error.
- La región secundaria es la **región emparejada** de la primaria y la asigna Azure.
- LRS es la **más barata**; GZRS y RA-GZRS, las **más caras**.

> [!warning] Confusión frecuente
> **La redundancia no es una copia de seguridad.** Si borras o modificas un archivo, el cambio se replica a todas las copias, incluida la del secundario. Para recuperarse de un borrado accidental hacen falta **soft delete, versionado o Azure Backup**.

> [!warning] Otra confusión típica
> Con **GRS "a secas" no puedes leer el secundario**. Está ahí pasivo, esperando una conmutación por error. Si el enunciado dice "acceso de solo lectura durante una caída de la región primaria", la respuesta es **RA-GRS o RA-GZRS**.

## 3. Casos de uso

- **LRS**: datos que se pueden regenerar fácilmente, entornos de desarrollo, copias de trabajo temporales. Es también la única opción para muchas cuentas Premium junto con ZRS.
- **ZRS**: aplicación crítica que debe seguir disponible si cae un centro de datos, con el requisito de que los datos **no salgan de la región** (por normativa de residencia).
- **GRS**: cumplimiento de plan de recuperación ante desastres regional con el coste más contenido.
- **GZRS**: aplicación empresarial que necesita a la vez alta disponibilidad zonal y recuperación regional.
- **RA-GRS / RA-GZRS**: aplicación de lectura global que debe seguir sirviendo consultas aunque la región primaria esté caída.

## 4. Comparaciones

**Cómo descifrar los nombres:**

- **L** = *Local* → un centro de datos.
- **Z** = *Zone* → tres zonas de disponibilidad de la misma región.
- **G** = *Geo* → una segunda región.
- **RA** = *Read-Access* → lectura del secundario siempre disponible.

| Requisito del enunciado | Respuesta |
|---|---|
| El coste más bajo | **LRS** |
| Sobrevivir a la caída de un centro de datos, sin salir de la región | **ZRS** |
| Sobrevivir a un desastre regional | **GRS** |
| Máxima durabilidad y disponibilidad combinadas | **GZRS** |
| Leer los datos del secundario en todo momento | **RA-GRS / RA-GZRS** |
| Los datos no pueden salir del país | **LRS o ZRS** |

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - **LRS: 3 copias, un centro de datos, 11 nueves, la más barata.**
> - **ZRS: 3 copias, 3 zonas de disponibilidad, misma región.**
> - **GRS: 6 copias, 2 regiones, secundario NO legible.**
> - **GZRS: ZRS en primaria + LRS en secundaria, 6 copias.**
> - **RA-\* = lectura del secundario siempre.**
> - Replicación al secundario **asíncrona**; en el secundario siempre **LRS**.
> - La redundancia se define **por cuenta de almacenamiento**.
> - Redundancia **no equivale a copia de seguridad**.

## 6. Tips para AZ-900

> [!tip] Palabras clave
> - "menor coste", "datos no críticos" → **LRS**
> - "zona de disponibilidad", "centro de datos", "los datos deben permanecer en la región" → **ZRS**
> - "desastre regional", "otra región", "recuperación ante desastres" → **GRS / GZRS**
> - "leer", "solo lectura", "durante la interrupción" → **RA-GRS / RA-GZRS**
> - "máxima durabilidad y disponibilidad" → **GZRS**

> [!warning] Trampas habituales
> - Combinar "residencia de datos en el país" con "protección frente a la caída de un centro de datos". Parece pedir GRS pero la respuesta es **ZRS**, porque GRS saca los datos de la región.
> - Preguntar la durabilidad de LRS. Son **11 nueves (99,999999999%)**, no "cinco nueves". Cinco nueves es una cifra de **disponibilidad**, no de durabilidad.
> - Ofrecer "GRS permite leer el secundario". Falso sin la conmutación por error, para eso está RA-GRS.
> - Creer que GZRS usa ZRS también en la región secundaria. Usa **LRS**.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** Una empresa debe proteger sus datos frente a la caída de un centro de datos completo, pero la normativa exige que los datos permanezcan dentro de la misma región de Azure. ¿Qué opción de redundancia debe elegir?

- A) LRS
- B) ZRS
- C) GRS
- D) RA-GRS

**Respuesta correcta: B.** ZRS replica en tres zonas de disponibilidad dentro de la misma región, cubriendo la caída de un centro de datos sin sacar los datos de la región. A no protege frente a la pérdida del centro de datos. C y D replican a una segunda región, lo que incumple el requisito de residencia.

---

**Pregunta 2.** Tu aplicación debe poder **leer** los datos aunque la región primaria sufra una interrupción, sin esperar a una conmutación por error. ¿Qué opción cumple el requisito con el menor coste?

- A) LRS
- B) ZRS
- C) GRS
- D) RA-GRS

**Respuesta correcta: D.** Solo las variantes con acceso de lectura (RA-GRS y RA-GZRS) permiten leer el secundario en todo momento. A y B no tienen región secundaria. C tiene la copia, pero no es legible hasta que se produce la conmutación por error.

---

**Pregunta 3.** ¿Cuál de las siguientes afirmaciones sobre GZRS es correcta?

- A) Replica los datos con ZRS en la región primaria y con LRS en la secundaria
- B) Replica con ZRS tanto en la primaria como en la secundaria
- C) Solo replica dentro de un único centro de datos
- D) Es la opción de redundancia más barata

**Respuesta correcta: A.** GZRS combina ZRS en la región primaria con LRS en la región secundaria. B es falso: en el secundario Azure siempre usa LRS. C describe LRS. D es falso, GZRS está entre las más caras.

## 🧠 Resumen para el examen

1. **Todas** las opciones guardan al menos **3 copias**.
2. **LRS** = 1 centro de datos, 11 nueves, la más barata.
3. **ZRS** = 3 zonas de disponibilidad en la **misma región**.
4. **GRS** = 6 copias en 2 regiones; el secundario **no se lee** sin conmutación por error.
5. **GZRS** = ZRS en primaria + **LRS** en secundaria.
6. **RA-GRS / RA-GZRS** añaden **lectura permanente** del secundario.
7. La copia a la región secundaria es **asíncrona** (posible pérdida mínima de datos).
8. La redundancia se elige **por cuenta** y **no sustituye a las copias de seguridad**.
9. "No puede salir de la región" → descarta cualquier opción con **G**.
