---
tags:
  - kubernetes
  - storage
aliases:
  - CSI Driver
  - CSI
  - Container Storage Interface
  - CSI Plugin
  - Driver CSI
---

# CSI Driver

### Qué es
Un **CSI Driver** es un plugin que implementa la **Container Storage Interface (CSI)**, un estándar abierto que define cómo un orquestador de contenedores (como Kubernetes) se comunica con un sistema de almacenamiento. Es el equivalente, para el almacenamiento, de lo que el [[Networking#CNI|CNI]] es para la red.

### Para qué sirve
Permite que cualquier proveedor de almacenamiento (AWS EBS, GCE PD, Azure Disk, Ceph, NetApp, Longhorn, NFS, etc.) se integre con Kubernetes **sin modificar el código fuente de Kubernetes**. Antes de CSI, los drivers vivían dentro del código de Kubernetes (*in-tree*). Hoy se instalan por separado (*out-of-tree*) y los antiguos drivers in-tree se migraron a CSI.

Un driver CSI suele desplegarse en dos partes:
1. **Controller plugin** (normalmente un Deployment): crea, borra, amplía y toma snapshots de volúmenes en el backend. Se apoya en *sidecars* como `external-provisioner` y `external-attacher`.
2. **Node plugin** (un DaemonSet en cada [[Worker Node]]): monta y desmonta el volumen en el nodo para que el [[kubelet]] lo entregue al [[Workloads#Pod|Pod]].

El driver se referencia desde el campo `provisioner` de una [[StorageClass]].

> [!tip] Snapshots de volúmenes
> Muchos drivers CSI soportan `VolumeSnapshot`, que permite crear copias puntuales de un [[PersistentVolumeClaim]] y restaurarlas en un PVC nuevo. Requiere los CRDs y el controlador de snapshots instalados.

### Ejemplo

**Comandos útiles:**
```bash
# Listar los drivers CSI registrados en el clúster
kubectl get csidrivers

# Ver qué drivers CSI están disponibles en cada nodo
kubectl get csinodes

# Ver los Pods del driver (ejemplo con el driver de AWS EBS)
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver

# Instalar el driver CSI de AWS EBS con Helm
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver -n kube-system
```

**Uso del driver desde una StorageClass (`storageclass.yaml`):**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
volumeBindingMode: WaitForFirstConsumer
```

### Relacionado
- [[StorageClass]]
- [[PersistentVolume]]
- [[PersistentVolumeClaim]]
- [[Volume]]
- [[kubelet]]
- [[Networking#CNI|CNI]]
- [[Helm]]
