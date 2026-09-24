---
tags:
  - kubernetes
  - storage
aliases:
  - PersistentVolumeClaim
  - PersistentVolumeClaims
  - PVC
  - Persistent Volume Claim
  - volumeClaimTemplates
---

# PersistentVolumeClaim

### Qué es
Un **PersistentVolumeClaim** (abreviado `pvc`) es una **solicitud de almacenamiento** hecha por un usuario o una aplicación. Especifica cuánto espacio necesita, con qué modo de acceso y, opcionalmente, de qué [[StorageClass]]. Kubernetes lo liga (*bind*) a un [[PersistentVolume]] que cumpla los requisitos. A diferencia del PV, el PVC **sí pertenece a un namespace**.

### Para qué sirve
Permite que los [[Workloads#Pod|Pod]]s consuman almacenamiento persistente sin conocer los detalles de la infraestructura (tipo de disco, proveedor de nube, servidor NFS). El Pod solo referencia el nombre del PVC. La relación PVC ↔ PV es **uno a uno**.

Flujo típico:
1. Se crea el PVC.
2. Si hay un PV compatible disponible, se liga. Si no y existe una [[StorageClass]], el [[CSI Driver]] aprovisiona uno nuevo dinámicamente.
3. El Pod monta el PVC como un [[Volume]].

> [!warning] Un PVC en `Pending`
> Un PVC se queda en `Pending` cuando no existe ningún PV compatible y no hay aprovisionamiento dinámico, o cuando la StorageClass usa `volumeBindingMode: WaitForFirstConsumer` y ningún Pod lo ha consumido todavía. Revisa los eventos con `kubectl describe pvc`.

> [!tip] PVC en StatefulSets
> Un [[Workloads#StatefulSet|StatefulSet]] no crea los PVC a mano: usa `volumeClaimTemplates` para generar un PVC por réplica (ej. `www-web-0`, `www-web-1`). Estos PVC **no se borran** al eliminar el StatefulSet.

### Ejemplo

**Comandos útiles:**
```bash
# Listar los PVC del namespace actual y verificar su estado (Bound / Pending)
kubectl get pvc

# Diagnosticar por qué un PVC no se liga
kubectl describe pvc data-pvc

# Ampliar el tamaño de un PVC (requiere allowVolumeExpansion: true en la StorageClass)
kubectl patch pvc data-pvc -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'

# Eliminar un PVC
kubectl delete pvc data-pvc
```

**Definición declarativa en YAML (`pvc.yaml`) y su uso en un Pod:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: app-con-datos
spec:
  containers:
    - name: app
      image: nginx:latest
      volumeMounts:
        - name: datos
          mountPath: /usr/share/nginx/html
  volumes:
    - name: datos
      persistentVolumeClaim:
        claimName: data-pvc
```

### Relacionado
- [[PersistentVolume]]
- [[StorageClass]]
- [[Volume]]
- [[CSI Driver]]
- [[Workloads#StatefulSet|StatefulSet]]
- [[Workloads#Pod|Pod]]
