---
tags: [az-900, azure, bases-de-datos, cosmos-db, nosql]
---

# Azure Cosmos DB

Clases 94 y 95 del curso.

## 1. Concepto

**Azure Cosmos DB** es una base de datos **NoSQL, multimodelo y distribuida globalmente**, entregada como **PaaS**: tú no administras servidores, parches ni réplicas.

**Qué problema resuelve:** dar respuesta en milisegundos a usuarios repartidos por todo el mundo, escalando de forma elástica y sin que el equipo tenga que operar la infraestructura de la base de datos.

**Para qué se usa:** aplicaciones web y móviles globales, IoT, catálogos de comercio electrónico, videojuegos y telemetría en tiempo real.

## 2. Características principales

- **Distribución global**: replicas tu base de datos en cualquier región de Azure con un clic, y puedes activar **escrituras en varias regiones** a la vez.
- **Latencia de milisegundos de un dígito** para lecturas y escrituras.
- **Escalado elástico** del rendimiento (medido en **RU/s**, unidades de solicitud) y del almacenamiento. Existe también modo **sin servidor** y **escalado automático**.
- **SLA de disponibilidad del 99,999%** para lecturas y escrituras en configuraciones multirregión.
- **Cinco niveles de coherencia** entre "fuerte" y "eventual", para equilibrar consistencia y rendimiento. *Detalle de nivel AZ-305, basta con saber que existen cinco.*
- **Sin esquema fijo**: cada documento puede tener campos distintos.

### APIs disponibles

Cosmos DB expone varias APIs para que puedas usar el modelo de datos que ya conoces:

| API | Modelo |
|---|---|
| **NoSQL** (la nativa) | Documentos JSON |
| **MongoDB** | Documentos, compatible con drivers de MongoDB |
| **Cassandra** | Columnas anchas |
| **Gremlin** | **Grafos** |
| **Table** | Clave-valor |
| **PostgreSQL** | Relacional distribuido |

> [!tip] Para AZ-900 no necesitas dominar cada API. Basta con reconocer que **Cosmos DB es NoSQL multimodelo** y que permite migrar apps de MongoDB o Cassandra sin reescribirlas.

> [!warning] Confusión frecuente
> **Cosmos DB no es Azure SQL Database.** Si el enunciado habla de **datos relacionales, tablas con relaciones, consultas SQL tradicionales o migrar SQL Server**, la respuesta es **Azure SQL Database**. Cosmos DB entra cuando aparecen "NoSQL", "global", "baja latencia" o "sin esquema".

## 3. Casos de uso

- Una tienda online con clientes en cinco continentes mantiene el catálogo replicado cerca de cada usuario.
- Una plataforma de IoT ingiere millones de lecturas de sensores por segundo.
- Un videojuego guarda perfiles y puntuaciones con lectura instantánea desde cualquier país.
- Una red social modela relaciones entre usuarios con la **API de Gremlin** (grafos).
- Una app existente en MongoDB se migra a Azure conservando su código gracias a la **API de MongoDB**.

## 4. Comparaciones

| | **Azure Cosmos DB** | **Azure SQL Database** | **Azure Table Storage** |
|---|---|---|---|
| Modelo de datos | NoSQL multimodelo | Relacional | NoSQL clave-valor simple |
| Distribución global | **Sí, nativa** | Limitada (réplicas geográficas) | No |
| Latencia | Milisegundos de un dígito | Buena, regional | Correcta, sin garantías de latencia |
| SLA | Hasta **99,999%** | 99,99% | Estándar de la cuenta |
| Coste | Más alto | Medio | **Muy bajo** |
| Elígelo cuando | App global, NoSQL, baja latencia | Datos relacionales y consultas SQL | Datos tabulares sencillos al mínimo coste |

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - Cosmos DB = **NoSQL, multimodelo, distribuido globalmente**, servicio **PaaS**.
> - **Latencia de milisegundos de un dígito** y **SLA de hasta 99,999%**.
> - **Escrituras en varias regiones (multi-master)**.
> - **Cinco niveles de coherencia**.
> - APIs: **NoSQL, MongoDB, Cassandra, Gremlin, Table, PostgreSQL**.
> - No es relacional; para SQL tradicional está **Azure SQL Database**.

## 6. Tips para AZ-900

> [!tip] Palabras clave
> - "global", "en todo el mundo", "usuarios en varios continentes" → **Cosmos DB**
> - "NoSQL", "sin esquema", "documentos JSON" → **Cosmos DB**
> - "latencia de milisegundos", "escalado masivo", "IoT" → **Cosmos DB**
> - "grafo", "relaciones entre entidades" → **API de Gremlin en Cosmos DB**
> - "relacional", "tablas y claves foráneas", "migrar SQL Server" → **Azure SQL Database**

> [!warning] Trampas habituales
> - Ofrecer Cosmos DB para una migración de SQL Server: la respuesta ahí es Azure SQL Managed Instance o Azure SQL Database.
> - Ofrecer Cosmos DB cuando el requisito es **coste mínimo** para datos tabulares simples: eso es **Table Storage**.
> - Pensar que Cosmos DB es IaaS. Es **PaaS**, totalmente administrado.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** Una aplicación móvil con usuarios en Europa, Asia y América necesita una base de datos NoSQL con latencia de milisegundos en todas las regiones. ¿Qué servicio eliges?

- A) Azure SQL Database
- B) Azure Cosmos DB
- C) Azure Table Storage
- D) Azure Blob Storage

**Respuesta correcta: B.** Cosmos DB está diseñado para distribución global con baja latencia y modelo NoSQL. A es relacional y no ofrece distribución global nativa con escrituras multirregión. C no proporciona distribución global ni garantías de latencia. D almacena objetos, no es una base de datos.

---

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones describe correctamente a Azure Cosmos DB?

- A) Es una base de datos relacional compatible con T-SQL
- B) Es una base de datos NoSQL multimodelo con distribución global
- C) Es un servicio de almacenamiento de objetos no estructurados
- D) Es una solución de máquinas virtuales para bases de datos

**Respuesta correcta: B.** Cosmos DB es NoSQL, admite varios modelos de datos mediante distintas APIs y se distribuye globalmente. A describe Azure SQL Database. C describe Blob Storage. D describe una instalación IaaS, y Cosmos DB es PaaS.

---

**Pregunta 3.** Una empresa tiene una aplicación que usa MongoDB en su centro de datos y quiere moverla a Azure sin reescribir el código de acceso a datos. ¿Qué opción es la más adecuada?

- A) Azure SQL Database
- B) Azure Cosmos DB con la API para MongoDB
- C) Azure Table Storage
- D) Azure Files

**Respuesta correcta: B.** Cosmos DB ofrece una API compatible con MongoDB, así que los drivers y el código existentes siguen funcionando. A obligaría a rehacer el modelo de datos a relacional. C usa un modelo clave-valor distinto. D es un recurso compartido de archivos, no una base de datos.

## 🧠 Resumen para el examen

1. Cosmos DB = **NoSQL multimodelo, global, PaaS y totalmente administrado**.
2. **Latencia de milisegundos de un dígito** y **SLA de hasta 99,999%**.
3. Replica en cualquier región con **escrituras en varias regiones**.
4. Ofrece **cinco niveles de coherencia**.
5. APIs: **NoSQL, MongoDB, Cassandra, Gremlin, Table, PostgreSQL**.
6. El rendimiento se mide en **RU/s** y escala de forma elástica.
7. **Relacional → Azure SQL Database.** **NoSQL global → Cosmos DB.** **Tabular barato → Table Storage.**
8. Casos típicos: **IoT, comercio electrónico global, gaming, apps en tiempo real**.
