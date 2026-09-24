---
tags:
  - kubernetes
  - storage
aliases:
  - Volume
  - Volumes
  - Volumen
  - K8s Volume
  - emptyDir
  - hostPath
---

# Volume

### Qué es
Un **Volume** (volumen) es un directorio accesible para los contenedores de un [[Workloads#Pod|Pod]] y que se declara a nivel de Pod en `spec.volumes`. Cada contenedor decide dónde montarlo mediante `volumeMounts`. El tipo de volumen determina de dónde provienen los datos y cuánto tiempo sobreviven (algunos viven lo mismo que el Pod, otros persisten más allá de él).

### Para qué sirve
Resuelve dos problemas del sistema de archivos del contenedor, que es efímero y privado:
1. **Persistencia**: evita que los datos se pierdan cuando un contenedor se reinicia.
2. **Compartición**: permite que varios contenedores del mismo Pod (por ejemplo, la aplicación y un *sidecar*) lean y escriban los mismos archivos.

Tipos más comunes:
- **`emptyDir`**: directorio vacío creado al programar el Pod en un nodo. Sobrevive a reinicios del contenedor, pero se borra cuando el Pod se elimina. Útil para caché o archivos temporales compartidos.
- **`hostPath`**: monta un directorio del sistema de archivos del nodo. Acopla el Pod al nodo y representa un riesgo de seguridad.
- **`configMap` / `secret`**: proyectan un [[ConfigMap]] o un [[Secret]] como archivos.
- **`persistentVolumeClaim`**: monta almacenamiento persistente solicitado mediante un [[PersistentVolumeClaim]].

> [!warning] `emptyDir` no es persistente
> Un `emptyDir` se borra junto con el Pod. Si el Pod se reprograma en otro nodo, los datos se pierden. Para datos que deben sobrevivir al Pod usa un [[PersistentVolumeClaim]].

> [!warning] Evita `hostPath` en producción
> `hostPath` expone el sistema de archivos del nodo al contenedor y ata el Pod a un nodo específico. Resérvalo para agentes del sistema (por ejemplo, recolectores de logs desplegados como DaemonSet).

### Ejemplo

**Comandos útiles:**
```bash
# Ver los volúmenes declarados y montados en un Pod
kubectl describe pod shared-pod

# Comprobar el contenido compartido desde el segundo contenedor
kubectl exec -it shared-pod -c reader -- cat /data/index.html
```

**Definición declarativa en YAML (`emptydir-pod.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-pod
spec:
  volumes:
    - name: shared-data
      emptyDir: {}
  containers:
    - name: writer
      image: busybox
      command: ["sh", "-c", "echo 'hola desde writer' > /data/index.html && sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data
    - name: reader
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data
```

### Relacionado
- [[Workloads#Pod|Pod]]
- [[PersistentVolume]]
- [[PersistentVolumeClaim]]
- [[StorageClass]]
- [[CSI Driver]]
- [[ConfigMap]]
- [[Secret]]
