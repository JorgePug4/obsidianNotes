---
tags:
  - docker
  - orchestration
aliases:
  - Docker Swarm
  - Swarm
  - Swarm mode
  - docker service
  - docker stack
  - Stack
---

# Docker Swarm

- [[#Docker Swarm]]
- [[#Servicios]]
- [[#Escalado, actualizaciones y rollback]]
- [[#Stacks]]

---

## Docker Swarm

### Qué es
**Docker Swarm** (*Swarm mode*) es el **orquestador de contenedores integrado en Docker**. Viene instalado pero **desactivado**; al activarlo, convierte uno o varios hosts Docker en un **clúster** gobernado de forma centralizada.

Tipos de nodo:
- **Manager**: gestiona el estado del clúster y **reparte el trabajo** entre los nodos. El primer nodo (`swarm init`) es manager.
- **Worker**: ejecuta las tareas (contenedores) que le asignan los managers. Se une con un `join token`.

### Para qué sirve
Superar las limitaciones de [[Docker Compose]] (un solo servidor, sin réplicas ni actualizaciones sin corte):
- **Réplicas y balanceo de carga** entre contenedores.
- **Escalado** horizontal en caliente.
- **Actualizaciones progresivas** (*rolling updates*) y **rollback** sin perder disponibilidad.
- **Alta disponibilidad**: si un contenedor o un nodo cae, Swarm lo recrea.
- Distribución de la carga entre **varios servidores**.

Es un **punto intermedio** entre lanzar contenedores "a pelo" y Kubernetes: mucho más sencillo, con la misma sintaxis que Compose, suficiente para aplicaciones que no necesitan la escala de Kubernetes.

| | Docker Compose | Docker Swarm | Kubernetes |
|---|---|---|---|
| Nodos | Uno | Varios | Varios (miles) |
| Unidad | Contenedor | Servicio (tareas replicadas) | Pod / Deployment |
| Réplicas, rolling update, rollback | No | Sí | Sí |
| Secrets nativos | Solo de fichero | Sí, cifrados en el clúster | Sí ([[Secret]]) |
| Complejidad | Muy baja | Baja | Alta |
| Ecosistema | — | Reducido | Enorme ([[Helm]], operadores, CRDs...) |

### Ejemplo
```bash
docker swarm init                     # activa Swarm; este nodo pasa a ser manager
# Swarm initialized: current node (...) is now a manager.
# To add a worker to this swarm, run the following command:
#   docker swarm join --token SWMTKN-1-... 192.168.1.10:2377

docker swarm join-token worker        # volver a ver el comando de unión
docker node ls                        # nodos del clúster

docker swarm leave --force            # desactivar Swarm en este nodo
```

### Relacionado
- [[Control Plane]]
- [[Worker Node]]

---

## Servicios

### Qué es
En Swarm no se trabaja con contenedores sueltos sino con **servicios**: la definición del estado deseado (imagen, réplicas, puertos...). Swarm crea las **tareas** (contenedores) necesarias y las reparte por los nodos.

### Para qué sirve
Desplegar una aplicación con **varias réplicas** de forma declarativa. Al publicar un puerto, Swarm usa su **routing mesh**: cualquier petición al puerto del host se **balancea** entre todas las réplicas (en un `curl` repetido verás responder a la réplica 1, 5, 2, 4, 3...).

> [!info] Publicar puertos en servicios
> En servicios se usa `--publish` (o `-p`) con la forma `publicado:destino`. Todas las réplicas escuchan en el mismo puerto interno (80) y el host expone uno solo (8080).

### Ejemplo
```bash
# Servicio de 5 réplicas de nginx, balanceado en el puerto 8080
docker service create --name web --replicas 5 --publish 8080:80 nginx

docker service ls                 # servicios, modo (replicated) y réplicas 5/5
docker service ps web             # tareas y en qué nodo corre cada una
docker service inspect --pretty web
docker service logs -f web        # logs de todas las réplicas (web.1, web.2...)

# Comprobar el balanceo
for i in $(seq 1 10); do curl -s localhost:8080 > /dev/null; done

docker service rm web
```

### Relacionado
- [[Workloads#Deployment|Deployment (Kubernetes)]]
- [[Networking#Kubernetes Service|Service (Kubernetes)]]

---

## Escalado, actualizaciones y rollback

### Qué es
Operaciones en caliente sobre un servicio:
- **`scale`**: cambiar el número de réplicas.
- **`update`**: cambiar la imagen u otra configuración; Swarm sustituye las réplicas **una a una** (cuando una está lista, pasa a la siguiente), sin perder disponibilidad.
- **`rollback`**: volver a la **versión anterior** del servicio.

### Para qué sirve
Reaccionar a picos de tráfico y desplegar nuevas versiones sin parar el servicio; si la nueva versión falla, volver atrás con un comando.

### Ejemplo
```bash
# Pico de tráfico: de 5 a 10 réplicas
docker service scale web=10

# Desplegar una nueva versión (aquí, cambiar a Apache para que se note)
docker service update --image ubuntu/apache2 web
docker service update --image miapp:2.0 --update-parallelism 2 --update-delay 10s web

# Algo ha ido mal: volver a la versión anterior
docker service rollback web
```

### Relacionado
- [[Workloads#Deployment|kubectl rollout en Kubernetes]]

---

## Stacks

### Qué es
Un **stack** agrupa **varios servicios** (con sus redes, volúmenes y secrets) definidos en un fichero YAML con **la misma sintaxis que Docker Compose**, y los despliega en el Swarm de una vez. Es el equivalente de `docker compose up` para un clúster.

### Para qué sirve
Desplegar toda una aplicación como código en el clúster. La sección **`deploy`** (réplicas, recursos, política de reinicio, estrategia de actualización) es la que más se aprovecha en Swarm.

> [!info] Diferencias con Compose
> - Opciones como `restart` o `build` se **ignoran**: Swarm ya mantiene los servicios siempre en marcha y necesita imágenes ya construidas en un registro.
> - El nombre de cada servicio es `<stack>_<servicio>` (por ejemplo `wordpress_db`). No hay `docker stack logs`: se usa `docker service logs <stack>_<servicio>`.
> - Crea una red `overlay` por defecto (`<stack>_default`).

### Ejemplo
```yaml
# test.yaml
services:
  web:
    image: nginx:1.27
    ports:
      - "8080:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```
```bash
docker stack deploy -c test.yaml web          # desplegar el stack "web"
docker stack ls
docker stack services web                     # servicios del stack (web_web)
docker stack ps web                           # tareas
docker service logs web_web

# Un compose.yaml existente también sirve (avisará de las opciones no soportadas)
docker stack deploy -c compose.yaml wordpress

docker stack rm web
```

### Relacionado
- [[Docker Compose]]
- [[Límites de recursos]]
