---
title: Claves y keyspace en Redis
aliases:
  - Keyspace
  - Claves en Redis
  - Convención de nombres de claves
  - SCAN vs KEYS
tags:
  - redis
  - backend
  - architecture
  - learning
  - redis/nivel-1
status: pending
level: beginner
type: lesson
course: redis
course_level: 1
order: 2
mastery: 0
created: 2026-11-03
last_review:
next_review:
---

# Claves y keyspace en Redis

> [!abstract] En 30 segundos
> - **Qué es:** el *keyspace* es el diccionario de todas las claves de una base lógica. Cada clave es un string binario que apunta a un valor de un tipo concreto.
> - **Para qué:** diseñar nombres de clave legibles y estables, gestionarlas (existir, borrar, renombrar, copiar) y recorrerlas sin bloquear el servidor.
> - **Recuerda:** convención `servicio:entidad:id[:subrecurso]`; `SCAN` en lugar de `KEYS`; `UNLINK` para borrar valores grandes; evita las bases lógicas numeradas.

> [!info] Lección 1.2 · Nivel 1 — Redis básico y comandos fundamentales
> ⬅️ [[redis-cli y Redis Insight]] · ➡️ [[Redis Strings]] · 🗺️ [[01 - Redis - Roadmap#Nivel 1 — Redis básico y comandos fundamentales|Roadmap]] · 📊 [[03 - Redis - Progreso|Progreso]]
> 🏷️ Core · `COPY` ≥6.2 · `SCAN … TYPE` ≥6.0 · Redis Cluster solo admite la base 0 (Valkey 9 admite bases numeradas en cluster: **Valkey≠**).

## 📌 Definición

Una **clave** es un string *binary-safe* (puede contener cualquier byte) que identifica un valor. El **keyspace** es la tabla hash que asocia cada clave con su valor y su tipo. Redis no tiene esquemas ni tablas: el «esquema» son tus **convenciones de nombres**.

Por defecto existen 16 **bases lógicas** (`databases 16`), numeradas del 0 al 15 y seleccionables con `SELECT`. Comparten proceso, memoria y configuración.

## 🎯 ¿Qué problema resuelve?

Sin una convención, en seis meses nadie sabe qué servicio escribió `u42`, si se puede borrar o cuánto ocupa. Con buenas claves:
- Puedes buscar, medir y limpiar por prefijo (`--scan --pattern 'shop:cart:*'`).
- Puedes dar permisos ACL por prefijo (`~shop:*`).
- Puedes analizar la memoria por prefijo en Redis Insight.
- Evitas colisiones entre servicios.

## 🧠 Conceptos fundamentales

### Convención de nombres

```text
servicio:entidad:id[:subrecurso][:versión]

shop:product:42
shop:product:42:stock
auth:session:9f1c2d
shop:cart:user:42
rate:api:ip:203.0.113.7:202611031015
shop:product:v2:42          ← versión del formato cuando cambia el esquema serializado
shop:{user:42}:cart         ← hash tag para Redis Cluster (Nivel 7)
```

- Separador `:` (convención universal; Redis Insight agrupa por él).
- Minúsculas, sin espacios, cortas pero legibles: la clave se guarda en memoria con cada valor.
- El prefijo de servicio permite ACL y limpieza por prefijo.

### Operaciones sobre claves

| Comando | Qué hace | Complejidad |
|---|---|---|
| `EXISTS k1 [k2 …]` | Cuántas de las claves existen | O(N) claves |
| `TYPE k` | Tipo del valor (`string`, `hash`, `list`…) | O(1) |
| `DEL k1 [k2 …]` | Borra; con colecciones libera la memoria en el hilo principal | O(M) elementos |
| `UNLINK k1 [k2 …]` | Desvincula en O(1) y libera la memoria en segundo plano | O(1) por clave |
| `RENAME k nueva` / `RENAMENX` | Renombra (sobrescribe / solo si la nueva no existe); conserva el TTL | O(1) |
| `COPY origen destino [DB n] [REPLACE]` | Copia una clave | O(N) |
| `SCAN cursor [MATCH patrón] [COUNT n] [TYPE tipo]` | Recorre el keyspace por lotes | O(1) por llamada |
| `KEYS patrón` | Devuelve todas las claves que coinciden. **Bloquea**: solo para depurar | O(N) |
| `DBSIZE` | Número de claves de la base | O(1) |
| `RANDOMKEY` | Una clave aleatoria | O(1) |
| `FLUSHDB` / `FLUSHALL [ASYNC]` | Vacía la base o todas | O(N) |

### `SCAN`: cómo recorrer sin bloquear
- Empieza con el cursor `0` y termina cuando el servidor devuelve `0`.
- `COUNT` es una **pista** de cuánto trabajo hacer por llamada, no el número de claves devueltas.
- `MATCH` se aplica **después** de recoger las claves: con un patrón muy selectivo, muchas llamadas devuelven listas vacías.
- Garantías: toda clave que exista durante todo el recorrido se devuelve **al menos una vez** (puede repetirse); las que se añaden o borran durante el recorrido pueden aparecer o no.

### Bases lógicas: por qué evitarlas
- Comparten memoria, `maxmemory`, política de desalojo y CPU: no aíslan nada importante.
- Redis Cluster solo admite la base 0.
- Muchas herramientas y clientes asumen la base 0.
- Para aislar, usa prefijos (y ACL por prefijo) o instancias distintas.

### Analogía
El keyspace es el **índice de un archivo de oficina**: cada carpeta tiene una etiqueta (la clave) y dentro hay un tipo de documento (el valor). Una buena convención de etiquetas («Ventas / Clientes / 42 / Contratos») permite encontrar cualquier cosa sin abrir todas las carpetas.
**Dónde se rompe:** en Redis no hay carpetas de verdad: `shop:product:42` es un nombre plano; la jerarquía es solo convención.

## ⚙️ Funcionamiento interno

- Cada base lógica tiene una tabla hash para las claves y otra para las expiraciones.
- `SCAN` recorre los *buckets* de la tabla hash con un cursor que tiene en cuenta el rehash incremental; por eso puede devolver duplicados, pero no se salta claves estables.
- `RENAME` mueve el valor a otra entrada y transfiere el TTL; si el destino existía, lo borra (con el mismo coste que un `DEL`).

## 💻 Ejemplos prácticos

```text
127.0.0.1:6379> SET shop:product:1 "teclado"
OK
127.0.0.1:6379> HSET shop:product:1:stock almacen 30 tienda 4
(integer) 2
127.0.0.1:6379> EXISTS shop:product:1 shop:product:2
(integer) 1
127.0.0.1:6379> TYPE shop:product:1:stock
hash
127.0.0.1:6379> SCAN 0 MATCH shop:product:* COUNT 100
1) "0"
2) 1) "shop:product:1"
   2) "shop:product:1:stock"
127.0.0.1:6379> SCAN 0 MATCH shop:* TYPE hash
1) "0"
2) 1) "shop:product:1:stock"
127.0.0.1:6379> COPY shop:product:1 shop:product:1:backup
(integer) 1
127.0.0.1:6379> UNLINK shop:product:1:backup
(integer) 1
```

**Renombrado atómico para publicar datos recalculados** (patrón muy útil):

```text
DEL ranking:tmp
ZADD ranking:tmp 100 ana 90 luis      # se construye la nueva versión aparte
RENAME ranking:tmp ranking:actual     # los lectores ven la vieja o la nueva, nunca una mezcla
```

## 🔧 Implementación en .NET

```csharp
using StackExchange.Redis;
using StackExchange.Redis.KeyspaceIsolation;

IDatabase db = mux.GetDatabase();

// Prefijo automático para todo un servicio: evita repetir "shop:" en cada llamada
IDatabase shop = db.WithKeyPrefix("shop:");
await shop.StringSetAsync("product:1", "teclado");     // escribe "shop:product:1"

bool exists = await db.KeyExistsAsync("shop:product:1");
RedisType type = await db.KeyTypeAsync("shop:product:1");             // RedisType.String
await db.KeyRenameAsync("ranking:tmp", "ranking:actual");             // RENAME
await db.KeyDeleteAsync("shop:product:1");                            // DEL
await db.ExecuteAsync("UNLINK", "coleccion:grande");                  // UNLINK (comando genérico)

// Recorrer claves con SCAN (para tareas de administración, no en peticiones)
IServer server = mux.GetServer(mux.GetEndPoints()[0]);
await foreach (RedisKey key in server.KeysAsync(pattern: "shop:product:*", pageSize: 500))
{
    Console.WriteLine(key);
}
```

> [!tip] Centraliza los nombres de clave
> Una clase estática `Keys` con métodos como `Keys.Product(id) => $"shop:product:{id}"` evita errores tipográficos y facilita cambiar la convención (o la versión) en un solo sitio.

## 🏗️ Casos de uso en arquitectura

| Decisión | Recomendación |
|---|---|
| Varios servicios comparten un Redis | Prefijo por servicio + usuario ACL limitado a ese prefijo (`~orders:*`) |
| Multi-tenant | Prefijo por tenant (`t:{tenantId}:…`) o instancias separadas si hay requisitos de aislamiento fuertes |
| Cambio de formato serializado | Versión en la clave (`v2`) para que convivan ambas durante el despliegue |
| Futuro Redis Cluster | Diseñar ya los *hash tags* para las claves que se usan juntas ([[Hash slots]]) |

### Cuándo NO usar bases lógicas
- Para separar entornos (desarrollo/producción) o servicios: comparten memoria y política; usa instancias o prefijos.

### Consideraciones de arquitectura
- **Memoria:** claves largas × millones de claves = muchos MB.
- **Seguridad:** el prefijo es la unidad de permisos en ACL.
- **Operación:** limpiar por prefijo con `SCAN` + `UNLINK`, o mejor con TTL.

## ⚠️ Errores y anti-patterns

| Error | Por qué duele | Cómo evitarlo |
|---|---|---|
| `KEYS *` en producción | Bloqueo del servidor | `SCAN` |
| Claves sin prefijo de servicio | Colisiones, ACL imposibles, limpieza imposible | Convención obligatoria |
| Usar `SELECT` para separar servicios | No aísla memoria ni política; incompatible con Cluster | Prefijos o instancias |
| `DEL` de colecciones enormes | Pico de latencia | `UNLINK` |
| Esperar que `SCAN … COUNT 10` devuelva 10 claves | Puede devolver 0 o 50 | Iterar hasta cursor `0` |
| Claves con datos sensibles en el nombre (emails, DNI) | Aparecen en logs, slowlog y herramientas | Usar identificadores opacos o hashes |

## ✅ Buenas prácticas

1. Convención `servicio:entidad:id[:subrecurso]` documentada y centralizada en código.
2. Solo base 0; aislamiento con prefijos, ACL o instancias.
3. `SCAN` para recorrer, `UNLINK` para borrar grande.
4. `RENAME` para publicar de forma atómica datos recalculados.
5. Nada de datos personales en los nombres de clave.

## 🎤 Preguntas de entrevista

> [!question]- 1. ¿Por qué no usar `KEYS` en producción y qué usas en su lugar?
> `KEYS` recorre todo el keyspace en una sola llamada, O(N), bloqueando el único hilo de ejecución. `SCAN` recorre por lotes con un cursor, cada llamada es corta y no bloquea; a cambio, puede devolver duplicados y no garantiza ver claves creadas durante el recorrido.

> [!question]- 2. ¿Usarías las bases lógicas para separar microservicios?
> No. Comparten memoria, `maxmemory`, política de desalojo y CPU, y Redis Cluster solo admite la base 0. Uso prefijos con ACL por prefijo, o instancias distintas si necesito aislamiento real.

> [!question]- 3. ¿Cómo publicarías un ranking recalculado sin que los lectores vean datos a medias?
> Construyéndolo en una clave temporal y haciendo `RENAME` a la clave definitiva: el cambio es atómico.

> [!question]- 4. ¿Qué diferencia hay entre `DEL` y `UNLINK`?
> `DEL` libera la memoria del valor en el hilo principal (O(N) para colecciones); `UNLINK` desvincula la clave en O(1) y libera la memoria en un hilo de fondo.

## 🧪 Ejercicios

### 🟢 Básicos
1. Define la convención de claves para un e-commerce: producto, stock, carrito, sesión, ranking diario.
2. Recorre con `SCAN` un prefijo con 10 000 claves y cuenta cuántas llamadas hacen falta con `COUNT 10` y con `COUNT 1000`.

### 🟡 Intermedios
3. Implementa en .NET una clase `Keys` con la convención del ejercicio 1 y úsala con `WithKeyPrefix`.
4. Escribe un job que borre con `SCAN` + `UNLINK` todas las claves `tmp:*` en lotes de 500.

### 🔴 Reto Senior
5. Tres equipos comparten una instancia. Diseña prefijos, usuarios ACL y una política para que ninguno pueda leer ni borrar datos de otro, y explica qué riesgos de compartir instancia **no** resuelve esa configuración.

> [!success]- Soluciones
> **2.** Con `COUNT 10` muchas más llamadas, varias devolviendo listas vacías; con `COUNT 1000`, pocas llamadas más pesadas.
> **5.** Un usuario por equipo con `~equipo:*` y `&equipo:*` y sin `@dangerous`. No resuelve: memoria y política de desalojo compartidas (un equipo puede provocar desalojos en otro), CPU compartida (un comando lento afecta a todos), ni el radio de impacto de un fallo o un upgrade. Para eso, instancias separadas.

### ✔️ Checklist de dominio
- [ ] Diseño claves con una convención clara.
- [ ] Recorro con `SCAN` entendiendo sus garantías.
- [ ] Sé cuándo usar `UNLINK`, `RENAME` y `COPY`.
- [ ] Sé por qué evitar las bases lógicas.

## 📝 Resumen

- Las claves son strings binarios en un diccionario plano; la jerarquía es convención.
- `servicio:entidad:id[:subrecurso]`, cortas y sin datos personales.
- `SCAN` en lugar de `KEYS`; `UNLINK` en lugar de `DEL` para valores grandes.
- `RENAME` para publicar de forma atómica.
- Solo base 0; aislamiento con prefijos, ACL o instancias.

## 🔗 Referencias

- [Keys and values](https://redis.io/docs/latest/develop/using-commands/keyspace/)
- [SCAN](https://redis.io/docs/latest/commands/scan/)
- [UNLINK](https://redis.io/docs/latest/commands/unlink/)

## 🔄 Notas relacionadas

- ⬅️ [[redis-cli y Redis Insight]] · ➡️ [[Redis Strings]]
- Más adelante: [[Expiración y TTL]] · [[Redis ACL]] · [[Hash slots]] · [[Modelado de datos en Redis]]
