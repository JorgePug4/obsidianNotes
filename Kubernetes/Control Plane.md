---
tags:
  - kubernetes
  - architecture
aliases:
  - Control Plane
  - Plano de Control
  - Master Node
  - Nodo Maestro
  - kube-apiserver
  - API Server
  - etcd
  - kube-scheduler
  - Scheduler
---

# Control Plane

### Qué es
El **Control Plane** (plano de control, antes llamado *master node*) es el conjunto de componentes que **toman las decisiones globales del clúster**: guardan el estado deseado, deciden en qué nodo corre cada [[Workloads#Pod|Pod]] y reaccionan a los cambios. Los contenedores de las aplicaciones corren en los [[Worker Node]]s.

### Para qué sirve
Mantiene el clúster en el **estado deseado** declarado por el usuario. Sus componentes son:
1. **kube-apiserver**: la puerta de entrada de todo el clúster. Expone la API REST de Kubernetes, autentica y autoriza las peticiones (de `kubectl`, del [[kubelet]], de los controladores) y es el **único** componente que habla directamente con etcd.
2. **etcd**: base de datos clave-valor distribuida y consistente donde se guarda todo el estado del clúster. Si se pierde etcd sin respaldo, se pierde el clúster.
3. **kube-scheduler**: observa los Pods sin nodo asignado y elige el nodo adecuado según recursos disponibles, afinidades, *taints/tolerations* y restricciones.
4. **[[Kube-controller-manager]]**: ejecuta los bucles de control que llevan el estado actual hacia el deseado (réplicas, nodos, endpoints, jobs...).
5. **[[Cloud Controller Manager]]** (opcional): integra el clúster con la API del proveedor de nube.

> [!tip] Flujo de un `kubectl apply`
> `kubectl` → **kube-apiserver** (valida y guarda en **etcd**) → el **controller-manager** crea el [[Workloads#ReplicaSet|ReplicaSet]] y los Pods → el **scheduler** asigna cada Pod a un nodo → el [[kubelet]] del nodo pide al [[Container Runtime]] que arranque los contenedores.

> [!warning] Alta disponibilidad
> En producción el Control Plane debe ser redundante: al menos **3 nodos** de control (o de etcd) para mantener el quórum. En servicios gestionados (EKS, GKE, AKS) el proveedor administra el Control Plane por ti.

### Ejemplo

**Comandos útiles:**
```bash
# Ver la dirección del API Server y servicios principales
kubectl cluster-info

# Listar los nodos y sus roles (control-plane / worker)
kubectl get nodes -o wide

# Ver los componentes del Control Plane (en clústeres kubeadm/minikube corren como static Pods)
kubectl get pods -n kube-system

# Comprobar la salud del API Server
kubectl get --raw='/readyz?verbose'

# Respaldar etcd (ejecutado en un nodo del Control Plane)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**Ubicación de los manifiestos del Control Plane (clústeres creados con kubeadm):**
```bash
ls /etc/kubernetes/manifests/
# etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

### Relacionado
- [[Worker Node]]
- [[Kube-controller-manager]]
- [[Cloud Controller Manager]]
- [[kubelet]]
- [[Networking#Kube-proxy|Kube-proxy]]
- [[Custom Controller]]
