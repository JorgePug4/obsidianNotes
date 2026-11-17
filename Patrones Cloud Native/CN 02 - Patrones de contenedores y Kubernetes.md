---
tags: [cloud-native, patrones, kubernetes, contenedores, sidecar]
aliases: [Sidecar, Ambassador, Adapter, Operator pattern, Patrones de Kubernetes]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 02 - Patrones de contenedores y Kubernetes

> [!info] Objetivo del capítulo
> Los patrones que usan el **contenedor y el Pod** como bloques de construcción. Fuentes: Brendan Burns, *Designing Distributed Systems*; Ibryam y Huß, *Kubernetes Patterns* (2.ª ed., 2023); Microsoft *Cloud Design Patterns* (Sidecar, Ambassador, Leader Election).
> Requisito: saber qué es un Pod → [[03 - Objetos y workloads]].

## Contenido
- [[#Clasificación]]
- [[#1. Sidecar]] · [[#2. Ambassador]] · [[#3. Adapter]] · [[#4. Init Container]]
- [[#5. Health Probe]] · [[#6. Predictable Demands]] · [[#7. Declarative Deployment]] · [[#8. Managed Lifecycle]]
- [[#9. Controller y Operator]] · [[#10. Leader Election (Singleton Service)]]
- [[#11. Patrones multi-nodo (Burns)]]
- [[#Comparación Sidecar vs Ambassador vs Adapter]]

---

## Clasificación

*Kubernetes Patterns* agrupa los patrones así (los más importantes en **negrita**):

| Categoría | Patrones | Idea |
|---|---|---|
| **Fundacionales** | **Health Probe**, **Predictable Demands**, **Declarative Deployment**, **Managed Lifecycle**, Automated Placement | Requisitos mínimos para que un contenedor sea un "buen ciudadano" del clúster |
| **De comportamiento** | Batch Job, Periodic Job, Daemon Service, **Singleton Service**, Stateless Service, Stateful Service, Service Discovery, Self Awareness | Tipos de carga y cómo los gestiona la plataforma |
| **Estructurales** | **Init Container**, **Sidecar**, **Adapter**, **Ambassador** | Cómo componer varios contenedores en un Pod |
| **De configuración** | EnvVar Configuration, Configuration Resource, Immutable Configuration, Configuration Template | Cómo inyectar configuración (ver [[CN 08 - Despliegue, configuración y entrega]]) |
| **De seguridad** | Process Containment, Network Segmentation, Secure Configuration, Access Control | Ver [[CN 09 - Seguridad]] |
| **Avanzados** | **Controller**, **Operator**, Elastic Scale, Image Builder | Extender Kubernetes |

> [!note] Por qué funcionan los patrones estructurales
> Los contenedores de un mismo Pod **comparten red** (`localhost`), pueden **compartir volúmenes** y se programan **en el mismo nodo**. Eso permite separar responsabilidades en procesos distintos sin pagar latencia de red.

---

## 1. Sidecar

**Problema.** Varias aplicaciones (en distintos lenguajes) necesitan la misma funcionalidad transversal: enviar logs, renovar certificados, sincronizar configuración, proxy de red. Reimplementarla en cada lenguaje es caro e inconsistente; meterla en el mismo proceso acopla ciclos de vida y fallos.

**Solución.** Desplegar esa funcionalidad como un **contenedor auxiliar dentro del mismo Pod** que extiende o mejora al contenedor principal sin modificarlo. Como un sidecar de moto: va pegado, comparte el viaje, pero es una pieza separada.

**Cómo funciona.**

```
┌──────────────── Pod ────────────────┐
│  ┌──────────────┐   ┌────────────┐  │
│  │ App (.NET)   │   │  Sidecar   │  │
│  │ escribe logs ├──►│ (Fluent    │──┼──► Elasticsearch / Log Analytics
│  │ en /var/log  │   │  Bit)      │  │
│  └──────────────┘   └────────────┘  │
│        volumen compartido emptyDir  │
└─────────────────────────────────────┘
```

**Ejemplo (Kubernetes ≥ 1.29, *native sidecar*).** Desde Kubernetes 1.29 (beta, activado por defecto; GA en 1.33) un sidecar se declara como `initContainer` con `restartPolicy: Always`: arranca **antes** que la app y termina **después**, lo que resolvía un problema clásico de los sidecars "normales" (que no terminaban y dejaban los `Job` colgados).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pedidos
spec:
  initContainers:
    - name: log-shipper
      image: fluent/fluent-bit:3.0
      restartPolicy: Always          # ← lo convierte en sidecar nativo
      volumeMounts:
        - { name: logs, mountPath: /var/log/app }
  containers:
    - name: app
      image: miregistro.azurecr.io/pedidos:1.4.2
      volumeMounts:
        - { name: logs, mountPath: /var/log/app }
  volumes:
    - name: logs
      emptyDir: {}
```

**Usos típicos reales:** proxy de service mesh (Envoy en Istio, `linkerd-proxy`), Dapr (`daprd`), agentes de logs, recarga de certificados, sincronizar ficheros desde Git (`git-sync`).

**Cuándo usarlo.** La funcionalidad es transversal, debe compartir ciclo de vida con la app, y quieres reutilizarla entre lenguajes o equipos.
**Cuándo no.** Si la comunicación entre app y sidecar necesita latencia mínima en un camino muy caliente; si una librería en el propio lenguaje resuelve lo mismo con menos coste; si el sidecar debería escalar de forma independiente (entonces es un servicio aparte).

**Trade-offs.**
- ✅ Separación de responsabilidades, independencia de lenguaje, reutilización.
- ❌ Más recursos por Pod (un sidecar × 500 Pods = 500 copias), más latencia (un salto `localhost`), más complejidad de depuración.
- Tendencia: por ese coste, han surgido alternativas **sin sidecar** (Istio *ambient mode*, Cilium con eBPF) que mueven la funcionalidad al nodo.

**Relacionados.** Ambassador y Adapter son especializaciones del Sidecar. Service Mesh ([[CN 03 - Comunicación, enrutamiento y APIs]]).

---

## 2. Ambassador

**Problema.** La aplicación necesita hablar con un servicio externo complejo (un clúster Redis particionado, una API con autenticación especial, una BD con réplicas de lectura) y no quieres meter esa lógica de conexión en cada app.

**Solución.** Un **sidecar que actúa como proxy de salida**: la app se conecta a `localhost` y el ambassador se encarga de la complejidad (enrutamiento, reintentos, TLS, sharding, descubrimiento).

```
App ──localhost:6379──► Ambassador ──► redis-shard-1
                                    ├─► redis-shard-2
                                    └─► redis-shard-3
```

**Ejemplos.** `twemproxy` o `envoy` delante de Redis; proxy de Cloud SQL (*Cloud SQL Auth Proxy*) en GCP; un Envoy que añade mTLS y reintentos a las llamadas salientes.

**Cuándo usarlo.** Para **delegar la conectividad** (retries, circuit breaking, TLS, monitorización de llamadas salientes) a una pieza común, sobre todo con apps *legacy* que no puedes modificar.
**Cuándo no.** Si la latencia añadida es crítica, o si una librería cliente en tu lenguaje ya lo resuelve bien.

**Trade-offs.** Simplifica la app; añade un salto de red y otro componente que operar.

---

## 3. Adapter

**Problema.** Tienes aplicaciones heterogéneas que exponen información (métricas, logs, health) en formatos distintos, y la plataforma espera **un formato único**.

**Solución.** Un **sidecar que traduce la interfaz de la app hacia fuera**, normalizándola. Es el inverso del Ambassador: el Ambassador simplifica el mundo para la app; el Adapter simplifica la app para el mundo.

```
App legacy (métricas en formato propio) ──► Adapter (exporter) ──► /metrics formato Prometheus
```

**Ejemplos.** Los *exporters* de Prometheus como sidecar (`redis_exporter`, `mysqld_exporter`, JMX exporter); un adapter que convierte logs de texto plano a JSON.

**Cuándo usarlo.** No puedes o no quieres modificar la app y necesitas que encaje con el estándar de la plataforma.

---

## 4. Init Container

**Problema.** Antes de arrancar la app hay que hacer tareas previas: esperar a que una dependencia esté lista, descargar configuración, ajustar permisos de un volumen, ejecutar una migración controlada.

**Solución.** Contenedores que se ejecutan **en orden, uno tras otro, hasta completarse con éxito**, antes de que arranquen los contenedores principales. Si uno falla, el Pod no arranca (se reintenta según `restartPolicy`).

```yaml
spec:
  initContainers:
    - name: esperar-bd
      image: busybox:1.36
      command: ['sh', '-c', 'until nc -z sqlserver 1433; do echo esperando; sleep 2; done']
  containers:
    - name: app
      image: miregistro.azurecr.io/pedidos:1.4.2
```

**Trade-offs.** Separa la inicialización de la imagen de la app (que no necesita herramientas como `nc` o `git`). Pero ralentiza el arranque, y **esperar a dependencias en un init container no sustituye a la resiliencia en tiempo de ejecución**: la BD puede caer cinco minutos después.

> [!warning] Migraciones en init containers
> Si 5 réplicas arrancan a la vez, ejecutan 5 migraciones en paralelo. Usa una herramienta que bloquee (EF Core no lo hacía hasta .NET 9, que añadió un bloqueo de migración para algunos proveedores — verifica tu versión y proveedor), o mejor un `Job` separado.

---

## 5. Health Probe

**Problema.** La plataforma necesita saber si un contenedor está **vivo** (si no, reiniciarlo) y si está **listo** para recibir tráfico (si no, sacarlo del balanceador). Que el proceso exista no significa que funcione.

**Solución.** La app expone endpoints de salud y Kubernetes los consulta con tres tipos de *probe*:

| Probe | Pregunta | Si falla | Debe comprobar |
|---|---|---|---|
| **startupProbe** | ¿Ya terminó de arrancar? | Sigue esperando; tras el umbral, reinicia | Inicialización lenta (cargar caché, JIT) |
| **livenessProbe** | ¿Está vivo o bloqueado? | **Reinicia** el contenedor | Sólo el propio proceso (deadlock, estado corrupto) |
| **readinessProbe** | ¿Puede atender tráfico ahora? | Lo **saca del Service** (no reinicia) | El proceso + dependencias imprescindibles |

```csharp
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy(), tags: ["live"])
    .AddSqlServer(connStr, tags: ["ready"]);   // paquete AspNetCore.HealthChecks.SqlServer

app.MapHealthChecks("/health/live",  new() { Predicate = r => r.Tags.Contains("live") });
app.MapHealthChecks("/health/ready", new() { Predicate = r => r.Tags.Contains("ready") });
```

```yaml
livenessProbe:
  httpGet: { path: /health/live, port: 8080 }
  periodSeconds: 10
  failureThreshold: 3
readinessProbe:
  httpGet: { path: /health/ready, port: 8080 }
  periodSeconds: 5
startupProbe:
  httpGet: { path: /health/live, port: 8080 }
  failureThreshold: 30
  periodSeconds: 2      # hasta 60 s para arrancar
```

> [!danger] El error más grave con probes
> **No compruebes dependencias externas en la liveness probe.** Si la BD cae y la liveness la comprueba, Kubernetes reiniciará **todos** los Pods en bucle, y al volver la BD recibirá una avalancha de conexiones. La liveness responde "¿está mi proceso roto?", no "¿está el mundo bien?".

Más detalle: [[07 - Salud, fiabilidad y despliegues]]. Patrón equivalente fuera de K8s: *Health Endpoint Monitoring* ([[CN 10 - Observabilidad]]).

---

## 6. Predictable Demands

**Problema.** Si el scheduler no sabe cuánta CPU y memoria necesita un contenedor, lo coloca mal: nodos sobrecargados, Pods expulsados (*evicted*), OOMKilled.

**Solución.** Declarar **requests** (lo que se reserva para planificar) y **limits** (el máximo permitido), además de las dependencias de runtime (volúmenes, ConfigMaps, puertos).

```yaml
resources:
  requests: { cpu: "250m", memory: "256Mi" }
  limits:   { memory: "512Mi" }     # muchos equipos omiten el límite de CPU para evitar throttling
```

| Clase QoS | Condición | Prioridad al expulsar |
|---|---|---|
| Guaranteed | requests = limits en CPU y memoria | La última en ser expulsada |
| Burstable | requests < limits | Intermedia |
| BestEffort | Sin requests ni limits | La primera |

> [!note] .NET y los límites
> El runtime de .NET (desde .NET Core 3.0) lee los límites del cgroup para dimensionar el GC y el pool de hilos. Sin límite de memoria, el GC asume la memoria del nodo. Detalle: [[06 - Scheduling y recursos]].

---

## 7. Declarative Deployment

**Problema.** Actualizar N réplicas a mano es lento y propenso a errores.
**Solución.** Declarar el **estado deseado** (`Deployment` con imagen v2) y dejar que un controlador lo alcance con una estrategia: **RollingUpdate** (por defecto) o **Recreate**. Blue-Green y Canary se construyen encima. Ver [[CN 08 - Despliegue, configuración y entrega]].

## 8. Managed Lifecycle

**Problema.** La plataforma va a parar tu contenedor (despliegue, escalado hacia abajo, drenaje de nodo). Si no reaccionas, cortas peticiones.
**Solución.** Reaccionar a los eventos de ciclo de vida:
- **`SIGTERM`** → dejar de aceptar trabajo nuevo, terminar el actual y salir antes de `terminationGracePeriodSeconds` (30 s por defecto); después llega `SIGKILL`.
- **Hook `preStop`** → por ejemplo, un `sleep 5` para dar tiempo a que el endpoint se retire de los balanceadores antes de que el proceso empiece a cerrarse (evita errores 502 durante los despliegues).
- **Hook `postStart`** → tareas justo tras arrancar (poco usado).

ASP.NET Core gestiona `SIGTERM` por defecto a través de `IHostApplicationLifetime`; ajusta `HostOptions.ShutdownTimeout` para que sea menor que el *grace period*.

---

## 9. Controller y Operator

**Problema.** Operar software con estado complejo (una base de datos con réplicas, backups, failover, actualizaciones) requiere conocimiento experto y pasos manuales.

**Solución.**
- **Controller:** un proceso que ejecuta un **bucle de reconciliación** (*reconcile loop*): observa el estado actual, lo compara con el deseado y actúa para acercarlos. Así funciona todo Kubernetes (Deployment controller, ReplicaSet controller…).
- **Operator:** un controller **más** una o varias **Custom Resource Definitions (CRD)** que codifican el conocimiento de un operador humano para una aplicación concreta. Término introducido por CoreOS en 2016.

```
           ┌────────── bucle de reconciliación ──────────┐
           ▼                                             │
  Observar (watch API) ─► Comparar deseado vs actual ─► Actuar (crear/borrar/actualizar)
```

```yaml
# Lo que escribe el usuario: una CRD de un operador de PostgreSQL (ejemplo: CloudNativePG)
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pedidos-db
spec:
  instances: 3              # el operador crea primario + 2 réplicas y gestiona el failover
  storage:
    size: 20Gi
  backup:
    barmanObjectStore:
      destinationPath: s3://backups/pedidos
```

**Ejemplos reales.** CloudNativePG y Zalando Postgres Operator, Strimzi (Kafka), Prometheus Operator, cert-manager, Argo CD (sus `Application` son CRDs).

**Cuándo usarlo.** Para software con estado que requiere operaciones complejas y repetibles. **Cuándo no.** Para una app *stateless*: un Deployment + Helm es suficiente. Escribir tu propio operador (con Operator SDK, Kubebuilder o, en .NET, KubeOps) es un proyecto serio que tendrás que mantener.

**Trade-offs.** Automatiza conocimiento experto; pero un operador con bugs puede causar daños automáticamente y a gran escala, y añade una dependencia crítica. Ver [[Custom Controller]] y [[Custom Resource Definition]].

---

## 10. Leader Election (Singleton Service)

**Problema.** Tienes varias réplicas por alta disponibilidad, pero una tarea debe ejecutarla **sólo una** a la vez (un scheduler de tareas, un consumidor que debe mantener orden, una tarea de mantenimiento).

**Solución.** Las instancias **compiten por un bloqueo distribuido con expiración** (*lease*); quien lo obtiene es el líder y lo renueva periódicamente. Si el líder muere, el lease expira y otra instancia lo toma.

**Implementaciones:**
- Kubernetes: objeto `Lease` (`coordination.k8s.io`). Así eligen líder `kube-controller-manager` y `kube-scheduler`.
- Azure: *lease* sobre un blob de Azure Storage (es el ejemplo de Microsoft en el patrón *Leader Election*).
- Redis: `SET clave valor NX PX 30000` (con cuidado: Redlock tiene críticas conocidas de Martin Kleppmann sobre su seguridad ante pausas y relojes).
- etcd, ZooKeeper, Consul.

**Alternativa más simple:** si sólo necesitas una instancia, `replicas: 1` con un `Deployment` (Kubernetes la recrea si muere) o un `CronJob`. Aceptas unos segundos sin servicio a cambio de no implementar elección de líder.

> [!warning] Split brain
> Un líder que sufre una pausa larga (GC, red) puede seguir creyéndose líder cuando el lease ya expiró. Para operaciones críticas, usa **fencing tokens**: un número creciente que el recurso protegido valida y que rechaza escrituras de líderes antiguos.

---

## 11. Patrones multi-nodo (Burns)

Brendan Burns describe patrones que reparten trabajo entre **varias máquinas**:

| Patrón | Idea | Ejemplo |
|---|---|---|
| **Replicated load-balanced services** | Réplicas idénticas *stateless* detrás de un balanceador | Deployment + Service de Kubernetes |
| **Sharded services** | Cada réplica atiende un subconjunto de datos/clientes, según una clave | Caché particionada; ver Sharding en [[CN 05 - Patrones de gestión de datos]] |
| **Scatter/Gather** | Enviar una petición a muchos nodos en paralelo y combinar respuestas | Búsqueda distribuida (cada nodo busca en su índice) |
| **Work Queue** | Una cola de tareas y un pool de workers | Ver Competing Consumers en [[CN 04 - Mensajería y arquitectura orientada a eventos]] |
| **Event-driven batch** | Etapas encadenadas por eventos | Ver Pipes and Filters |
| **Ownership election** | Elección de líder | Ver sección 10 |

---

## Comparación Sidecar vs Ambassador vs Adapter

| | Sidecar | Ambassador | Adapter |
|---|---|---|---|
| Dirección | Cualquiera / auxiliar | **Salida** (app → mundo) | **Entrada** (mundo → app) |
| Propósito | Extender la app | Simplificar el acceso a servicios externos | Normalizar la interfaz de la app |
| Ejemplo | Agente de logs, Dapr | Proxy hacia Redis particionado | Exporter de Prometheus |
| Pregunta mnemotécnica | "¿Qué le **añado**?" | "¿Cómo **sale**?" | "¿Cómo la **ven**?" |

## Puntos clave

- Un Pod con varios contenedores comparte red y volúmenes: eso habilita los patrones estructurales.
- **Liveness ≠ readiness**: liveness reinicia, readiness saca del balanceo. Nunca dependencias en liveness.
- Requests/limits = **Predictable Demands**; sin ellos, el scheduler adivina.
- Operator = controller + CRD + conocimiento operativo codificado.
- Leader Election sólo cuando realmente necesites un único ejecutor; cuidado con el *split brain*.

## Errores comunes

- Sidecars "clásicos" en `Job`: el Job nunca termina porque el sidecar sigue vivo (resuelto con sidecars nativos).
- Liveness que consulta la BD → reinicios en cascada.
- No manejar `SIGTERM` → errores 502/503 en cada despliegue.
- Escribir un Operator para algo que un Helm chart resuelve.
- Usar Redis `SETNX` como lock sin expiración: si el proceso muere, el lock queda para siempre.

## 🎯 Para entrevistas

- *"¿Diferencia entre Sidecar, Ambassador y Adapter?"* → tabla anterior: extender, proxy de salida, normalizar hacia fuera.
- *"¿Qué pasa si la liveness probe comprueba la BD y la BD cae?"* → reinicios en bucle de todos los Pods; la readiness es la adecuada.
- *"¿Qué es un Operator?"* → controller con CRDs que automatiza la operación de una app específica (backups, failover, upgrades).
- *"¿Cómo garantizas que un job sólo lo ejecuta una réplica?"* → Leader Election con `Lease`, o una sola réplica / CronJob con `concurrencyPolicy: Forbid`.

## Referencias

- Microsoft, *Sidecar pattern*: https://learn.microsoft.com/azure/architecture/patterns/sidecar
- Microsoft, *Ambassador pattern*: https://learn.microsoft.com/azure/architecture/patterns/ambassador
- Microsoft, *Leader Election pattern*: https://learn.microsoft.com/azure/architecture/patterns/leader-election
- Kubernetes, *Sidecar Containers*: https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/
- Kubernetes, *Configure Liveness, Readiness and Startup Probes*: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- Ibryam y Huß, *Kubernetes Patterns*, 2.ª ed. (O'Reilly, 2023).
- Brendan Burns, *Designing Distributed Systems* (O'Reilly).
- Martin Kleppmann, *How to do distributed locking* (2016): https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html

---
⬅️ [[CN 01 - Fundamentos Cloud Native]] · ➡️ [[CN 03 - Comunicación, enrutamiento y APIs]]
