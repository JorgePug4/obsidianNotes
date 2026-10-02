---
tags:
  - docker
  - cheatsheet
aliases:
  - Docker cheatsheet
  - Comandos Docker
  - Chuleta Docker
  - Atajos Docker
---

# Docker - Cheatsheet de comandos

> [!tip] Cómo usar esta chuleta
> Tenla a mano mientras practicas y repásala antes de cada sesión: el objetivo es crear **memoria muscular** con la terminal. Unos 20 comandos cubren el 90 % del día a día.

### Contenedores → [[Contenedor]]
```bash
docker run -d --name web -p 8080:80 nginx:1.27   # crear y arrancar en segundo plano
docker run --rm -it ubuntu bash                  # interactivo y borrar al salir
docker run -e CLAVE=valor --env-file .env img    # variables de entorno
docker run -v datos:/ruta -v ./src:/app img      # volumen y bind mount
docker run --network red --restart unless-stopped img
docker run --cpus 1 --memory 512m img            # límites de recursos

docker ps            # en ejecución            | docker ps -a   # todos
docker logs -f web   # logs en vivo            | docker logs --tail 100 web
docker exec -it web sh                         # shell dentro del contenedor
docker attach web                              # engancharse al proceso principal
docker stop web | docker start web | docker restart web | docker kill web
docker rm web        # borrar parado           | docker rm -f web   # parar y borrar
docker container prune                         # borrar todos los parados
docker inspect web | docker top web | docker stats | docker port web
docker cp web:/ruta/fichero .                  # copiar contenedor -> host
docker update --cpus 2 --restart always web    # cambiar config en caliente
```

### Imágenes → [[Imagen]]
```bash
docker pull postgres:16-alpine
docker image ls                       # = docker images
docker search nginx
docker history nginx                  # capas / pasos de construcción
docker image inspect nginx
docker rmi nginx:latest               # = docker image rm
docker image prune                    # dangling   | docker image prune -a   # todas sin uso
```

### Construcción → [[Dockerfile]] · [[Multi-stage y Distroless]] · [[Buildx y multi-arquitectura]]
```bash
docker build -t app:1.0 .
docker build -f Dockerfile.prod -t app:prod .
docker build --build-arg VERSION=3.12 --no-cache -t app:1.0 .
docker build --target build -t app:build .                # hasta una etapa
docker buildx create --name multibuilder --use
docker buildx build --platform linux/amd64,linux/arm64 -t user/app:1.0 --push .
docker run --platform linux/amd64 ubuntu uname -m
```

### Registro y distribución → [[Registro de imágenes]]
```bash
docker login | docker logout
docker tag app:1.0 usuario/app:1.0
docker push usuario/app:1.0
docker commit web nginx-modificada        # snapshot de un contenedor (evitar)
docker save -o app.tar app:1.0 | docker load -i app.tar
```

### Volúmenes → [[Volúmenes]]
```bash
docker volume create datos | docker volume ls | docker volume inspect datos
docker volume rm datos     | docker volume prune
```

### Redes → [[Redes en Docker]]
```bash
docker network create red-web | docker network ls | docker network inspect red-web
docker network connect red-web cliente | docker network disconnect red-web cliente
docker network rm red-web | docker network prune
```

### Compose → [[Docker Compose]]
```bash
docker compose up -d [--build]   | docker compose down [-v]
docker compose ps | docker compose logs -f servicio | docker compose exec servicio sh
docker compose restart servicio | docker compose pull | docker compose config
docker compose -f otro.yaml up -d
```

### Swarm → [[Docker Swarm]]
```bash
docker swarm init | docker swarm join-token worker | docker node ls | docker swarm leave --force
docker service create --name web --replicas 5 -p 8080:80 nginx
docker service ls | docker service ps web | docker service logs -f web
docker service scale web=10
docker service update --image nginx:1.27 web | docker service rollback web
docker stack deploy -c compose.yaml app | docker stack services app | docker stack rm app
```

### Sistema y limpieza
```bash
docker version | docker info
docker system df              # espacio usado
docker system prune           # parados + redes sin uso + dangling + caché
docker system prune -a --volumes   # ⚠️ limpieza total (incluye volúmenes sin uso)
docker builder prune          # caché de build
```

### Equivalencias de sintaxis
| Forma corta (clásica) | Forma por objeto |
|---|---|
| `docker run` | `docker container run` |
| `docker ps` | `docker container ls` |
| `docker rm` | `docker container rm` |
| `docker images` | `docker image ls` |
| `docker rmi` | `docker image rm` |

> [!tip] Comandos multilínea
> Para partir un comando largo en varias líneas: `\` en bash/zsh (Linux, Mac) y la **comilla invertida** `` ` `` en PowerShell (Windows).

### Relacionado
- [[Docker - Índice]]
