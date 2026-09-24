---
tags:
  - kubernetes
  - architecture
aliases:
  - Kube-controller-manager
  - kube-controller-manager
  - Controller Manager
  - Controladores
  - Reconciliation Loop
  - Bucle de reconciliación
---

# Kube-controller-manager

### Qué es
El **kube-controller-manager** es un componente del [[Control Plane]] que ejecuta, en un solo proceso, los **controladores integrados de Kubernetes**. Un controlador es un bucle infinito (*control loop*) que observa el estado del clúster a través del API Server y actúa para acercar el **estado actual** al **estado deseado**.

### Para qué sirve
Es el responsable del comportamiento "auto-reparable" de Kubernetes. Algunos de los controladores que incluye:
- **ReplicaSet controller**: garantiza que exista el número de [[Workloads#Pod|Pod]]s declarado en cada [[Workloads#ReplicaSet|ReplicaSet]] (si uno muere, crea otro).
- **Deployment controller**: gestiona los ReplicaSets de un [[Workloads#Deployment|Deployment]] durante actualizaciones y *rollbacks*.
- **StatefulSet, DaemonSet, Job y CronJob controllers**.
- **Node controller**: detecta nodos que dejan de responder y los marca como `NotReady`.
- **EndpointSlice controller**: mantiene la lista de IPs de Pods detrás de cada [[Networking#Kubernetes Service|Kubernetes Service]].
- **ServiceAccount y Namespace controllers**.
- **PersistentVolume controller**: liga cada [[PersistentVolumeClaim]] con su [[PersistentVolume]].

> [!tip] El patrón de reconciliación
> Todos los controladores siguen el mismo ciclo: **observar → comparar → actuar**. Es el mismo patrón que se usa para escribir un [[Custom Controller]] o un Operator.

> [!warning] Elección de líder
> En un Control Plane con alta disponibilidad hay varias copias del controller-manager, pero solo **una está activa** (líder) gracias a *leader election*. Las demás esperan para tomar el relevo.

### Ejemplo

**Comandos útiles:**
```bash
# Ver el Pod del controller-manager (clústeres kubeadm/minikube)
kubectl get pods -n kube-system -l component=kube-controller-manager

# Revisar sus logs (útil cuando los Pods de un ReplicaSet no se crean)
kubectl logs -n kube-system -l component=kube-controller-manager

# Ver qué instancia es el líder actual
kubectl get lease kube-controller-manager -n kube-system -o yaml

# Observar la reconciliación: borrar un Pod de un Deployment y ver cómo se recrea
kubectl delete pod <pod-de-un-deployment>
kubectl get pods -w
```

**Manifiesto del controller-manager (extracto de `/etc/kubernetes/manifests/kube-controller-manager.yaml`):**
```yaml
spec:
  containers:
    - name: kube-controller-manager
      command:
        - kube-controller-manager
        - --kubeconfig=/etc/kubernetes/controller-manager.conf
        - --leader-elect=true
        - --controllers=*,bootstrapsigner,tokencleaner
        - --node-monitor-grace-period=40s
```

### Relacionado
- [[Control Plane]]
- [[Cloud Controller Manager]]
- [[Custom Controller]]
- [[Workloads#ReplicaSet|ReplicaSet]]
- [[Workloads#Deployment|Deployment]]
