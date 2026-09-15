---
tags: [az-900, azure, almacenamiento, niveles-acceso, archive]
---

# Azure Archive Storage y niveles de acceso

Clases 86 y 87 del curso. Este es uno de los temas con más probabilidad de aparecer en el examen.

## 1. Concepto

Los **niveles de acceso (access tiers)** son la forma que tiene Azure de ajustar el precio del almacenamiento de blobs a la frecuencia con que usas los datos. Se aplican **solo a [[Azure Blob Storage]]**.

**Qué problema resuelve:** pagar poco por datos que casi nunca se tocan. Guardar todo en el nivel más rápido sería carísimo cuando el 90% de los datos son backups de hace tres años.

**La regla que gobierna todo el tema:** cuanto **más barato es guardar**, **más caro es acceder**.

## 2. Los cuatro niveles

| Nivel | Estado | Coste de almacenar | Coste de acceder | Retención mínima | Latencia de acceso |
|---|---|---|---|---|---|
| **Hot** (frecuente) | En línea | El más alto | El más bajo | Ninguna | Milisegundos |
| **Cool** (esporádico) | En línea | Bajo | Medio | **30 días** | Milisegundos |
| **Cold** (poco frecuente) | En línea | Más bajo | Más alto | **90 días** | Milisegundos |
| **Archive** (archivo) | **Sin conexión** | **El más bajo** | **El más alto** | **180 días** | **Horas** (rehidratación) |

- **Hot, Cool y Cold son niveles en línea**: los datos se leen al instante.
- **Archive está sin conexión**. Para leer un blob archivado hay que **rehidratarlo** primero a Hot o Cool, y eso tarda horas.
- Prioridades de rehidratación: **Estándar** (hasta unas 15 horas) y **Alta** (menos de 1 hora para objetos pequeños, más cara).

> [!warning] Penalización por eliminación temprana
> Si borras o mueves un blob antes de cumplir su retención mínima, **pagas igualmente los días que faltaban**. Ejemplo: mueves un archivo a Archive y lo borras a los 45 días → te cobran los 135 días restantes.
> Ojo: la retención mínima es una **regla de facturación**, no un bloqueo. Puedes leer o borrar cuando quieras, simplemente pagas la penalización.

### Dónde se aplica cada nivel

- El nivel **predeterminado de la cuenta** se establece en Hot o Cool.
- **Archive solo se puede establecer a nivel de blob individual**, nunca como predeterminado de la cuenta.
- La **administración del ciclo de vida** mueve blobs entre niveles automáticamente según su antigüedad o su último acceso. Es la respuesta correcta cuando piden "automatizar el ahorro".

> [!warning] Confusión frecuente
> **Nivel de acceso ≠ redundancia ≠ tipo de cuenta.** Hot/Cool/Cold/Archive controlan el **coste según la frecuencia de uso**. LRS/ZRS/GRS/GZRS controlan **cuántas copias y dónde**. Son ajustes independientes: puedes tener un blob en Archive con GRS.

## 3. Casos de uso

- **Hot**: fotos de producto del catálogo activo, ficheros de una app en uso diario.
- **Cool**: backups del último mes, informes que se consultan de vez en cuando.
- **Cold**: copias de seguridad trimestrales, datos de cumplimiento que se inspeccionan muy de vez en cuando pero sin esperar horas.
- **Archive**: registros médicos o financieros que hay que conservar 7 o 10 años por ley y que casi nunca se abren; cintas de backup antiguas digitalizadas.

## 4. Comparaciones

**Cool vs Cold** (la diferencia nueva que más confunde):

| | **Cool** | **Cold** |
|---|---|---|
| Estado | En línea | En línea |
| Coste de almacenar | Mayor que Cold | Menor que Cool |
| Coste de acceder | Menor que Cold | Mayor que Cool |
| Retención mínima | 30 días | 90 días |
| Elígelo si | Accedes cada pocas semanas | Accedes un par de veces al año pero necesitas los datos **al instante** |

**Cold vs Archive** (la que decide muchas preguntas):

| | **Cold** | **Archive** |
|---|---|---|
| ¿Datos disponibles al momento? | **Sí** | **No**, hay que rehidratar |
| Tiempo de recuperación | Milisegundos | Horas |
| Coste de almacenamiento | Bajo | **El más bajo de todos** |
| Retención mínima | 90 días | 180 días |

## 5. Conceptos que debo memorizar

> [!important] Números exactos del examen
> - Retención mínima: **Cool 30 días, Cold 90 días, Archive 180 días**. Hot no tiene.
> - **Archive está sin conexión** y requiere **rehidratación**, que tarda **horas**.
> - Rehidratación **Estándar hasta ~15 h**, **Alta menos de 1 h**.
> - **Archive solo se configura por blob**, no como predeterminado de la cuenta.
> - Los niveles aplican **solo a Blob**, nunca a Files ni a discos.
> - La **administración del ciclo de vida** automatiza los cambios de nivel.

## 6. Tips para AZ-900

> [!tip] Palabras clave que deciden la respuesta
> - "acceso frecuente", "uso diario", "producción activa" → **Hot**
> - "acceso esporádico", "al menos 30 días", "backups recientes" → **Cool**
> - "raro pero necesito los datos al instante", "90 días" → **Cold**
> - "requisito legal", "conservar durante años", "el coste más bajo posible", "puedo esperar horas" → **Archive**
> - "reducir costes automáticamente conforme envejecen los datos" → **administración del ciclo de vida**

> [!warning] Trampas habituales
> - El enunciado pide el **coste más bajo** pero añade "los datos deben estar disponibles de inmediato". Entonces **Archive queda descartado** aunque sea el más barato: la respuesta es Cold o Cool.
> - Ofrecer "Archive" como nivel predeterminado de la cuenta. No es posible.
> - Dar por válido que Archive es "lento porque es un disco lento". Es lento porque está **sin conexión**.
> - Aplicar niveles a Azure Files o a discos administrados. No existen ahí.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** Una empresa debe conservar registros financieros durante siete años por normativa. Es poco probable que se consulten y, si se hace, se acepta esperar varias horas. ¿Qué nivel de acceso minimiza el coste?

- A) Hot
- B) Cool
- C) Cold
- D) Archive

**Respuesta correcta: D.** Archive ofrece el coste de almacenamiento más bajo y el escenario tolera explícitamente horas de espera para la rehidratación. A es el más caro de almacenar. B y C serían más caros de almacenar que Archive y su ventaja (acceso inmediato) no hace falta aquí.

---

**Pregunta 2.** Un archivo se mueve al nivel Archive y se elimina 60 días después. ¿Qué ocurre con la facturación?

- A) No se cobra nada adicional
- B) Se cobra una penalización equivalente a los 120 días restantes
- C) Se bloquea la eliminación hasta cumplir 180 días
- D) El archivo pasa automáticamente a Cool

**Respuesta correcta: B.** Archive tiene una retención mínima de 180 días; al borrar a los 60, se factura el equivalente a los 120 días que faltaban. A ignora la penalización. C es falso: la retención mínima es una regla de facturación, no un bloqueo. D no sucede automáticamente.

---

**Pregunta 3.** Necesitas el menor coste de almacenamiento posible para datos que se consultan una o dos veces al año, pero la recuperación debe ser **inmediata**. ¿Qué nivel eliges?

- A) Hot
- B) Cool
- C) Cold
- D) Archive

**Respuesta correcta: C.** Cold es un nivel **en línea** más barato que Cool, pensado justo para datos con acceso muy poco frecuente que aun así deben recuperarse al instante. A y B cuestan más almacenar. D sería más barato, pero no ofrece recuperación inmediata.

## 🧠 Resumen para el examen

1. Los niveles de acceso existen **solo en Blob Storage**.
2. Orden de coste de almacenamiento, de mayor a menor: **Hot > Cool > Cold > Archive**. El coste de acceso va justo al revés.
3. **Hot, Cool y Cold están en línea; Archive está sin conexión.**
4. Retenciones mínimas: **Cool 30 / Cold 90 / Archive 180** días.
5. Recuperar de Archive exige **rehidratar** y tarda **horas** (Estándar ~15 h, Alta < 1 h).
6. Si la pregunta exige **acceso inmediato**, Archive nunca es la respuesta.
7. **Archive se establece por blob**, no como predeterminado de la cuenta.
8. La **administración del ciclo de vida** mueve y borra blobs automáticamente.
9. El nivel de acceso es independiente de la [[Redundancia de almacenamiento]].
