---
tags:
  - kubernetes
  - storage
aliases:
  - PersistentVolume
  - PersistentVolumes
  - PV
  - Persistent Volume
  - Volumen Persistente
---

# PersistentVolume

### Qué es
Un **PersistentVolume** (abreviado `pv`) es un recurso del clúster que representa una pieza de almacenamiento real (un disco en la nube, un recurso NFS, un volumen iSCSI, etc.). Existe de forma independiente de cualquier [[Workloads#Pod|Pod]]: su ciclo de vida no está atado al de las aplicaciones que lo usan. A diferencia de la mayoría de los recursos, **no pertenece a un namespace** (es de ámbito de clúster).

### Para qué sirve
Separa **quién provee el almacenamiento** (el administrador o una [[StorageClass]]) de **quién lo consume** (el desarrollador mediante un [[PersistentVolumeClaim]]). Un PV puede crearse de dos formas:
1. **Aprovisionamiento estático**: el administrador crea el PV a mano.
2. **Aprovisionamiento dinámico**: una [[StorageClass]] y su [[CSI Driver]] crean el PV automáticamente cuando aparece un PVC.

Propiedades clave:
- **`accessModes`**: `ReadWriteOnce` (RWO, lectura/escritura desde un solo nodo), `ReadOnlyMany` (ROX), `ReadWriteMany` (RWX, varios nodos) y `ReadWriteOncePod` (RWOP, un solo Pod).
- **`persistentVolumeReclaimPolicy`**: qué hacer con el volumen cuando se libera su PVC. `Retain` conserva los datos y requiere limpieza manual. `Delete` borra el PV y el disco subyacente. (`Recycle` está obsoleto).
- **Estados (`STATUS`)**: `Available` → `Bound` → `Released` → (`Failed`).

> [!warning] `Released` no significa reutilizable
> Con la política `Retain`, al borrar el PVC el PV pasa a `Released` pero **no** vuelve a `Available`: todavía guarda la referencia al PVC anterior y sus datos. Para reutilizarlo hay que limpiar los datos y eliminar `spec.claimRef`, o borrar y recrear el PV.

> [!tip] RWO es por nodo, no por Pod
> `ReadWriteOnce` permite que varios Pods lo monten **si están en el mismo nodo**. Si necesitas exclusividad real de un solo Pod, usa `ReadWriteOncePod`.

### Ejemplo

**Comandos útiles:**
```bash
# Listar los PersistentVolumes (no llevan namespace)
kubectl get pv

# Ver capacidad, modo de acceso, política de reclamo y a qué PVC está ligado
kubectl describe pv pv-local-demo

# Eliminar un PersistentVolume
kubectl delete pv pv-local-demo
```

**Definición declarativa en YAML (`pv.yaml`) — aprovisionamiento estático:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local-demo
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

### Relacionado
- [[PersistentVolumeClaim]]
- [[StorageClass]]
- [[CSI Driver]]
- [[Volume]]
- [[Workloads#StatefulSet|StatefulSet]]

> [!info] 📚 Estudio guiado
> Capítulo: [[05 - Configuración y almacenamiento]] · Índice: [[00 - Kubernetes - Índice]]
