---
title: Contadores atómicos con INCR y DECR
aliases:
  - INCR
  - DECR
  - INCREX
  - Contadores en Redis
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
order: 4
mastery: 0
created: 2026-11-03
last_review:
next_review:
---

# Contadores atómicos con INCR y DECR

> [!abstract] En 30 segundos
> - **Qué es:** comandos que suman o restan a un número guardado en un string, **de forma atómica**, devolviendo el nuevo valor.
> - **Para qué:** contadores de visitas, cuotas, generación de IDs, stock, rate limiting.
> - **Recuerda:** `INCR` + `EXPIRE` son dos comandos y no son atómicos entre sí; desde Redis 8.8, `INCREX` incrementa, acota y fija la expiración en uno solo.

> [!info] Lección 1.4 · Nivel 1 — Redis básico y comandos fundamentales
> ⬅️ [[Redis Strings]] · ➡️ [[Expiración y TTL]] · 🗺️ [[01 - Redis - Roadmap#Nivel 1 — Redis básico y comandos fundamentales|Roadmap]] · 📊 [[03 - Redis - Progreso|Progreso]]
> 🏷️ Core · `INCREX` ≥8.8 (Redis Open Source y Redis Software; consulta la tabla de compatibilidad de Redis Cloud antes de usarlo).

## 📌 Definición

| Comando | Qué hace |
|---|---|
| `INCR k` / `DECR k` | Suma o resta 1 |
| `INCRBY k n` / `DECRBY k n` | Suma o resta un entero de 64 bits |
| `INCRBYFLOAT k f` | Suma un número decimal (puede ser negativo) |
| `INCREX k [BYINT n / BYFLOAT f] [LBOUND l] [UBOUND u] [SATURATE] [EX…/PX…/EXAT…/PXAT…/PERSIST] [ENX]` (🏷️ ≥8.8) | Incrementa con límites y control de expiración en una sola operación atómica |

Si la clave no existe, se parte de `0`. Si el valor no es un número, el comando falla con `ERR value is not an integer or out of range`. Los enteros son de 64 bits con signo: un desbordamiento devuelve error en lugar de dar la vuelta.

## 🎯 ¿Qué problema resuelve?

La versión ingenua de un contador («leo 41, sumo 1, escribo 42») pierde incrementos cuando dos procesos lo hacen a la vez (el *lost update* que viste en [[Qué es Redis]]). `INCR` hace la lectura, la suma y la escritura dentro de un único comando que nadie puede interrumpir, así que 1000 instancias pueden incrementar el mismo contador sin perder ni uno.

## 🧠 Conceptos fundamentales

1. **Atomicidad por comando:** cada incremento es indivisible, gracias al [[Modelo de ejecución single-threaded]].
2. **Devuelven el nuevo valor:** sirven para tomar decisiones sin una lectura extra («si el resultado supera 100 → rechazar»).
3. **Contadores con ventana:** una clave por periodo (`visitas:2026-11-03`, `rate:ip:…:202611031015`) con TTL.
4. **El problema de `INCR` + `EXPIRE`:** son dos comandos. Si el proceso cae entre ambos, la clave queda sin TTL para siempre. Soluciones:
   - `EXPIRE … NX` (≥7.0) en cada petición: el TTL se fija la primera vez que alguien lo consigue.
   - Un script Lua o una transacción `MULTI`/`EXEC`.
   - **`INCREX … EX 60 ENX`** (≥8.8): incrementa y fija el TTL solo si la clave no lo tenía, en un comando.
5. **`INCREX` y los límites:** con `UBOUND 100`, un incremento que superaría 100 no se aplica y devuelve `[valor_actual, 0]`; con `SATURATE`, se recorta al límite. Responde siempre un array: `[nuevo_valor, incremento_aplicado]`.
6. **Floats:** `INCRBYFLOAT` guarda el resultado como string; acumular decimales tiene los problemas de redondeo habituales: para dinero, usa enteros en céntimos.

### Analogía
Un **torniquete de estadio**: cada persona que pasa suma uno al contador, y es imposible que dos personas pasen «a la vez» y cuenten como una. `INCREX` es un torniquete con aforo: cuando llega al máximo, deja de girar.
**Dónde se rompe:** el torniquete no se resetea solo cada hora; un contador por ventana sí, gracias al TTL.

## ⚙️ Funcionamiento interno

- El valor se guarda con codificación `int` cuando cabe en 64 bits, así que incrementar no requiere parsear texto.
- `INCR` sobre un valor `embstr`/`raw` que representa un número lo convierte; si no es numérico, error.
- `INCREX` comprueba los límites antes de aplicar el cambio: si se viola un límite (y no hay `SATURATE`), no toca ni el valor ni el TTL.

## 💻 Ejemplos prácticos

```text
127.0.0.1:6379> INCR visitas:home
(integer) 1
127.0.0.1:6379> INCRBY visitas:home 10
(integer) 11
127.0.0.1:6379> DECRBY stock:producto:42 3
(integer) -3
127.0.0.1:6379> INCRBYFLOAT temperatura:sensor:7 0.5
"0.5"
127.0.0.1:6379> SET nombre "ana"
OK
127.0.0.1:6379> INCR nombre
(error) ERR value is not an integer or out of range
```

**Generador de IDs** (secuencia global, sin huecos salvo que una transacción de negocio falle después):

```text
127.0.0.1:6379> INCR seq:pedido
(integer) 1001
```

**Rate limiting por minuto con `INCREX`** (🏷️ ≥8.8):

```text
127.0.0.1:6379> INCREX rate:user:42:202611031015 BYINT 1 UBOUND 100 EX 60 ENX
1) (integer) 1
2) (integer) 1
…                                        # 99 peticiones después
127.0.0.1:6379> INCREX rate:user:42:202611031015 BYINT 1 UBOUND 100 EX 60 ENX
1) (integer) 100
2) (integer) 1
127.0.0.1:6379> INCREX rate:user:42:202611031015 BYINT 1 UBOUND 100 EX 60 ENX
1) (integer) 100
2) (integer) 0                           # límite alcanzado: rechazar con 429
```

Para versiones anteriores a 8.8:

```text
INCR rate:user:42:202611031015
EXPIRE rate:user:42:202611031015 60 NX
```

## 🔧 Implementación en .NET

```csharp
using StackExchange.Redis;

IDatabase db = mux.GetDatabase();

long visitas = await db.StringIncrementAsync("visitas:home");          // INCR
long stock = await db.StringDecrementAsync("stock:producto:42", 3);    // DECRBY
double temp = await db.StringIncrementAsync("temperatura:sensor:7", 0.5); // INCRBYFLOAT

// Contador diario con TTL (Redis < 8.8): dos comandos; el TTL se fija una sola vez
string dayKey = $"visitas:{DateTime.UtcNow:yyyy-MM-dd}";
long hoy = await db.StringIncrementAsync(dayKey);
await db.KeyExpireAsync(dayKey, TimeSpan.FromDays(2), ExpireWhen.HasNoExpiry);

// Rate limit atómico con INCREX (Redis ≥ 8.8), con el comando genérico
static async Task<bool> TryConsumeAsync(IDatabase db, string user, int limitPerMinute)
{
    string key = $"rate:user:{user}:{DateTime.UtcNow:yyyyMMddHHmm}";
    RedisResult result = await db.ExecuteAsync("INCREX", key,
        "BYINT", 1, "UBOUND", limitPerMinute, "EX", 60, "ENX");
    RedisResult[] parts = (RedisResult[])result!;
    long applied = (long)parts[1];
    return applied > 0;     // 0 = límite alcanzado
}
```

> [!info] 🏷️ API tipada
> Las versiones recientes de StackExchange.Redis 3.x incluyen una sobrecarga de `StringIncrementAsync` con `Expiration` y límites que se traduce a `INCREX`. El comando genérico funciona en cualquier versión del cliente.

## 🏗️ Casos de uso en arquitectura

| Caso | Diseño |
|---|---|
| Visitas por página y día | `INCR page:{id}:views:{fecha}` + TTL |
| Cuotas de API por cliente y mes | `INCRBY quota:{cliente}:{yyyyMM}` comparado con el plan |
| IDs secuenciales legibles | `INCR seq:{entidad}` (ojo: Redis debe ser durable para no repetir IDs tras un reinicio) |
| Stock reservado | `DECRBY` + comprobación; con límite inferior, Lua o `INCREX … LBOUND 0` |
| Rate limiting de ventana fija | `INCREX` con `UBOUND` y `ENX` ([[Rate limiting con Redis]]) |

### Cuándo NO usarlo
- IDs que deben ser únicos aunque Redis pierda datos: usa GUID/ULID o la secuencia de la base de datos.
- Contar elementos **únicos** (visitantes distintos): un contador cuenta repeticiones; usa Sets o [[Redis HyperLogLog]].

### Consideraciones de arquitectura

| Dimensión | Qué pensar |
|---|---|
| Durabilidad | Un contador de negocio (cuotas facturables) necesita persistencia y réplicas, o volcados periódicos a la BD |
| Escalabilidad | Un contador global muy caliente es una *hot key*: fragmenta en N subcontadores y suma al leer |
| Consistencia | Con réplicas, leer de una réplica puede dar un valor atrasado |

## ⚠️ Errores y anti-patterns

| Error | Por qué duele | Cómo evitarlo |
|---|---|---|
| `GET` + `SET` para incrementar | Pérdida de incrementos | `INCR` |
| `INCR` + `EXPIRE` sin protección | Claves sin TTL que nunca caducan | `EXPIRE … NX`, Lua o `INCREX … ENX` |
| Dinero con `INCRBYFLOAT` | Errores de redondeo | Enteros en céntimos |
| IDs de negocio con `INCR` en un Redis sin persistencia | Duplicados tras reinicio | Persistencia o IDs de la BD |
| Un único contador global para millones de escrituras | *Hot key* | Contadores fragmentados |

## ✅ Buenas prácticas

1. Usa el valor devuelto para decidir, sin lecturas extra.
2. Una clave por ventana de tiempo, con TTL.
3. `INCREX` en Redis 8.8+; `EXPIRE … NX` o Lua en versiones anteriores.
4. Enteros para importes.
5. Fragmenta los contadores muy calientes.

## 🎤 Preguntas de entrevista

> [!question]- 1. ¿Por qué `INCR` es seguro con muchos clientes concurrentes?
> Porque es un único comando y Redis ejecuta los comandos uno a uno en un hilo: la lectura, la suma y la escritura no pueden intercalarse con las de otro cliente.

> [!question]- 2. ¿Qué problema tiene `INCR` seguido de `EXPIRE` y cómo lo resuelves?
> No son atómicos juntos: si el cliente cae entre ambos, la clave queda sin TTL y el contador no se reinicia nunca. Se resuelve con `EXPIRE … NX` en cada llamada, con un script Lua o `MULTI`/`EXEC`, o en Redis 8.8+ con `INCREX … EX n ENX`.

> [!question]- 3. Un contador recibe 200 000 incrementos por segundo y es el cuello de botella. ¿Qué haces?
> Lo fragmento en N claves (`counter:{0..N-1}`), cada incremento va a una al azar o por hash, y al leer sumo las N con `MGET`. Si basta una aproximación o un valor algo retrasado, también puedo acumular en memoria local y volcar cada segundo.

## 🧪 Ejercicios

### 🟢 Básicos
1. Crea un contador diario de visitas con TTL de 2 días.
2. Provoca el error de `INCR` sobre un valor no numérico.

### 🟡 Intermedios
3. Lanza 10 000 incrementos concurrentes desde .NET (`Task.WhenAll`) y comprueba que el resultado es exactamente 10 000. Repite con `GET`+`SET` y compara.
4. Implementa `TryConsumeAsync` con `INCREX` y, como alternativa, con `INCR` + `EXPIRE NX`.

### 🔴 Reto Senior
5. Diseña la gestión del stock de una venta flash: 1000 unidades y 50 000 compradores simultáneos. Debe ser imposible vender más de 1000.

> [!success]- Soluciones
> **3.** Con `INCR`, 10 000 exactos; con `GET`+`SET` concurrentes, bastante menos (incrementos perdidos).
> **5.** `INCREX stock:vendidos BYINT 1 UBOUND 1000` (≥8.8): si el incremento aplicado es 0, se agotó. En versiones anteriores, un script Lua que lea, compare y decremente atómicamente. Después, la reserva se confirma de forma idempotente en la BD; si el pago falla, se devuelve la unidad.

### ✔️ Checklist de dominio
- [ ] Uso `INCR`/`INCRBY`/`DECRBY`/`INCRBYFLOAT` y sé sus errores.
- [ ] Explico el problema de `INCR` + `EXPIRE` y tres soluciones.
- [ ] Sé usar `INCREX` y en qué versión existe.
- [ ] Sé cuándo un contador no es la herramienta adecuada.

## 📝 Resumen

- Los incrementos son atómicos y devuelven el nuevo valor.
- Contadores por ventana = clave por periodo + TTL.
- `INCR` + `EXPIRE` no es atómico: `EXPIRE NX`, Lua o `INCREX`.
- `INCREX` (8.8) añade límites (`UBOUND`/`LBOUND`, `SATURATE`) y TTL (`ENX`).
- Enteros para dinero; contadores fragmentados si están muy calientes.

## 🔗 Referencias

- [INCR](https://redis.io/docs/latest/commands/incr/)
- [INCREX](https://redis.io/docs/latest/commands/increx/)
- [INCRBYFLOAT](https://redis.io/docs/latest/commands/incrbyfloat/)

## 🔄 Notas relacionadas

- ⬅️ [[Redis Strings]] · ➡️ [[Expiración y TTL]]
- Relacionadas: [[Rate limiting con Redis]] · [[Hot keys]] · [[Redis HyperLogLog]] · [[Lua scripting en Redis]]
