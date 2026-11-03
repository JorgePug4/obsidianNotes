---
tags:
  - kubernetes
  - architecture
aliases:
  - kubelet
  - Kubelet
  - Node Agent
  - Static Pod
  - Static Pods
---

# kubelet

### Qué es
El **kubelet** es el **agente principal que corre en cada nodo** del clúster (tanto en los [[Worker Node]]s como, normalmente, en los nodos del [[Control Plane]]). A diferencia de casi todo lo demás en Kubernetes, no corre como un [[Workloads#Pod|Pod]]: es un proceso del sistema operativo (normalmente un servicio de `systemd`).

### Para qué sirve
Es el puente entre el [[Control Plane]] y los contenedores del nodo:
1. **Observa** en el API Server los Pods asignados a su nodo (los *PodSpecs*).
2. **Ordena** al [[Container Runtime]] (vía la interfaz CRI) descargar imágenes y crear/detener contenedores.
3. **Monta volúmenes** (incluidos [[ConfigMap]], [[Secret]] y [[PersistentVolumeClaim]] a través del [[CSI Driver]]) y configura la red del Pod mediante el [[Networking#CNI|CNI]].
4. **Ejecuta las sondas de salud** (*liveness*, *readiness* y *startup probes*) y reinicia contenedores según la `restartPolicy`.
5. **Reporta** al API Server el estado del nodo y de sus Pods, y desaloja Pods cuando el nodo se queda sin recursos (*node-pressure eviction*).

> [!tip] Static Pods
> El kubelet también puede ejecutar Pods leyendo manifiestos directamente desde un directorio local (por defecto `/etc/kubernetes/manifests`), sin pasar por el API Server. Así es como kubeadm arranca los componentes del [[Control Plane]] (kube-apiserver, etcd, scheduler, controller-manager).

> [!warning] Si el kubelet cae, el nodo queda `NotReady`
> Sin kubelet, los contenedores existentes pueden seguir corriendo, pero el nodo deja de reportar su estado, no recibe Pods nuevos y, pasado el tiempo de tolerancia, sus Pods se desalojan.

### Ejemplo

**Comandos útiles (ejecutados en el nodo):**
```bash
# Ver el estado del servicio kubelet
systemctl status kubelet

# Revisar los logs del kubelet para diagnosticar problemas de Pods o del nodo
journalctl -u kubelet -f

# Reiniciar el kubelet tras cambiar su configuración
sudo systemctl restart kubelet

# Ver la configuración del kubelet
cat /var/lib/kubelet/config.yaml

# Ver la versión del kubelet de cada nodo desde kubectl
kubectl get nodes -o wide
```

**Sondas de salud que el kubelet ejecuta (`probes.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-con-sondas
spec:
  containers:
    - name: app
      image: nginx:latest
      ports:
        - containerPort: 80
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /
          port: 80
        periodSeconds: 5
```

### Relacionado
- [[Worker Node]]
- [[Control Plane]]
- [[Container Runtime]]
- [[Workloads#Pod|Pod]]
- [[Networking#CNI|CNI]]
- [[CSI Driver]]

> [!info] 📚 Estudio guiado
> Capítulo: [[02 - Fundamentos y arquitectura]] · [[07 - Salud, fiabilidad y despliegues]] · Índice: [[00 - Kubernetes - Índice]]
