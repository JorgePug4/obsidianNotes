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

> [!abstract] En 30 segundos
> - **Qué es:** un objeto que guarda **configuración no secreta** (pares clave-valor o archivos completos) fuera de la imagen.
> - **Analogía:** el archivo `appsettings.json` que cambias por entorno, pero gestionado por Kubernetes.
> - **Recuerda:** como **variable de entorno no se refresca**; como **archivo montado sí** (con retraso, y nunca con `subPath`).

---

## Qué es

Un **ConfigMap** (abreviado `cm`) es un objeto de la API que guarda **datos de configuración no confidenciales**. Pertenece a un namespace.

## Para qué sirve

Desacopla la configuración de la imagen: **la misma imagen** corre en desarrollo, pruebas y producción cambiando solo el ConfigMap.

## Tres formas de consumirlo

| Forma | Cómo | ¿Se actualiza si cambia el ConfigMap? |
|---|---|---|
| **Variables de entorno** | `env.valueFrom.configMapKeyRef` (una) o `envFrom` (todas) | ❌ No. Hay que reiniciar los Pods |
| **Archivos montados** | Como [[Volume]]: cada clave se convierte en un archivo | ✅ Sí, tras un breve retraso (❌ si se monta con `subPath`) |
| **Argumentos del comando** | `args: ["--level=$(LOG_LEVEL)"]` usando las variables anteriores | ❌ No |

## Errores comunes

> [!warning] No guardes datos sensibles
> Un ConfigMap se almacena y se muestra en **texto plano**. Contraseñas, tokens y certificados van en un [[Secret]].

> [!warning] "Cambié el ConfigMap y la app no se enteró"
> Si lo consumes como variables de entorno, reinicia los Pods:
> `kubectl rollout restart deployment <nombre>`.
> Con Helm, el patrón habitual es una anotación `checksum/config` en la plantilla del Pod que fuerza el reinicio cuando cambia el contenido.

> [!tip] Límite de tamaño
> Un ConfigMap no puede superar **1 MiB**. Para archivos grandes usa un [[Volume]] o un [[PersistentVolumeClaim]]. Si nunca va a cambiar, márcalo `immutable: true` (reduce carga en el API Server y evita cambios accidentales).

---

## Ejemplo

### Comandos

```bash
# Crear desde valores literales
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=LOG_LEVEL=info

# Crear desde un archivo
kubectl create configmap nginx-conf --from-file=nginx.conf

# Ver el contenido
kubectl get configmap
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Reiniciar un Deployment para que tome los nuevos valores
kubectl rollout restart deployment python-django-app
```

### YAML: ConfigMap + consumo en un Deployment

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
          image: python-django-app:1.0.0
          envFrom:                     # ① todas las claves como variables de entorno
            - configMapRef:
                name: app-config
          volumeMounts:                # ② settings.json como archivo en /etc/app
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

---

## Relacionado
[[Secret]] · [[Volume]] · [[Workloads#Pod|Pod]] · [[Workloads#Deployment|Deployment]]

> [!info] 📚 Estudio guiado
> Capítulo: [[05 - Configuración y almacenamiento]] · Índice: [[00 - Kubernetes - Índice]]
