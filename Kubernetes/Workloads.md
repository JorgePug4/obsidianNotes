---
tags:
  - kubernetes
  - workloads
aliases:
  - Pod
  - Pods
  - K8s Pod
  - ReplicaSet
  - ReplicaSets
  - RS
  - Replication Controller
  - Deployment
  - Deployments
  - deploy
  - StatefulSet
  - StatefulSets
  - sts
---

# Workloads

- [[#Pod]]
- [[#ReplicaSet]]
- [[#Deployment]]
- [[#StatefulSet]]

---

## Pod

### Qué es
Un **Pod** es la unidad mínima y más básica de ejecución y despliegue dentro de Kubernetes. Representa una envoltura o *wrapper* abstracto sobre uno o más contenedores que comparten la misma dirección IP, el mismo espacio de nombres de red, el almacenamiento y la configuración de ejecución.

### Para qué sirve
Sirve para encapsular y ejecutar aplicaciones o microservicios en los nodos de trabajo (*Worker Nodes*). Aunque el patrón más común es ejecutar un solo contenedor por Pod, se pueden empaquetar múltiples contenedores dentro del mismo Pod en casos específicos (como contenedores auxiliares *sidecar* o *init containers*) para que compartan comunicación mediante `localhost` y volúmenes de datos locales.

> [!note] Corrección (auditoría 2026-10): qué se comparte realmente
> Los contenedores de un Pod comparten **red** (misma IP y puertos: se hablan por `localhost`, y dos contenedores no pueden escuchar el mismo puerto) e **IPC**, y siempre se programan **en el mismo nodo**. **No comparten sistema de archivos**: cada uno tiene el suyo, y solo comparten los [[Volume|volúmenes]] que se declaren en el Pod y se monten explícitamente en cada contenedor. Los **sidecars nativos** (`initContainers` con `restartPolicy: Always`) son **GA desde 1.33**. Ciclo de vida completo, init containers y terminación ordenada en [[03 - Objetos y workloads#Pod]].

> [!tip] Políticas de reinicio (`restartPolicy`)
> La propiedad `restartPolicy` se especifica a nivel de Pod pero actúa sobre sus contenedores. Admite tres valores: `Always` (reintenta reiniciar el contenedor continuamente si se detiene), `OnFailure` (solo reinicia si el contenedor termina con un código de error distinto de 0) y `Never` (nunca reinicia el contenedor).

> [!warning] Naturaleza efímera de los Pods
> Los Pods son recursos efímeros e intercambiables. Si un Pod muere o es eliminado, pierde su dirección IP interna y todo dato guardado dentro del sistema de archivos del contenedor que no esté respaldado en un volumen externo. En entornos de producción no se recomienda desplegar Pods de forma individual, sino gestionarlos mediante cargas de trabajo como [[Workloads#Deployment|Deployment]] o [[Workloads#StatefulSet|StatefulSet]].

### Ejemplo

**Creación imperativa mediante comandos:**
```bash
# Crear y ejecutar un Pod de Nginx
kubectl run nginx1 --image=nginx

# Crear un Pod de Apache especificando el puerto
kubectl run apache1 --image=httpd --port=80

# Listar los Pods en el namespace actual
kubectl get pods -o wide

# Inspeccionar eventos y detalles del Pod
kubectl describe pod nginx1

# Ver los logs del contenedor dentro del Pod
kubectl logs nginx1

# Ejecutar un comando interactivo dentro del Pod
kubectl exec -it nginx1 -- /bin/bash

# Eliminar un Pod
kubectl delete pod nginx1
```

**Definición declarativa en YAML (`pod.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: enginex-pod
  labels:
    app: frontend
spec:
  restartPolicy: Always
  containers:
    - name: enginex-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

### Relacionado
- [[Workloads#Deployment|Deployment]]
- [[Workloads#ReplicaSet|ReplicaSet]]
- [[Workloads#StatefulSet|StatefulSet]]
- [[Container Runtime]]
- [[kubelet]]
- [[Networking#CNI|CNI]]
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[ConfigMap]]
- [[Secret]]
- [[Volume]]

---

## ReplicaSet

### Qué es
Un **ReplicaSet** (sucesor del antiguo *ReplicationController*, ver corrección abajo) es un objeto y controlador del plano de control de Kubernetes cuyo objetivo principal es garantizar que un número exacto y determinado de réplicas de un [[Workloads#Pod|Pod]] idéntico se encuentren en estado de ejecución en todo momento.

### Para qué sirve
Proporciona escalabilidad y auto-recuperación (*auto-healing*). Monitorea de forma continua el clúster: si un Pod falla, es eliminado accidentalmente o se cae el nodo donde residía, el ReplicaSet detecta la discrepancia entre el estado deseado y el estado real y crea un nuevo Pod de forma automática para reemplazarlo. Identifica qué Pods le pertenecen utilizando etiquetas y selectores (`labels` y `selectors`).

> [!tip] Uso indirecto recomendado
> Aunque es posible definir y crear un ReplicaSet de forma independiente, en la práctica habitual de DevOps no se gestionan directamente. En su lugar, se utilizan los [[Workloads#Deployment|Deployment]], los cuales administran automáticamente el ciclo de vida de los ReplicaSets subyacentes.

> [!note] Corrección (auditoría 2026-10): ReplicaSet ≠ ReplicationController
> No es un cambio de nombre: son **dos objetos distintos** que conviven en la API. `ReplicationController` (`v1`) es el original y solo admite selectores por **igualdad** (`app=web`). `ReplicaSet` (`apps/v1`) es su sucesor y admite selectores **por conjuntos** (`matchExpressions` con `In`, `NotIn`, `Exists`). El RC está desaconsejado; y, en la práctica, tampoco se crean ReplicaSets a mano: se usan Deployments.

> [!warning] Dependencia de Labels y Selectors
> El ReplicaSet vincula los Pods mediante sus etiquetas (`metadata.labels`). Si un usuario modifica manualmente la etiqueta de un Pod en ejecución, el ReplicaSet dejará de contabilizarlo y creará una nueva réplica de inmediato para reponer la cantidad deseada.

### Ejemplo

**Comandos útiles para consultar ReplicaSets:**
```bash
# Listar los ReplicaSets activos
kubectl get rs

# Listar ReplicaSets con más detalle sobre las etiquetas y plantillas
kubectl get rs -o wide

# Ver todos los recursos relacionados
kubectl get deploy,rs,pods -l app=nginx
```

**Definición declarativa en YAML (`replicaset.yaml`):**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

### Relacionado
- [[Workloads#Pod|Pod]]
- [[Workloads#Deployment|Deployment]]
- [[Kube-controller-manager]]
- [[Networking#Kubernetes Service|Kubernetes Service]]

---

## Deployment

### Qué es
Un **Deployment** es un objeto de carga de trabajo (*workload*) de alto nivel en Kubernetes que gestiona de manera declarativa la implementación, actualización y escalado de aplicaciones sin estado (*stateless*).

### Para qué sirve
Abstrae la gestión directa de Pods y [[Workloads#ReplicaSet|ReplicaSet]]. Permite definir el estado deseado de una aplicación (número de réplicas, imagen del contenedor, variables de entorno) y se encarga de ejecutar actualizaciones graduales sin tiempo de inactividad (*Rolling Updates*), estrategias como *Canary* o *Blue-Green*, escalado manual o automático (vía HPA) y la capacidad de realizar reversiones (*rollbacks*) si una nueva versión falla.

> [!tip] Requisito de Readiness Probes en Rolling Updates
> Para garantizar que un *Rolling Update* se realice sin caídas de servicio, se deben configurar los `ReadinessProbe` en la plantilla del Pod. Esto evita que Kubernetes elimine la versión antigua antes de verificar que el nuevo contenedor está listo para recibir tráfico de red.

> [!warning] Comportamiento ante errores de imagen
> Si al realizar un despliegue la nueva imagen falla al descargarse (provocando un error `ImagePullBackOff` o `CrashLoopBackOff`), el Deployment detendrá la actualización paulatinamente tras fallar la primera réplica nueva, manteniendo las réplicas antiguas activas para proteger la disponibilidad del servicio.

> [!note] Corrección (auditoría 2026-10)
> **1. Estrategias nativas.** `spec.strategy.type` solo admite **`RollingUpdate`** (por defecto) y **`Recreate`**. *Canary* y *Blue-Green* **no son estrategias del Deployment**: se construyen encima (dos Deployments con un Service/Ingress/Gateway que reparte el tráfico) o con herramientas como **Argo Rollouts**, **Flagger** o una service mesh. Ver [[07 - Salud, fiabilidad y despliegues#Estrategias de despliegue]].
>
> **2. Qué pasa realmente cuando falla un rollout.** El Deployment no tiene lógica de "parar al primer fallo": crea Pods nuevos según `maxSurge`, y como esos Pods **nunca llegan a Ready**, `maxUnavailable` impide seguir borrando Pods antiguos. El rollout queda **atascado** (los Pods viejos siguen sirviendo). Pasado `progressDeadlineSeconds` (600 s por defecto) el Deployment se marca con la condición `Progressing=False` (`ProgressDeadlineExceeded`) y `kubectl rollout status` devuelve error. **No hay rollback automático**: hay que ejecutar `kubectl rollout undo` (o automatizarlo en el pipeline o con Argo Rollouts).
>
> **3. Versiones de imagen.** Los ejemplos usan `nginx:latest` y `nginx:1.14.2` (de 2018). Fija siempre una versión actual y concreta: con `latest`, `imagePullPolicy` pasa a ser `Always` por defecto y dos réplicas pueden acabar ejecutando versiones distintas.

### Ejemplo

**Comandos imperativos y gestión de Deployments:**
```bash
# Crear un Deployment básico de Apache
kubectl create deployment apache-deploy --image=httpd

# Listar deployments
kubectl get deploy

# Escalar el número de réplicas manualmente a 5
kubectl scale deployment apache-deploy --replicas=5

# Aplicar cambios desde un archivo manifiesto
kubectl apply -f deployment.yaml

# Inspeccionar el estado del Deployment
kubectl describe deployment apache-deploy
```

**Definición declarativa en YAML (`deployment.yaml`):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### Relacionado
- [[Workloads#Pod|Pod]]
- [[Workloads#ReplicaSet|ReplicaSet]]
- [[Workloads#StatefulSet|StatefulSet]]
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[ConfigMap]]
- [[Secret]]

---

## StatefulSet

### Qué es
Un **StatefulSet** es el objeto de carga de trabajo (*workload*) en Kubernetes diseñado específicamente para gestionar aplicaciones con estado (*stateful*), como bases de datos (MySQL, PostgreSQL, MongoDB), motores de búsqueda o sistemas de mensajería (Kafka, Redis).

### Para qué sirve
A diferencia de un [[Workloads#Deployment|Deployment]], un StatefulSet mantiene una identidad persistente y fija (*sticky identity*) para cada una de sus réplicas de Pod. Otorga a cada Pod un índice numérico secuencial ordenado (ej. `web-0`, `web-1`, `web-2`), asigna nombres de dominio DNS individuales mediante un [[Networking#Headless Service|Headless Service]] y vincula de forma persistente e independiente un volumen de datos exclusivo a cada Pod.

> [!warning] Persistencia de PVC tras la eliminación
> Al borrar o escalar hacia abajo un StatefulSet, Kubernetes **no elimina automáticamente** los [[PersistentVolumeClaim]] (PVC) ni los volúmenes físicos asociados. Esta regla de seguridad evita la pérdida accidental de datos. Para eliminar el almacenamiento se debe ejecutar el comando `kubectl delete pvc <nombre-pvc>` explícitamente.

> [!warning] Creación y borrado estrictamente secuencial
> Los Pods gestionados por un StatefulSet no se crean ni se borran en paralelo. La segunda réplica (`web-1`) únicamente comenzará a crearse cuando la primera (`web-0`) esté completamente activa y en estado *Running*. El borrado se realiza en orden inverso (de la réplica más alta a la menor).

> [!note] Corrección (auditoría 2026-10)
> - **El orden secuencial no es obligatorio.** Es el comportamiento de `podManagementPolicy: OrderedReady` (por defecto), y la réplica siguiente espera a que la anterior esté **Running y Ready** (no solo *Running*): una *readiness probe* mal configurada bloquea el escalado. Con `podManagementPolicy: Parallel` los Pods se crean y borran a la vez (la identidad estable se mantiene). Las **actualizaciones** (`updateStrategy: RollingUpdate`) van en orden inverso, de la réplica más alta a la `0`, y admiten `partition` para canaries.
> - **Los PVC sí se pueden borrar automáticamente** desde **1.32 (GA)** con `persistentVolumeClaimRetentionPolicy`:
>   ```yaml
>   spec:
>     persistentVolumeClaimRetentionPolicy:
>       whenDeleted: Retain   # o Delete: borra los PVC al eliminar el StatefulSet
>       whenScaled: Retain    # o Delete: borra los PVC de las réplicas eliminadas al escalar hacia abajo
>   ```
>   `Retain`/`Retain` sigue siendo el valor por defecto, así que la advertencia anterior es correcta **si no se configura**.
> - El campo `serviceName` debe apuntar a un [[Networking#Headless Service|Headless Service]] que **hay que crear aparte**: el StatefulSet no lo crea.

> [!tip] Resolución DNS individual de réplicas
> Gracias al enlace con un [[Networking#Headless Service|Headless Service]], cada Pod dentro de un StatefulSet obtiene un registro de dominio FQDN propio (por ejemplo, `mysql-0.mydb-headless.default.svc.cluster.local`), lo que permite que clientes o réplicas secundarias se conecten directamente a un nodo específico (como un nodo maestro de lectura/escritura).

### Ejemplo

**Comandos de gestión de StatefulSets:**
```bash
# Listar StatefulSets
kubectl get statefulset

# Aplicar archivo de configuración
kubectl apply -f statefulset.yaml

# Eliminar explícitamente un PVC persistente de un StatefulSet
kubectl delete pvc www-web-0
```

**Definición declarativa en YAML (`statefulset.yaml`):**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx-headless"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi
```

### Relacionado
- [[Workloads#Pod|Pod]]
- [[Workloads#Deployment|Deployment]]
- [[Networking#Headless Service|Headless Service]]
- [[PersistentVolume]]
- [[PersistentVolumeClaim]]
- [[StorageClass]]

> [!info] 📚 Estudio guiado
> Capítulo: [[03 - Objetos y workloads]] · [[07 - Salud, fiabilidad y despliegues]] · Índice: [[00 - Kubernetes - Índice]]
