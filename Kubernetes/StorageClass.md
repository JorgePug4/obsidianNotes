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

> [!abstract] En 30 segundos
> - **Qué es:** un **perfil de almacenamiento** ("SSD rápido cifrado", "HDD barato") que permite crear discos automáticamente.
> - **Analogía:** el menú de un restaurante. El [[PersistentVolumeClaim|PVC]] pide un plato del menú; la cocina ([[CSI Driver]]) lo prepara.
> - **Recuerda:** el `reclaimPolicy` por defecto es **`Delete`**: borrar el PVC **destruye el disco**.

---

## Qué es

Una **StorageClass** (`sc`) es un recurso de ámbito de clúster que describe una clase de almacenamiento: **qué aprovisionador** la crea (normalmente un [[CSI Driver]]) y **con qué parámetros** (tipo de disco, rendimiento, replicación, cifrado…).

## Para qué sirve

Habilita el **aprovisionamiento dinámico**: nadie crea [[PersistentVolume|PVs]] a mano. Cuando un PVC pide una StorageClass, el aprovisionador crea el disco y el PV.

## Campos clave

| Campo | Qué controla | Valores / ejemplo |
|---|---|---|
| `provisioner` | Qué driver crea los volúmenes | `ebs.csi.aws.com`, `pd.csi.storage.gke.io`, `disk.csi.azure.com`, `k8s.io/minikube-hostpath` |
| `parameters` | Opciones del proveedor | `type: gp3`, `encrypted: "true"` |
| `reclaimPolicy` | Qué pasa con el disco al borrar el PVC | `Delete` *(por defecto)* · `Retain` |
| `volumeBindingMode` | Cuándo se crea el disco | `Immediate` · `WaitForFirstConsumer` |
| `allowVolumeExpansion` | Si se pueden ampliar PVCs existentes | `true` / `false` |

> [!tip] Usa `WaitForFirstConsumer` en la nube
> Con `Immediate` el disco se crea en cualquier zona en cuanto aparece el PVC… y el Pod puede acabar programado en **otra zona**, donde no puede montarlo. `WaitForFirstConsumer` espera a saber en qué nodo va el Pod y crea el disco ahí.

> [!tip] StorageClass por defecto
> Si un PVC no indica `storageClassName`, se usa la clase con la anotación `storageclass.kubernetes.io/is-default-class: "true"`. Para forzar un PV estático **sin** clase: `storageClassName: ""`.

> [!danger] `reclaimPolicy: Delete` borra el disco
> Con el valor por defecto, eliminar el PVC destruye también el disco en el proveedor. Para bases de datos productivas usa una clase con `Retain` y backups (VolumeSnapshots, Velero).

---

## Ejemplo (AWS EBS)

### YAML

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
reclaimPolicy: Retain                       # no borrar el disco con el PVC
volumeBindingMode: WaitForFirstConsumer     # crear el disco en la zona del Pod
allowVolumeExpansion: true
```

### Comandos

```bash
# Ver las clases y cuál es la predeterminada (default)
kubectl get storageclass

# Ver aprovisionador y parámetros
kubectl describe sc fast-ssd

# Marcar una clase como predeterminada
kubectl patch storageclass fast-ssd -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## Relacionado
[[PersistentVolume]] · [[PersistentVolumeClaim]] · [[CSI Driver]] · [[Volume]] · [[Workloads#StatefulSet|StatefulSet]]

> [!info] 📚 Estudio guiado
> Capítulo: [[05 - Configuración y almacenamiento]] · Índice: [[00 - Kubernetes - Índice]]
