---
tags:
  - docker
  - build
aliases:
  - Buildx
  - docker buildx
  - Multi-arquitectura
  - Multi-plataforma
  - Multi-arch
  - --platform
  - ARM64
  - BuildKit
---

# Buildx y multi-arquitectura

### Qué es
Una imagen se construye para una **arquitectura de CPU** concreta: `linux/amd64` (x86-64, la de torres y la mayoría de servidores) o `linux/arm64` (Mac con chips M, Raspberry Pi, AWS Graviton, portátiles Windows con Qualcomm). El sistema operativo siempre es `linux` porque los contenedores son una característica del kernel de Linux, aunque los ejecutes desde Windows o Mac.

**`docker buildx`** es la versión extendida de `docker build` (basada en **BuildKit**) que permite **construir para otras arquitecturas, o para varias a la vez**, y publicar una **imagen multi-arquitectura**: un único tag con un *manifest list* que apunta a una variante por plataforma.

### Para qué sirve
- **Ejecutar** imágenes de otra arquitectura con `--platform` (por ejemplo, depurar en un Mac M1 un servicio que irá a un servidor `amd64`). Docker lo emula con **QEMU**: funciona, pero más lento.
- **Construir** desde un Mac ARM imágenes para servidores AMD64, o viceversa.
- **Publicar** un solo tag que funciona en cualquier máquina: al hacer `pull`, Docker elige automáticamente la variante de la arquitectura local.

> [!warning] Conflicto de arquitectura en local
> Docker no guarda dos arquitecturas del mismo tag en local (con el almacén clásico). Si descargaste `ubuntu` con `--platform linux/amd64` y luego lo ejecutas sin `--platform` en un ARM, verás un aviso de que la imagen no coincide con tu plataforma. Solución: `docker pull ubuntu` para sobrescribirla con la arquitectura correcta.

> [!tip] Requisitos para construir multi-plataforma
> - Activar el **almacén de imágenes containerd**: en Docker Desktop, *Settings → General → Use containerd for pulling and storing images*. En Docker Engine, en `/etc/docker/daemon.json` activar la *feature* `containerd-snapshotter` y reiniciar el servicio.
> - O bien crear un **builder** propio (driver `docker-container`) con `docker buildx create`.
> - Sin almacén containerd, `--load` solo puede cargar en local **una** plataforma; para varias, usa `--push` directamente al registro.

### Ejemplo

**Ejecutar con otra arquitectura:**
```bash
docker run --rm --platform linux/amd64 ubuntu uname -m   # x86_64 (emulado)
docker pull ubuntu                                       # vuelve a la arquitectura nativa
docker run --rm ubuntu uname -m                          # aarch64 en un Mac M
```

**Construir y publicar para varias arquitecturas a la vez:**
```bash
# Crear y seleccionar un builder
docker buildx create --name multibuilder --use
docker buildx inspect --bootstrap
docker buildx ls

# Construir para AMD64 y ARM64 y subir en un solo paso
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t pabpereza/nginx:1.0 \
  --push .

# Comprobar las plataformas del tag publicado
docker buildx imagetools inspect pabpereza/nginx:1.0
```

**Usar la plataforma en el Dockerfile (compilación cruzada sin emular):**
```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.23 AS build
ARG TARGETOS TARGETARCH
WORKDIR /src
COPY . .
RUN GOOS=$TARGETOS GOARCH=$TARGETARCH CGO_ENABLED=0 go build -o /app

FROM scratch
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

### Relacionado
- [[Dockerfile#Argumentos de build (ARG)]]
- [[Imagen#Tags y versiones]]
- [[Registro de imágenes]]
