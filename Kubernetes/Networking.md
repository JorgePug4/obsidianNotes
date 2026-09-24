---
tags:
  - kubernetes
  - networking
aliases:
  - Kubernetes Service
  - Service
  - SVC
  - K8s Service
  - ClusterIP
  - Cluster IP
  - Internal Service
  - NodePort
  - Node Port
  - NodePort Service
  - LoadBalancer
  - Load Balancer Service
  - External LoadBalancer
  - Headless Service
  - Headless Services
  - None ClusterIP
  - Headless SVC
  - Ingress
  - Ingress Resource
  - K8s Ingress
  - Ingress API
  - Ingress Controller
  - Ingress Controllers
  - NGINX Ingress Controller
  - ALB Ingress Controller
  - Kubernetes Gateway API
  - Gateway API
  - GatewayClass
  - HTTPRoute
  - K8s Gateway API
  - Kube-proxy
  - Kube Proxy
  - Cube proxy
  - Service Mesh
  - Mesh de Servicios
  - Istio
  - Envoy Proxy
  - CNI
  - Container Network Interface
  - CNI Plugin
---

# Networking

- [[#Kubernetes Service]]
- [[#ClusterIP]]
- [[#NodePort]]
- [[#LoadBalancer]]
- [[#Headless Service]]
- [[#Ingress]]
- [[#Ingress Controller]]
- [[#Kubernetes Gateway API]]
- [[#Kube-proxy]]
- [[#Service Mesh]]
- [[#CNI]]

---

## Kubernetes Service

### Qué es
Un **Kubernetes Service** (abreviado frecuentemente como `svc`) es un recurso y objeto de abstracción en Kubernetes que actúa como un proxy frente a un conjunto de [[Workloads#Pod|Pod]]s. Proporciona una IP estable y un nombre DNS fijo que abstrae el acceso a la aplicación.

### Para qué sirve
Resuelve el problema de la naturaleza efímera e IP cambiante de los Pods cuando son recreados o escalados, ofreciendo tres funciones fundamentales:
1. **Service Discovery (Descubrimiento de Servicios)**: Rastrea e identifica dinámicamente los Pods utilizando etiquetas y selectores (`labels` y `selectors`) en lugar de depender de direcciones IP fijas.
2. **Balanceo de carga (Load Balancing)**: Distribuye las peticiones entrantes de manera equitativa entre las réplicas activas de los Pods (manejado internamente a nivel de red por [[Networking#Kube-proxy|Kube-proxy]]).
3. **Exposición de aplicaciones**: Permite definir el alcance de acceso a las aplicaciones, ya sea internamente dentro del clúster o externamente hacia la organización o el mundo.

### Ejemplo

**Comandos imperativos de gestión:**
```bash
# Consultar los servicios en el namespace actual
kubectl get svc

# Inspeccionar información detallada de un servicio
kubectl describe svc python-django-app-service

# Editar la configuración de un servicio activo
kubectl edit svc python-django-app-service

# Eliminar un servicio
kubectl delete svc python-django-app-service
```

**Definición declarativa en YAML (`service.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-django-app-service
spec:
  selector:
    app: sample-python-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
```

### Relacionado
- [[Workloads#Pod|Pod]]
- [[Networking#ClusterIP|ClusterIP]]
- [[Networking#NodePort|NodePort]]
- [[Networking#LoadBalancer|LoadBalancer]]
- [[Networking#Headless Service|Headless Service]]
- [[Networking#Ingress|Ingress]]
- [[Networking#Kube-proxy|Kube-proxy]]

---

## ClusterIP

### Qué es
**ClusterIP** es el tipo de [[Networking#Kubernetes Service|Kubernetes Service]] por defecto (*default type*). Asigna una dirección IP virtual interna que únicamente es alcanzable desde dentro del clúster de Kubernetes.

### Para qué sirve
Sirve para habilitar la comunicación y el balanceo de carga interno entre los distintos microservicios o [[Workloads#Pod|Pod]]s dentro del clúster (por ejemplo, la comunicación entre un servicio frontend y su backend o base de datos). Impide de forma predeterminada que usuarios u origen de tráfico externos a la red del clúster puedan acceder al servicio.

### Ejemplo

**Comandos útiles:**
```bash
# Consultar los servicios y verificar la dirección IP interna asignada
kubectl get svc
```

**Definición declarativa en YAML (`clusterip-service.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-internal-service
spec:
  type: ClusterIP
  selector:
    app: sample-python-app
  ports:
    - port: 80
      targetPort: 8000
```

### Relacionado
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Networking#NodePort|NodePort]]
- [[Networking#LoadBalancer|LoadBalancer]]
- [[Workloads#Pod|Pod]]
- [[Networking#Kube-proxy|Kube-proxy]]

---

## NodePort

### Qué es
**NodePort** es un tipo de [[Networking#Kubernetes Service|Kubernetes Service]] que expone la aplicación mapeando un puerto específico e idéntico en la dirección IP de cada uno de los nodos de trabajo (*Worker Nodes*) del clúster.

### Para qué sirve
Permite que la aplicación sea accesible desde fuera del clúster a cualquier usuario o sistema que tenga acceso a la red de los nodos o a sus direcciones IP (como las instancias EC2 o máquinas virtuales en una VPC/red corporativa). Redirige el tráfico recibido en el puerto del nodo (`nodePort`) hacia el puerto del servicio (`port`), y de ahí al puerto del contenedor (`targetPort`).

> [!warning] Rango de puertos predefinido y seguridad
> El valor del campo `nodePort` requiere obligatoriamente estar dentro del rango estricto entre **30000 y 32767**. Además, abrir puertos directamente en los nodos no se considera la opción más eficiente ni segura para entornos de producción, ya que expone los *Worker Nodes* a la red directa.

### Ejemplo

**Comandos de prueba:**
```bash
# Obtener la dirección IP del nodo de Kubernetes (ejemplo en Minikube)
minikube ip

# Acceder al servicio desde la terminal mediante la IP del nodo y el NodePort
curl -L http://<NODE-IP>:30007/demo
```

**Definición declarativa en YAML (`nodeport-service.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-django-app-service
spec:
  type: NodePort
  selector:
    app: sample-python-app
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

### Relacionado
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Networking#ClusterIP|ClusterIP]]
- [[Networking#LoadBalancer|LoadBalancer]]
- [[Networking#Ingress|Ingress]]
- [[Networking#Kube-proxy|Kube-proxy]]

---

## LoadBalancer

### Qué es
**LoadBalancer** es un tipo de [[Networking#Kubernetes Service|Kubernetes Service]] que expone la aplicación públicamente al mundo exterior conectándose e integrándose con el balanceador de carga nativo del proveedor de nube (*Cloud Provider*), como AWS (ELB/ALB), GCP o Azure.

### Para qué sirve
Genera una dirección IP pública estática y externa mediante la intervención del componente `Cloud Controller Manager`, permitiendo que cualquier usuario desde Internet pueda acceder a la aplicación.

> [!warning] Comportamiento en clústeres locales o Minikube
> El tipo `LoadBalancer` solo funciona de forma nativa en proveedores de nube públicos. Si se aplica en entornos locales como Minikube o KIND sin controladores adicionales (como MetalLB), la columna de IP externa permanecerá de forma indefinida en estado `<pending>`.

> [!warning] Costo financiero por balanceador
> Los proveedores de nube cobran una tarifa por cada balanceador de carga e IP pública estática provisionada. Crear un servicio de tipo `LoadBalancer` individual para cada microservicio en arquitecturas con decenas o cientos de aplicaciones incrementa considerablemente los costos. En esos escenarios se recomienda utilizar [[Networking#Ingress|Ingress]].

### Ejemplo

**Comandos útiles:**
```bash
# Cambiar un servicio de tipo NodePort a LoadBalancer
kubectl edit svc python-django-app-service

# Verificar la asignación de la dirección IP pública externa
kubectl get svc
```

**Definición declarativa en YAML (`loadbalancer-service.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: sample-python-app
  ports:
    - port: 80
      targetPort: 8000
```

### Relacionado
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Networking#ClusterIP|ClusterIP]]
- [[Networking#NodePort|NodePort]]
- [[Networking#Ingress|Ingress]]
- [[Cloud Controller Manager]]

---

¿Te gustaría proceder con las siguientes notas del área de **Networking** (como `[[Networking#Headless Service|Headless Service]]`, `[[Networking#Ingress|Ingress]]`, `[[Networking#Ingress Controller|Ingress Controller]]`, `[[Networking#Kube-proxy|Kube-proxy]]`, `[[Networking#Service Mesh|Service Mesh]]`, `[[Networking#CNI|CNI]]`) o prefieres cambiar a otra categoría?
Aquí tienes las notas completas en formato Markdown para tu bóveda de Obsidian, correspondientes a los conceptos solicitados del área de **Networking**: **Headless Service**, **Ingress**, **Ingress Controller** y **Kubernetes Gateway API**.

---

## Headless Service

### Qué es
Un **Headless Service** es un tipo de [[Networking#Kubernetes Service|Kubernetes Service]] que se define explícitamente asignando el campo `clusterIP` con el valor `None` (`clusterIP: None`). A diferencia de los servicios convencionales, no se le asigna una dirección IP virtual fija ni realiza proxying o balanceo de carga aleatorio entre réplicas.

### Para qué sirve
En lugar de redirigir el tráfico a través de una IP de clúster, un Headless Service utiliza el servidor DNS interno de Kubernetes para devolver directamente los registros A (*DNS A records*) que contienen las direcciones IP individuales de cada [[Workloads#Pod|Pod]] asociado. Es fundamental al desplegar aplicaciones con estado ([[Workloads#StatefulSet|StatefulSet]]) como bases de datos (MySQL, Redis, MongoDB), donde los clientes o las distintas réplicas de los Pods necesitan comunicarse directamente con una réplica específica (por ejemplo, conectar un nodo maestro de lectura/escritura o sincronizar datos entre nodos primarios y secundarios) utilizando nombres de dominio DNS individuales asignados por Pod.

> [!tip] Orden de creación recomendado
> Debido a que las aplicaciones con estado y sus Pods requieren resolver sus nombres de dominio individuales para inicializarse y sincronizarse correctamente, es una buena práctica crear el manifiesto del Headless Service antes de desplegar el [[Workloads#StatefulSet|StatefulSet]].

### Ejemplo

**Definición declarativa en YAML (`headless-svc.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
```

**Verificación y prueba de resolución DNS por Pod:**
```bash
# Crear el Headless Service y verificar que no tiene ClusterIP asignada
kubectl apply -f headless-svc.yaml
kubectl get svc

# Probar la resolución DNS de un Pod específico dentro de un StatefulSet
kubectl run -it busybox --image=busybox -- nslookup mysql-0.my-db-headless.default.svc.cluster.local
```

### Relacionado
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Workloads#StatefulSet|StatefulSet]]
- [[Workloads#Pod|Pod]]
- [[Networking#ClusterIP|ClusterIP]]

---

## Ingress

### Qué es
**Ingress** es un recurso y objeto API nativo de Kubernetes (perteneciente al grupo `networking.k8s.io`) que actúa como la capa de enrutamiento y punto de entrada centralizado para administrar el tráfico HTTP y HTTPS proveniente desde fuera del clúster hacia los servicios internos ([[Networking#Kubernetes Service|Kubernetes Service]]).

### Para qué sirve
Resuelve el problema de costos y complejidad que genera exponer aplicaciones mediante [[Networking#LoadBalancer|LoadBalancer]] (donde los proveedores de nube cobran por cada IP estática pública creada) o [[Networking#NodePort|NodePort]] (que expone puertos directos en los nodos). Ingress permite gestionar múltiples aplicaciones y microservicios mediante una sola dirección IP externa pública, proporcionando:
- **Enrutamiento por rutas de contexto** (*path-based routing*, ej. `domain.com/app1` frente a `domain.com/app2`).
- **Enrutamiento por nombres de dominio** (*host-based routing*, ej. `app1.domain.com` frente a `app2.domain.com`).
- **Terminación y seguridad SSL/TLS** (soporta esquemas de *offloading*, *passthrough* y *bridging/re-encrypt* respaldados por objetos [[Secret]]).
- **Mapeo por comodines** (*wildcard hosts*).

> [!warning] Requisito de un Ingress Controller activo
> Crear únicamente el archivo de manifiesto YAML del recurso `Ingress` no realiza ninguna acción de enrutamiento de red por sí solo. El campo de dirección IP del Ingress permanecerá vacío indefinidamente a menos que exista un [[Networking#Ingress Controller|Ingress Controller]] instalado en el clúster para leer y aplicar la configuración.

> [!warning] Limitaciones de la especificación nativa de Ingress
> La especificación estándar del objeto Ingress solo soporta de forma nativa la asignación de servicio y ruta. Para implementar capacidades avanzadas (como *rate limiting*, WAF, reescritura de URLs o despliegues *Canary*), los controladores tuvieron que recurrir a un uso extensivo y complejo de anotaciones en formato JSON, lo que derivó en la creación de la [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]].

### Ejemplo

**Definición declarativa en YAML (`ingress.yaml`):**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
spec:
  ingressClassName: nginx
  rules:
    - host: foo.bar.com
      http:
        paths:
          - path: /bar
            pathType: Prefix
            backend:
              service:
                name: my-service-name
                port:
                  number: 80
```

**Comandos de inspección y prueba:**
```bash
# Aplicar y consultar el Ingress
kubectl apply -f ingress.yaml
kubectl get ingress
kubectl describe ingress ingress-example

# Probar el acceso pasando la cabecera de host simulada
curl -H "Host: foo.bar.com" http://<INGRESS-IP>/bar
```

### Relacionado
- [[Networking#Ingress Controller|Ingress Controller]]
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Networking#LoadBalancer|LoadBalancer]]
- [[Networking#NodePort|NodePort]]
- [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]]
- [[Secret]]

---

## Ingress Controller

### Qué es
Un **Ingress Controller** es una aplicación o controlador en ejecución (desplegado habitualmente como un [[Workloads#Pod|Pod]] o conjunto de Pods dentro del clúster, o integrado con balanceadores externos como NGINX, HAProxy, AWS ALB Controller, F5 o Ambassador) que monitorea de forma continua el servidor de la API de Kubernetes en busca de objetos [[Networking#Ingress|Ingress]].

### Para qué sirve
Sirve como el componente ejecutor que traduce las reglas lógicas declaradas en los manifiestos [[Networking#Ingress|Ingress]] y las evalúa en tiempo real para configurar y actualizar el software proxy o el balanceador de carga subyacente (por ejemplo, actualizando dinámicamente el archivo de configuración `nginx.conf` o las reglas del balanceador de la nube). Sin un Ingress Controller activo, los objetos Ingress no tienen ningún efecto en el clúster.

### Ejemplo

**Comandos de instalación y diagnóstico:**
```bash
# Habilitar el add-on del NGINX Ingress Controller en Minikube
minikube add-ons enable ingress

# Verificar los Pods en ejecución del Ingress Controller
kubectl get pods -n ingress-nginx

# Revisar los logs del Ingress Controller para confirmar la sincronización de reglas
kubectl logs -n ingress-nginx <nombre-del-pod-ingress-controller>
```

### Relacionado
- [[Networking#Ingress|Ingress]]
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]]
- [[Helm]]
- [[Custom Controller]]

---

## Kubernetes Gateway API

### Qué es
La **Kubernetes Gateway API** es un estándar y conjunto de recursos personalizados ([[Custom Resource Definition]]) que evolucionan y mejoran el concepto de [[Networking#Ingress|Ingress]] para la gestión del tráfico de red de entrada (norte-sur).

### Para qué sirve
Supera las deficiencias arquitectónicas de [[Networking#Ingress|Ingress]] al estructurar el enrutamiento en tres objetos con responsabilidades y roles claramente separados:
1. **`GatewayClass`**: Definido por el administrador de la infraestructura/plataforma para especificar qué controlador proxy se utiliza.
2. **`Gateway`**: Definido por los administradores del clúster para establecer los puntos de escucha (*listeners*), puertos y certificados TLS.
3. **`HTTPRoute`**: Definido por los ingenieros DevOps o desarrolladores para declarar las reglas de enrutamiento hacia los servicios.

Sustituye la necesidad de usar anotaciones complejas y propietarias al ofrecer soporte nativo para funciones avanzadas de producción como la división de tráfico por peso (*weight-based traffic splitting / Canary*), reescritura de URLs (*URL rewrite*), redirecciones de tráfico, *rate limiting* y filtrado de cabeceras.

### Ejemplo

**Definición declarativa en YAML con división de tráfico por peso (`gateway-setup.yaml`):**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg-gateway-class
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: eg-gateway
spec:
  gatewayClassName: eg-gateway-class
  listeners:
    - name: http
      protocol: HTTP
      port: 80
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-app-route
spec:
  parentRefs:
    - name: eg-gateway
  hostnames:
    - "backends.example"
  rules:
    - backendRefs:
        - name: backend-service-1
          port: 3000
          weight: 80
        - name: backend-service-2
          port: 3000
          weight: 20
```

**Comandos de gestión:**
```bash
# Aplicar las definiciones de la Gateway API
kubectl apply -f gateway-setup.yaml

# Consultar los recursos de la Gateway API
kubectl get gatewayclass,gateway,httproute
```

### Relacionado
- [[Networking#Ingress|Ingress]]
- [[Networking#Ingress Controller|Ingress Controller]]
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Custom Resource Definition]]
- [[Custom Controller]]

---

¿Te gustaría continuar con las notas del siguiente grupo de conceptos (como los de **Storage**: `[[Volume]]`, `[[PersistentVolume]]`, `[[PersistentVolumeClaim]]`, `[[StorageClass]]`, `[[CSI Driver]]`) o prefieres seleccionar otro tema?
Aquí tienes las notas completas en formato Markdown para tu bóveda de Obsidian, correspondientes a los tres conceptos solicitados del área de **Networking**: **Kube-proxy**, **Service Mesh** y **CNI**.

---

## Kube-proxy

### Qué es
**Kube-proxy** es un componente de red fundamental que se ejecuta como un proceso o Pod en cada uno de los nodos del clúster (tanto en el [[Control Plane]] como en los [[Worker Node]]s).

### Para qué sirve
Es el encargado de configurar y mantener las reglas de red y enrutamiento en el núcleo (*kernel*) de cada nodo (utilizando principalmente `iptables` en sistemas Linux, o en su defecto `ipvs`). Permite la comunicación de red entre los distintos [[Workloads#Pod|Pod]]s y gestiona el balanceo de carga básico por defecto (*round-robin*). Cuando un usuario crea un objeto de tipo [[Networking#Kubernetes Service|Kubernetes Service]] (como un [[Networking#ClusterIP|ClusterIP]] o [[Networking#NodePort|NodePort]]), `kube-proxy` entiende esta configuración y actualiza las tablas IP para redirigir automáticamente el tráfico enviado a la dirección IP o puerto del servicio hacia las direcciones IP dinámicas de los Pods de respaldo. Además, incluye lógica para priorizar el enrutamiento hacia los Pods que se ejecuten en el mismo nodo local a fin de reducir la sobrecarga de red.

### Ejemplo

**Comandos para verificar el estado y los componentes de Kube-proxy:**
```bash
# Listar los pods de la red del sistema (donde corre una réplica de kube-proxy por cada nodo)
kubectl get pods -n kube-system -o wide

# Inspeccionar logs de kube-proxy para diagnosticar reglas de iptables o red
kubectl logs -n kube-system <pod-kube-proxy-nombre>

# Consultar la configuración del servicio mapeado por kube-proxy
kubectl get svc
```

### Relacionado
- [[Networking#Kubernetes Service|Kubernetes Service]]
- [[Networking#ClusterIP|ClusterIP]]
- [[Networking#NodePort|NodePort]]
- [[Worker Node]]
- [[Control Plane]]
- [[Workloads#Pod|Pod]]
- [[Networking#CNI|CNI]]

---

## Service Mesh

### Qué es
Una **Service Mesh** (malla de servicios) es una capa de infraestructura dedicada a gestionar, asegurar y monitorear la comunicación directa de servicio a servicio dentro del clúster de Kubernetes, también conocida como tráfico este-oeste (*east-west traffic*). Herramientas populares como Istio se integran en Kubernetes extendiendo la API mediante [[Custom Resource Definition]] y [[Custom Controller]]s.

### Para qué sirve
Añade capacidades avanzadas de red y seguridad a las aplicaciones sin necesidad de modificar su código fuente:
1. **Seguridad mediante Mutual TLS (mTLS)**: Autentica y cifra de forma bidireccional la comunicación entre microservicios usando certificados digitales gestionados automáticamente.
2. **Gestión avanzada de tráfico**: Facilita despliegues progresivos como *Canary*, *Blue-Green* o pruebas A/B dividiendo el tráfico por porcentajes o pesos entre versiones mediante objetos como `VirtualService` y `DestinationRule`.
3. **Observabilidad e inspección**: Registra métricas de tráfico y permite visualizar la topología de la red mediante herramientas como Kiali.
4. **Resiliencia**: Proporciona funciones de cortacircuitos (*circuit breaking*), límites de tasa y reintentos automáticos.

Funciona inyectando un contenedor proxy secundario (*sidecar container*, habitualmente Envoy Proxy) dentro del mismo [[Workloads#Pod|Pod]] de la aplicación mediante un controlador de admisión dinámico (*Mutating Admission Webhook*).

> [!warning] Impacto en recursos y latencia
> La inyección de un contenedor *sidecar* en cada Pod intercepta todo el tráfico entrante y saliente. Esto puede añadir latencia adicional a las peticiones (debido al procesamiento y validación de cifrado) y consume recursos de memoria y CPU adicionales en cada nodo del clúster.

### Ejemplo

**Comandos de instalación y habilitación de Istio Service Mesh:**
```bash
# Agregar repositorio e instalar los CRDs e infraestructura de Istio con Helm
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

# Instalación mediante istioctl usando el perfil demo
istioctl install --set profile=demo -y

# Habilitar la inyección automática de sidecars en un namespace
kubectl label namespace default istio-injection=enabled

# Consultar los recursos de la Service Mesh (Virtual Services y Destination Rules)
kubectl get virtualservices
kubectl get destinationrules
```

### Relacionado
- [[Workloads#Pod|Pod]]
- [[Custom Resource Definition]]
- [[Custom Controller]]
- [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]]
- [[Networking#Ingress|Ingress]]
- [[Helm]]

---

## CNI

### Qué es
**CNI** (Container Network Interface) es una especificación y software de red externo a Kubernetes que se encarga de habilitar y gestionar la conectividad de red de los [[Workloads#Pod|Pod]]s en el clúster.

### Para qué sirve
Dado que ni Kubernetes ni el motor de ejecución de contenedores ([[Container Runtime]] como Docker) tienen la capacidad nativa de gestionar asignaciones complejas de red a gran escala, el plugin CNI es el software responsable de:
1. Asignar direcciones IP únicas a cada [[Workloads#Pod|Pod]] desde un rango o bloque de red asignado a los nodos.
2. Garantizar que todos los Pods puedan comunicarse entre sí a través del clúster (incluso si están en diferentes nodos de trabajo) sin necesidad de NAT.

Ejemplos de softwares CNI mencionados en las fuentes incluyen **Calico**, **WeaveNet**, **Flannel** y plugins nativos de nube como AWS VPC CNI.

> [!question] Revisar: Ejemplo de manifiesto YAML para la instalación de un CNI (las fuentes mencionan el uso de WeaveNet, Calico, Flannel y AWS VPC CNI, pero no proporcionan el manifiesto YAML completo de instalación del plugin).

### Ejemplo

**Comandos de verificación y diagnóstico de la red CNI:**
```bash
# Consultar los pods del sistema para verificar el estado del plugin CNI (ej. Calico o WeaveNet)
kubectl get pods -n kube-system -o wide

# Verificar la conectividad directa por IP asignada por el CNI a un Pod
curl http://<POD-IP>:80
```

### Relacionado
- [[Workloads#Pod|Pod]]
- [[Worker Node]]
- [[Networking#Kube-proxy|Kube-proxy]]
- [[Container Runtime]]
- [[Networking#Kubernetes Service|Kubernetes Service]]
