---
tags:
  - docker
  - containers
aliases:
  - Contenedor
  - Container
  - docker run
  - docker ps
  - docker exec
  - docker logs
  - Detached
  - Publicar puertos
  - Port mapping
  - Políticas de reinicio
  - Restart policy
---

# Contenedor

- [[#Contenedor]]
- [[#Ejecutar contenedores (docker run)]]
- [[#Modos de ejecución attached, detached e interactivo]]
- [[#Publicación de puertos]]
- [[#Políticas de reinicio]]
- [[#Logs]]
- [[#Ciclo de vida]]

---

## Contenedor

### Qué es
Un **contenedor** es una **instancia en ejecución de una [[Imagen]]**: un **proceso** que se ejecuta en un **entorno aislado** con su propio sistema de archivos, red y árbol de procesos, compartiendo el kernel del anfitrión.

> [!important] Un contenedor es un proceso
> No es una máquina virtual ni un sistema operativo. Se ejecuta con un **propósito o comando principal** (definido con `CMD` o `ENTRYPOINT` en el [[Dockerfile]]). **Cuando ese comando termina, el contenedor también termina.** Puede ser una tarea que acaba (un script) o un servicio que se queda escuchando (un servidor web).

### Para qué sirve
- Ejecutar una aplicación con todas sus dependencias en segundos, sin instalar nada en el host (arrancar un nginx no requiere `apt install nginx` ni configurarlo).
- Lanzar **muchas instancias** de la misma imagen: una vez descargada, crear contenedores es prácticamente instantáneo.
- Gestionarlo como cualquier proceso Unix: arrancar, parar, reiniciar, ver su salida, ejecutar comandos dentro...

Borrar un contenedor **no borra la imagen**. Los datos escritos dentro del contenedor se pierden al borrarlo, salvo que estén en un [[Volúmenes|volumen]].

> [!info] Estados de un contenedor
> `created` → `running` → (`paused`) → `exited` (parado) → eliminado. Un contenedor parado **sigue existiendo** con sus datos y su configuración hasta que se borra con `docker rm`.

### Ejemplo
```bash
docker run nginx          # crea y arranca un contenedor
docker ps                 # contenedores en ejecución
docker ps -a              # todos, incluidos los parados
```
Columnas de `docker ps`: `CONTAINER ID` (hash corto), `IMAGE`, `COMMAND`, `CREATED`, `STATUS`, `PORTS`, `NAMES` (si no das nombre, Docker genera uno aleatorio tipo `festive_thompson`, combinando un adjetivo y el apellido de un científico o hacker famoso).

### Relacionado
- [[Imagen]]
- [[Workloads#Pod|Pod (Kubernetes)]]

---

## Ejecutar contenedores (docker run)

### Qué es
`docker run [opciones] imagen [comando]` **crea y arranca** un contenedor. Si la imagen no está en local, primero hace `docker pull`.

### Para qué sirve
Es el comando central de Docker. Opciones más usadas:

| Opción | Para qué |
|---|---|
| `-d` / `--detach` | Ejecutar en **segundo plano** |
| `-it` | Modo **interactivo** con terminal (`-i` mantiene STDIN abierto, `-t` asigna una TTY) |
| `--name web` | Dar un **nombre** (si no, Docker genera uno) |
| `-p 8080:80` | **Publicar** el puerto 80 del contenedor en el 8080 del host |
| `-e CLAVE=valor` / `--env-file .env` | Pasar **variables de entorno** |
| `-v volumen:/ruta` | Montar un **volumen** o carpeta del host |
| `--rm` | **Borrar** el contenedor automáticamente al pararse o terminar |
| `--restart unless-stopped` | **Política de reinicio** |
| `--network red` | Conectar a una **red** concreta |
| `--cpus 1 --memory 512m` | **Limitar recursos** |
| `--platform linux/amd64` | Ejecutar una imagen de **otra arquitectura** (emulada) |

### Ejemplo
```bash
# Servidor web en segundo plano, con nombre y puerto publicado
docker run -d --name web -p 8080:80 nginx:1.27

# Contenedor de usar y tirar: abre una shell en Ubuntu y se borra al salir
docker run --rm -it ubuntu bash

# Pasar un comando distinto al de por defecto
docker run --rm ubuntu cat /etc/os-release
```

### Relacionado
- [[Volúmenes]]
- [[Redes en Docker]]
- [[Dockerfile#Variables de entorno (ENV)]]
- [[Límites de recursos]]

---

## Modos de ejecución attached, detached e interactivo

### Qué es
- **Attached** (por defecto): la terminal queda **enganchada a la salida del proceso principal**. `Ctrl+C` envía la señal de parada y **detiene el contenedor**.
- **Detached** (`-d`): el contenedor corre en **segundo plano** y la terminal queda libre. Docker imprime el ID completo del contenedor.
- **Interactivo** (`-it`): abre una **terminal** dentro del contenedor para escribir comandos.

### Para qué sirve
- `docker attach <id>`: volver a **engancharse** al proceso principal de un contenedor en segundo plano (para interactuar con él). Cuidado: `Ctrl+C` lo detendrá.
- `docker exec <id> <comando>`: **ejecutar un comando adicional** dentro de un contenedor que ya está corriendo, sin afectar al proceso principal. Con `-it` y una shell (`bash`, `sh`, `zsh`) abres una sesión para **depurar** y navegar por su sistema de archivos.

> [!tip] ¿`bash` o `sh`?
> Muchas imágenes ligeras (Alpine, distroless) no traen `bash`. Prueba con `sh`. Las imágenes [[Multi-stage y Distroless#Distroless y scratch|distroless]] no tienen ni shell.

> [!tip] IDs abreviados
> En cualquier comando puedes referirte a un contenedor por su **nombre**, su **ID completo** o solo **los primeros caracteres del ID** (`docker stop d7`), siempre que no sean ambiguos.

### Ejemplo
```bash
# Lanzar en segundo plano
docker run -d --name web -p 8080:80 nginx

# Ejecutar un comando suelto dentro
docker exec web ls /etc/nginx

# Abrir una shell interactiva para depurar (salir con exit)
docker exec -it web bash

# Engancharse al proceso principal
docker attach web
```

### Relacionado
- [[#Logs]]
- [[Docker Compose#Comandos principales|docker compose exec]]

---

## Publicación de puertos

### Qué es
Por defecto un contenedor está aislado en una red virtual: aunque el proceso escuche en el puerto 80, **desde el host no se puede acceder**. Publicar un puerto (`-p host:contenedor`) crea una regla de **NAT** que redirige el tráfico de un puerto del host al puerto del contenedor.

> [!example] La bola de bolos
> Un contenedor sin puertos publicados es como una bola de bolos sin agujeros: está ahí funcionando, pero no puedes "agarrarla".

### Para qué sirve
Acceder a servicios web, bases de datos o APIs que corren en contenedores. Permite además **varias instancias de la misma imagen** en el mismo host, cada una en un puerto distinto del host (el puerto interno puede repetirse porque cada contenedor está aislado).

> [!warning] Reglas de los puertos
> - La relación es **uno a uno**: un puerto del host solo puede apuntar a **un** contenedor. Si ya está ocupado, `docker run` falla con `port is already allocated`.
> - En **rangos**, el origen y el destino deben tener **el mismo tamaño** (`-p 8080-8090:80-90`); no se puede mandar un rango a un único puerto.
> - Si el puerto 80 del host está ocupado (en Windows a veces lo usan Skype o IIS), usa otro como 8080.

> [!warning] `-p 5432:5432` expone en todas las interfaces
> Por defecto Docker publica en `0.0.0.0`: el servicio queda accesible **desde fuera del servidor**. Para bases de datos, no publiques el puerto (los contenedores de la misma red se ven entre sí) o publícalo solo en local: `-p 127.0.0.1:5432:5432`. Además, en Linux Docker escribe sus propias reglas de `iptables` y **puede saltarse UFW**.

> [!info] `EXPOSE` no publica
> La instrucción `EXPOSE` del [[Dockerfile]] solo **documenta** qué puerto usa la app. Para acceder desde el host hace falta `-p` (o `-P`, que publica todos los `EXPOSE` en puertos aleatorios).

### Ejemplo
```bash
# Puerto 8080 del host -> 80 del contenedor
docker run -d -p 8080:80 nginx

# Varios puertos
docker run -d -p 8080:80 -p 8081:81 nginx

# Rangos (mismo tamaño en origen y destino)
docker run -d -p 8080-8090:8080-8090 mi-app

# Solo accesible desde el propio host
docker run -d -p 127.0.0.1:5432:5432 -e POSTGRES_PASSWORD=secreto postgres:16

# Tres servidores web independientes
docker run -d --name web1 -p 8081:80 nginx
docker run -d --name web2 -p 8082:80 nginx
docker run -d --name web3 -p 8083:80 nginx

# Ver qué puertos tiene publicados un contenedor
docker port web1
```

### Relacionado
- [[Redes en Docker]]
- [[Networking#Kubernetes Service|Service (Kubernetes)]]

---

## Políticas de reinicio

### Qué es
La opción `--restart` indica **qué hace Docker cuando el contenedor se detiene** o cuando se reinicia el daemon/servidor.

| Política | Comportamiento |
|---|---|
| `no` (defecto) | Nunca reinicia |
| `on-failure[:N]` | Reinicia solo si sale con **código de error** (≠ 0), opcionalmente hasta N intentos |
| `always` | Reinicia **siempre**, incluso tras reiniciar el servidor. Si lo paras a mano, vuelve a arrancar cuando se reinicie el daemon |
| `unless-stopped` | Como `always`, pero **no** lo arranca si lo paraste manualmente |

### Para qué sirve
En un servidor, que los servicios **se recuperen solos** si fallan y **arranquen solos** tras un reinicio de la máquina. Sin política de reinicio, tras reiniciar el servidor tu aplicación no volverá a levantarse.

### Ejemplo
```bash
docker run -d --restart unless-stopped --name web -p 8080:80 nginx

# Cambiar la política de un contenedor existente
docker update --restart always web
```

### Relacionado
- [[Docker en producción]]
- [[Workloads#Pod|restartPolicy en Kubernetes]]

---

## Logs

### Qué es
`docker logs` muestra lo que el proceso principal del contenedor escribe en **STDOUT y STDERR**.

### Para qué sirve
Saber qué pasa dentro: si el servicio está listo para aceptar conexiones, en qué puerto arrancó, errores, accesos denegados... Muchas imágenes muestran ahí información útil, como la **contraseña generada** de MariaDB con `MARIADB_RANDOM_ROOT_PASSWORD=yes`.

> [!tip] Escribe tus logs en STDOUT
> En tus propias imágenes, envía los logs a la salida estándar en lugar de a ficheros: así `docker logs`, Compose, Swarm y Kubernetes los recogen sin configuración extra.

### Ejemplo
```bash
docker logs web            # salida hasta el momento
docker logs -f web         # seguir en tiempo real (Ctrl+C para salir, el contenedor sigue)
docker logs --tail 100 web # últimas 100 líneas
docker logs --since 10m web

# Buscar la contraseña generada por MariaDB
docker run -d --name mariadb -e MARIADB_RANDOM_ROOT_PASSWORD=yes -p 3306:3306 mariadb:jammy
docker logs mariadb | grep "GENERATED ROOT PASSWORD"
```

### Relacionado
- [[Docker Compose#Comandos principales|docker compose logs]]
- [[Docker Swarm#Servicios|docker service logs]]

---

## Ciclo de vida

### Qué es
Comandos para **parar, arrancar, reiniciar, borrar y limpiar** contenedores.

### Para qué sirve
Gestionar contenedores sin acumular basura: cada `docker run` crea un contenedor nuevo, y los parados se quedan ocupando espacio.

> [!warning] Conflictos de nombre
> Los nombres de contenedor son **únicos**. Un contenedor **parado sigue reservando su nombre**: hasta que no lo borres no puedes crear otro igual (`Conflict. The container name "/web1" is already in use`).
> Además, si `docker run` falla por un puerto ocupado, el contenedor **llega a crearse** (queda en estado `created`) y también reserva el nombre: hay que borrarlo antes de reintentar.

> [!warning] No se borra un contenedor en marcha
> `docker rm` falla si el contenedor está `running`. Opciones: `docker stop` y luego `docker rm`, o `docker rm -f` (para y borra de golpe; cómodo en desarrollo, con cuidado en producción).

### Ejemplo
```bash
docker stop web            # parada ordenada (SIGTERM y, tras 10 s, SIGKILL)
docker start web           # arrancar un contenedor parado (conserva ID, datos y config)
docker restart web
docker kill web            # parada inmediata (SIGKILL)
docker pause web / docker unpause web

docker rm web              # borrar un contenedor parado
docker rm -f web           # parar y borrar
docker rm 606 a57          # varios a la vez por ID corto

docker container prune     # borrar TODOS los contenedores parados (pide confirmación)
docker system prune        # contenedores parados + redes sin uso + imágenes dangling + caché de build

docker inspect web         # configuración completa en JSON (IP, redes, montajes, variables...)
docker top web             # procesos dentro del contenedor
docker stats               # consumo de CPU/RAM en vivo
```

### Relacionado
- [[Imagen#Gestión de imágenes en local]]
- [[Docker - Cheatsheet de comandos]]
