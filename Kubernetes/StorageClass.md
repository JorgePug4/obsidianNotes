---
tags:
  - kubernetes
  - storage
aliases:
  - StorageClass
  - StorageClasses
  - SC
  - Storage Class
  - Aprovisionamiento dinámico
  - Dynamic Provisioning
---

# StorageClass

### Qué es
Una **StorageClass** (abreviado `sc`) es un recurso de ámbito de clúster que describe una **"clase" o perfil de almacenamiento** disponible: qué aprovisionador (*provisioner*, normalmente un [[CSI Driver]]) lo crea y con qué parámetros (tipo de disco, rendimiento, replicación, cifrado, etc.).

### Para qué sirve
Habilita el **aprovisionamiento dinámico**: en lugar de que un administrador cree cada [[PersistentVolume]] a mano, cuando un [[PersistentVolumeClaim]] pide una StorageClass, el aprovisionador crea el disco y el PV automáticamente.

Campos clave:
- **`provisioner`**: el driver que crea los volúmenes (ej. `ebs.csi.aws.com`, `pd.csi.storage.gke.io`, `disk.csi.azure.com`, `k8s.io/minikube-hostpath`).
- **`parameters`**: opciones específicas del proveedor (ej. `type: gp3`).
- **`reclaimPolicy`**: `Delete` (por defecto) o `Retain` para los PV que cree.
- **`volumeBindingMode`**: `Immediate` crea el volumen en cuanto aparece el PVC. `WaitForFirstConsumer` espera a que un Pod lo use para crearlo en la zona/nodo correcto.
- **`allowVolumeExpansion`**: permite ampliar PVC existentes.

> [!tip] StorageClass por defecto
> Si un PVC no especifica `storageClassName`, se usa la StorageClass marcada con la anotación `storageclass.kubernetes.io/is-default-class: "true"`. Si quieres forzar un PV estático sin clase, usa `storageClassName: ""`.

> [!warning] `reclaimPolicy: Delete` borra el disco
> Con la política por defecto (`Delete`), eliminar el PVC destruye también el disco en el proveedor. Para bases de datos productivas considera `Retain`.

### Ejemplo

**Comandos útiles:**
```bash
# Listar las StorageClasses y ver cuál es la predeterminada
kubectl get storageclass

# Inspeccionar el aprovisionador y los parámetros
kubectl describe sc fast-ssd

# Marcar una StorageClass como predeterminada
kubectl patch storageclass fast-ssd -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

**Definición declarativa en YAML (`storageclass.yaml`) — ejemplo en AWS EBS:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

### Relacionado
- [[PersistentVolume]]
- [[PersistentVolumeClaim]]
- [[CSI Driver]]
- [[Volume]]
- [[Workloads#StatefulSet|StatefulSet]]
