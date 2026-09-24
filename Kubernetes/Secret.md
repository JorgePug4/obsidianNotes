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

### Qué es
Un **Secret** es un objeto de la API de Kubernetes, muy parecido a un [[ConfigMap]], pero pensado para **datos sensibles**: contraseñas, tokens, llaves SSH, credenciales de registros de imágenes o certificados TLS. Pertenece a un namespace.

### Para qué sirve
Evita incluir información confidencial en la imagen del contenedor o en el manifiesto del [[Workloads#Pod|Pod]]. Se consume igual que un ConfigMap: como variables de entorno o como archivos montados en un [[Volume]]. Tipos más comunes (`type`):
- **`Opaque`**: datos arbitrarios definidos por el usuario (tipo por defecto).
- **`kubernetes.io/tls`**: certificado (`tls.crt`) y llave privada (`tls.key`), usado por un [[Networking#Ingress|Ingress]] o la [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]] para terminar HTTPS.
- **`kubernetes.io/dockerconfigjson`**: credenciales para descargar imágenes de registros privados (`imagePullSecrets`).
- **`kubernetes.io/service-account-token`**: tokens de ServiceAccounts (en versiones modernas se prefieren tokens proyectados de corta duración).

> [!warning] Base64 no es cifrado
> Los valores de `data` están codificados en **Base64**, que se decodifica trivialmente (`echo <valor> | base64 -d`). Por defecto los Secrets se guardan **sin cifrar en etcd**. Para protegerlos de verdad:
> - Habilita el **cifrado en reposo** (`EncryptionConfiguration`, idealmente con un proveedor KMS).
> - Restringe con **RBAC** quién puede hacer `get`/`list` sobre Secrets.
> - Considera gestores externos (HashiCorp Vault, AWS Secrets Manager) con *External Secrets Operator* o el *Secrets Store CSI Driver*.
> - No subas manifiestos de Secrets en texto plano a Git (usa Sealed Secrets o SOPS).

> [!tip] `stringData` para escribir en texto plano
> En el YAML puedes usar `stringData` en lugar de `data` para escribir los valores sin codificar. Kubernetes los convierte a Base64 al guardarlos.

### Ejemplo

**Comandos imperativos de gestión:**
```bash
# Crear un Secret genérico desde literales
kubectl create secret generic db-credentials --from-literal=DB_USER=admin --from-literal=DB_PASSWORD='S3gur0!'

# Crear un Secret TLS a partir de un certificado y su llave
kubectl create secret tls app-tls --cert=tls.crt --key=tls.key

# Crear un Secret para un registro privado de imágenes
kubectl create secret docker-registry regcred --docker-server=registry.example.com --docker-username=user --docker-password=pass

# Listar y decodificar un valor
kubectl get secrets
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

**Definición declarativa en YAML (`secret.yaml`) y su consumo en un Pod:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
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
      image: python-django-app:latest
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: DB_PASSWORD
      volumeMounts:
        - name: creds
          mountPath: /etc/creds
          readOnly: true
  volumes:
    - name: creds
      secret:
        secretName: db-credentials
```

### Relacionado
- [[ConfigMap]]
- [[Volume]]
- [[Workloads#Pod|Pod]]
- [[Workloads#Deployment|Deployment]]
- [[Networking#Ingress|Ingress]]
- [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]]
