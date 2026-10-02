---
tags:
  - docker
  - images
aliases:
  - Imagen
  - Imágenes
  - Docker image
  - Imagen de contenedor
  - Capas
  - Layers
  - Tag
  - Tags
  - latest
  - Alpine
  - docker pull
---

# Imagen

- [[#Imagen]]
- [[#Capas (layers)]]
- [[#Tags y versiones]]
- [[#Gestión de imágenes en local]]

---

## Imagen

### Qué es
Una **imagen** es un **archivo binario de solo lectura que contiene todo lo necesario para ejecutar un contenedor**. Es la **plantilla** a partir de la cual se crean uno o muchos [[Contenedor|contenedores]] (un contenedor es una *instancia* de una imagen en ejecución).

Una imagen contiene:
1. **La aplicación**: tu código o una aplicación de terceros ya empaquetada (nginx, PostgreSQL...).
2. **Librerías del lenguaje**: por ejemplo, el framework Spring Boot o los paquetes de `pip`.
3. **Librerías y paquetes del SO**: `curl`, `wget`, `cat`, `libpq`... de los que dependen muchas aplicaciones.
4. **Configuración**: usuario que ejecuta la app, comando de arranque, puertos que expone, variables de entorno.

> [!example] Imagen de una app Java
> Necesitaría: el **JRE** (o JDK) instalado, la aplicación compilada (`app.jar`) con sus librerías (Spring Boot) y la definición de cómo arrancarla (`java -jar app.jar`).

### Para qué sirve
- **Distribuir** una aplicación lista para ejecutarse, sin conflictos con el SO o con otras apps.
- **Versionar** la aplicación: cada versión es una imagen nueva, como commits en git.
- Garantizar **integridad**: lo que se prueba es exactamente lo que se despliega.

Las imágenes se crean a partir de un [[Dockerfile]] con `docker build`. También se pueden crear a partir del estado de un contenedor (`docker commit`, ver [[Registro de imágenes#docker commit]]), pero **no es buena práctica**.

> [!important] Inmutabilidad
> Una imagen **no se modifica** una vez creada. Para cambiarla se construye una **nueva**. Esto garantiza que, al moverla de un entorno a otro, no se pierda ni cambie nada, y permite un control de versiones fiable.

> [!tip] Imágenes estáticas
> La buena práctica es que la imagen **no dependa del estado de ejecución**: desde el primer arranque debe comportarse igual en cualquier entorno. La configuración que cambia entre entornos se inyecta con [[Dockerfile#Variables de entorno (ENV)|variables de entorno]] y los datos se guardan en [[Volúmenes]].

### Ejemplo
```bash
# Descargar una imagen (por defecto de Docker Hub y con el tag latest)
docker pull nginx

# docker run hace el pull automáticamente si la imagen no está en local
docker run -d -p 8080:80 nginx
```

### Relacionado
- [[Contenedor]]
- [[Dockerfile]]
- [[Registro de imágenes]]

---

## Capas (layers)

### Qué es
Una imagen **no es un bloque monolítico**: está formada por **capas apiladas**, y **cada capa corresponde a una instrucción** del [[Dockerfile]] que se usó para construirla (`FROM`, `RUN`, `COPY`...). Cada capa se identifica por un **hash** (digest) único.

Al ejecutar un contenedor, Docker añade encima una **capa fina de escritura** propia del contenedor; las capas de la imagen siguen siendo de solo lectura.

### Para qué sirve
- **Reutilización**: si una app de Python y otra de Node parten de la misma base (`alpine`, `debian`...), **comparten esas capas**. Se descargan y almacenan una sola vez.
- **Descargas más rápidas**: al hacer `pull`, Docker compara los hashes y **omite las capas que ya tiene** (`Already exists`).
- **Almacenamiento eficiente**: los registros pueden alojar millones de imágenes sin que el espacio crezca exponencialmente.
- **Caché de construcción**: al reconstruir, las capas que no cambiaron se reutilizan (ver [[Dockerfile#docker build y la caché|caché de build]]).

> [!info] Etiquetar no duplica
> `docker tag` no copia la imagen: crea **otra referencia al mismo conjunto de capas**. Por eso en `docker image ls` dos tags pueden mostrar el mismo `IMAGE ID`.

### Ejemplo
```bash
# Descargar alpine y luego una imagen basada en alpine: la capa base no se vuelve a bajar
docker pull alpine:3.19
docker pull nginx:stable-alpine
# ... 4abcf2066143: Already exists   <- capa compartida

# Ver el historial de construcción (una fila por capa/instrucción)
docker history nginx
docker history --no-trunc nginx   # comandos completos
```

### Relacionado
- [[Dockerfile]]
- [[Multi-stage y Distroless]]

---

## Tags y versiones

### Qué es
Un **tag** es la **etiqueta de versión** de una imagen. El nombre completo de una imagen sigue el formato:

```text
[registro/][usuario-u-organización/]repositorio[:tag][@digest]

nginx                         -> docker.io/library/nginx:latest
postgres:16-alpine            -> versión 16 de PostgreSQL sobre Alpine
pabpereza/quotes:1.2.0        -> imagen de un usuario en Docker Hub
ghcr.io/miorg/api:2.3.1       -> imagen en GitHub Container Registry
nginx@sha256:4c0f...          -> versión fijada por digest (inmutable)
```

Si no se indica tag, Docker usa **`latest`**.

### Para qué sirve
- Elegir **versiones concretas** (`postgres:14`, `postgres:15.1`).
- Elegir **variantes** de la misma versión: `-alpine` (muy ligera), `-slim`, `-bookworm` (Debian)... Por ejemplo `postgres:14-alpine3.17` = PostgreSQL 14 sobre Alpine 3.17.
- **Un mismo build puede tener varios tags** (`1.27.3`, `1.27`, `1`, `stable`, `latest` apuntando al mismo digest).
- Los tags son **históricos**: en las imágenes oficiales no se borran versiones antiguas.

> [!warning] `latest` no significa "la última versión"
> `latest` es **solo un nombre de tag por defecto**. En imágenes oficiales suele apuntar a la última versión publicada, pero en imágenes propias o de terceros puede apuntar a cualquier cosa (o estar desactualizada). Además, **cambia con el tiempo**: un `docker pull postgres` hoy puede bajar una versión mayor distinta a la de ayer.

> [!tip] Fija siempre la versión
> En `docker run`, Compose y `FROM` usa tags explícitos (`postgres:16.4-alpine`, no `postgres`). Así evitas actualizaciones mayores inesperadas que rompan tu aplicación o tus datos. Para máxima reproducibilidad, fija el **digest**.

> [!info] Alpine
> **Alpine Linux** es una distribución mínima (≈ 7 MB) orientada a la seguridad. Muchas imágenes ofrecen variante `-alpine` porque reduce mucho el tamaño y la superficie de ataque. Ojo: usa `musl` en lugar de `glibc` y `apk` en lugar de `apt`, lo que puede dar problemas con algunos binarios.

> [!info] Arquitecturas
> En Docker Hub cada tag indica las **arquitecturas soportadas** (`linux/amd64`, `linux/arm64`...). Docker descarga automáticamente la que corresponde a tu equipo. Ver [[Buildx y multi-arquitectura]].

### Ejemplo
```bash
docker pull postgres                  # = postgres:latest
docker pull postgres:14-alpine3.17    # versión y variante concretas
docker pull mariadb:jammy             # tag con nombre (puede coincidir en hash con otros tags)

# Dos versiones de PostgreSQL a la vez en el mismo equipo, en puertos distintos
docker run -d --name postgres-alfa -e POSTGRES_PASSWORD=mypass1 -p 5432:5432 postgres
docker run -d --name postgres-beta -e POSTGRES_PASSWORD=mypass1 -p 5433:5432 postgres:14-alpine3.17
```

### Relacionado
- [[Registro de imágenes]]
- [[Contenedor#Publicación de puertos]]

---

## Gestión de imágenes en local

### Qué es
El conjunto de comandos `docker image ...` para **listar, buscar, inspeccionar y borrar** imágenes del equipo.

### Para qué sirve
Mantener el disco limpio y saber qué hay dentro de cada imagen.

> [!warning] No se puede borrar una imagen en uso
> Docker impide borrar una imagen mientras exista **algún contenedor (en marcha o parado)** creado a partir de ella, porque ese contenedor la necesitaría para volver a arrancar. Con `-f` se fuerza, pero puedes dejar **contenedores huérfanos** que ya no funcionarán. Borra primero los contenedores.

> [!tip] Imágenes de confianza
> En Docker Hub prioriza las insignias **Docker Official Image** y **Verified Publisher**: siguen buenas prácticas y están revisadas. Las de la etiqueta *Sponsored OSS* o de usuarios anónimos pueden contener cualquier cosa; revísalas antes de usarlas.

### Ejemplo
```bash
# Listar imágenes locales
docker image ls            # = docker images

# Buscar imágenes en Docker Hub desde la terminal (la oficial aparece con [OK])
docker search nginx

# Inspeccionar metadatos (capas, variables, puertos, comando por defecto...)
docker image inspect nginx

# Borrar una imagen (por nombre:tag, ID completo o los primeros caracteres del ID)
docker image rm nginx:latest     # = docker rmi nginx:latest
docker rmi 3f8a 9c1b             # varias a la vez por ID corto
docker rmi -f nginx              # forzar (cuidado: contenedores huérfanos)

# Borrar imágenes "dangling" (sin tag, restos de builds)
docker image prune

# Borrar TODAS las imágenes no usadas por ningún contenedor
docker image prune -a

# Ver cuánto ocupa todo (imágenes, contenedores, volúmenes, caché de build)
docker system df
```

### Relacionado
- [[Contenedor#Ciclo de vida]]
- [[Docker - Cheatsheet de comandos]]
