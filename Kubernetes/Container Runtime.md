---
tags:
  - kubernetes
  - architecture
aliases:
  - Container Runtime
  - Runtime de contenedores
  - Motor de contenedores
  - CRI
  - Container Runtime Interface
  - containerd
  - CRI-O
  - dockershim
---

# Container Runtime

### Qué es
El **Container Runtime** (entorno de ejecución de contenedores) es el software de cada nodo que **descarga imágenes y ejecuta los contenedores**. Kubernetes se comunica con él a través de la **CRI** (*Container Runtime Interface*), una API estándar que permite usar distintos runtimes sin cambiar Kubernetes.

### Para qué sirve
Cuando el [[kubelet]] recibe un [[Workloads#Pod|Pod]], le pide al runtime (vía CRI) que:
1. Descargue (*pull*) las imágenes de los contenedores.
2. Cree el *sandbox* del Pod (espacios de nombres de red, IPC, etc.) y los contenedores.
3. Los arranque, detenga, elimine y reporte su estado y logs.

Hay dos niveles:
- **Runtime de alto nivel (compatible con CRI)**: `containerd` (el más usado, por defecto en EKS, GKE, AKS y kubeadm) y `CRI-O` (por defecto en OpenShift).
- **Runtime de bajo nivel (OCI)**: `runc` (o alternativas como `crun`, `gVisor`, `Kata Containers`), que crea el proceso aislado usando *namespaces* y *cgroups* del kernel de Linux.

> [!warning] Docker Engine ya no es el runtime de Kubernetes
> Desde **Kubernetes 1.24** se eliminó el `dockershim`, así que Docker Engine no se usa directamente como runtime. **Las imágenes creadas con Docker siguen funcionando**, porque cumplen el estándar OCI y containerd/CRI-O las ejecutan igual. Si necesitas Docker Engine como runtime existe el adaptador externo `cri-dockerd`.

> [!tip] `crictl` en lugar de `docker`
> Para depurar contenedores directamente en un nodo usa `crictl`, que habla CRI con cualquier runtime compatible.

### Ejemplo

**Comandos útiles:**
```bash
# Ver qué runtime y versión usa cada nodo (columna CONTAINER-RUNTIME)
kubectl get nodes -o wide

# --- En el nodo ---
# Listar contenedores y Pods gestionados por el runtime
sudo crictl ps
sudo crictl pods

# Listar imágenes descargadas en el nodo
sudo crictl images

# Ver los logs de un contenedor desde el nodo
sudo crictl logs <container-id>

# Estado del servicio containerd
systemctl status containerd
```

**Elegir un runtime alternativo con `RuntimeClass` (ej. gVisor para mayor aislamiento):**
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
---
apiVersion: v1
kind: Pod
metadata:
  name: app-aislada
spec:
  runtimeClassName: gvisor
  containers:
    - name: app
      image: nginx:latest
```

### Relacionado
- [[kubelet]]
- [[Worker Node]]
- [[Workloads#Pod|Pod]]
- [[Networking#CNI|CNI]]

> [!info] 📚 Estudio guiado
> Capítulo: [[02 - Fundamentos y arquitectura]] · Índice: [[00 - Kubernetes - Índice]]
