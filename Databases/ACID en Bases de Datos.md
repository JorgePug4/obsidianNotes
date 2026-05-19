ACID es un acrónimo que describe 4 propiedades fundamentales de una transacción en bases de datos.

|Letra|Significado|Traducción|
|---|---|---|
|A|Atomicity|Atomicidad|
|C|Consistency|Consistencia|
|I|Isolation|Aislamiento|
|D|Durability|Durabilidad|

---

# ¿Qué es una transacción?

Una transacción es un grupo de operaciones que deben ejecutarse como una sola unidad.

Ejemplo:

```text
Restar $100 de cuenta A
+
Agregar $100 a cuenta B
```

Si una operación falla:

```text
Toda la transacción debe revertirse
```

---

# A — Atomicity (Atomicidad)

La transacción ocurre completamente o no ocurre.

```text
Todo o nada
```

## Ejemplo

```text
1. Cobrar tarjeta
2. Crear orden
3. Descontar inventario
```

Si falla el paso 3:

```sql
ROLLBACK;
```

La base revierte todos los cambios.

---

# C — Consistency (Consistencia)

La base de datos siempre pasa de un estado válido a otro estado válido.

Las reglas e integridad de los datos nunca deben romperse.

## Ejemplo

```text
inventario >= 0
```

La base nunca debería permitir:

```text
inventario = -5
```

---

# I — Isolation (Aislamiento)

Las transacciones concurrentes no deben interferir entre sí.

```text
Cada transacción trabaja de forma aislada
```

## Ejemplo

```text
Stock = 1
```

Dos usuarios intentan comprar el último producto al mismo tiempo.

Sin aislamiento:

```text
Usuario A compra ✔
Usuario B compra ✔
```

Resultado:

```text
Stock = -1
```

Con aislamiento:

```text
Usuario A compra ✔
Usuario B recibe "sin stock"
```

---

# D — Durability (Durabilidad)

Una vez confirmada la transacción:

```sql
COMMIT;
```

Los cambios no se pierden aunque exista:

- reinicio del servidor
    
- crash del sistema
    
- apagón
    

```text
Los datos persisten
```

---

# Ejemplo completo ACID

## Compra en e-commerce

```text
1. Crear orden
2. Cobrar pago
3. Descontar inventario
4. Generar factura
```

Todo ocurre dentro de una transacción.

Si algo falla:

```sql
ROLLBACK;
```

Nada queda incompleto.

---

# ¿Por qué es importante?

ACID es crítico en sistemas como:

- banca
    
- pagos
    
- e-commerce
    
- ERP
    
- inventario
    
- facturación
    

Porque garantiza:

- integridad de datos
    
- consistencia
    
- seguridad transaccional
    
- recuperación ante fallos
    

---

# Ejemplo SQL real

```sql
BEGIN TRANSACTION;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Si ocurre un error:

```sql
ROLLBACK;
```

---

# Resumen rápido

```text
A → All or nothing
C → Correct data
I → Independent transactions
D → Data persists
```

---

# Respuesta corta para entrevista

> ACID garantiza integridad transaccional mediante atomicidad, consistencia, aislamiento y durabilidad, asegurando que las operaciones críticas se ejecuten de forma segura y confiable.