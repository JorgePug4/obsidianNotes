---
tags:
  - kubernetes
  - extensibility
aliases:
  - Custom Resource Definition
  - Custom Resource Definitions
  - CRD
  - CRDs
  - Custom Resource
  - CR
  - Recurso personalizado
---

# Custom Resource Definition

### Qué es
Una **Custom Resource Definition** (CRD) es un recurso de la API que permite **agregar tipos de objetos nuevos a Kubernetes** sin modificar su código. Al aplicar una CRD, el API Server empieza a aceptar un nuevo `kind` (por ejemplo `Certificate`, `VirtualService` o `HTTPRoute`), y a cada objeto de ese tipo se le llama **Custom Resource** (CR).

### Para qué sirve
Extiende la API de Kubernetes para modelar conceptos propios de una herramienta o de un dominio. Muchas piezas del ecosistema se construyen así:
- La [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]] (`GatewayClass`, `Gateway`, `HTTPRoute`).
- Una [[Networking#Service Mesh|Service Mesh]] como Istio (`VirtualService`, `DestinationRule`).
- cert-manager (`Certificate`, `Issuer`), Argo CD (`Application`), Prometheus Operator (`ServiceMonitor`).

Por sí sola, una CRD **solo guarda datos**: el API Server valida y almacena los objetos en etcd, pero nada ocurre en el clúster. El comportamiento lo aporta un [[Custom Controller]] que observa esos objetos y actúa. CRD + controlador = **Operator**.

> [!tip] Validación con OpenAPI
> La CRD define un esquema `openAPIV3Schema` que el API Server usa para rechazar objetos mal formados, igual que con los recursos nativos. También puede exponer columnas extra en `kubectl get` con `additionalPrinterColumns`.

> [!warning] Borrar una CRD borra todos sus objetos
> Al eliminar una CRD, Kubernetes elimina **todos los Custom Resources de ese tipo** en todos los namespaces. Ten cuidado al desinstalar operadores (algunos charts de [[Helm]] no borran las CRDs precisamente por esto).

### Ejemplo

**Comandos útiles:**
```bash
# Listar las CRDs instaladas en el clúster
kubectl get crd

# Ver el esquema y las versiones de una CRD
kubectl describe crd backups.example.com

# Consultar la documentación del nuevo tipo como si fuera nativo
kubectl explain backup.spec

# Trabajar con los Custom Resources igual que con cualquier recurso
kubectl apply -f backup.yaml
kubectl get backups
kubectl get bk   # nombre corto definido en la CRD
```

**Definición de la CRD (`crd.yaml`) y un Custom Resource (`backup.yaml`):**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.example.com   # <plural>.<group>
spec:
  group: example.com
  scope: Namespaced
  names:
    kind: Backup
    plural: backups
    singular: backup
    shortNames: ["bk"]
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              required: ["database", "schedule"]
              properties:
                database:
                  type: string
                schedule:
                  type: string
                retentionDays:
                  type: integer
                  minimum: 1
---
apiVersion: example.com/v1
kind: Backup
metadata:
  name: backup-diario-mysql
spec:
  database: mysql-0
  schedule: "0 3 * * *"
  retentionDays: 7
```

### Relacionado
- [[Custom Controller]]
- [[Kube-controller-manager]]
- [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]]
- [[Networking#Service Mesh|Service Mesh]]
- [[Helm]]
