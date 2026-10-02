---
tags: [kubernetes, laboratorios, practica]
---
# 15 - Laboratorios

> Laboratorios locales con **kind** (Kubernetes in Docker), **minikube**, `kubectl` y `helm`. Cada uno: objetivo, requisitos, arquitectura, pasos, resultado esperado, verificación, errores comunes, limpieza y reflexión.

## Contenido
- [[#Preparación del entorno]]
- [[#Lab 01 - Tu primer clúster con kind]]
- [[#Lab 02 - Workloads, rollouts y rollbacks]]
- [[#Lab 03 - Services, DNS e Ingress]]
- [[#Lab 04 - Configuración y persistencia con PostgreSQL]]
- [[#Lab 05 - Probes, fallos y troubleshooting]]
- [[#Lab 06 - Scheduling con taints, affinity y spread]]
- [[#Lab 07 - Autoescalado con HPA]]
- [[#Lab 08 - Seguridad RBAC y NetworkPolicies]]
- [[#Lab 09 - Observabilidad con kube-prometheus-stack]]
- [[#Lab 10 - Tu propio chart de Helm]]
- [[#Lab 11 - API .NET en kind]]

---

## Preparación del entorno

| Herramienta | Para qué | Instalación (Windows con winget) |
|---|---|---|
| Docker Desktop | Motor de contenedores para kind | `winget install Docker.DockerDesktop` |
| kubectl | CLI de Kubernetes | `winget install Kubernetes.kubectl` |
| kind | Clústeres en contenedores Docker (multi-nodo, rápido) | `winget install Kubernetes.kind` |
| minikube | Clúster local con add-ons y CNIs seleccionables | `winget install Kubernetes.minikube` |
| Helm | Gestor de paquetes | `winget install Helm.Helm` |
| k9s (opcional) | TUI | `winget install Derailed.k9s` |

```bash
docker version && kubectl version --client && kind version && helm version
```
> [!tip] Recursos de Docker Desktop
> Asigna al menos **4 CPU y 8 GB de RAM** (Settings → Resources) para los labs multi-nodo y de observabilidad. Docker Desktop también puede activar un Kubernetes de un nodo (Settings → Kubernetes); estos labs usan kind para poder tener varios nodos.

---

## Lab 01 - Tu primer clúster con kind

1. **Objetivo**: crear un clúster de 3 nodos, explorar la arquitectura y ver la reconciliación en acción.
2. **Requisitos**: Docker Desktop, kind, kubectl.
3. **Arquitectura**: 1 nodo *control-plane* + 2 *workers*, cada uno es un contenedor Docker.
4. **Pasos y YAML**:
```yaml
# kind-lab.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: lab
nodes:
  - role: control-plane
    extraPortMappings:            # para el Ingress del Lab 03
      - { containerPort: 80, hostPort: 80, protocol: TCP }
      - { containerPort: 443, hostPort: 443, protocol: TCP }
  - role: worker
  - role: worker
```
5. **Comandos**:
```bash
kind create cluster --config kind-lab.yaml
kubectl cluster-info --context kind-lab
kubectl get nodes -o wide
kubectl get pods -n kube-system -o wide       # etcd, apiserver, scheduler, controller-manager, coredns, kube-proxy, kindnet

# Reconciliación
kubectl create deployment web --image=nginx:1.27 --replicas=3
kubectl get pods -o wide -w                    # Ctrl+C cuando estén Running
kubectl delete pod -l app=web --wait=false
kubectl get pods -l app=web                    # se recrean con otros nombres
kubectl get events --sort-by=.lastTimestamp | tail -20
```
6. **Resultado esperado**: 3 nodos `Ready`; 3 Pods de `web` repartidos entre los workers; tras borrarlos, 3 nuevos.
7. **Verificación**: `kubectl get deploy web` → `READY 3/3`; en los eventos verás `Scheduled`, `Pulling`, `Pulled`, `Created`, `Started` y `SuccessfulCreate` del ReplicaSet.
8. **Errores comunes**: Docker Desktop parado (`Cannot connect to the Docker daemon`); puertos 80/443 ocupados en el host (quita `extraPortMappings` o libera los puertos); contexto equivocado (`kubectl config current-context`).
9. **Limpieza**: `kubectl delete deployment web` (el clúster se reutiliza en los siguientes labs). Para borrarlo todo: `kind delete cluster --name lab`.
10. **Reflexión**: ¿qué componente creó los Pods nuevos? ¿Por qué los nombres cambian? ¿Qué pasaría si borraras el ReplicaSet en lugar de los Pods?

---

## Lab 02 - Workloads, rollouts y rollbacks

1. **Objetivo**: hacer un rolling update, provocar un rollout fallido y recuperarlo; ejecutar Jobs y CronJobs.
2. **Requisitos**: clúster del Lab 01.
3. **Arquitectura**: Deployment `web` (4 réplicas) + Job + CronJob en el namespace `lab02`.
4. **YAML**:
```yaml
# lab02.yaml
apiVersion: v1
kind: Namespace
metadata: { name: lab02 }
---
apiVersion: apps/v1
kind: Deployment
metadata: { name: web, namespace: lab02 }
spec:
  replicas: 4
  selector: { matchLabels: { app: web } }
  strategy: { rollingUpdate: { maxSurge: 1, maxUnavailable: 0 } }
  minReadySeconds: 5
  progressDeadlineSeconds: 60
  template:
    metadata: { labels: { app: web } }
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports: [{ containerPort: 80 }]
          readinessProbe: { httpGet: { path: /, port: 80 }, periodSeconds: 2 }
---
apiVersion: batch/v1
kind: Job
metadata: { name: pi, namespace: lab02 }
spec:
  backoffLimit: 2
  ttlSecondsAfterFinished: 600
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: pi
          image: perl:5.40
          command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(500)"]
---
apiVersion: batch/v1
kind: CronJob
metadata: { name: hello, namespace: lab02 }
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 2
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: hello
              image: busybox:1.36
              command: ["sh", "-c", "date; echo Hola desde el CronJob"]
```
5. **Comandos**:
```bash
kubectl apply -f lab02.yaml
kubectl rollout status deploy/web -n lab02

# Rolling update correcto
kubectl set image deploy/web nginx=nginx:1.28 -n lab02
kubectl annotate deploy/web kubernetes.io/change-cause="nginx 1.28" -n lab02
kubectl get rs -n lab02 -w                     # observa cómo sube el RS nuevo y baja el viejo

# Rollout fallido (la imagen no existe)
kubectl set image deploy/web nginx=nginx:9.99 -n lab02
kubectl rollout status deploy/web -n lab02     # falla tras progressDeadlineSeconds (60 s)
kubectl get pods -n lab02                      # 1 Pod en ImagePullBackOff; los 4 antiguos siguen Ready
kubectl describe deploy web -n lab02 | grep -A3 Conditions

# Recuperación
kubectl rollout history deploy/web -n lab02
kubectl rollout undo deploy/web -n lab02
kubectl rollout status deploy/web -n lab02

# Job y CronJob
kubectl wait --for=condition=complete job/pi -n lab02 --timeout=120s
kubectl logs job/pi -n lab02
kubectl get cronjob,jobs -n lab02 -w           # cada minuto aparece un Job nuevo
```
6. **Resultado esperado**: el update a 1.28 se completa sin bajar de 4 Pods Ready; el update a 9.99 se atasca con los Pods viejos sirviendo; `undo` lo arregla.
7. **Verificación**: `kubectl get deploy web -n lab02 -o jsonpath='{.spec.template.spec.containers[0].image}'` → `nginx:1.28`.
8. **Errores comunes**: esperar un rollback automático (no existe); no ver el error por mirar el Deployment en lugar de los Pods (`describe pod` → `Failed to pull image`).
9. **Limpieza**: `kubectl delete namespace lab02`.
10. **Reflexión**: ¿por qué con `maxUnavailable: 0` el servicio no se vio afectado por la imagen rota? ¿Qué pasaría con `maxUnavailable: 50%`? ¿Cómo automatizarías el rollback en un pipeline?

---

## Lab 03 - Services, DNS e Ingress

1. **Objetivo**: exponer dos versiones de una app, depurar DNS y enrutar por ruta con Ingress; opcionalmente con Gateway API.
2. **Requisitos**: clúster del Lab 01 (con los puertos 80/443 mapeados).
3. **Arquitectura**: `Ingress → Service app-v1 / app-v2 → Pods http-echo`.
4. **YAML**:
```yaml
# lab03.yaml
apiVersion: v1
kind: Namespace
metadata: { name: lab03 }
---
apiVersion: apps/v1
kind: Deployment
metadata: { name: app-v1, namespace: lab03 }
spec:
  replicas: 2
  selector: { matchLabels: { app: echo, version: v1 } }
  template:
    metadata: { labels: { app: echo, version: v1 } }
    spec:
      containers:
        - name: echo
          image: hashicorp/http-echo:1.0
          args: ["-text=Hola desde v1", "-listen=:5678"]
          ports: [{ name: http, containerPort: 5678 }]
---
apiVersion: apps/v1
kind: Deployment
metadata: { name: app-v2, namespace: lab03 }
spec:
  replicas: 1
  selector: { matchLabels: { app: echo, version: v2 } }
  template:
    metadata: { labels: { app: echo, version: v2 } }
    spec:
      containers:
        - name: echo
          image: hashicorp/http-echo:1.0
          args: ["-text=Hola desde v2", "-listen=:5678"]
          ports: [{ name: http, containerPort: 5678 }]
---
apiVersion: v1
kind: Service
metadata: { name: app-v1, namespace: lab03 }
spec:
  selector: { app: echo, version: v1 }
  ports: [{ port: 80, targetPort: http }]
---
apiVersion: v1
kind: Service
metadata: { name: app-v2, namespace: lab03 }
spec:
  selector: { app: echo, version: v2 }
  ports: [{ port: 80, targetPort: http }]
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata: { name: echo, namespace: lab03 }
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - { path: /v1, pathType: Prefix, backend: { service: { name: app-v1, port: { number: 80 } } } }
          - { path: /v2, pathType: Prefix, backend: { service: { name: app-v2, port: { number: 80 } } } }
```
5. **Comandos**:
```bash
kubectl apply -f lab03.yaml

# Service y endpoints
kubectl get svc,endpointslices -n lab03
kubectl port-forward svc/app-v1 8080:80 -n lab03     # en otra terminal: curl localhost:8080

# DNS desde dentro del clúster
kubectl run tmp --rm -it --image=busybox:1.36 -n lab03 -- sh
#   nslookup app-v1
#   nslookup app-v1.lab03.svc.cluster.local
#   wget -qO- http://app-v1
#   wget -qO- http://app-v2.lab03
#   cat /etc/resolv.conf
#   exit

# Ingress Controller para el lab (ingress-nginx está RETIRADO desde marzo de 2026: válido para
# aprender, no para producción; versión fijada para que la URL siga funcionando)
kubectl label node lab-control-plane ingress-ready=true
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/kind/deploy.yaml
kubectl wait -n ingress-nginx --for=condition=ready pod -l app.kubernetes.io/component=controller --timeout=180s

curl http://localhost/v1      # Hola desde v1
curl http://localhost/v2      # Hola desde v2
```
6. **Resultado esperado**: cada ruta responde con su versión; los nombres cortos resuelven dentro del mismo namespace.
7. **Verificación**: `kubectl describe ingress echo -n lab03` muestra los backends con IPs de Pods; `kubectl get endpointslices -n lab03` lista 2 endpoints para v1 y 1 para v2.
8. **Errores comunes**: Ingress sin dirección (controlador no instalado o `ingressClassName` incorrecto); 404 del controlador (ruta no coincide); 503 (Service sin endpoints: revisa labels).
9. **Limpieza**: `kubectl delete ns lab03` (y `kubectl delete ns ingress-nginx` si no lo vas a reutilizar).
10. **Reflexión**: ¿por qué el Service de v1 no incluye los Pods de v2 aunque compartan `app: echo`? ¿Cómo harías un canary 90/10 entre v1 y v2? (pista: Gateway API con `weight`).

> [!example] Parte B (opcional): el mismo enrutamiento con Gateway API y Envoy Gateway
> ```bash
> # Instala Envoy Gateway (incluye las CRDs de Gateway API). Consulta la versión actual en gateway.envoyproxy.io
> helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.5.0 -n envoy-gateway-system --create-namespace
> kubectl wait --timeout=5m -n envoy-gateway-system deployment/envoy-gateway --for=condition=Available
> ```
> ```yaml
> apiVersion: gateway.networking.k8s.io/v1
> kind: GatewayClass
> metadata: { name: eg }
> spec: { controllerName: gateway.envoyproxy.io/gatewayclass-controller }
> ---
> apiVersion: gateway.networking.k8s.io/v1
> kind: Gateway
> metadata: { name: eg, namespace: lab03 }
> spec:
>   gatewayClassName: eg
>   listeners: [{ name: http, protocol: HTTP, port: 80 }]
> ---
> apiVersion: gateway.networking.k8s.io/v1
> kind: HTTPRoute
> metadata: { name: echo-canary, namespace: lab03 }
> spec:
>   parentRefs: [{ name: eg }]
>   rules:
>     - backendRefs:
>         - { name: app-v1, port: 80, weight: 90 }
>         - { name: app-v2, port: 80, weight: 10 }
> ```
> ```bash
> ENVOY_SERVICE=$(kubectl get svc -n envoy-gateway-system \
>   --selector=gateway.envoyproxy.io/owning-gateway-namespace=lab03,gateway.envoyproxy.io/owning-gateway-name=eg \
>   -o jsonpath='{.items[0].metadata.name}')
> kubectl -n envoy-gateway-system port-forward service/$ENVOY_SERVICE 8888:80 &
> for i in $(seq 1 20); do curl -s localhost:8888; done | sort | uniq -c    # ~18 v1 / ~2 v2
> ```

---

## Lab 04 - Configuración y persistencia con PostgreSQL

1. **Objetivo**: ConfigMap + Secret + StatefulSet con PVC; comprobar que los datos sobreviven al Pod.
2. **Requisitos**: clúster del Lab 01 (kind trae la StorageClass `standard` con *local-path-provisioner*).
3. **Arquitectura**: Headless Service + StatefulSet `pg` (1 réplica) + PVC por `volumeClaimTemplates`.
4. **YAML**:
```yaml
# lab04.yaml
apiVersion: v1
kind: Namespace
metadata: { name: lab04 }
---
apiVersion: v1
kind: Secret
metadata: { name: pg-secret, namespace: lab04 }
type: Opaque
stringData:
  POSTGRES_PASSWORD: "cambia-esto-en-serio"
---
apiVersion: v1
kind: ConfigMap
metadata: { name: pg-config, namespace: lab04 }
data:
  POSTGRES_DB: shop
  POSTGRES_USER: shop
---
apiVersion: v1
kind: Service
metadata: { name: pg, namespace: lab04 }
spec:
  clusterIP: None
  selector: { app: pg }
  ports: [{ port: 5432 }]
---
apiVersion: apps/v1
kind: StatefulSet
metadata: { name: pg, namespace: lab04 }
spec:
  serviceName: pg
  replicas: 1
  selector: { matchLabels: { app: pg } }
  template:
    metadata: { labels: { app: pg } }
    spec:
      containers:
        - name: postgres
          image: postgres:16-alpine
          envFrom:
            - configMapRef: { name: pg-config }
            - secretRef: { name: pg-secret }
          env:
            - { name: PGDATA, value: /var/lib/postgresql/data/pgdata }
          ports: [{ containerPort: 5432 }]
          readinessProbe:
            exec: { command: ["sh", "-c", "pg_isready -U $POSTGRES_USER -d $POSTGRES_DB"] }
            periodSeconds: 5
          volumeMounts: [{ name: data, mountPath: /var/lib/postgresql/data }]
  volumeClaimTemplates:
    - metadata: { name: data }
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard
        resources: { requests: { storage: 1Gi } }
```
5. **Comandos**:
```bash
kubectl apply -f lab04.yaml
kubectl get pvc,pv -n lab04
kubectl wait --for=condition=ready pod/pg-0 -n lab04 --timeout=120s

kubectl exec -it pg-0 -n lab04 -- psql -U shop -d shop \
  -c "CREATE TABLE orders(id serial primary key, total numeric); INSERT INTO orders(total) VALUES (42.50);"

kubectl delete pod pg-0 -n lab04
kubectl wait --for=condition=ready pod/pg-0 -n lab04 --timeout=120s
kubectl exec -it pg-0 -n lab04 -- psql -U shop -d shop -c "SELECT * FROM orders;"

# El Secret no está cifrado
kubectl get secret pg-secret -n lab04 -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d; echo
```
6. **Resultado esperado**: tras borrar `pg-0`, se recrea **con el mismo nombre** y la fila sigue ahí.
7. **Verificación**: `kubectl get pvc -n lab04` → `data-pg-0 Bound`; el PV tiene `RECLAIM POLICY Delete` (la de la StorageClass).
8. **Errores comunes**: PVC `Pending` si la StorageClass no existe; montar en `/var/lib/postgresql/data` sin `PGDATA` en un subdirectorio (initdb falla si hay `lost+found` en volúmenes de bloque); cambiar la contraseña en el Secret y esperar que Postgres la cambie (solo se usa en la **primera** inicialización).
9. **Limpieza**: `kubectl delete ns lab04` (borra también el PVC y, por `reclaimPolicy: Delete`, el volumen).
10. **Reflexión**: si borras el StatefulSet pero no el namespace, ¿qué pasa con el PVC? ¿Qué te falta para usar esto en producción? (backups, HA, monitorización, upgrades → operador como CloudNativePG o servicio gestionado).

---

## Lab 05 - Probes, fallos y troubleshooting

1. **Objetivo**: provocar a propósito los fallos más comunes y diagnosticarlos con el método de [[16 - Troubleshooting]].
2. **Requisitos**: clúster del Lab 01.
3. **Arquitectura**: cinco Pods rotos en el namespace `lab05`.
4. **YAML** (cada uno tiene **un error intencional**; intenta diagnosticarlo antes de leer la solución):
```yaml
# lab05.yaml
apiVersion: v1
kind: Namespace
metadata: { name: lab05 }
---
apiVersion: v1
kind: Pod
metadata: { name: broken-image, namespace: lab05 }
spec:
  containers: [{ name: app, image: nginx:1.277 }]
---
apiVersion: v1
kind: Pod
metadata: { name: broken-crash, namespace: lab05 }
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "echo 'Conectando a la BD...'; sleep 3; echo 'ERROR: DB_HOST no definido' >&2; exit 1"]
---
apiVersion: v1
kind: Pod
metadata: { name: broken-oom, namespace: lab05 }
spec:
  containers:
    - name: app
      image: polinux/stress          # imagen del ejemplo oficial de la documentación de Kubernetes
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
      resources:
        requests: { memory: 50Mi }
        limits: { memory: 100Mi }
---
apiVersion: v1
kind: Pod
metadata: { name: broken-pending, namespace: lab05 }
spec:
  containers:
    - name: app
      image: nginx:1.27
      resources: { requests: { cpu: "64", memory: 1Gi } }
---
apiVersion: v1
kind: Pod
metadata: { name: broken-probe, namespace: lab05 }
spec:
  containers:
    - name: app
      image: nginx:1.27
      readinessProbe: { httpGet: { path: /healthz, port: 80 }, periodSeconds: 3 }
      livenessProbe: { httpGet: { path: /, port: 8080 }, periodSeconds: 5, failureThreshold: 2 }
```
5. **Comandos de diagnóstico**:
```bash
kubectl apply -f lab05.yaml
kubectl get pods -n lab05 -w
kubectl describe pod <pod> -n lab05            # sección Events y Last State
kubectl logs <pod> -n lab05 --previous
kubectl get events -n lab05 --sort-by=.lastTimestamp
kubectl get pod broken-oom -n lab05 -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}'
```
6. **Resultado esperado y soluciones**:

> [!success]- `broken-image` → ImagePullBackOff
> Evento `Failed to pull image "nginx:1.277": ... not found`. Tag inexistente. Solución: `nginx:1.27`.

> [!success]- `broken-crash` → CrashLoopBackOff
> `kubectl logs --previous` muestra `ERROR: DB_HOST no definido`; `Last State: Terminated, Exit Code: 1`. Falta configuración: inyectar la variable desde un ConfigMap.

> [!success]- `broken-oom` → OOMKilled → CrashLoopBackOff
> `Last State: Terminated, Reason: OOMKilled, Exit Code: 137`. Intenta usar 250M con límite 100Mi. Subir el límite o corregir el consumo.

> [!success]- `broken-pending` → Pending
> Evento `FailedScheduling: 0/3 nodes are available: 3 Insufficient cpu`. Pide 64 CPU. Ajustar requests (o, en la nube, el Cluster Autoscaler añadiría nodos solo si existe un tipo de VM así).

> [!success]- `broken-probe` → Running pero 0/1, y reinicios
> Readiness a `/healthz` (nginx devuelve 404 → nunca Ready, sin tráfico) **y** liveness al puerto 8080 (nada escucha → reinicios cada ~10 s). Dos errores distintos con síntomas distintos: `READY 0/1` + `RESTARTS` creciendo. Corregir ruta y puerto.

7. **Verificación**: puedes explicar cada estado con una línea de `describe` o `logs`.
8. **Errores comunes**: leer `kubectl logs` sin `--previous` en un CrashLoop (ves el intento actual, vacío); mirar solo `get pods` y no los eventos.
9. **Limpieza**: `kubectl delete ns lab05`.
10. **Reflexión**: ¿qué Pods "hacen daño" al clúster además de no funcionar? (el OOM consume ciclos de reinicio; un `livenessProbe` agresivo en producción podría provocar cascadas).

---

## Lab 06 - Scheduling con taints, affinity y spread

1. **Objetivo**: dedicar un nodo, atraer Pods a él y repartir réplicas.
2. **Requisitos**: clúster del Lab 01 (nodos `lab-worker` y `lab-worker2`).
3. **Arquitectura**: `lab-worker2` dedicado a "gpu" (simulado); Deployment `web` repartido.
4. **Pasos y comandos**:
```bash
kubectl taint nodes lab-worker2 dedicated=gpu:NoSchedule
kubectl label nodes lab-worker2 dedicated=gpu

kubectl create deployment web --image=nginx:1.27 --replicas=4
kubectl get pods -l app=web -o wide          # todos en lab-worker (y quizá control-plane no: tiene taint)
```
```yaml
# lab06-gpu.yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: trainer }
spec:
  replicas: 2
  selector: { matchLabels: { app: trainer } }
  template:
    metadata: { labels: { app: trainer } }
    spec:
      tolerations: [{ key: dedicated, operator: Equal, value: gpu, effect: NoSchedule }]
      nodeSelector: { dedicated: gpu }
      containers: [{ name: t, image: busybox:1.36, command: ["sleep", "3600"] }]
---
apiVersion: apps/v1
kind: Deployment
metadata: { name: api }
spec:
  replicas: 4
  selector: { matchLabels: { app: api } }
  template:
    metadata: { labels: { app: api } }
    spec:
      tolerations: [{ key: dedicated, operator: Equal, value: gpu, effect: NoSchedule }]
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector: { matchLabels: { app: api } }
      containers: [{ name: a, image: nginx:1.27 }]
```
```bash
kubectl apply -f lab06-gpu.yaml
kubectl get pods -o wide -l 'app in (trainer,api)'
```
5. **Resultado esperado**: `web` nunca va a `lab-worker2`; `trainer` solo va a `lab-worker2`; `api` queda 2-2 entre los dos workers.
6. **Verificación**: `kubectl describe node lab-worker2 | grep Taints`; columna `NODE` de `get pods -o wide`.
7. **Experimento**: quita el `nodeSelector` de `trainer` → sus Pods pueden caer en cualquier worker (la toleration **permite**, no **obliga**). Cambia `api` a 5 réplicas con `DoNotSchedule` → reparto 3-2.
8. **Errores comunes**: olvidar que el control-plane de kind tiene taint `node-role.kubernetes.io/control-plane:NoSchedule`; confundir label y taint.
9. **Limpieza**:
```bash
kubectl delete deploy web trainer api
kubectl taint nodes lab-worker2 dedicated=gpu:NoSchedule-
kubectl label nodes lab-worker2 dedicated-
```
10. **Reflexión**: ¿por qué hacen falta taint **y** nodeSelector para dedicar nodos? ¿Qué pasaría con `required` pod anti-affinity y 5 réplicas en 2 nodos?

---

## Lab 07 - Autoescalado con HPA

1. **Objetivo**: instalar Metrics Server, crear un HPA y verlo escalar con carga.
2. **Requisitos**: clúster del Lab 01.
3. **Arquitectura**: Deployment `php-apache` (ejemplo oficial) + Service + HPA + generador de carga.
4. **Comandos y YAML**:
```bash
# Metrics Server (en kind hay que desactivar la verificación TLS del kubelet; SOLO en labs)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type=json \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
kubectl wait -n kube-system --for=condition=available deploy/metrics-server --timeout=120s
kubectl top nodes
```
```yaml
# lab07.yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: php-apache }
spec:
  selector: { matchLabels: { run: php-apache } }
  template:
    metadata: { labels: { run: php-apache } }
    spec:
      containers:
        - name: php-apache
          image: registry.k8s.io/hpa-example
          ports: [{ containerPort: 80 }]
          resources:
            requests: { cpu: 200m }
            limits: { cpu: 500m }
---
apiVersion: v1
kind: Service
metadata: { name: php-apache }
spec:
  selector: { run: php-apache }
  ports: [{ port: 80 }]
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: php-apache }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: php-apache }
  minReplicas: 1
  maxReplicas: 8
  metrics: [{ type: Resource, resource: { name: cpu, target: { type: Utilization, averageUtilization: 50 } } }]
  behavior:
    scaleDown: { stabilizationWindowSeconds: 60 }
```
```bash
kubectl apply -f lab07.yaml
kubectl get hpa php-apache -w            # terminal 1

# terminal 2: generar carga
kubectl run load-generator --rm -it --image=busybox:1.36 --restart=Never -- \
  /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
# Ctrl+C para parar la carga y observa el scale down (~1 min por la ventana de estabilización)
```
5. **Resultado esperado**: `TARGETS` sube por encima del 50 %, `REPLICAS` crece (2, 4, 6...), y vuelve a 1 al parar.
6. **Verificación**: `kubectl describe hpa php-apache` → eventos `SuccessfulRescale` con el motivo.
7. **Errores comunes**: `TARGETS <unknown>` (Metrics Server no listo o falta `requests.cpu`); esperar escalado instantáneo.
8. **Limpieza**: `kubectl delete -f lab07.yaml`.
9. **Reflexión**: con `requests.cpu: 50m`, ¿escalaría antes o después? ¿Qué pasaría si quitas `limits.cpu`? ¿Por qué no escalar por memoria en .NET?
10. **Reto**: añade una segunda métrica (memoria) y explica qué valor elige el HPA.

---

## Lab 08 - Seguridad RBAC y NetworkPolicies

1. **Objetivo**: crear una ServiceAccount con permisos mínimos y aislar Pods con NetworkPolicies.
2. **Requisitos**: **minikube** con un CNI que aplique políticas (kindnet de kind tiene soporte limitado según versión):
```bash
minikube start -p netpol --cni=calico --cpus=2 --memory=4096
kubectl config use-context netpol
```
3. **Arquitectura**: namespace `lab08` con `web` (nginx), `client` (busybox) y `intruder` (busybox).
4. **Parte A - RBAC**:
```bash
kubectl create namespace lab08
kubectl create serviceaccount reader -n lab08
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n lab08
kubectl create rolebinding reader-binding --role=pod-reader --serviceaccount=lab08:reader -n lab08

kubectl auth can-i list pods -n lab08 --as=system:serviceaccount:lab08:reader      # yes
kubectl auth can-i delete pods -n lab08 --as=system:serviceaccount:lab08:reader    # no
kubectl auth can-i list secrets -n lab08 --as=system:serviceaccount:lab08:reader   # no
kubectl auth can-i list pods -n default --as=system:serviceaccount:lab08:reader    # no

# Usar el token de la SA (caduca: token proyectado)
TOKEN=$(kubectl create token reader -n lab08 --duration=10m)
kubectl get pods -n lab08 --token="$TOKEN"
kubectl get secrets -n lab08 --token="$TOKEN"       # Forbidden
```
5. **Parte B - NetworkPolicies**:
```bash
kubectl run web --image=nginx:1.27 -n lab08 --labels=app=web --port=80
kubectl expose pod web -n lab08 --port=80
kubectl run client --image=busybox:1.36 -n lab08 --labels=role=client -- sleep 3600
kubectl run intruder --image=busybox:1.36 -n lab08 --labels=role=intruder -- sleep 3600
kubectl exec -n lab08 intruder -- wget -qO- -T 3 http://web     # funciona: red abierta por defecto
```
```yaml
# lab08-netpol.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: default-deny-ingress, namespace: lab08 }
spec:
  podSelector: {}
  policyTypes: [Ingress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: web-allow-client, namespace: lab08 }
spec:
  podSelector: { matchLabels: { app: web } }
  policyTypes: [Ingress]
  ingress:
    - from: [{ podSelector: { matchLabels: { role: client } } }]
      ports: [{ protocol: TCP, port: 80 }]
```
```bash
kubectl apply -f lab08-netpol.yaml
kubectl exec -n lab08 client -- wget -qO- -T 3 http://web      # OK
kubectl exec -n lab08 intruder -- wget -qO- -T 3 http://web    # timeout
```
6. **Resultado esperado**: la SA solo lee Pods de su namespace; `intruder` deja de llegar a `web`.
7. **Verificación**: salidas `yes`/`no` de `auth can-i`; `wget` con timeout para `intruder`.
8. **Errores comunes**: probar en un CNI sin soporte de políticas (todo sigue abierto); aplicar *default deny* de egress y romper el DNS.
9. **Limpieza**: `minikube delete -p netpol` y `kubectl config use-context kind-lab`.
10. **Reflexión**: añade una política de *egress* *default deny* y comprueba que `client` ya no resuelve `web`. ¿Qué regla falta? (DNS a `kube-dns`, ver [[04 - Networking y tráfico#NetworkPolicy]]).

---

## Lab 09 - Observabilidad con kube-prometheus-stack

1. **Objetivo**: instalar Prometheus + Grafana + Alertmanager, explorar dashboards y escribir consultas PromQL.
2. **Requisitos**: clúster del Lab 01, Helm, ~4 GB libres.
3. **Arquitectura**: chart `kube-prometheus-stack` en el namespace `monitoring`.
4. **Comandos**:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kps prometheus-community/kube-prometheus-stack -n monitoring --create-namespace \
  --set grafana.adminPassword=admin-lab
kubectl get pods -n monitoring -w

kubectl port-forward -n monitoring svc/kps-grafana 3000:80          # http://localhost:3000 (admin / admin-lab)
kubectl port-forward -n monitoring svc/kps-kube-prometheus-stack-prometheus 9090:9090   # http://localhost:9090
```
5. **Ejercicios**:
   - En Grafana: dashboard *Kubernetes / Compute Resources / Namespace (Pods)*.
   - En Prometheus, consulta: `sum(rate(container_cpu_usage_seconds_total{namespace="kube-system"}[5m])) by (pod)`.
   - Repite el Lab 05 (`broken-oom`) y busca: `increase(kube_pod_container_status_restarts_total[10m]) > 0`.
   - Mira las alertas precargadas en *Alerting* (p. ej. `KubePodCrashLooping`).
6. **Resultado esperado**: dashboards con datos de nodos, Pods y namespaces; la alerta de CrashLooping se activa tras unos minutos con el Pod roto.
7. **Verificación**: `kubectl get servicemonitors -n monitoring`; *Status → Targets* en Prometheus todos `UP` (algunos del Control Plane pueden aparecer `DOWN` en kind: es esperable).
8. **Errores comunes**: nombre del Service distinto si cambias el nombre del release (`kubectl get svc -n monitoring`); falta de memoria en Docker Desktop.
9. **Limpieza**: `helm uninstall kps -n monitoring && kubectl delete ns monitoring` (las CRDs de Prometheus Operator quedan: `kubectl get crd | grep monitoring.coreos.com`).
10. **Reflexión**: ¿qué alertas despertarían a alguien de madrugada y cuáles no deberían? ¿Cómo añadirías las métricas de tu propia API? (ServiceMonitor, [[10 - Observabilidad#Métricas con Prometheus y Grafana]]).

---

## Lab 10 - Tu propio chart de Helm

1. **Objetivo**: crear, parametrizar, instalar, actualizar y revertir un chart.
2. **Requisitos**: clúster del Lab 01, Helm.
3. **Arquitectura**: chart `web` con Deployment, Service y ConfigMap con checksum.
4. **Comandos**:
```bash
helm create web
# Edita web/values.yaml:
#   image.repository: nginx   image.tag: "1.27"   replicaCount: 2
# Añade web/templates/configmap.yaml:
cat > web/templates/configmap.yaml <<'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "web.fullname" . }}
data:
  MESSAGE: {{ .Values.message | default "hola" | quote }}
EOF
# En web/templates/deployment.yaml, bajo spec.template.metadata, añade:
#   annotations:
#     checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}

helm lint ./web
helm template demo ./web | less
helm upgrade --install demo ./web -n lab10 --create-namespace --wait
helm upgrade demo ./web -n lab10 --set message="nuevo mensaje" --wait    # Pods recreados por el checksum
helm upgrade demo ./web -n lab10 --set image.tag=9.99 --wait --timeout 60s --atomic   # falla y revierte solo
helm history demo -n lab10
helm rollback demo 1 -n lab10
helm test demo -n lab10            # el chart generado incluye un test de conexión
```
5. **Resultado esperado**: revisión 2 con el mensaje nuevo y Pods recreados; el upgrade con imagen rota falla y `--atomic` lo revierte.
6. **Verificación**: `helm history` muestra estados `deployed`, `superseded`, `failed`/`rolled back`.
7. **Errores comunes**: indentación en plantillas (usa `nindent`); valores con tipos incorrectos (`"1.27"` como string, no número).
8. **Limpieza**: `helm uninstall demo -n lab10 && kubectl delete ns lab10`.
9. **Reflexión**: ¿qué cambiarías para tener `values-dev.yaml` y `values-prod.yaml`? ¿Cómo empaquetarías el chart en un registro OCI?
10. **Reto**: añade un HPA condicional (`{{- if .Values.autoscaling.enabled }}`) y elimina `replicas` del Deployment cuando esté activo.

---

## Lab 11 - API .NET en kind

1. **Objetivo**: llevar una API ASP.NET Core al clúster con health checks, probes, configuración y apagado ordenado.
2. **Requisitos**: .NET SDK 10, Docker Desktop, clúster del Lab 01.
3. **Arquitectura**: `shop-api` (Deployment 2 réplicas + Service + ConfigMap), acceso con `port-forward`.
4. **Pasos**:
```bash
dotnet new webapi -n Shop.Api -o shop-api
cd shop-api
```
```csharp
// Program.cs (añadir a lo generado)
using Microsoft.Extensions.Diagnostics.HealthChecks;
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy(), tags: ["live"]);
builder.Services.Configure<HostOptions>(o => o.ShutdownTimeout = TimeSpan.FromSeconds(20));
builder.Logging.AddJsonConsole();
// ... después de var app = builder.Build();
app.MapHealthChecks("/health/live", new() { Predicate = r => r.Tags.Contains("live") });
app.MapHealthChecks("/health/ready");
app.MapGet("/info", (IConfiguration cfg) => new {
    Pod = Environment.GetEnvironmentVariable("POD_NAME"),
    Message = cfg["Shop:Message"]
});
```
```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:10.0-noble-chiseled
WORKDIR /app
COPY --from=build /app .
USER $APP_UID
EXPOSE 8080
ENTRYPOINT ["dotnet", "Shop.Api.dll"]
```
```bash
docker build -t shop-api:0.1 .
kind load docker-image shop-api:0.1 --name lab       # copia la imagen a los nodos de kind (sin registro)
```
```yaml
# k8s.yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: shop-api-config }
data:
  Shop__Message: "Hola desde Kubernetes"
  ASPNETCORE_ENVIRONMENT: Production
---
apiVersion: apps/v1
kind: Deployment
metadata: { name: shop-api }
spec:
  replicas: 2
  selector: { matchLabels: { app: shop-api } }
  template:
    metadata: { labels: { app: shop-api } }
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - name: api
          image: shop-api:0.1
          imagePullPolicy: IfNotPresent
          ports: [{ name: http, containerPort: 8080 }]
          envFrom: [{ configMapRef: { name: shop-api-config } }]
          env:
            - name: POD_NAME
              valueFrom: { fieldRef: { fieldPath: metadata.name } }
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits: { memory: 256Mi }
          startupProbe:   { httpGet: { path: /health/live,  port: http }, periodSeconds: 2, failureThreshold: 15 }
          readinessProbe: { httpGet: { path: /health/ready, port: http }, periodSeconds: 5 }
          livenessProbe:  { httpGet: { path: /health/live,  port: http }, periodSeconds: 10 }
          lifecycle: { preStop: { sleep: { seconds: 5 } } }
---
apiVersion: v1
kind: Service
metadata: { name: shop-api }
spec:
  selector: { app: shop-api }
  ports: [{ port: 80, targetPort: http }]
```
```bash
kubectl apply -f k8s.yaml
kubectl rollout status deploy/shop-api
kubectl port-forward svc/shop-api 8080:80
curl localhost:8080/info               # repite: el Pod cambia según la conexión
curl localhost:8080/weatherforecast
kubectl logs -l app=shop-api --prefix  # logs JSON

# Cambio de configuración: las variables no se refrescan solas
kubectl patch configmap shop-api-config -p '{"data":{"Shop__Message":"Mensaje nuevo"}}'
kubectl rollout restart deploy/shop-api
```
5. **Resultado esperado**: 2 Pods Ready; `/info` devuelve el nombre del Pod y el mensaje del ConfigMap; tras el `rollout restart`, el mensaje nuevo.
6. **Verificación**: `kubectl get pods -l app=shop-api` → `1/1 Running`; `kubectl describe pod` sin eventos de probes fallidas.
7. **Errores comunes**: `ErrImagePull` por no ejecutar `kind load` o por `imagePullPolicy: Always`; probes al puerto 80 (la app escucha en 8080); `port-forward` siempre conecta al mismo Pod (es una conexión a un único Pod: para ver el balanceo, usa un Pod cliente dentro del clúster).
8. **Limpieza**: `kubectl delete -f k8s.yaml`. Fin de los labs: `kind delete cluster --name lab`.
9. **Reflexión**: ¿qué falta para producción? (registro, Helm, secretos, Data Protection, forwarded headers, HPA, PDB, NetworkPolicy, OTel → [[13 - Kubernetes para .NET]]).
10. **Reto**: añade Redis (Deployment `redis:7-alpine` + Service) y usa `AddStackExchangeRedisCache` con `Redis__Endpoint` desde el ConfigMap; añade su health check en `ready`.

### Relacionado
- [[16 - Troubleshooting]] · [[20 - Proyecto final]] · [[00 - Kubernetes - Índice]]
