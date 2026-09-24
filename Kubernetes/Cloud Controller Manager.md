---
tags:
  - kubernetes
  - architecture
  - cloud
aliases:
  - Cloud Controller Manager
  - cloud-controller-manager
  - CCM
  - Cloud Provider
---

# Cloud Controller Manager

### Qué es
El **Cloud Controller Manager** (CCM) es un componente opcional del [[Control Plane]] que contiene la **lógica específica de cada proveedor de nube** (AWS, Azure, GCP, DigitalOcean, OpenStack, etc.). Separa ese código del núcleo de Kubernetes para que cada proveedor lo mantenga y publique a su propio ritmo.

### Para qué sirve
Conecta los objetos de Kubernetes con los recursos reales de la nube. Incluye principalmente tres controladores:
1. **Node controller**: al registrarse un nodo, consulta a la nube su zona, tipo de instancia e IPs, y elimina el objeto `Node` cuando la VM se borra en el proveedor.
2. **Route controller**: configura las rutas de la red de la nube para que los Pods de distintos nodos se alcancen (depende del modelo de red).
3. **Service controller**: cuando se crea un [[Networking#Kubernetes Service|Kubernetes Service]] de tipo [[Networking#LoadBalancer|LoadBalancer]], crea el balanceador de carga en la nube, le asigna una IP pública y lo mantiene actualizado.

> [!tip] ¿Por qué mi Service LoadBalancer se queda en `<pending>`?
> En clústeres locales (minikube, kind, bare-metal) no hay un CCM que cree el balanceador, así que `EXTERNAL-IP` queda en `<pending>`. Soluciones: `minikube tunnel`, MetalLB en bare-metal, o usar [[Networking#NodePort|NodePort]] / [[Networking#Ingress|Ingress]].

> [!warning] No confundir con el CSI Driver
> El CCM ya no se encarga de los discos. Los volúmenes en la nube se gestionan con un [[CSI Driver]] y una [[StorageClass]].

### Ejemplo

**Comandos útiles:**
```bash
# Ver si el clúster ejecuta un cloud-controller-manager
kubectl get pods -n kube-system | grep -i cloud-controller

# Crear un Service LoadBalancer y ver cómo el CCM asigna la IP externa
kubectl expose deployment python-django-app --type=LoadBalancer --port=80 --target-port=8000
kubectl get svc python-django-app -w

# Revisar los eventos del Service si el balanceador no se crea
kubectl describe svc python-django-app

# Ver los datos que el CCM agregó al nodo (providerID, zona, tipo de instancia)
kubectl get node <nodo> -o jsonpath='{.spec.providerID}'
kubectl get node <nodo> --show-labels
```

**Anotaciones que lee el CCM para configurar el balanceador (ejemplo en AWS):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-publico
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

### Relacionado
- [[Control Plane]]
- [[Kube-controller-manager]]
- [[Networking#LoadBalancer|LoadBalancer]]
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Worker Node]]
- [[CSI Driver]]
