---
tags:
  - docker
  - orchestration
aliases:
  - Docker Compose
  - Compose
  - docker compose
  - docker-compose
  - compose.yaml
  - docker-compose.yml
---

# Docker Compose

- [[#Docker Compose]]
- [[#Comandos principales]]
- [[#Servicios con build]]
- [[#Volúmenes y redes en Compose]]
- [[#Dockerizar una aplicación (ejemplo completo)]]

---

## Docker Compose

### Qué es
**Docker Compose** es la herramienta para **definir y ejecutar aplicaciones de varios contenedores** con **un único fichero YAML** (`compose.yaml`) y **un solo comando**. En el fichero se declaran:
- **`services`**: cada servicio equivale a un contenedor (o varias réplicas) con su imagen, puertos, variables, volúmenes, redes...
- **`volumes`**, **`networks`**, **`configs`** y **`secrets`**: recursos compartidos.

Todo lo que antes se hacía con comandos sueltos (`docker run`, `docker volume create`, `docker network create`) queda **como código**.

| Comando suelto | En Compose |
|---|---|
| `docker run --name web -p 8080:80 nginx` | `services: web: image: nginx, ports: ["8080:80"]` |
| `docker volume create datos` | `volumes: datos:` |
| `docker network create backend` | `networks: backend:` |
| `docker run -e CLAVE=valor` | `environment:` / `env_file:` |

### Para qué sirve
- **No repetir** largos `docker run`: la configuración queda versionada en git.
- Levantar **toda una aplicación** (frontend, API, base de datos, caché) en cualquier servidor con `docker compose up -d`.
- Orquestar contenedores **en un solo servidor** (para varios nodos, ver [[Docker Swarm]] o Kubernetes).

> [!info] `docker compose` vs `docker-compose`
> La versión actual (Compose V2) es un **plugin** de la CLI: `docker compose` (con espacio). Viene incluido en Docker Desktop y se instala en Docker Engine con el paquete `docker-compose-plugin`. El antiguo binario `docker-compose` (con guion) está obsoleto. La clave `version:` en la cabecera del YAML también está obsoleta y se ignora.

> [!tip] Repositorio `awesome-compose`
> Docker mantiene el repositorio **docker/awesome-compose** con decenas de aplicaciones de ejemplo definidas en Compose (WordPress, Nextcloud, stacks de Node, Python, Go...), listas para reutilizar.

### Ejemplo

**El caso más básico:**
```yaml
# compose.yaml
services:
  web:
    image: nginx:1.27
    ports:
      - "8080:80"
```
```bash
docker compose up        # lee compose.yaml de la carpeta actual
```

**WordPress + MariaDB** (ejemplo de la documentación oficial):
```yaml
services:
  db:
    image: mariadb:10.6
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress    # ⚠️ cambia las credenciales de ejemplo
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    volumes:
      - db_data:/var/lib/mysql
    expose:
      - "3306"

  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db                 # nombre del servicio, no localhost
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html

volumes:
  db_data:
  wp_data:
```

> [!info] `expose` vs `ports`
> - `ports` **publica** el puerto en el host (accesible desde fuera).
> - `expose` solo **documenta** el puerto para otros servicios. En la práctica, los servicios de la misma red de Compose **ya se ven entre sí en todos sus puertos**, así que `expose` es opcional.

> [!important] Revisa la documentación de cada imagen
> Muchas imágenes **no arrancan** sin ciertas variables: MariaDB necesita contraseña de root; WordPress necesita los datos de conexión a la base de datos. La página de la imagen en Docker Hub lista las variables obligatorias.

### Relacionado
- [[Contenedor]]
- [[Redes en Docker]]
- [[Volúmenes]]
- [[Helm|Helm (equivalente en Kubernetes)]]

---

## Comandos principales

### Qué es
La CLI `docker compose` opera sobre **todos los servicios del fichero** del directorio actual (o del indicado con `-f`), y la mayoría de comandos aceptan un **nombre de servicio** para actuar solo sobre él.

### Para qué sirve
Lanzar todo en conjunto, pero **depurar por separado**: logs, `exec` o reinicios de un solo servicio. `docker compose ps` solo muestra los contenedores de **ese** proyecto, lo que da una salida limpia aunque haya muchas aplicaciones en el servidor.

> [!info] Nombres de los recursos
> Compose prefija todo con el **nombre del proyecto** (por defecto, la carpeta): contenedores `proyecto-web-1`, red `proyecto_default`, volúmenes `proyecto_db_data`. Se cambia con `-p nombre` o la clave `name:` del YAML.

### Ejemplo
```bash
docker compose up                  # crea y arranca todo en primer plano (logs mezclados)
docker compose up -d               # en segundo plano
docker compose up -d --build       # reconstruye las imágenes con build: antes de arrancar
docker compose -f despliegue.yaml up -d   # otro nombre de fichero

docker compose ps                  # contenedores de este proyecto
docker compose logs                # logs de todos los servicios
docker compose logs -f web1        # logs en vivo de un servicio
docker compose exec web1 sh        # shell dentro de un servicio
docker compose restart web1
docker compose pull                # descargar las últimas versiones de las imágenes
docker compose config              # ver el YAML final resuelto (variables incluidas)

docker compose stop                # parar sin borrar
docker compose down                # parar y BORRAR contenedores y red por defecto
docker compose down -v             # ... y también los volúmenes (⚠️ borra los datos)
```

> [!tip] Actualizar en producción
> Cambias el `compose.yaml` (o haces `git pull`) y vuelves a lanzar `docker compose up -d`: Compose **solo recrea los servicios que han cambiado**.

### Relacionado
- [[Contenedor#Logs]]
- [[Docker - Cheatsheet de comandos]]

---

## Servicios con build

### Qué es
Un servicio puede usar `image:` (imagen ya construida) o `build:` para **construir la imagen desde un [[Dockerfile]]** al levantar el proyecto.

### Para qué sirve
Tener tu propia aplicación y sus dependencias (bases de datos, colas...) en el mismo fichero, sin construir la imagen a mano antes.

### Ejemplo
```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"

  web1:
    build: .                     # Dockerfile en la carpeta actual
    ports:
      - "8081:80"

  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
      args:
        PYTHON_VERSION: "3.12"
    image: miusuario/api:1.0     # nombre con el que se etiqueta la imagen construida
```

### Relacionado
- [[Dockerfile#docker build y la caché]]

---

## Volúmenes y redes en Compose

### Qué es
Los volúmenes y redes se **declaran en la raíz** del fichero y se **referencian desde cada servicio**. Si no existen, Compose los crea; si existen, los reutiliza.

### Para qué sirve
- **Red por defecto**: aunque no declares redes, Compose crea una (`<proyecto>_default`) y conecta todos los servicios, aislados del resto de contenedores del host. Dentro, cada servicio se resuelve **por su nombre** ([[Redes en Docker#Resolución por nombre (DNS interno)|DNS interno]]): un contenedor llega a otro con `curl web` o a la base de datos con `host=postgres`.
- **Redes propias** para segregar: en el ejemplo de Nextcloud de `awesome-compose`, MariaDB y Redis están en redes distintas y solo Nextcloud (que necesita ambas) está conectado a las dos. **Si algo no necesita conexión, limítalo.**
- **Volúmenes** declarados vacíos usan la configuración por defecto; también admiten drivers, opciones o montar rutas del host.

### Ejemplo
```yaml
services:
  nextcloud:
    image: nextcloud:apache
    ports:
      - "80:80"
    networks:
      - redisnet
      - dbnet
    volumes:
      - nc_data:/var/www/html

  redis:
    image: redis:alpine
    networks:
      - redisnet

  db:
    image: mariadb:10.6
    networks:
      - dbnet
    volumes:
      - db_data:/var/lib/mysql

networks:
  redisnet:
  dbnet:

volumes:
  nc_data:
  db_data:
    driver: local
    driver_opts:          # montar una ruta concreta del host como volumen
      type: none
      o: bind
      device: /srv/datos/mariadb
```

**Bind mounts y solo lectura en un servicio:**
```yaml
services:
  web:
    image: nginx
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
```

### Relacionado
- [[Volúmenes]]
- [[Redes en Docker]]

---

## Dockerizar una aplicación (ejemplo completo)

### Qué es
El proceso completo para llevar una aplicación propia a contenedores: **Dockerfile → configuración por entorno → Compose con sus dependencias → persistencia**. Ejemplo: una API de citas célebres en **FastAPI** con **PostgreSQL**.

### Para qué sirve
Pasos y lecciones clave:
1. **Dockerfile** con las dependencias del SO, las librerías del lenguaje (separadas del código para aprovechar la caché) y el comando de arranque.
2. **Sacar la configuración del código**: usuario, contraseña, host y base de datos se leen de variables de entorno (`os.getenv`). Al principio el código tenía las credenciales escritas a mano; eso hace la imagen inservible fuera de tu equipo.
3. **Compose** con la API y su base de datos, usando el **nombre del servicio** como host.
4. **Volumen** para que los datos de PostgreSQL sobrevivan.
5. No publicar el puerto de la base de datos si solo la usa la API.

> [!warning] Las credenciales deben coincidir
> Las variables que configuran PostgreSQL (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`) y las que lee la API tienen que tener los mismos valores. Lo más limpio es compartirlas desde un único fichero `.env` (ver [[Docker en producción#Variables y secretos fuera del código]]).

> [!tip] Errores típicos de sintaxis
> Las claves son `environment` y `volumes` (no `enviroment` ni `volume`), y la indentación YAML importa. `docker compose config` valida el fichero antes de lanzarlo.

### Ejemplo

**`main.py` (configuración leída del entorno):**
```python
import os
DB_CONFIG = {
    "user": os.getenv("POSTGRES_USER"),
    "password": os.getenv("POSTGRES_PASSWORD"),
    "host": os.getenv("POSTGRES_HOST", "postgres"),
    "dbname": os.getenv("POSTGRES_DB"),
}
```

**`compose.yaml`:**
```yaml
services:
  backend:
    build: .
    ports:
      - "8080:80"
    env_file: .env
    environment:
      POSTGRES_HOST: postgres        # el nombre del servicio de la BD
    depends_on:
      postgres:
        condition: service_healthy   # espera a que la BD esté lista
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    env_file: .env
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      retries: 5
    restart: unless-stopped
    # sin "ports": solo accesible para backend dentro de la red de Compose

volumes:
  postgres_data:
```

**`.env`** (fuera de git):
```bash
POSTGRES_USER=quotes
POSTGRES_PASSWORD=cambia-esto
POSTGRES_DB=quotes
```

```bash
docker compose up -d --build
curl http://localhost:8080/quote
```

### Relacionado
- [[Dockerfile]]
- [[Docker en producción]]
