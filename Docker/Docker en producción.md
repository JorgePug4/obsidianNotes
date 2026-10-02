---
tags:
  - docker
  - operations
  - security
aliases:
  - Docker en producción
  - Buenas prácticas Docker
  - Puesta en producción
  - Healthcheck
  - env_file
  - .env
---

# Docker en producción

- [[#Preparar el servidor]]
- [[#Desplegar con Compose]]
- [[#Reinicios y healthchecks]]
- [[#Variables y secretos fuera del código]]
- [[#Seguridad del servidor]]
- [[#Buenas prácticas de imágenes]]

---

## Preparar el servidor

### Qué es
El entorno de producción típico para empezar: un **servidor Linux** (por ejemplo Ubuntu Server 24.04) con **Docker Engine** y el plugin de **Docker Compose**, sin interfaz gráfica.

### Para qué sirve
Ejecutar tus aplicaciones con rendimiento nativo y sin licencias (ver [[Docker#Docker Engine vs Docker Desktop]]).

> [!warning] Usuario y permisos
> Tras instalar, tu usuario no tiene permisos sobre Docker. Dos opciones: usar `sudo docker ...` o añadir el usuario al grupo `docker`. En ambos casos **el daemon se ejecuta como root**, y pertenecer al grupo `docker` equivale a tener root en la máquina. Para reducir riesgos existe el modo **rootless** de Docker.

### Ejemplo
```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER          # opcional (cerrar sesión y volver a entrar)

docker run --rm hello-world
docker compose version                 # Compose ya viene como plugin
# Instalaciones antiguas: sudo apt install docker-compose-plugin
```

### Relacionado
- [[Docker]]

---

## Desplegar con Compose

### Qué es
El flujo de despliegue y actualización de una aplicación definida en [[Docker Compose]] en un servidor.

### Para qué sirve
Desplegar y actualizar con pocos comandos y de forma repetible.

> [!warning] No publiques la base de datos
> `ports: ["5432:5432"]` publica en `0.0.0.0`, es decir, **en Internet** si el servidor es público. Si solo la usa tu API, **quita `ports`** del servicio de base de datos: dentro de la red de Compose los contenedores ya se ven. El contenedor seguirá mostrando `5432/tcp` en `docker ps` (lo abre el proceso), pero sin vinculación al host. Para administrarla desde fuera, usa un **túnel SSH** o publica solo en `127.0.0.1`.

### Ejemplo
```bash
# Primer despliegue
git clone https://github.com/usuario/quotes.git && cd quotes
cp .env.example .env && nano .env        # credenciales reales, solo en el servidor
docker compose up -d --build

# Actualizar a una nueva versión
git pull
docker compose up -d --build             # reconstruye si cambió el código; solo recrea lo modificado

docker compose ps
docker compose logs -f backend

# Acceder a la BD no publicada mediante un túnel SSH desde tu equipo
ssh -L 5432:localhost:5432 usuario@servidor   # requiere publicarla en 127.0.0.1 del servidor
```

### Relacionado
- [[Docker Compose#Dockerizar una aplicación (ejemplo completo)]]

---

## Reinicios y healthchecks

### Qué es
- **`restart`**: política de reinicio del contenedor (ver [[Contenedor#Políticas de reinicio]]).
- **`deploy.restart_policy`**: política detallada (condición, espera entre intentos, máximo de intentos y ventana de tiempo). Pensada para Swarm; Compose la aplica parcialmente.
- **`healthcheck`**: una **prueba periódica** (por ejemplo, una petición a `/health`) que marca el contenedor como `healthy` o `unhealthy`.

### Para qué sirve
- **`restart: always` o `unless-stopped` es obligatorio en producción**: sin él, si se reinicia el servidor tu aplicación no vuelve a arrancar.
- Limitar los reintentos evita un bucle infinito cuando la imagen está rota y nunca va a arrancar.
- El healthcheck detecta aplicaciones "vivas pero colgadas" y permite que otros servicios esperen a que estén listas (`depends_on: condition: service_healthy`).

> [!warning] Un healthcheck fallido no reinicia el contenedor (sin Swarm)
> Con `docker run` o Compose, el estado `unhealthy` **solo se informa**; el contenedor no se reinicia. Quien **sustituye** las réplicas no sanas es **Swarm** (o Kubernetes con sus *liveness probes*). Fuera de Swarm, combina el healthcheck con monitorización o con una herramienta como `autoheal`.

### Ejemplo
```yaml
services:
  backend:
    build: .
    restart: unless-stopped
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s          # espera entre reinicios
        max_attempts: 3    # deja de intentarlo tras 3 fallos...
        window: 120s       # ...dentro de una ventana de 120 s
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 10s        # cada cuánto se comprueba
      timeout: 10s         # tiempo máximo de cada prueba
      retries: 3           # fallos seguidos para marcarlo unhealthy
      start_period: 2m     # margen de arranque para apps lentas
```
```bash
docker ps        # STATUS: Up 2 minutes (healthy)
docker inspect --format '{{json .State.Health}}' backend
```

### Relacionado
- [[Workloads#Pod|Probes en Kubernetes]]
- [[Docker Swarm]]

---

## Variables y secretos fuera del código

### Qué es
Sacar del `compose.yaml` las credenciales y la configuración sensible a un fichero **`.env`** que **no se sube a git**, y cargarlo con `env_file`.

### Para qué sirve
- No publicar usuarios y contraseñas en el repositorio.
- **Compartir** variables repetidas entre servicios (el usuario, la contraseña y la base de datos los usan tanto la API como PostgreSQL).
- Tener **credenciales distintas por servidor** con el mismo `compose.yaml`: cada servidor guarda su propio `.env`.

> [!tip] Plantilla versionada
> Sube a git un **`.env.example`** con las claves y valores ficticios, y añade `.env` al `.gitignore`.

> [!info] Dos usos de `.env` en Compose
> - Un `.env` en la carpeta del proyecto se usa automáticamente para **interpolar** variables en el YAML (`image: miapp:${TAG}`).
> - `env_file:` **inyecta** las variables dentro del contenedor.

> [!tip] Secrets
> Para secretos más serios, Compose permite `secrets:` montados como ficheros en `/run/secrets/<nombre>` (muchas imágenes oficiales aceptan variantes `_FILE`, como `POSTGRES_PASSWORD_FILE`). En Swarm los secrets se cifran y distribuyen por el clúster; en Kubernetes existe el objeto [[Secret]].

### Ejemplo
```bash
# .env  (en .gitignore)
POSTGRES_USER=quotes
POSTGRES_PASSWORD=una-contraseña-larga-y-aleatoria
POSTGRES_DB=quotes
```
```yaml
services:
  backend:
    build: .
    env_file: .env
  postgres:
    image: postgres:16-alpine
    env_file: .env
```
```bash
echo ".env" >> .gitignore
docker compose exec backend env | grep POSTGRES     # comprobar que llegan
```

**Con secrets de fichero:**
```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Relacionado
- [[Dockerfile#Variables de entorno (ENV)]]
- [[Azure Key Vault]]

---

## Seguridad del servidor

### Qué es
Medidas básicas de *hardening* del Linux que aloja los contenedores.

### Para qué sirve
Un servidor público recibe intentos de acceso constantes. Lo mínimo:
1. **No permitir login como root por SSH** (`PermitRootLogin no`; en Ubuntu Server ya viene desactivado).
2. **Autenticación por clave SSH** en lugar de contraseña: permite una contraseña de usuario larga y aleatoria sin sufrirla a diario. Después se puede desactivar el login por contraseña (`PasswordAuthentication no`).
3. **Fail2ban**: bloquea (vía iptables/firewall) las IPs que fallan varios intentos de conexión SSH (por defecto, unos 5). Con claves SSH no te bloquearás a ti mismo por teclear mal.
4. **Firewall**: abre solo los puertos necesarios. Recuerda que Docker escribe sus propias reglas de iptables y puede **saltarse UFW** con los puertos publicados.
5. **Actualizar** el sistema y Docker Engine/Desktop con frecuencia, sobre todo ante versiones de seguridad.
6. **Monitorizar**: `docker stats` para el consumo y, a medio plazo, un sistema que recoja logs y **avise** de caídas o falta de recursos; si no, nunca te enterarás de que algo falla.

### Ejemplo
```bash
# En tu equipo: crear claves (si no tienes) y copiar la pública al servidor
ssh-keygen -t ed25519
ssh-copy-id usuario@servidor
ssh usuario@servidor             # ya no pide contraseña

# En el servidor
sudo apt update && sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

> [!tip] Alta disponibilidad sin Kubernetes
> Para tener la aplicación en varios servidores sin ir a Kubernetes: [[Docker Swarm]], o varios servidores con Compose detrás de un **proxy inverso / balanceador de carga**.

### Relacionado
- [[Confianza cero (Zero Trust)]]

---

## Buenas prácticas de imágenes

### Qué es
Recomendaciones para que las imágenes que llegan a producción sean **pequeñas, seguras y reproducibles**.

### Para qué sirve
Menos tamaño → despliegues más rápidos; menos paquetes → menos vulnerabilidades; versiones fijadas → builds reproducibles.

| Práctica | Cómo |
|---|---|
| Fijar versiones | `FROM python:3.12.5-slim`, nunca `latest` (ver [[Imagen#Tags y versiones]]) |
| Imágenes mínimas | `-slim`, `-alpine`, distroless o `scratch` ([[Multi-stage y Distroless]]) |
| Multi-stage | Compiladores y herramientas solo en la etapa de build |
| No ejecutar como root | `USER appuser`, `COPY --chown=...` |
| Orden de capas | Dependencias antes que código ([[Dockerfile#docker build y la caché]]) |
| `.dockerignore` | Excluir `.git`, `.env`, `node_modules`... |
| Sin secretos en la imagen | Ni en `ENV`, ni en `ARG`, ni copiados: inyectarlos en ejecución |
| Imágenes de confianza | Oficiales o de *Verified Publisher* |
| Escanear vulnerabilidades | `docker scout cves miimagen:tag`, Trivy... |
| Un proceso por contenedor | Logs a STDOUT; `ENTRYPOINT`/`CMD` en forma exec |
| Límites de recursos | `deploy.resources.limits` ([[Límites de recursos]]) |

### Ejemplo
```bash
# Analizar vulnerabilidades de una imagen
docker scout quickview miapp:1.0
docker scout cves miapp:1.0

# Ejecutar con un sistema de archivos de solo lectura y sin privilegios extra
docker run -d --read-only --tmpfs /tmp --cap-drop ALL --security-opt no-new-privileges miapp:1.0
```

### Relacionado
- [[Multi-stage y Distroless]]
- [[Límites de recursos]]
- [[Docker - Cheatsheet de comandos]]
