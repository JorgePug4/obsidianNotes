---
tags:
  - docker
  - build
  - security
aliases:
  - Multi-stage
  - Multistage
  - Multi-stage build
  - Distroless
  - scratch
  - Imágenes mínimas
---

# Multi-stage y Distroless

- [[#Multi-stage builds]]
- [[#Distroless y scratch]]

---

## Multi-stage builds

### Qué es
Un **multi-stage build** es un [[Dockerfile]] con **varias instrucciones `FROM`**. Cada `FROM` empieza una **etapa** nueva (a la que se le puede dar un alias con `AS`), y una etapa puede **copiar archivos de otra** con `COPY --from=<etapa>`. **Solo la última etapa** forma la imagen final.

### Para qué sirve
Separar el **entorno de construcción** del **entorno de ejecución** en un único Dockerfile:
- En la etapa de build tienes compiladores, SDK, herramientas y dependencias de desarrollo (JDK + Maven, Go toolchain, `npm` con *devDependencies*...).
- En la etapa final solo copias **el artefacto** (el `.jar`, el binario, el `dist/`) sobre una imagen mínima (JRE, `scratch`, distroless, `nginx`).

Beneficios:
- **Imágenes mucho más pequeñas**: una app Java de "Hola mundo" construida a lo bruto con la imagen de Maven puede ocupar cerca de 1 GB; separando build y runtime se reduce mucho.
- **Más seguridad**: menos librerías → **menos superficie de ataque** → menos vulnerabilidades.
- **Despliegues más rápidos**: menos datos que transferir al registro, al servidor o al clúster.
- **Un solo Dockerfile** en lugar de mantener uno para compilar y otro para ejecutar.

> [!tip] Quién más se beneficia
> Los lenguajes **compilados** (Go, Rust, C/C++, Java): el resultado es un binario o un `.jar` que no necesita el compilador. También se aplica a **transpilados** (TypeScript → JS) e incluso **interpretados** (Python, instalando dependencias en una etapa y copiando el *virtualenv*), aunque la ganancia es menor.

> [!tip] Buenas prácticas en la etapa final
> - Partir de la imagen **más pequeña** que funcione (JRE en lugar de JDK, `-slim`, distroless).
> - **No ejecutar como root**: crear un usuario y usar `USER`, copiando los archivos con `COPY --chown=usuario:grupo`.
> - No arrastrar cachés de gestores de paquetes (no tiene sentido cachear en una imagen inmutable).
> - Fijar las versiones de las imágenes base.

### Ejemplo

**Java con Maven (build) + Temurin JRE (runtime), usuario no root:**
```dockerfile
# ---- Etapa 1: construcción ----
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline          # dependencias cacheadas en su propia capa
COPY src ./src
RUN mvn package -DskipTests

# ---- Etapa 2: ejecución ----
FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S app && adduser -S appuser -G app
WORKDIR /app
COPY --from=build --chown=appuser:app /app/target/app.jar app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Frontend: build con Node, servir con nginx:**
```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:1.27-alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

```bash
docker build -t app:old -f Dockerfile.old .
docker build -t app:multistage .
docker image ls app     # comparar tamaños

# Construir solo hasta una etapa concreta (útil para tests)
docker build --target build -t app:build .
```

### Relacionado
- [[#Distroless y scratch]]
- [[Docker en producción]]

---

## Distroless y scratch

### Qué es
**Distroless** lleva el multi-stage al extremo: la imagen final **no tiene distribución de Linux "completa"**: ni gestor de paquetes, ni **shell**, ni utilidades (`ls`, `curl`...). Solo el runtime mínimo del lenguaje (o nada) y tu aplicación. Su principal impulsor y mantenedor es **Google** (`gcr.io/distroless/...`), con variantes basadas en Debian para Java, Python, Node.js, C/C++ (`cc`), estáticas (`static`), etc.

**`scratch`** es la imagen **vacía**, el punto de partida del que se construyen las imágenes base como Debian o Alpine. Sirve para binarios **estáticos** que no necesitan nada más (típico en Go).

### Para qué sirve
Conseguir las imágenes **más pequeñas y seguras posibles**: sin shell ni herramientas, un atacante que entre en el contenedor apenas tiene con qué trabajar, y los escáneres de vulnerabilidades encuentran muchos menos paquetes afectados. Un "Hola mundo" en Go sobre `scratch` pesa **unos pocos MB**, poco más que el propio binario.

> [!warning] Más difíciles de depurar
> Al no haber shell, `docker exec -it ... sh` no funciona. Para depurar usa `docker debug` (Docker Desktop), un contenedor auxiliar que comparta el espacio de procesos/red, las variantes `:debug` de las imágenes distroless (incluyen BusyBox) o *ephemeral containers* en Kubernetes.

> [!info] Certificados y zona horaria en `scratch`
> `scratch` no trae certificados CA ni `tzdata`. Si tu binario hace HTTPS o usa zonas horarias, cópialos desde la etapa de build (`COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/`) o usa `gcr.io/distroless/static`.

### Ejemplo

**Java sobre distroless:**
```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY . .
RUN mvn package -DskipTests

FROM gcr.io/distroless/java21-debian12
COPY --from=build /app/target/app.jar /app/app.jar
WORKDIR /app
CMD ["app.jar"]          # la imagen ya define java como ENTRYPOINT
```

**Go sobre scratch (binario estático):**
```dockerfile
FROM golang:1.23-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app

FROM scratch
COPY --from=build /app /app
USER 65534:65534
ENTRYPOINT ["/app"]
```

```bash
docker build -t hello-go .
docker image ls hello-go     # ~2-3 MB
```

### Relacionado
- [[#Multi-stage builds]]
- [[Docker en producción]]
- [[Go routines|Go]]
