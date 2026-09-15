---
tags: [az-900, azure, almacenamiento, certificacion]
tipo: MOC
---

# AZ-900 · Almacenamiento en Azure (Índice)

Mapa de contenidos de la Sección 7 del curso. Cada nota sigue la misma estructura: concepto, características, casos de uso, comparaciones, memorización, tips de examen y preguntas tipo.

## Notas de la sección

| #   | Nota                                          | Qué cubre                                                      |
| --- | --------------------------------------------- | -------------------------------------------------------------- |
| 1   | [[Azure Storage - Cuentas de almacenamiento]] | Visión general, tipos de cuenta, servicios de datos, endpoints |
| 2   | [[Azure Blob Storage]]                        | Objetos, contenedores, tipos de blob, sitio web estático       |
| 3   | [[Azure Disk Storage]]                        | Discos administrados para máquinas virtuales                   |
| 4   | [[Azure Files]]                               | Recursos compartidos SMB/NFS y File Sync                       |
| 5   | [[Azure Archive Storage y niveles de acceso]] | Hot, Cool, Cold, Archive y rehidratación                       |
| 6   | [[Redundancia de almacenamiento]]             | LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS                           |
| 7   | [[Movimiento y migración de datos]]           | AzCopy, Storage Explorer, File Sync, Data Box, Azure Migrate   |
| 8   | [[Almacenamiento Premium]]                    | Rendimiento estándar vs premium                                |
| 9   | [[Azure Cosmos DB]]                           | Base de datos NoSQL global                                     |
| 10  | [[Repaso final de Almacenamiento]]            | Tablas, confusiones y 10 preguntas de repaso                   |

## Peso en el examen

El almacenamiento entra dentro del dominio **"Describir la arquitectura y los servicios de Azure" (35-40%)**, en el apartado de servicios de almacenamiento. Se pregunta a nivel de *qué servicio elijo* y *qué diferencia hay entre estos dos*, nunca de configuración avanzada.

> [!tip] Cómo estudiar esta sección
> El 80% de las preguntas de almacenamiento del AZ-900 caen en tres bloques: elegir entre Blob/Files/Disk, elegir el nivel de acceso correcto (Hot/Cool/Cold/Archive) y elegir la redundancia correcta (LRS/ZRS/GRS/GZRS). Domina esos tres y el resto es propina.

> [!warning] Temas del curso con poco peso en AZ-900
> Las clases de tipo **[DEMO]** (creación de blob storage, blob en acción, mi primera web, demos de disk/file/archive/premium) no aportan contenido nuevo evaluable: son práctica de portal. Están integradas dentro de la nota del servicio correspondiente. El examen no te pide pasos del portal.
