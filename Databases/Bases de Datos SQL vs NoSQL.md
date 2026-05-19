
## Introducción
La elección entre una base de datos relacional (SQL) y una no relacional (NoSQL) depende principalmente de la **estructura de tus datos** y los **requisitos de escalabilidad** de tu proyecto.

---

## 1. Bases de Datos Relacionales (SQL)
*Ejemplos: PostgreSQL, MySQL, SQL Server.*

### ¿Cuándo utilizar SQL?
- **Estructura fija:** Los datos tienen un esquema predefinido y no cambian constantemente.
- **Integridad ACID:** Necesitas transacciones seguras donde la consistencia es crítica (ej. sistemas bancarios).
- **Consultas complejas:** Necesitas realizar múltiples `JOINs` para relacionar datos entre tablas.

[[ACID en Bases de Datos]]

### Características Principales
- **Estructura:** Basada en tablas (filas y columnas).
- **Lenguaje:** SQL (Structured Query Language).
- **Escalabilidad:** Principalmente vertical (añadir más potencia al servidor).

---

## 2. Bases de Datos No Relacionales (NoSQL)
*Ejemplos: MongoDB, DynamoDB, Redis.*

### ¿Cuándo utilizar NoSQL?
- **Estructura flexible:** Los datos pueden variar entre registros (documentos, clave-valor, etc.).
- **Escalabilidad masiva:** Necesitas escalar horizontalmente (añadir más servidores al clúster) de forma rápida.
- **Velocidad y disponibilidad:** Proyectos donde la latencia baja y el alto volumen de escritura son más importantes que la consistencia estricta.

### Tipos Comunes
- **Documentales:** (Ej. MongoDB) Almacenan datos en formato similar a JSON.
- **Clave-Valor:** (Ej. Redis) Ideales para caché y sesiones.
- **Grafos:** (Ej. Neo4j) Optimizadas para relaciones complejas.

---

## Tabla Comparativa Rápida

| Característica | SQL (Relacional) | NoSQL (No Relacional) |
| :--- | :--- | :--- |
| **Esquema** | Rígido / Predefinido | Flexible / Dinámico |
| **Escalabilidad** | Vertical | Horizontal |
| **Relaciones** | Fuertes (JOINs) | Débiles o inexistentes |
| **Consistencia** | Alta (ACID) | Variable (BASE) |

---

## Conclusión: ¿Cómo decidir?
1. **Elige SQL si:** La consistencia, los datos estructurados y las relaciones complejas son el núcleo de tu aplicación.
2. **Elige NoSQL si:** Priorizas la velocidad de desarrollo, el manejo de grandes volúmenes de datos no estructurados o una escalabilidad horizontal sencilla.



