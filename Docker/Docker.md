---
tags:
  - docker
  - architecture
aliases:
  - Docker
  - Contenedores
  - Containers
  - Docker Engine
  - Docker Desktop
  - OCI
  - Open Container Initiative
  - Contenedores vs Máquinas virtuales
---

# Docker

- [[#Docker y los contenedores]]
- [[#Contenedores vs Máquinas virtuales]]
- [[#OCI (Open Container Initiative)]]
- [[#Docker Engine vs Docker Desktop]]

---

## Docker y los contenedores

### Qué es
**Docker** es la plataforma que popularizó los **contenedores**: una forma de **empaquetar una aplicación con todas sus dependencias** (librerías del lenguaje, paquetes del sistema operativo, configuración) para **distribuirla y ejecutarla en un entorno aislado** de forma idéntica en cualquier máquina. Lo lanzó la empresa Docker Inc. en **2013**.

Docker no inventó los contenedores (existían desde los 90, por ejemplo **LXC**, *Linux Containers*), pero los hizo sencillos de **construir, distribuir y ejecutar**. Pasa como con el "pan Bimbo": la marca acabó dando nombre a la tecnología.

Un contenedor **no es una máquina virtual**: es un **proceso** aislado que **comparte el kernel** del sistema anfitrión. El aislamiento lo dan dos características del kernel de Linux:
- **Namespaces**: cada contenedor ve su propio árbol de procesos, red, sistema de archivos, usuarios, hostname...
- **cgroups** (*control groups*): limitan y contabilizan cuánta CPU, memoria, red o disco puede usar (ver [[Límites de recursos]]).

### Para qué sirve
Resuelve el clásico **"en mi máquina funciona"**:
- Un desarrollador nuevo no tiene que instalar a mano MariaDB, Redis y MongoDB **en versiones concretas** (y distintas para cada proyecto) en Windows, Mac o Linux: levanta un contenedor por servicio con un comando.
- Varias versiones de la misma base de datos pueden convivir en el mismo equipo sin conflictos, porque cada contenedor lleva sus propias dependencias.
- Lo que pruebas en local es **exactamente** lo que corre en CI y en producción: la misma imagen, las mismas librerías.
- Montar, borrar, cambiar de versión o hacer *rollback* no deja basura ni configuraciones a medias en tu equipo.

**Casos de uso más comunes:**
| Caso de uso | Por qué encajan los contenedores |
|---|---|
| **Testing y CI/CD** | La construcción y las pruebas son siempre homogéneas: mismas librerías, mismos paquetes |
| **Microservicios** | Cada servicio pequeño en su propio contenedor, sin conflictos de librerías entre ellos (ver [[🏗️ Diseño de Microservicios]]) |
| **Cloud y multi-nube** | El estándar OCI hace que la misma imagen funcione en cualquier proveedor y en Kubernetes |
| **Escalabilidad y alta disponibilidad** | Lanzar 10 contenedores más ante un pico de carga y quitarlos después es rápido y barato |
| **Desarrollo multiplataforma** | El mismo contenedor corre en Linux, Mac y Windows |

> [!info] Flujo típico de trabajo con Docker
> 1. Escribes tu aplicación y un [[Dockerfile]].
> 2. **Construyes** la [[Imagen]] (`docker build`), manualmente o automatizado en CI (Jenkins, GitHub Actions...).
> 3. **Subes** la imagen a un [[Registro de imágenes|registro]] como Docker Hub (`docker push`).
> 4. El servidor **descarga** esa versión concreta (`docker pull`) y la **ejecuta** como [[Contenedor]] (`docker run`).
> 5. Si una versión falla en producción, vuelves a la anterior simplemente ejecutando la imagen previa.

### Ejemplo

**Comprobar la instalación con el contenedor de prueba oficial:**
```bash
# Lista todos los comandos disponibles
docker

# Descarga la imagen hello-world y ejecuta un contenedor que imprime un mensaje de bienvenida
docker run hello-world

# Información del cliente, del daemon y del sistema
docker version
docker info
```

> [!tip] Dos sintaxis equivalentes
> Docker reorganizó la CLI por objetos (`docker container ...`, `docker image ...`, `docker volume ...`, `docker network ...`). Las formas cortas antiguas siguen funcionando por retrocompatibilidad:
> - `docker run` = `docker container run`
> - `docker ps` = `docker container ls`
> - `docker images` = `docker image ls`
> - `docker rmi` = `docker image rm`

### Relacionado
- [[Contenedor]]
- [[Imagen]]
- [[Dockerfile]]
- [[Container Runtime]]

---

## Contenedores vs Máquinas virtuales

### Qué es
Ambas tecnologías permiten ejecutar muchas aplicaciones aisladas sobre el mismo hardware, pero **virtualizan a niveles distintos**:

```text
   MÁQUINAS VIRTUALES                    CONTENEDORES
┌───────┬───────┬───────┐        ┌───────┬───────┬───────┐
│ App A │ App B │ App C │        │ App A │ App B │ App C │
│ Bins/ │ Bins/ │ Bins/ │        │ Bins/ │ Bins/ │ Bins/ │
│ Libs  │ Libs  │ Libs  │        │ Libs  │ Libs  │ Libs  │
│SO inv.│SO inv.│SO inv.│        ├───────┴───────┴───────┤
├───────┴───────┴───────┤        │ Motor de contenedores │
│      Hipervisor       │        │ SO anfitrión (kernel) │
├───────────────────────┤        ├───────────────────────┤
│ Infraestructura (HW)  │        │ Infraestructura (HW)  │
└───────────────────────┘        └───────────────────────┘
```

- La **VM** virtualiza **hardware**: el hipervisor (VMware ESXi, Hyper-V, KVM...) crea hardware virtual y en cada VM se instala un **sistema operativo completo** con su propio kernel.
- El **contenedor** virtualiza **a nivel de sistema operativo**: todos comparten el **kernel del anfitrión** y cada uno lleva solo lo mínimo imprescindible (la app, sus binarios y librerías).

### Para qué sirve
Saber cuándo usar cada uno:

| Criterio | Máquina virtual | Contenedor |
|---|---|---|
| Qué virtualiza | Hardware | Sistema operativo |
| Kernel | Uno propio por VM | **Compartido** con el anfitrión |
| Tamaño | GB (un Ubuntu Server ya ocupa 2-3 GB) | MB (un `nginx` ronda los 50-200 MB; `alpine` ≈ 7 MB) |
| Arranque | Minutos | Milisegundos o segundos |
| Aislamiento | Fuerte (frontera de hardware) | Más ligero (frontera de proceso) |
| Portabilidad | Pesada de mover | Muy fácil de empaquetar y transportar |
| Uso típico | SO distintos, aislamiento fuerte, cargas legacy | Apps y microservicios, CI/CD, escalado rápido |

> [!warning] Un contenedor no es un sistema operativo completo
> Como un contenedor trae paquetes del SO (`apt`, `curl`, una shell...), a veces se confunde con una VM. Pero **no tiene kernel propio**: se lo presta el anfitrión. Por eso los contenedores Linux necesitan un kernel Linux, y en Windows/Mac Docker Desktop los ejecuta dentro de una pequeña VM Linux.

> [!tip] No son excluyentes
> En la nube lo habitual es ejecutar contenedores **dentro** de máquinas virtuales (los nodos de un clúster de Kubernetes son VMs). Las VMs siguen teniendo sus casos de uso; los contenedores no las sustituyen.

### Ejemplo
```bash
# Comparar tamaños: un servidor web completo frente a un SO mínimo
docker pull nginx
docker pull alpine
docker image ls
# REPOSITORY   TAG      SIZE
# nginx        latest   ~190MB
# alpine       latest   ~8MB
```

### Relacionado
- [[Contenedor]]
- [[Límites de recursos]]
- [[04 - Contenedores - ACI, AKS y Container Apps]]

---

## OCI (Open Container Initiative)

### Qué es
La **OCI** (*Open Container Initiative*) es el **estándar abierto** que define **cómo se construye una imagen de contenedor, cómo se distribuye y cómo se ejecuta**. La crearon en 2015 las principales empresas del sector: Docker, Google, Red Hat, CoreOS, entre otras.

Define tres especificaciones:
- **Image spec**: formato de la imagen (capas + manifiesto + configuración).
- **Runtime spec**: cómo se ejecuta un contenedor a partir de esa imagen (lo implementa `runc`).
- **Distribution spec**: cómo se suben y descargan imágenes de un registro.

### Para qué sirve
Gracias a OCI, **una imagen construida con Docker funciona en cualquier sitio**: Kubernetes, Google Cloud, Azure, AWS, Podman... y se puede guardar en cualquier registro compatible (Docker Hub, GitHub Container Registry, GitLab, Amazon ECR, Azure Container Registry, Google Artifact Registry).

> [!info] Docker y Kubernetes
> Kubernetes ya no usa Docker Engine como runtime (desde la 1.24 usa `containerd` o `CRI-O`), pero **las imágenes creadas con Docker siguen funcionando** porque son imágenes OCI. Ver [[Container Runtime]].

### Ejemplo
```bash
# La misma imagen se puede ejecutar con Docker o con Podman (otra herramienta compatible con OCI)
docker run -d -p 8080:80 nginx:1.27
podman run -d -p 8080:80 docker.io/library/nginx:1.27
```

### Relacionado
- [[Imagen]]
- [[Registro de imágenes]]
- [[Container Runtime]]

---

## Docker Engine vs Docker Desktop

### Qué es
- **Docker Engine**: el **motor** que crea y ejecuta contenedores. Se compone del **daemon** `dockerd` (servicio en segundo plano que gestiona imágenes, contenedores, redes y volúmenes), su API y el **cliente CLI** `docker`. Es **gratuito y sin restricciones**, pero **solo se instala en Linux** y se usa **solo por terminal**.
- **Docker Desktop**: aplicación de escritorio para **Windows, Mac y Linux** que incluye Docker Engine **más** interfaz gráfica, Docker Compose, Buildx, un clúster de Kubernetes opcional, extensiones y funciones de equipo/empresa.

### Para qué sirve
| | Docker Engine | Docker Desktop |
|---|---|---|
| Sistemas | Solo Linux | Windows, Mac, Linux |
| Interfaz | Solo CLI | CLI + GUI (contenedores, imágenes, volúmenes, builds, logs, terminal, ficheros) |
| Rendimiento | **Nativo** | Ejecuta los contenedores en una **VM Linux** ligera (incluso en Linux, para homogeneizar la experiencia) |
| Licencia | Gratis siempre | Gratis para uso personal, educativo y empresas de **menos de 250 empleados y menos de 10 M$ de facturación**; si no, suscripción de pago |
| Uso habitual | **Servidores** (desarrollo y producción) | **Equipos de desarrollo** |

> [!tip] En Linux, mejor Docker Engine
> En Linux Docker Desktop monta otra VM dentro de tu Linux. Docker Engine es más rápido (nativo) y no tiene restricciones de licencia.

> [!info] Configuración útil de Docker Desktop
> En *Settings* puedes limitar la **CPU, RAM, swap y disco virtual** que usa la VM de Docker, gestionar la compartición de ficheros, activar **containerd** como almacén de imágenes (necesario para [[Buildx y multi-arquitectura|builds multi-arquitectura]]) y activar un **clúster de Kubernetes** local.

> [!warning] Aprende primero la terminal
> En servidores y entornos productivos casi nunca tendrás interfaz gráfica. Domina la CLI; la GUI de Docker Desktop es un atajo cómodo una vez entiendes los conceptos (en *Images* incluso puedes ver los comandos con los que se construyó cada imagen).

### Ejemplo

**Windows y Mac:** descargar el instalador desde la web oficial de Docker Desktop y seguir el asistente. En Mac hay versión para Apple Silicon (chips M) y para Intel.

**Linux (Docker Engine) con el script oficial de conveniencia** (detecta la distribución e instala todo):
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Comprobar que funciona
sudo docker run hello-world

# Opcional: usar docker sin sudo (cierra sesión y vuelve a entrar después)
sudo usermod -aG docker $USER
```

> [!warning] El grupo `docker` equivale a root
> Quien pertenece al grupo `docker` puede montar `/` del host en un contenedor y obtener privilegios de root. Añade solo usuarios de confianza. Ver [[Docker en producción]].

**Alternativa por paquetes (Ubuntu):** configurar el repositorio `apt` oficial de Docker (clave GPG + repositorio) e instalar `docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`. Los pasos exactos cambian por distribución: sigue la documentación oficial *Install Docker Engine*.

### Relacionado
- [[Docker en producción]]
- [[Docker Compose]]
- [[Buildx y multi-arquitectura]]
- [[Docker - Índice]]
