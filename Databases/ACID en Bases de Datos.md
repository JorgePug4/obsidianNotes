---
tags: [bases-de-datos, sql, transacciones, acid]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [ACID]
---

# ACID en Bases de Datos

> [!info] Acrónimo acuñado por Theo Härder y Andreas Reuter (1983), a partir del trabajo de Jim Gray sobre transacciones.

## ¿Qué es?

**ACID** describe las **4 propiedades que garantiza una transacción** en una base de datos para que los datos sigan siendo correctos incluso ante errores, concurrencia y fallos de hardware.

| Letra | Significado | Traducción | En una frase |
|---|---|---|---|
| **A** | Atomicity | Atomicidad | Todo o nada |
| **C** | Consistency | Consistencia | De un estado válido a otro estado válido |
| **I** | Isolation | Aislamiento | Las transacciones concurrentes no se interfieren |
| **D** | Durability | Durabilidad | Lo confirmado no se pierde |

### ¿Qué es una transacción?

Una transacción es un **grupo de operaciones que deben ejecutarse como una sola unidad**.

```text
Restar $100 de la cuenta A
+
Sumar  $100 a la cuenta B
```

Si una operación falla, **toda la transacción debe revertirse**: no puede quedar el dinero restado de A y no sumado a B.

## ¿Para qué sirve?

ACID es crítico en sistemas donde un dato incorrecto tiene consecuencias reales:

- Banca y pagos
- E-commerce (pedidos, inventario)
- ERP, facturación, contabilidad
- Reservas (asientos, habitaciones)

Garantiza **integridad de datos, consistencia, seguridad transaccional y recuperación ante fallos** sin que la aplicación tenga que programar esas garantías a mano.

## Conceptos relacionados

- [[Bases de Datos SQL vs NoSQL]] → las relacionales ofrecen ACID completo; muchas NoSQL lo ofrecen por documento o partición.
- [[Teorema CAP y BASE]] → **BASE** es el modelo alternativo de los sistemas distribuidos que relajan ACID. Ojo: la "C" de ACID y la "C" de CAP **no significan lo mismo**.
- [[Saga Pattern]] → cómo conseguir "atomicidad de negocio" entre microservicios cuando no hay una transacción ACID que los abarque.
- [[Transactional Outbox]] → aprovecha la atomicidad de la transacción local para publicar eventos de forma fiable.
- [[Idempotencia]] → una restricción `UNIQUE` es la forma más simple de idempotencia en base de datos.
- [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] → un Agregado es el límite de una transacción.

## ¿Cómo funciona? Propiedad por propiedad

### A — Atomicity (Atomicidad)

La transacción **ocurre completamente o no ocurre**. No hay estados intermedios visibles ni persistentes.

```text
1. Cobrar tarjeta
2. Crear orden
3. Descontar inventario
```

Si falla el paso 3, `ROLLBACK` revierte los pasos 1 y 2. La base de datos lo implementa con un **log de deshacer** (*undo log*) que guarda los valores anteriores.

### C — Consistency (Consistencia)

La base de datos siempre pasa **de un estado válido a otro estado válido**. Las reglas de integridad definidas (restricciones `CHECK`, `UNIQUE`, `NOT NULL`, `FOREIGN KEY`, *triggers*) nunca se rompen al confirmar.

```sql
ALTER TABLE productos ADD CONSTRAINT stock_no_negativo CHECK (stock >= 0);
```

Con esa restricción la base **nunca** permitirá `stock = -5`: la transacción que lo intente fallará y se revertirá.

> [!note] La consistencia es responsabilidad compartida
> La base de datos garantiza las reglas **que le has declarado**. Si la regla "un pedido no puede tener más de 50 líneas" solo existe en tu cabeza, la base no la protegerá. Por eso la "C" de ACID depende en parte de la aplicación (y en DDD, de las invariantes del Agregado).

### I — Isolation (Aislamiento)

Las transacciones **concurrentes no deben interferir entre sí**; el resultado debe ser como si se hubieran ejecutado una tras otra.

#### El problema clásico

```text
Stock = 1. Dos usuarios compran el último producto al mismo tiempo.

Sin aislamiento adecuado:
  T1 lee stock = 1  ✔          T2 lee stock = 1  ✔
  T1 escribe stock = 0         T2 escribe stock = 0
  → Ambos compran. Stock real: -1 (o 0 con dos ventas).

Con aislamiento correcto:
  T1 compra ✔ y bloquea la fila
  T2 espera, relee stock = 0 → "sin stock"
```

> [!warning] Aislamiento no es automático en todos los niveles
> En el nivel por defecto de la mayoría de bases (*Read Committed*), el escenario anterior **sí puede fallar**: ambas transacciones leen `1` antes de que ninguna escriba. Para protegerse hay que usar una **escritura condicional** (`UPDATE productos SET stock = stock - 1 WHERE id = 7 AND stock > 0` y comprobar las filas afectadas), un **bloqueo explícito** (`SELECT ... FOR UPDATE`), **concurrencia optimista** (columna de versión, `RowVersion` en EF Core) o el nivel *Serializable*.

#### Anomalías de concurrencia

| Anomalía | Qué ocurre |
|---|---|
| **Lectura sucia** (*dirty read*) | T2 lee un dato que T1 modificó pero **aún no confirmó**; si T1 hace rollback, T2 leyó algo que nunca existió |
| **Lectura no repetible** (*non-repeatable read*) | T1 lee una fila, T2 la modifica y confirma, T1 vuelve a leerla y **obtiene otro valor** |
| **Lectura fantasma** (*phantom read*) | T1 ejecuta `SELECT ... WHERE precio > 100`, T2 inserta una fila que cumple, T1 repite la consulta y **aparecen filas nuevas** |
| **Actualización perdida** (*lost update*) | Dos transacciones leen el mismo valor y ambas escriben; la segunda **sobrescribe** la primera (el ejemplo del stock) |

#### Niveles de aislamiento (SQL estándar)

| Nivel | Lectura sucia | No repetible | Fantasma | Rendimiento | Notas |
|---|---|---|---|---|---|
| **Read Uncommitted** | ❌ Posible | ❌ Posible | ❌ Posible | Máximo | Casi nunca se usa |
| **Read Committed** | ✅ Evitada | ❌ Posible | ❌ Posible | Alto | **Por defecto** en PostgreSQL, SQL Server, Oracle |
| **Repeatable Read** | ✅ Evitada | ✅ Evitada | ❌ Posible* | Medio | **Por defecto** en MySQL/InnoDB (*que además evita fantasmas en la práctica) |
| **Serializable** | ✅ Evitada | ✅ Evitada | ✅ Evitada | Menor | Equivale a ejecución secuencial; puede abortar transacciones por conflicto |

Además, SQL Server ofrece **Snapshot** y PostgreSQL implementa los niveles con **MVCC** (*Multi-Version Concurrency Control*): los lectores no bloquean a los escritores porque cada transacción ve una "foto" de los datos.

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
-- ...
COMMIT;
```

### D — Durability (Durabilidad)

Una vez que la transacción hace `COMMIT`, los cambios **no se pierden** aunque ocurra un reinicio del servidor, un *crash* del proceso o un apagón.

Se implementa con el **log de transacciones** (*Write-Ahead Log*, WAL, o *transaction log*): antes de confirmar, la base escribe el cambio en el log **en disco**. Si el servidor cae, al arrancar **reproduce el log** para recuperar todo lo confirmado.

> [!tip] Durabilidad en la nube
> En servicios gestionados, la durabilidad se refuerza replicando el log en varias zonas. Ver [[Redundancia de almacenamiento]] para el equivalente en Azure Storage.

## Ejemplo completo

### Compra en e-commerce

```text
1. Crear orden
2. Cobrar pago
3. Descontar inventario
4. Generar factura
```

Todo ocurre dentro de **una transacción**. Si algo falla, `ROLLBACK` y nada queda incompleto.

### En SQL

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Si la cuenta 1 tiene CHECK (balance >= 0) y se queda negativa,
-- el UPDATE falla y hacemos:
-- ROLLBACK;

COMMIT;
```

### En .NET con EF Core

`SaveChangesAsync()` envuelve todos los cambios pendientes en **una transacción** automáticamente:

```csharp
using var tx = await _db.Database.BeginTransactionAsync(); // solo si necesitas varios SaveChanges

var origen  = await _db.Accounts.FindAsync(1);
var destino = await _db.Accounts.FindAsync(2);

origen.Balance  -= 100;
destino.Balance += 100;

await _db.SaveChangesAsync();   // un solo INSERT/UPDATE transaccional
await tx.CommitAsync();
```

Para concurrencia optimista, EF Core usa una columna de versión:

```csharp
public class Product
{
    public int Id { get; set; }
    public int Stock { get; set; }
    [Timestamp] public byte[] RowVersion { get; set; } // SQL Server rowversion
}
// Si otro proceso modificó la fila entre la lectura y SaveChanges:
// → DbUpdateConcurrencyException. Reintentar o informar al usuario.
```

## ACID en el mundo distribuido

| Escenario | ¿ACID? | Alternativa |
|---|---|---|
| Una base de datos relacional | ✅ Completo | — |
| Una base NoSQL (MongoDB, Cosmos DB) | ✅ Por documento o partición; multi-documento en algunas | Diseñar el documento como unidad de consistencia |
| Varias bases / varios microservicios | ❌ No hay transacción que las abarque | **2PC** (bloqueante, poco usado) o **[[Saga Pattern]]** (consistencia eventual) |

## Comparación: ACID vs BASE

| | ACID | BASE |
|---|---|---|
| Significado | Atomicity, Consistency, Isolation, Durability | **B**asically **A**vailable, **S**oft state, **E**ventually consistent |
| Prioridad | Corrección de los datos | Disponibilidad y escalabilidad |
| Consistencia | Fuerte e inmediata | Eventual |
| Típico en | Relacionales, sistemas transaccionales | NoSQL distribuidas, microservicios |
| Ver | Esta nota | [[Teorema CAP y BASE]] |

## Puntos clave

```text
A → All or nothing        (todo o nada)
C → Correct data          (reglas declaradas nunca se rompen)
I → Independent           (concurrentes no se pisan)
D → Data persists         (lo confirmado sobrevive)
```

- El **aislamiento tiene niveles**; el por defecto (*Read Committed*) **no** evita la actualización perdida.
- La durabilidad se apoya en el **log de transacciones (WAL)**.
- Entre microservicios no hay ACID: hay **Sagas** y consistencia eventual.

## Errores comunes

- Creer que "usar transacciones" protege automáticamente contra el problema del stock (hace falta el nivel de aislamiento o el bloqueo adecuado).
- Transacciones **largas** que mantienen bloqueos mientras se llama a una API externa.
- Confundir la "C" de ACID (integridad declarada) con la "C" de CAP (todos los nodos ven el mismo dato).
- Poner en la aplicación reglas que deberían ser restricciones de la base (`UNIQUE`, `CHECK`).
- Asumir que un `SaveChanges` en EF Core y una publicación a RabbitMQ son atómicos juntos (ver [[Transactional Outbox]]).

## 🎯 Para entrevistas y exámenes

- Definir las 4 propiedades con un ejemplo cada una.
- *"¿Qué nivel de aislamiento usa tu base por defecto y qué anomalías permite?"* → Read Committed en SQL Server / PostgreSQL: permite lecturas no repetibles y fantasmas.
- *"¿Cómo evitas la actualización perdida?"* → Bloqueo pesimista (`FOR UPDATE`), concurrencia optimista (versión), o escritura condicional.
- *"¿Cómo garantizas atomicidad entre dos microservicios?"* → No se puede con ACID; se usa Saga con compensaciones.

### Respuesta corta para entrevista

> ACID garantiza integridad transaccional mediante atomicidad, consistencia, aislamiento y durabilidad, asegurando que las operaciones críticas se ejecuten de forma segura y confiable incluso ante fallos y concurrencia.

## Referencias

- Härder & Reuter, *Principles of Transaction-Oriented Database Recovery* (1983)
- Martin Kleppmann, *Designing Data-Intensive Applications* (O'Reilly, 2017), capítulo 7 *Transactions*
- PostgreSQL, *Transaction Isolation*: https://www.postgresql.org/docs/current/transaction-iso.html
- Microsoft Learn, *SET TRANSACTION ISOLATION LEVEL*: https://learn.microsoft.com/sql/t-sql/statements/set-transaction-isolation-level-transact-sql
- EF Core, *Handling Concurrency Conflicts*: https://learn.microsoft.com/ef/core/saving/concurrency

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
