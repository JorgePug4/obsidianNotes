---
tags:
  - docker
  - build
aliases:
  - Dockerfile
  - Docker file
  - docker build
  - Build context
  - Caché de construcción
  - ENTRYPOINT
  - CMD
  - ARG
  - ENV
  - Variables de entorno
  - .dockerignore
---

# Dockerfile

- [[#Dockerfile]]
- [[#docker build y la caché]]
- [[#ENTRYPOINT vs CMD]]
- [[#Argumentos de build (ARG)]]
- [[#Variables de entorno (ENV)]]

---

## Dockerfile

### Qué es
Un **Dockerfile** es un **archivo de texto con instrucciones secuenciales** que le dicen a Docker **cómo construir una [[Imagen]]**. Es como escribir en un Linux los comandos necesarios para que tu aplicación funcione, pero **como código**, para que el proceso sea replicable en cualquier sitio. Cada instrucción genera una [[Imagen#Capas (layers)|capa]].

Instrucciones principales:

| Instrucción | Qué hace |
|---|---|
| `FROM imagen:tag` | **Imagen base** de la que se parte (un SO como `ubuntu`, un runtime como `python:3.12-slim` o un servicio final como `nginx`). Suele ser la primera |
| `WORKDIR /app` | Directorio de trabajo para las instrucciones siguientes (lo crea si no existe). Se puede usar varias veces |
| `COPY origen destino` | Copia archivos del **contexto de build** (tu máquina) a la imagen |
| `ADD` | Como `COPY`, pero además descomprime `.tar` y acepta URLs. Usa `COPY` salvo que necesites eso |
| `RUN comando` | **Ejecuta un comando** durante la construcción (instalar paquetes, compilar...) |
| `ENV CLAVE=valor` | Variable de entorno disponible en build **y en ejecución** |
| `ARG NOMBRE=defecto` | Variable **solo de build** |
| `EXPOSE 80` | Documenta el puerto que usa la app (no lo publica) |
| `USER appuser` | Usuario con el que se ejecutan las instrucciones siguientes y el contenedor |
| `CMD [...]` | Comando **por defecto** al arrancar (sobrescribible) |
| `ENTRYPOINT [...]` | Ejecutable **fijo** al arrancar |
| `HEALTHCHECK` | Comando para comprobar si el contenedor está sano |
| `VOLUME /datos` | Declara un punto de montaje para datos persistentes |
| `LABEL clave=valor` | Metadatos (autor, versión, repositorio...) |

### Para qué sirve
- **Empaquetar** tu aplicación con sus dependencias de forma reproducible.
- **Compartir** el proceso de construcción (no solo la imagen) para que otro lo replique.
- Versionar la infraestructura junto al código en git.

> [!tip] Servicio final vs runtime
> - Si partes de un **servicio final** (`nginx`, una base de datos), basta con copiar tu contenido o configuración: un Dockerfile de 2 líneas sirve una web estática (ideal para el `dist/` de Angular o React).
> - Si partes de un **runtime** (`python`, `node`, `eclipse-temurin`), tienes que instalar dependencias, copiar el código y definir el arranque.

> [!tip] `.dockerignore`
> Igual que `.gitignore`: excluye del contexto de build `node_modules/`, `.git/`, `.env`, binarios... Builds más rápidos y evitas meter secretos en la imagen.

### Ejemplo

**Web estática con nginx:**
```dockerfile
FROM nginx:1.27-alpine
COPY index.html /usr/share/nginx/html/index.html
```

**Servidor Apache sobre Ubuntu:**
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y apache2 && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/
EXPOSE 80
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

**Script de Python:**
```dockerfile
FROM python:3.12-alpine
WORKDIR /app
COPY script.py .
CMD ["python", "script.py"]
```

**API FastAPI con PostgreSQL (versión completa):**
```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Dependencias del SO (librería cliente de PostgreSQL)
RUN apt-get update && apt-get install -y --no-install-recommends libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Primero dependencias (cambian poco) -> capa cacheada
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Después el código (cambia mucho)
COPY . .

EXPOSE 80
CMD ["fastapi", "run", "main.py", "--port", "80"]
```

### Relacionado
- [[Imagen]]
- [[Multi-stage y Distroless]]
- [[Docker Compose#Dockerizar una aplicación (ejemplo completo)]]

---

## docker build y la caché

### Qué es
`docker build` construye una imagen a partir de un Dockerfile y un **contexto**:

```bash
docker build -t nombre:tag <contexto>
```
- **Contexto**: la carpeta (normalmente `.`) cuyos archivos se envían al motor de build; es lo que `COPY` puede ver.
- `-t`: nombre y tag de la imagen (sin tag → `latest`).
- `-f`: Dockerfile con otro nombre o ubicación (`-f Dockerfile.prod`).

### Para qué sirve
Convertir tu código en una imagen ejecutable. Docker **cachea cada capa**: si una instrucción y los archivos que usa no han cambiado, reutiliza la capa anterior (`CACHED`). En cuanto **una capa cambia, se invalidan todas las siguientes**.

> [!important] Ordena de lo que menos cambia a lo que más cambia
> Como la caché es secuencial, el orden de las instrucciones decide la velocidad del build:
> ```dockerfile
> # ❌ Mal: cada cambio de código reinstala TODAS las dependencias
> COPY . .
> RUN pip install -r requirements.txt
>
> # ✅ Bien: las dependencias solo se reinstalan si cambia requirements.txt
> COPY requirements.txt .
> RUN pip install -r requirements.txt
> COPY . .
> ```
> Con un `requirements.txt` realista (Flask, NumPy, pandas...) la diferencia entre ambos es de minutos a segundos en cada build. Lo mismo aplica a `package.json` (npm/yarn), `go.mod`, `Cargo.toml`, `pom.xml`...

### Ejemplo
```bash
# Construir en la carpeta actual
docker build -t app-nginx .
docker build -t app-python:v1 .

# Usar un Dockerfile con otro nombre
docker build -f Dockerfile.test -t app:test .

# Ignorar la caché por completo
docker build --no-cache -t app:v2 .

# Ejecutar la imagen construida
docker run -d -p 8080:80 --name web app-nginx
docker run -it app-python:v1
```

> [!tip] No reconstruyas en cada cambio durante el desarrollo
> Para iterar rápido monta tu código con un [[Volúmenes#Bind mount|bind mount]] o usa **Dev Containers**, en lugar de hacer `docker build` cada vez que tocas una línea.

### Relacionado
- [[Imagen#Capas (layers)]]
- [[Multi-stage y Distroless]]

---

## ENTRYPOINT vs CMD

### Qué es
Ambas definen **qué se ejecuta al arrancar el contenedor**, con una diferencia clave:
- **`ENTRYPOINT`**: el **ejecutable fijo**. No se sobrescribe con los argumentos de `docker run` (solo con `--entrypoint`). Todo lo que venga después (de `CMD` o de `docker run`) se le **añade como argumentos**.
- **`CMD`**: el **comando o argumentos por defecto**. **Cualquier argumento** que pases en `docker run imagen ...` **lo sustituye**.

| Dockerfile | `docker run img` | `docker run img docker maníaco` |
|---|---|---|
| Solo `CMD ["echo","Hola mundo"]` | `Hola mundo` | ejecuta `docker maníaco` como comando (falla) |
| Solo `ENTRYPOINT ["echo","Hola"]` | `Hola` | `Hola docker maníaco` |
| `ENTRYPOINT ["echo","Hola"]` + `CMD ["mundo"]` | `Hola mundo` | `Hola docker maníaco` |

### Para qué sirve
El patrón **`ENTRYPOINT` + `CMD`** convierte la imagen en un **ejecutable con argumentos por defecto**: por ejemplo, `ENTRYPOINT ["python", "app.py"]` y `CMD ["--help"]`. Si el usuario no pasa nada, ve la ayuda; si hace `docker run mi-app param1 param2`, se ejecuta `python app.py param1 param2`.

> [!warning] Forma exec vs forma shell
> - **Exec** (JSON, recomendada): `CMD ["nginx", "-g", "daemon off;"]` → el proceso es el **PID 1** y recibe las señales (`docker stop` lo para limpiamente).
> - **Shell**: `CMD nginx -g "daemon off;"` → se ejecuta como `/bin/sh -c ...`; la shell es el PID 1 y puede **no reenviar `SIGTERM`**, de modo que `docker stop` espera 10 s y mata el proceso con `SIGKILL`.

### Ejemplo
```dockerfile
FROM alpine
ENTRYPOINT ["echo", "Hola"]
CMD ["mundo"]
```
```bash
docker build -t test .
docker run test                      # Hola mundo
docker run test docker maníaco       # Hola docker maníaco
docker run --entrypoint ls test /    # sobrescribe el ENTRYPOINT
```

### Relacionado
- [[Contenedor#Contenedor]]
- [[Workloads#Pod|command/args en Kubernetes]]

---

## Argumentos de build (ARG)

### Qué es
`ARG` define **variables que solo existen durante la construcción** de la imagen. Se usan como `$NOMBRE` o `${NOMBRE}` en el resto de instrucciones y pueden tener un **valor por defecto**, que se sobrescribe con `--build-arg`.

### Para qué sirve
**Parametrizar el Dockerfile** sin editarlo: versiones de la imagen base, nombres de servicio, rutas, flags de compilación... Cambias un valor y se aplica a todas las instrucciones que lo usan.

> [!warning] `ARG` no llega al contenedor ni es secreto
> Un `ARG` **no existe en tiempo de ejecución** (para eso está `ENV`). Además su valor queda registrado en `docker history`: **no pases contraseñas ni tokens con `ARG`**; usa los *build secrets* de BuildKit (`RUN --mount=type=secret`).

### Ejemplo
```dockerfile
ARG PYTHON_VERSION=3.12
FROM python:${PYTHON_VERSION}-slim

ARG NOMBRE=Mundo
RUN echo "Hola $NOMBRE" > /mensaje
ENTRYPOINT ["cat", "/mensaje"]
```
```bash
docker build -t saludo .                               # Hola Mundo
docker build --build-arg NOMBRE=YouTube -t saludo .    # Hola YouTube
docker build --build-arg PYTHON_VERSION=3.11 -t saludo .
```

### Relacionado
- [[#Variables de entorno (ENV)]]
- [[Buildx y multi-arquitectura]]

---

## Variables de entorno (ENV)

### Qué es
Las **variables de entorno** son pares clave-valor disponibles **dentro del contenedor en ejecución**. Se pueden definir:
- En el Dockerfile con `ENV` (valor por defecto de la imagen).
- Al ejecutar: `docker run -e CLAVE=valor` o `--env-file fichero.env`.
- En [[Docker Compose]] con `environment:` o `env_file:`.

### Para qué sirve
Hacer la imagen **agnóstica del entorno**: la misma imagen sirve para desarrollo, pruebas, producción o para 200 clientes distintos, cambiando solo la configuración (URL de la API, credenciales de la base de datos, modo debug...).

- **Imágenes de terceros**: su documentación en Docker Hub lista las variables que aceptan. Por ejemplo `POSTGRES_PASSWORD` (obligatoria), `POSTGRES_USER`, `POSTGRES_DB`; en MariaDB `MARIADB_ROOT_PASSWORD`, `MARIADB_DATABASE`, `MARIADB_USER`...; en nginx `NGINX_ENTRYPOINT_QUIET_LOGS=1` silencia los logs de arranque.
- **Tus propias imágenes**: lee la configuración del entorno en el código (`os.getenv` en Python, `process.env` en Node, `Environment.GetEnvironmentVariable` en .NET).

> [!important] Nunca configuración escrita en el código
> Un error típico al empezar es dockerizar la app **dejando las credenciales y hosts hardcodeados**. El objetivo de una imagen es que funcione en cualquier entorno: todo lo que cambie entre entornos se inyecta desde fuera.

> [!warning] Las variables de entorno no son un almacén seguro
> Se ven con `docker inspect` y las hereda cualquier proceso. Para secretos en producción usa ficheros `.env` fuera de git, *secrets* de Compose/Swarm o el [[Secret]] de Kubernetes. Ver [[Docker en producción]].

### Ejemplo
```python
# app.py
import os, time
while True:
    if os.getenv("DEBUG"):
        print("Modo depuración activado")
    else:
        print("Modo depuración desactivado")
    time.sleep(1)
```
```bash
docker run --rm test                      # Modo depuración desactivado
docker run --rm -e DEBUG=true test        # Modo depuración activado

# Base de datos configurada por entorno
docker run -d --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 postgres:16

# Varias variables desde un fichero
docker run -d --env-file .env mi-api:1.0

# Ver las variables de un contenedor
docker exec some-postgres env
```

### Relacionado
- [[Docker Compose]]
- [[ConfigMap]]
- [[Secret]]
