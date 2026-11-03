---
title: redis-cli y Redis Insight
aliases:
  - redis-cli
  - Redis Insight
  - RedisInsight
  - Herramientas de Redis
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
order: 1
mastery: 0
created: 2026-11-03
last_review:
next_review:
---

# redis-cli y Redis Insight

> [!abstract] En 30 segundos
> - **Qué es:** `redis-cli` es el cliente de línea de comandos oficial; Redis Insight, la interfaz gráfica oficial.
> - **Para qué:** explorar datos, ejecutar comandos, diagnosticar (latencia, big keys, estadísticas) y cargar datos de forma masiva.
> - **Recuerda:** `redis-cli` tiene modos de diagnóstico muy potentes (`--scan`, `--bigkeys`, `--stat`, `--latency`). Nunca uses `KEYS` ni `MONITOR` en producción: usa `--scan` y `SLOWLOG`.

> [!info] Lección 1.1 · Nivel 1 — Redis básico y comandos fundamentales
> ⬅️ [[Laboratorio 00 - Entorno Redis y .NET]] · ➡️ [[Claves y keyspace en Redis]] · 🗺️ [[01 - Redis - Roadmap#Nivel 1 — Redis básico y comandos fundamentales|Roadmap]] · 📊 [[03 - Redis - Progreso|Progreso]]
> 🏷️ Core · `--user`/`--pass` ≥6.0 · `--memkeys` ≥6.0 · `--latency-percentiles` ≥8.10.

## 📌 Definición

- **`redis-cli`** viene con Redis (también en la imagen de Docker). Funciona en modo **interactivo** (REPL con autocompletado y ayuda), en modo **comando** (`redis-cli GET clave`, ideal para scripts) y en **modos especiales** de diagnóstico y carga.
- **Redis Insight** es una aplicación gratuita de Redis Inc. (escritorio o contenedor) para navegar por las claves, ejecutar comandos en un *Workbench*, analizar la memoria, ver el slow log y perfilar comandos.

## 🎯 ¿Qué problema resuelve?

En un incidente necesitas respuestas en minutos: ¿cuál es la clave más grande?, ¿cuántas operaciones por segundo hay?, ¿cuál es la latencia ahora mismo? `redis-cli` las da sin instalar nada más. En desarrollo, Redis Insight acelera la exploración de los datos y el modelado.

## 🧠 Conceptos fundamentales

| Modo de `redis-cli` | Para qué |
|---|---|
| Interactivo (`redis-cli`) | Explorar, probar comandos; `HELP SET`, `HELP @hash` |
| Comando (`redis-cli INCR x`) | Scripts y automatización |
| `-r N -i S` | Repetir un comando N veces cada S segundos |
| `--scan --pattern 'app:*'` | Listar claves con `SCAN`, sin bloquear |
| `--bigkeys` / `--memkeys` | Claves más grandes por número de elementos / por memoria |
| `--hotkeys` | Claves más accedidas (requiere política de desalojo LFU) |
| `--stat` | Estadísticas en vivo: claves, memoria, clientes, peticiones |
| `--latency` / `--latency-history` / `--latency-dist` | Latencia de ida y vuelta |
| `--intrinsic-latency N` | Latencia del sistema (en el servidor) |
| `--pipe` | Carga masiva leyendo protocolo RESP de la entrada estándar |
| `--eval fichero.lua claves , args` | Ejecutar scripts Lua desde un fichero |
| `--cluster …` | Crear, comprobar y reequilibrar un Redis Cluster |
| `-3` | Usar RESP3 (los hashes se muestran como mapas) |

**Conexión:** `-h host -p puerto`, `--user` y `--pass` (o `--askpass` para no dejarla en el historial), `--tls`, `-n db`, o una URI `-u redis://usuario:clave@host:6379/0`.

### Analogía
`redis-cli` es el **multímetro** del electricista: pequeño, siempre en la caja de herramientas y suficiente para el 90 % de los diagnósticos. Redis Insight es el **osciloscopio**: más visual y potente, pero no lo llevas a cualquier sitio.
**Dónde se rompe:** el multímetro no altera el circuito; algunas funciones de las herramientas (`MONITOR`, el *profiler* de Insight) sí cargan el servidor.

## ⚙️ Funcionamiento interno

- `redis-cli` es un cliente RESP normal: lo que ves es lo que viaja por la red.
- `--scan` usa `SCAN` con cursor, así que no bloquea.
- `--bigkeys` recorre el keyspace con `SCAN` y consulta el tamaño de cada clave: es seguro pero genera carga; en producción, mejor en una réplica.
- El *Profiler* de Redis Insight se basa en `MONITOR`, que hace que el servidor copie cada comando a ese cliente: en producción puede reducir el throughput de forma drástica ([[MONITOR y sus riesgos]]).

## 💻 Ejemplos prácticos

```bash
# Ayuda integrada
redis-cli HELP SET
redis-cli HELP @sorted-set

# Ejecutar un comando y usar el resultado en un script
VISITAS=$(redis-cli GET visitas:home)

# Listar claves de forma segura
redis-cli --scan --pattern 'app:session:*' | head -20

# Contar claves que encajan con un patrón (sin KEYS)
redis-cli --scan --pattern 'app:session:*' | wc -l

# Estadísticas en vivo cada segundo
redis-cli --stat

# Claves más grandes
redis-cli --bigkeys
redis-cli --memkeys

# Repetir un comando: 5 veces, una por segundo
redis-cli -r 5 -i 1 INFO clients | grep connected_clients
```

**Carga masiva con `--pipe`** (mucho más rápida que un comando por llamada):

```bash
# genera 100 000 comandos SET en formato de texto y los envía en modo pipe
for i in $(seq 1 100000); do echo "SET producto:$i valor$i"; done | redis-cli --pipe
```

`--pipe` acepta comandos en línea (*inline*), aunque para datos binarios conviene generar RESP directamente.

**Redis Insight** (contenedor del [[Laboratorio 00 - Entorno Redis y .NET]], `http://localhost:5540`):
1. *Browser*: navega por las claves con filtros por patrón y tipo, edita valores y TTL.
2. *Workbench*: ejecuta comandos con resultados formateados y guarda scripts.
3. *Analysis tools*: análisis de memoria por prefijos y tipos.
4. *Slow Log*: lectura de `SLOWLOG` con interfaz.
5. *Profiler*: solo en desarrollo (usa `MONITOR`).

## 🔧 Implementación en .NET

Lo que haces con `--scan` y `--stat`, desde código (por ejemplo, para un endpoint de administración interno):

```csharp
using StackExchange.Redis;

IServer server = mux.GetServer(mux.GetEndPoints()[0]);

// Equivalente a: redis-cli --scan --pattern 'app:session:*'  (usa SCAN por debajo)
int total = 0;
await foreach (RedisKey key in server.KeysAsync(pattern: "app:session:*", pageSize: 1000))
{
    total++;
}
Console.WriteLine($"Sesiones: {total}");

// Una foto similar a --stat
long dbSize = await server.DatabaseSizeAsync();
var stats = (await server.InfoAsync("stats")).SelectMany(g => g).ToDictionary(kv => kv.Key, kv => kv.Value);
Console.WriteLine($"Claves: {dbSize} · ops/s: {stats["instantaneous_ops_per_sec"]}");
```

> [!warning] `IServer.KeysAsync` usa `SCAN`, pero sigue recorriendo todo el keyspace
> No bloquea al servidor, pero en una base con 50 millones de claves tarda y genera carga. Úsalo para tareas de administración, nunca en el camino de una petición.

## 🏗️ Casos de uso en arquitectura

| Situación | Herramienta |
|---|---|
| Incidente de latencia | `redis-cli --latency`, `SLOWLOG GET`, `INFO commandstats` |
| Memoria creciendo | `--memkeys`, `MEMORY STATS`, análisis de Redis Insight |
| Migración de datos | `--pipe`, `DUMP`/`RESTORE`, `MIGRATE` |
| Exploración del modelo en desarrollo | Redis Insight |
| Operación de un Cluster | `redis-cli --cluster check/reshard/rebalance` |

### Consideraciones de arquitectura
- **Seguridad:** las herramientas se conectan con un usuario ACL; usa usuarios de solo lectura para exploración (`+@read`).
- **Producción:** diagnósticos pesados (`--bigkeys`, análisis de memoria) mejor contra una réplica.

## ⚠️ Errores y anti-patterns

| Error | Por qué duele | Cómo evitarlo |
|---|---|---|
| `redis-cli KEYS '*'` en producción | Bloquea el servidor | `--scan --pattern` |
| Usar el *Profiler* o `MONITOR` en producción | Reduce mucho el throughput y expone datos | `SLOWLOG`, `latencystats`, métricas |
| `-a contraseña` en la línea de comandos | Queda en el historial y en la lista de procesos | `--askpass` o variable `REDISCLI_AUTH` |
| Conectar como administrador para explorar | Un error borra datos | Usuario ACL de solo lectura |

## ✅ Buenas prácticas

1. Ten `redis-cli` a mano en el mismo host o contenedor que Redis.
2. Explora con `--scan`, nunca con `KEYS`.
3. Diagnósticos pesados, en réplicas.
4. Usuarios ACL específicos para herramientas.
5. Redis Insight para desarrollo y análisis; en producción, con cuidado y sin profiler.

## 🎤 Preguntas de entrevista

> [!question]- 1. ¿Cómo encontrarías la clave que más memoria ocupa sin bloquear Redis?
> `redis-cli --memkeys` (o `--bigkeys` por número de elementos), que recorre el keyspace con `SCAN` y mide cada clave; mejor ejecutarlo contra una réplica. Para una clave concreta, `MEMORY USAGE`.

> [!question]- 2. ¿Cuál es la forma más rápida de cargar millones de claves?
> El modo `--pipe` de `redis-cli`, que envía los comandos en un flujo continuo sin esperar cada respuesta y al final informa de errores y respuestas. Desde código, pipelining o comandos variádicos (`MSET`, `HSET` con muchos campos).

> [!question]- 3. ¿Qué riesgo tiene el profiler de Redis Insight?
> Que usa `MONITOR`: el servidor envía una copia de cada comando a ese cliente, lo que puede reducir el throughput a menos de la mitad y expone todos los datos que pasan.

## 🧪 Ejercicios

### 🟢 Básicos
1. Usa `HELP @string` y `HELP @hash` y apunta tres comandos que no conocías.
2. Carga 100 000 claves con `--pipe` y cuéntalas con `--scan … | wc -l` y con `DBSIZE`.

### 🟡 Intermedios
3. Ejecuta `--bigkeys` y `--memkeys` tras crear un hash con 10 000 campos. ¿Coinciden los resultados? ¿Por qué no siempre?
4. Conecta Redis Insight y localiza las claves de un prefijo con su TTL.

### 🔴 Reto Senior
5. Escribe un script de diagnóstico de 10 líneas que, ante un incidente, recoja en un fichero: versión, `INFO` completo, `SLOWLOG GET 50`, `LATENCY LATEST`, `CLIENT LIST` y las 10 claves más grandes.

> [!success]- Soluciones
> **3.** `--bigkeys` ordena por número de elementos; `--memkeys`, por bytes. Una lista con pocos elementos muy grandes puede ser la mayor en memoria y no en elementos.
> **5.** `redis-cli INFO server`, `INFO`, `SLOWLOG GET 50`, `LATENCY LATEST`, `CLIENT LIST` y `--memkeys`, cada uno redirigido con `>>` a un fichero con marca de tiempo; ejecútalo contra la réplica si el primario está saturado.

### ✔️ Checklist de dominio
- [ ] Uso `redis-cli` en modo interactivo, de comando y de diagnóstico.
- [ ] Sé qué modos son seguros en producción y cuáles no.
- [ ] Exploro datos con Redis Insight.

## 📝 Resumen

- `redis-cli`: REPL, comandos sueltos, diagnóstico (`--scan`, `--bigkeys`, `--memkeys`, `--stat`, `--latency`) y carga masiva (`--pipe`).
- Redis Insight: exploración visual, Workbench, memoria y slow log.
- Nunca `KEYS` ni `MONITOR` en producción.
- Credenciales con `--askpass` o `REDISCLI_AUTH`, y usuarios ACL específicos.

## 🔗 Referencias

- [Redis CLI](https://redis.io/docs/latest/develop/tools/cli/)
- [Redis Insight](https://redis.io/docs/latest/develop/tools/insight/)
- [SCAN](https://redis.io/docs/latest/commands/scan/)

## 🔄 Notas relacionadas

- ⬅️ [[Laboratorio 00 - Entorno Redis y .NET]] · ➡️ [[Claves y keyspace en Redis]]
- Más adelante: [[MONITOR y sus riesgos]] · [[Big keys]] · [[Hot keys]] · [[Diagnóstico de Redis]]
