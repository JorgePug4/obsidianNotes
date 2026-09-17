---
tags: [bases-de-datos, sql, nosql, arquitectura]
up: "[[🗺️ Índice - Ingeniería de Software]]"
---

# Bases de Datos SQL vs NoSQL

> [!info] No es "cuál es mejor", sino **cuál encaja con la forma de tus datos, tus patrones de acceso y tus requisitos de consistencia y escala**.

## ¿Qué es?

- **SQL / relacional**: datos organizados en **tablas** con filas y columnas, un **esquema fijo**, relaciones mediante claves foráneas y un lenguaje estándar (**SQL**). Ejemplos: PostgreSQL, MySQL/MariaDB, SQL Server, Oracle, SQLite.
- **NoSQL / no relacional** (*Not only SQL*): familia heterogénea de bases de datos que **no usan el modelo relacional** como estructura principal. Priorizan flexibilidad de esquema, escalado horizontal y patrones de acceso concretos. Ejemplos: MongoDB, Redis, Cassandra, DynamoDB, Neo4j, [[Azure Cosmos DB]].

La elección depende principalmente de la **estructura de tus datos**, tus **patrones de consulta** y los **requisitos de escalabilidad y consistencia** del proyecto.

## ¿Para qué sirve cada una?

### Cuándo utilizar SQL

- **Estructura conocida y estable**: los datos tienen un esquema predefinido que no cambia constantemente.
- **Integridad transaccional ([[ACID en Bases de Datos|ACID]])**: transacciones donde la consistencia es crítica (banca, facturación, inventario).
- **Consultas complejas y no previstas**: múltiples `JOIN`, agregaciones, informes *ad hoc*. SQL es el lenguaje de consulta más potente y universal.
- **Relaciones muchos-a-muchos** frecuentes.
- **Por defecto**: si no tienes una razón clara para NoSQL, PostgreSQL es una elección segura para la mayoría de aplicaciones.

### Cuándo utilizar NoSQL

- **Esquema flexible o heterogéneo**: cada registro puede tener campos distintos (catálogos de productos con atributos variables, perfiles de usuario).
- **Escalado horizontal masivo**: distribuir datos en muchos nodos de forma nativa (*sharding* automático).
- **Patrones de acceso conocidos y simples**: "dame el documento con esta clave" a muy baja latencia y alto volumen de escritura.
- **Casos especializados**: caché y sesiones (clave-valor), relaciones profundas (grafos), series temporales, búsqueda de texto completo.
- **Disponibilidad sobre consistencia**: cuando toleras leer un dato ligeramente desactualizado a cambio de que el sistema nunca deje de responder ([[Teorema CAP y BASE]]).

## Conceptos relacionados

- [[ACID en Bases de Datos|ACID]] → garantías transaccionales clásicas del mundo relacional.
- [[Teorema CAP y BASE]] → por qué muchas NoSQL distribuidas relajan la consistencia.
- [[🏗️ Diseño de Microservicios]] → *Database-per-Service* permite elegir el motor adecuado para cada servicio (**persistencia políglota**).
- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → escritura en SQL, lectura en un almacén NoSQL optimizado.
- [[Event Sourcing]] → un *event store* es un modelo de persistencia distinto de ambos.
- [[Azure Cosmos DB]] → NoSQL multi-modelo gestionado de Azure, con niveles de consistencia configurables.
- [[Clean Architecture]] → la base de datos es un detalle de infraestructura; el dominio no debería saber cuál es.

## ¿Cómo funciona? Modelos de datos

### Bases de datos relacionales

```
Clientes                 Pedidos                    Productos
┌────┬────────┐          ┌────┬───────────┬───────┐ ┌────┬────────┬───────┐
│ id │ nombre │ 1 ───∞  │ id │ cliente_id│ fecha │ │ id │ nombre │ precio│
└────┴────────┘          └────┴───────────┴───────┘ └────┴────────┴───────┘
                                   │ ∞                      ∞ │
                                   └──── LineasPedido ────────┘
```

- **Normalización**: cada dato vive en un solo sitio; se reconstruye con `JOIN`.
- **Esquema estricto**: la base rechaza datos que no cumplan la estructura.
- **Índices B-tree**, vistas, procedimientos almacenados, restricciones (`UNIQUE`, `FOREIGN KEY`, `CHECK`).

### Tipos de NoSQL

| Tipo | Modelo | Ejemplos | Casos de uso típicos |
|---|---|---|---|
| **Documental** | Documentos JSON/BSON anidados, agrupados en colecciones | **MongoDB**, Couchbase, [[Azure Cosmos DB]] (API NoSQL), Firestore | Catálogos, perfiles, CMS, datos semiestructurados |
| **Clave-valor** | Diccionario gigante: clave → valor opaco | **Redis**, Memcached, DynamoDB (parcial), etcd | Caché, sesiones, colas ligeras, contadores, *rate limiting* |
| **Columnar / familia de columnas** (*wide-column*) | Filas con columnas dinámicas, particionadas por clave | **Apache Cassandra**, HBase, ScyllaDB, Bigtable | Series temporales, IoT, escritura masiva distribuida |
| **Grafos** | Nodos y aristas con propiedades | **Neo4j**, Amazon Neptune, Cosmos DB (Gremlin) | Redes sociales, recomendaciones, detección de fraude, grafos de conocimiento |
| **Búsqueda** | Índices invertidos sobre texto | **Elasticsearch**, OpenSearch, Meilisearch | Búsqueda de texto completo, logs, analítica |
| **Series temporales** | Puntos (timestamp, valor) optimizados para rangos de tiempo | InfluxDB, TimescaleDB (sobre PostgreSQL) | Métricas, monitorización, sensores |
| **Vectorial** | Vectores de alta dimensión con búsqueda por similitud | pgvector, Pinecone, Qdrant, Milvus | Búsqueda semántica, RAG para aplicaciones con IA |

> [!example] Documento en MongoDB (desnormalizado)
> ```json
> {
>   "_id": "ped-1001",
>   "cliente": { "id": "c-9", "nombre": "Ana" },
>   "fecha": "2026-09-01",
>   "lineas": [
>     { "producto": "Laptop", "precio": 1299.99, "cantidad": 1 },
>     { "producto": "Mouse",  "precio": 25.50,   "cantidad": 2 }
>   ]
> }
> ```
> Todo lo que hace falta para mostrar el pedido está en **una lectura**, sin `JOIN`. El coste: el nombre del cliente está duplicado en cada pedido; si cambia, hay que actualizarlo en todos (o aceptar que el pedido guarda el nombre *en el momento de la compra*, que a menudo es lo correcto).

## Tabla comparativa

| Característica | SQL (relacional) | NoSQL (no relacional) |
|---|---|---|
| **Esquema** | Rígido, definido de antemano (*schema-on-write*) | Flexible, validado al leer o de forma opcional (*schema-on-read*) |
| **Modelo** | Tablas normalizadas | Documentos, clave-valor, columnas, grafos… |
| **Relaciones** | Nativas, con `JOIN` y claves foráneas | Débiles; se resuelven desnormalizando o en la aplicación (los grafos son la excepción) |
| **Transacciones** | ACID completas, multi-tabla | Varía: ACID por documento/partición es común; multi-documento en algunos (MongoDB ≥ 4.0, Cosmos DB dentro de una partición) |
| **Consistencia** | Fuerte por defecto | Configurable; a menudo **eventual** por defecto en modo distribuido |
| **Escalabilidad** | Principalmente **vertical**; horizontal con réplicas de lectura, particionado manual o **SQL distribuido** (CockroachDB, YugabyteDB, Spanner, Citus) | **Horizontal** nativa (*sharding* automático) |
| **Lenguaje de consulta** | SQL estándar | Propio de cada motor (MQL, CQL, Cypher, comandos Redis…) |
| **Consultas ad hoc** | Excelentes | Limitadas al diseño de las claves e índices |
| **Madurez / herramientas** | Décadas; ORMs, BI, tooling universal | Variable según el motor |
| **Curva de aprendizaje** | Conocida por casi todo el equipo | Requiere pensar en **patrones de acceso** primero |

> [!warning] Dos ideas antiguas que ya no son del todo ciertas
> 1. **"NoSQL no tiene transacciones"**: MongoDB soporta transacciones ACID multi-documento desde la versión 4.0 (2018); Cosmos DB, DynamoDB y otros ofrecen transacciones dentro de una partición.
> 2. **"SQL no escala horizontalmente"**: las bases **SQL distribuidas** (CockroachDB, YugabyteDB, Google Spanner, Azure SQL Hyperscale, Citus) escalan horizontalmente manteniendo SQL y ACID. Y PostgreSQL con `JSONB` cubre muchos casos "documentales".
> La frontera entre ambos mundos se ha difuminado; la decisión hoy es más sobre **modelo de datos y patrones de acceso** que sobre "escala sí/no".

## Ejemplo de decisión en un sistema de e-commerce

| Servicio | Necesidad | Elección | Por qué |
|---|---|---|---|
| Pedidos y pagos | Transacciones, integridad, informes | **PostgreSQL / SQL Server** | ACID, consultas complejas |
| Catálogo de productos | Atributos variables por categoría, lecturas masivas | **MongoDB / Cosmos DB** | Esquema flexible, un documento por producto |
| Carrito y sesiones | Latencia mínima, datos efímeros | **Redis** | Clave-valor en memoria, TTL |
| Búsqueda | Texto completo, facetas, tolerancia a errores tipográficos | **Elasticsearch** | Índices invertidos |
| Recomendaciones | "Clientes que compraron X también…" | **Neo4j** | Recorrido de grafos |

Esto es **persistencia políglota**: cada [[🏗️ Diseño de Microservicios|microservicio]] elige su motor. El precio es operar varias tecnologías.

## Ventajas y desventajas resumidas

### SQL

- ✅ Integridad garantizada, consultas potentes, madurez, estándar conocido.
- ❌ Esquema rígido (migraciones), escalado horizontal más complejo, rendimiento con JOINs masivos a gran escala.

### NoSQL

- ✅ Flexibilidad, escalado horizontal, rendimiento excelente en su patrón de acceso, alta disponibilidad.
- ❌ Consultas limitadas fuera del diseño inicial, duplicación de datos, consistencia eventual que la aplicación debe manejar, cada motor tiene su lenguaje.

## Puntos clave

- **SQL**: esquema fijo, relaciones, ACID, consultas ad hoc. Elección **por defecto** para la mayoría de aplicaciones de negocio.
- **NoSQL**: esquema flexible, escalado horizontal, optimizado para **un patrón de acceso**. Siete familias principales; elige por caso de uso, no por moda.
- En NoSQL se **diseña desde las consultas**, no desde las entidades.
- Las fronteras se difuminan: PostgreSQL con JSONB, MongoDB con transacciones, SQL distribuido.
- **Persistencia políglota** en microservicios: el motor correcto para cada servicio.

## Errores comunes

- Elegir NoSQL "porque escala" para una aplicación con 10 000 usuarios y datos relacionales.
- Modelar en MongoDB como si fuera relacional (una colección por tabla y "JOINs" en la aplicación).
- Ignorar la consistencia eventual y mostrar al usuario datos que "desaparecen" tras guardar.
- Usar Redis como base de datos principal sin persistencia configurada.
- Meter JSON sin estructura en una columna SQL para "no hacer migraciones" y perder las garantías de ambos mundos.

## 🎯 Para entrevistas y exámenes

- *"¿Cuándo elegirías NoSQL sobre SQL?"* → Esquema flexible, escalado horizontal, patrón de acceso simple y conocido, alta disponibilidad prioritaria.
- *"Tipos de NoSQL y un ejemplo de cada"* → Documental (MongoDB), clave-valor (Redis), columnar (Cassandra), grafos (Neo4j).
- *"¿Escala SQL horizontalmente?"* → Con réplicas de lectura, sharding o SQL distribuido; más complejo que en NoSQL nativo.
- *"¿Qué es la persistencia políglota?"* → Usar varios motores, cada uno para lo que mejor hace.
- En **AZ-900**: Azure SQL Database (relacional gestionado) vs [[Azure Cosmos DB]] (NoSQL multi-modelo, distribución global).

## Referencias

- Martin Fowler & Pramod Sadalage, *NoSQL Distilled* (Addison-Wesley, 2012)
- Martin Kleppmann, *Designing Data-Intensive Applications* (O'Reilly, 2017), capítulos 2 y 5
- Microsoft Learn, *Relational vs. NoSQL data*: https://learn.microsoft.com/dotnet/architecture/cloud-native/relational-vs-nosql-data
- MongoDB, *Transactions*: https://www.mongodb.com/docs/manual/core/transactions/

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
