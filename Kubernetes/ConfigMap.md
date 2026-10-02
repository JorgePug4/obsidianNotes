---
tags:
  - kubernetes
  - configuration
aliases:
  - ConfigMap
  - ConfigMaps
  - CM
  - Config Map
---

# ConfigMap

### Qué es
Un **ConfigMap** (abreviado `cm`) es un objeto de la API de Kubernetes que guarda **datos de configuración no confidenciales** en forma de pares clave-valor (o archivos completos). Pertenece a un namespace.

### Para qué sirve
Desacopla la configuración de la imagen del contenedor: la misma imagen puede ejecutarse en desarrollo, pruebas y producción cambiando solo el ConfigMap. Un [[Workloads#Pod|Pod]] puede consumirlo de tres maneras:
1. **Variables de entorno** individuales (`env.valueFrom.configMapKeyRef`) o todas a la vez (`envFrom`).
2. **Archivos montados** como [[Volume]] (cada clave se convierte en un archivo).
3. **Argumentos de línea de comandos** del contenedor, usando las variables de entorno anteriores.

> [!warning] No guardes datos sensibles
> Un ConfigMap se almacena y se muestra en texto plano. Contraseñas, tokens y certificados deben ir en un [[Secret]].

> [!warning] Las variables de entorno no se actualizan en caliente
> Si el ConfigMap cambia, las variables de entorno del contenedor **no** se actualizan: hay que reiniciar los Pods (ej. `kubectl rollout restart deployment <nombre>`). Los ConfigMaps montados como volumen sí se actualizan tras un breve retraso (excepto si se montan con `subPath`).

> [!tip] Límite de tamaño
> Un ConfigMap no puede superar **1 MiB**. Para archivos grandes usa un [[Volume]] o un [[PersistentVolumeClaim]].

### Ejemplo

**Comandos imperativos de gestión:**
```bash
# Crear un ConfigMap desde valores literales
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=LOG_LEVEL=info

# Crear un ConfigMap desde un archivo
kubectl create configmap nginx-conf --from-file=nginx.conf

# Listar y ver el contenido
kubectl get configmap
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Reiniciar un Deployment para que tome los nuevos valores
kubectl rollout restart deployment python-django-app
```

**Definición declarativa en YAML (`configmap.yaml`) y su consumo en un Deployment:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  settings.json: |
    { "featureX": true, "maxConnections": 50 }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-django-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-python-app
  template:
    metadata:
      labels:
        app: sample-python-app
    spec:
      containers:
        - name: app
          image: python-django-app:latest
          envFrom:
            - configMapRef:
                name: app-config
          volumeMounts:
            - name: config-files
              mountPath: /etc/app
      volumes:
        - name: config-files
          configMap:
            name: app-config
            items:
              - key: settings.json
                path: settings.json
```

### Relacionado
- [[Secret]]
- [[Volume]]
- [[Workloads#Pod|Pod]]
- [[Workloads#Deployment|Deployment]]

> [!info] 📚 Estudio guiado
> Capítulo: [[05 - Configuración y almacenamiento]] · Índice: [[00 - Kubernetes - Índice]]
