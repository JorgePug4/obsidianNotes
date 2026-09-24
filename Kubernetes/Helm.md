---
tags:
  - kubernetes
  - tooling
aliases:
  - Helm
  - Helm Chart
  - Helm Charts
  - Chart
  - Charts
  - Helm Release
  - values.yaml
---

# Helm

### Qué es
**Helm** es el **gestor de paquetes de Kubernetes**. Empaqueta todos los manifiestos de una aplicación ([[Workloads#Deployment|Deployment]], [[Networking#Kubernetes Service|Kubernetes Service]], [[ConfigMap]], [[Networking#Ingress|Ingress]], etc.) en una unidad reutilizable llamada **chart**, parametrizable mediante plantillas.

### Para qué sirve
Evita copiar y mantener a mano decenas de archivos YAML por entorno. Conceptos clave:
- **Chart**: el paquete (plantillas + `Chart.yaml` + `values.yaml`).
- **Values**: parámetros que personalizan el chart (réplicas, imagen, recursos, dominio...). Se sobrescriben con `-f valores.yaml` o `--set clave=valor`.
- **Release**: una instalación concreta de un chart en el clúster, con nombre y namespace. Un mismo chart puede instalarse varias veces.
- **Repository**: servidor donde se publican charts (repositorios HTTP o registros OCI).

Además gestiona el **historial de versiones** de cada release, lo que permite `upgrade` y `rollback` en un solo comando. Es la forma habitual de instalar componentes como un [[Networking#Ingress Controller|Ingress Controller]], una [[Networking#Service Mesh|Service Mesh]] (Istio) o un [[CSI Driver]].

> [!tip] Desde Helm 3 ya no existe Tiller
> Helm 3 eliminó el componente de servidor *Tiller*. Helm habla directamente con el API Server usando tu `kubeconfig` y respeta tus permisos RBAC. El estado de cada release se guarda como un [[Secret]] en el namespace de la release.

> [!warning] Las CRDs y Helm
> Las [[Custom Resource Definition]] incluidas en la carpeta `crds/` de un chart se instalan una sola vez: `helm upgrade` **no las actualiza** y `helm uninstall` **no las borra** (para no eliminar datos). Por eso muchos proyectos piden instalar o actualizar las CRDs por separado.

### Ejemplo

**Comandos de gestión:**
```bash
# Agregar un repositorio de charts y actualizar el índice
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Buscar charts y ver sus valores configurables
helm search repo ingress-nginx
helm show values ingress-nginx/ingress-nginx > values-default.yaml

# Instalar (o actualizar si ya existe) una release en su propio namespace
helm upgrade --install my-ingress ingress-nginx/ingress-nginx \
  -n ingress-nginx --create-namespace -f mis-valores.yaml

# Renderizar las plantillas sin instalar (ideal para revisar el YAML final)
helm template my-app ./mi-chart -f values-prod.yaml

# Listar releases, ver historial y volver a una versión anterior
helm list -A
helm history my-ingress -n ingress-nginx
helm rollback my-ingress 1 -n ingress-nginx

# Desinstalar
helm uninstall my-ingress -n ingress-nginx

# Crear el esqueleto de un chart propio
helm create mi-chart
```

**Estructura de un chart y uso de valores en una plantilla:**
```text
mi-chart/
├── Chart.yaml          # nombre, versión del chart y appVersion
├── values.yaml         # valores por defecto
├── crds/               # CRDs (opcional)
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── _helpers.tpl
```

```yaml
# values.yaml
replicaCount: 2
image:
  repository: python-django-app
  tag: "1.0.0"
service:
  port: 80
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 8000
```

### Relacionado
- [[Custom Resource Definition]]
- [[Custom Controller]]
- [[Networking#Ingress Controller|Ingress Controller]]
- [[Networking#Service Mesh|Service Mesh]]
- [[Workloads#Deployment|Deployment]]
- [[Secret]]
