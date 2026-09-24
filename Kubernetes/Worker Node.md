---
tags:
  - kubernetes
  - architecture
aliases:
  - Worker Node
  - Worker Nodes
  - Nodo de trabajo
  - Nodo worker
  - Node
  - Nodo
---

# Worker Node

### Qué es
Un **Worker Node** (nodo de trabajo) es una máquina, física o virtual, que forma parte del clúster y **ejecuta los [[Workloads#Pod|Pod]]s de las aplicaciones**. Es gestionado por el [[Control Plane]].

### Para qué sirve
Aporta la capacidad de cómputo (CPU, memoria, almacenamiento, red) donde realmente corren los contenedores. Cada Worker Node ejecuta tres componentes:
1. **[[kubelet]]**: agente que recibe del API Server los Pods asignados al nodo y se asegura de que sus contenedores estén corriendo y sanos.
2. **[[Networking#Kube-proxy|Kube-proxy]]**: mantiene las reglas de red (iptables/IPVS) que implementan los [[Networking#Kubernetes Service|Kubernetes Service]].
3. **[[Container Runtime]]**: software que descarga imágenes y ejecuta los contenedores (containerd, CRI-O).

Además, en cada nodo corren el plugin [[Networking#CNI|CNI]] (red de Pods) y, si aplica, el plugin de nodo del [[CSI Driver]] (almacenamiento).

> [!tip] Mantenimiento de un nodo: `cordon` y `drain`
> Antes de actualizar o apagar un nodo:
> - `kubectl cordon <nodo>`: lo marca como *unschedulable* (no recibe Pods nuevos).
> - `kubectl drain <nodo> --ignore-daemonsets --delete-emptydir-data`: desaloja los Pods de forma ordenada para que se reprogramen en otros nodos.
> - `kubectl uncordon <nodo>`: lo vuelve a habilitar.

> [!warning] Nodo en estado `NotReady`
> Si el [[kubelet]] deja de reportar al API Server, el nodo pasa a `NotReady` y, tras un tiempo, sus Pods se desalojan y se recrean en otros nodos (solo si están gestionados por un [[Workloads#Deployment|Deployment]], [[Workloads#StatefulSet|StatefulSet]], etc.). Los Pods sueltos no se recrean.

### Ejemplo

**Comandos útiles:**
```bash
# Listar nodos con IP, versión del kubelet y runtime
kubectl get nodes -o wide

# Ver capacidad, recursos asignados, condiciones y taints de un nodo
kubectl describe node worker-1

# Ver consumo de CPU/memoria por nodo (requiere metrics-server)
kubectl top nodes

# Agregar una etiqueta para programar Pods en nodos específicos
kubectl label node worker-1 disktype=ssd

# Aplicar un taint para que solo ciertos Pods se programen en el nodo
kubectl taint nodes worker-1 dedicated=gpu:NoSchedule

# Mantenimiento
kubectl cordon worker-1
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data
kubectl uncordon worker-1
```

**Programar un Pod en un nodo con una etiqueta concreta (`nodeSelector`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-en-ssd
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: app
      image: nginx:latest
```

### Relacionado
- [[Control Plane]]
- [[kubelet]]
- [[Container Runtime]]
- [[Networking#Kube-proxy|Kube-proxy]]
- [[Networking#CNI|CNI]]
- [[Networking#NodePort|NodePort]]
- [[Workloads#Pod|Pod]]
