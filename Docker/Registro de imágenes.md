---
tags:
  - docker
  - images
  - registry
aliases:
  - Registro de imágenes
  - Registry
  - Container Registry
  - Docker Hub
  - dockerhub
  - docker push
  - docker tag
  - docker commit
  - docker save
  - docker load
---

# Registro de imágenes

- [[#Registro (Docker Hub y otros)]]
- [[#docker tag y docker push]]
- [[#docker commit]]
- [[#docker save y docker load]]

---

## Registro (Docker Hub y otros)

### Qué es
Un **registro** (*registry*) es un servidor que **almacena y distribuye imágenes** de contenedor, organizado en **repositorios** (uno por imagen) con sus **tags** (versiones). Es "como GitHub, pero para imágenes".

**Docker Hub** es el registro de Docker y el **predeterminado**: si no indicas registro, `docker pull nginx` descarga `docker.io/library/nginx:latest`. Gracias al estándar [[Docker#OCI (Open Container Initiative)|OCI]] hay muchos registros compatibles:

| Registro | Prefijo de la imagen |
|---|---|
| Docker Hub | `docker.io/usuario/imagen` (o simplemente `usuario/imagen`) |
| GitHub Container Registry | `ghcr.io/usuario/imagen` |
| GitLab Container Registry | `registry.gitlab.com/grupo/proyecto/imagen` |
| Amazon ECR | `<cuenta>.dkr.ecr.<región>.amazonaws.com/imagen` |
| Azure Container Registry | `<nombre>.azurecr.io/imagen` |
| Google Artifact Registry | `<región>-docker.pkg.dev/<proyecto>/<repo>/imagen` |
| Registro propio | `mi-registro.local:5000/imagen` (imagen oficial `registry`) |

### Para qué sirve
- **Consumir** imágenes oficiales, verificadas o de la comunidad (en Docker Hub cada imagen tiene documentación, tags, arquitecturas y comandos de uso).
- **Publicar** tus imágenes para desplegarlas en servidores, compartirlas con el equipo o con la comunidad.
- **Versionar** releases con `push`/`pull`, igual que con un repositorio de código.

> [!info] Plan gratuito de Docker Hub
> Los repositorios públicos son ilimitados, pero el plan gratuito solo permite **un repositorio privado** y tiene límites de descargas (*rate limit*) para usuarios anónimos. Iniciar sesión (`docker login`) amplía ese límite.

### Ejemplo
```bash
# Iniciar sesión (Docker Hub por defecto; en Docker Desktop también desde el icono de la bandeja)
docker login
docker login ghcr.io -u miusuario        # otro registro (pide un token como contraseña)

# Registro privado local para pruebas
docker run -d -p 5000:5000 --name registry registry:2
docker tag nginx localhost:5000/nginx
docker push localhost:5000/nginx

docker logout
```

### Relacionado
- [[13 - Azure Container Registry]]
- [[Imagen#Tags y versiones]]

---

## docker tag y docker push

### Qué es
- **`docker tag origen destino`**: crea un **nuevo nombre (referencia)** para una imagen existente. No duplica capas; es como un alias o un "renombrado" que conserva el original.
- **`docker push`**: **sube** una imagen al registro indicado en su nombre.

### Para qué sirve
Para subir una imagen, su nombre debe **incluir el registro y el usuario/organización** del repositorio destino. El flujo es: crear el repositorio (en Docker Hub se crea también al hacer el primer push) → etiquetar la imagen local con ese nombre → `push`.

### Ejemplo
```bash
# Renombrar/versionar una imagen local
docker tag nginx-modificada nginx:v3.2

# Preparar para Docker Hub (usuario: ciberstriker, repositorio: nginx-modificado)
docker tag nginx-modificada ciberstriker/nginx-modificado:latest
docker tag nginx-modificada ciberstriker/nginx-modificado:1.0.0

# Subir
docker push ciberstriker/nginx-modificado:latest
docker push ciberstriker/nginx-modificado --all-tags

# En cualquier otro equipo
docker pull ciberstriker/nginx-modificado:1.0.0
```

> [!tip] Versiona con semver y no solo con `latest`
> Sube cada release con su versión (`1.4.2`) y, si quieres, mueve también `latest`. Así un despliegue o un *rollback* apunta siempre a una versión concreta.

### Relacionado
- [[Imagen#Capas (layers)]]
- [[Buildx y multi-arquitectura|docker buildx build --push]]

---

## docker commit

### Qué es
`docker commit` crea una **nueva imagen a partir del estado actual de un contenedor**: incluye los cambios en su sistema de archivos y sus metadatos, como una **snapshot** de una máquina virtual.

### Para qué sirve
Guardar rápidamente un contenedor que has modificado a mano (para depurar, analizar un incidente o conservar un estado).

> [!warning] No es la forma correcta de crear imágenes
> Una imagen creada con `commit` **no es reproducible**: nadie sabe qué pasos se hicieron dentro. La buena práctica es **editar el [[Dockerfile]] y reconstruir**. Además, `commit` **no incluye los datos de los volúmenes** montados.

### Ejemplo
```bash
docker run -d --name web nginx
docker exec -it web bash          # ... haces cambios dentro ...

# ID o nombre del contenedor + nombre de la nueva imagen
docker commit web nginx-modificada
docker image ls
# nginx              latest
# nginx-modificada   latest
```

### Relacionado
- [[Dockerfile]]

---

## docker save y docker load

### Qué es
- **`docker save`**: **exporta** una o varias imágenes (con todas sus capas, nombre y tags) a un archivo **`.tar`**.
- **`docker load`**: **importa** ese `.tar` como imagen, conservando nombre y tag.

### Para qué sirve
Mover imágenes **sin usar un registro**: entornos sin acceso a Internet (*air-gapped*), copias de archivo en un disco o en la nube, o pasar una imagen a otro equipo con un USB.

> [!info] No confundir con `export`/`import`
> `docker export` exporta el **sistema de archivos de un contenedor** (aplanado, sin capas ni metadatos) y `docker import` lo convierte en imagen. `save`/`load` trabajan con **imágenes** completas.

### Ejemplo
```bash
# Exportar
docker save -o nginx.tar nginx-modificada
docker save nginx:1.27 redis:7 | gzip > imagenes.tar.gz

# Borrar e importar de nuevo
docker rmi nginx-modificada
docker load -i nginx.tar
docker image ls        # nginx-modificada vuelve a aparecer con su tag
```

### Relacionado
- [[Volúmenes#Copias de seguridad de volúmenes]]
- [[Imagen#Gestión de imágenes en local]]
