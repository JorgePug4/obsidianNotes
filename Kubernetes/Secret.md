---
tags:
  - kubernetes
  - configuration
  - security
aliases:
  - Secret
  - Secrets
  - K8s Secret
  - TLS Secret
  - Opaque Secret
---

# Secret

> [!abstract] En 30 segundos
> - **Qué es:** un [[ConfigMap]] pensado para **datos sensibles** (contraseñas, tokens, certificados).
> - **Analogía:** un sobre cerrado… pero **sin candado**. Base64 solo oculta el contenido a simple vista; cualquiera con permisos lo abre.
> - **Recuerda:** **Base64 no es cifrado.** La seguridad real viene de RBAC, cifrado en reposo y gestores externos.

---

## Qué es

Un **Secret** es un objeto de la API muy parecido a un [[ConfigMap]], pero para información confidencial. Pertenece a un namespace y se consume igual: como **variables de entorno** o como **archivos montados** en un [[Volume]].

## Tipos más comunes

| `type` | Para qué |
|---|---|
| `Opaque` *(por defecto)* | Datos arbitrarios definidos por ti |
| `kubernetes.io/tls` | Certificado (`tls.crt`) y llave (`tls.key`) para terminar HTTPS en un [[Networking#Ingress\|Ingress]] o [[Networking#Kubernetes Gateway API\|Gateway]] |
| `kubernetes.io/dockerconfigjson` | Credenciales de un registro privado de imágenes (`imagePullSecrets`) |
| `kubernetes.io/service-account-token` | Token **sin caducidad** de una ServiceAccount. Ver advertencia abajo |

> [!warning] Tokens de ServiceAccount: ya no uses Secrets
> Desde **Kubernetes 1.24** no se crea automáticamente un Secret de token por cada ServiceAccount (y desde 1.29-1.30 los antiguos sin usar se limpian). Los Pods reciben un **token proyectado** (TokenRequest API): **caduca** y está **ligado al Pod**. Se monta en `/var/run/secrets/kubernetes.io/serviceaccount/token`.
> - Token puntual: `kubectl create token <serviceaccount>`.
> - Crear a mano un Secret de este tipo solo tiene sentido en casos muy concretos y es un riesgo. Ver [[09 - Seguridad]].

## Cómo protegerlos de verdad

> [!danger] Base64 no es cifrado
> Los valores de `data` se decodifican trivialmente (`echo <valor> | base64 -d`) y, por defecto, se guardan **sin cifrar en etcd**.

| Capa | Medida |
|---|---|
| **Almacenamiento** | Cifrado en reposo (`EncryptionConfiguration`, idealmente con un proveedor **KMS**) |
| **Acceso** | **RBAC**: limita quién puede hacer `get`/`list` sobre Secrets |
| **Origen** | Gestores externos (Vault, AWS Secrets Manager, Azure Key Vault) con *External Secrets Operator* o *Secrets Store CSI Driver* |
| **Git** | Nunca subas Secrets en texto plano: usa **Sealed Secrets** o **SOPS** |

> [!tip] `stringData` para escribir en texto plano
> En el YAML puedes usar `stringData` en lugar de `data` y escribir los valores sin codificar. Kubernetes los convierte a Base64 al guardarlos.

---

## Ejemplo

### Comandos

```bash
# Secret genérico desde literales
kubectl create secret generic db-credentials --from-literal=DB_USER=admin --from-literal=DB_PASSWORD='S3gur0!'

# Secret TLS desde un certificado y su llave
kubectl create secret tls app-tls --cert=tls.crt --key=tls.key

# Secret para un registro privado de imágenes
kubectl create secret docker-registry regcred --docker-server=registry.example.com --docker-username=user --docker-password=pass

# Listar y decodificar un valor (demuestra que Base64 no protege nada)
kubectl get secrets
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

### YAML: Secret + consumo en un Pod

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:                      # texto plano; se guarda en Base64
  DB_USER: admin
  DB_PASSWORD: "S3gur0!"
---
apiVersion: v1
kind: Pod
metadata:
  name: app-con-secret
spec:
  containers:
    - name: app
      image: python-django-app:1.0.0
      env:                       # ① una clave como variable de entorno
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: DB_PASSWORD
      volumeMounts:              # ② todas las claves como archivos (preferible)
        - name: creds
          mountPath: /etc/creds
          readOnly: true
  volumes:
    - name: creds
      secret:
        secretName: db-credentials
```

> [!tip] ¿Variable de entorno o archivo?
> Mejor **archivo**: las variables de entorno aparecen en volcados de proceso, logs de errores y herramientas de diagnóstico, y no se actualizan sin reiniciar.

---

## Relacionado
[[ConfigMap]] · [[Volume]] · [[Workloads#Pod|Pod]] · [[Workloads#Deployment|Deployment]] · [[Networking#Ingress|Ingress]] · [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]]

> [!info] 📚 Estudio guiado
> Capítulo: [[05 - Configuración y almacenamiento]] · [[09 - Seguridad]] · Índice: [[00 - Kubernetes - Índice]]
