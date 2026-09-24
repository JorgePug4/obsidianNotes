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
Un **ReplicaSet** (anteriormente conocido en versiones previas como *Replication Controller*) es un objeto y controlador del plano de control de Kubernetes cuyo objetivo principal es garantizar que un número exacto y determinado de réplicas de un [[Workloads#Pod|Pod]] idéntico se encuentren en estado de ejecución en todo momento.

### Para qué sirve
Proporciona escalabilidad y auto-recuperación (*auto-healing*). Monitorea de forma continua el clúster: si un Pod falla, es eliminado accidentalmente o se cae el nodo donde residía, el ReplicaSet detecta la discrepancia entre el estado deseado y el estado real y crea un nuevo Pod de forma automática para reemplazarlo. Identifica qué Pods le pertenecen utilizando etiquetas y selectores (`labels` y `selectors`).

> [!tip] Uso indirecto recomendado
> Aunque es posible definir y crear un ReplicaSet de forma independiente, en la práctica habitual de DevOps no se gestionan directamente. En su lugar, se utilizan los [[Workloads#Deployment|Deployment]], los cuales administran automáticamente el ciclo de vida de los ReplicaSets subyacentes.

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
