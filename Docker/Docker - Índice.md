---
tags:
  - docker
  - MOC
aliases:
  - Índice Docker
  - Curso Docker
  - MOC Docker
---

# 🐳 Docker - Índice

> [!info] Origen
> Notas generadas a partir del cuaderno de NotebookLM **"🐳 Curso Docker - Aprende sobre contenedores, de cero a experto"**: el curso de Docker en español de **pabpereza** (episodios 1-17, documentación escrita en *pabpereza.dev*), complementado con la serie *Docker - Guía práctica de uso para desarrolladores* (Postgres, MariaDB, logs, variables de entorno). Se han corregido y ampliado algunos detalles con la documentación oficial de Docker.

## Mapa de notas

```text
🐳 Docker
├── 🧱 Fundamentos
│   ├── [[Docker]]                      qué es · contenedor vs VM · OCI · Engine vs Desktop · instalación
│   ├── [[Imagen]]                      capas · tags y latest · gestión local
│   └── [[Contenedor]]                  docker run · -d/-it · exec · puertos · restart · logs · ciclo de vida
├── 🏗️ Construcción y distribución
│   ├── [[Dockerfile]]                  instrucciones · build y caché · ENTRYPOINT vs CMD · ARG · ENV
│   ├── [[Multi-stage y Distroless]]    imágenes pequeñas y seguras
│   ├── [[Buildx y multi-arquitectura]] amd64 / arm64
│   └── [[Registro de imágenes]]        Docker Hub · tag/push · commit · save/load
├── 💾 Datos y red
│   ├── [[Volúmenes]]                   volúmenes · bind mounts · docker cp · backups
│   └── [[Redes en Docker]]             drivers · connect/disconnect · DNS interno
├── 🎼 Orquestación
│   ├── [[Docker Compose]]              multi-contenedor · dockerizar una app
│   └── [[Docker Swarm]]                servicios · réplicas · rolling update · stacks
├── 🚀 Operación
│   ├── [[Límites de recursos]]         CPU · memoria · reservas · docker update
│   └── [[Docker en producción]]        despliegue · healthchecks · .env · hardening · buenas prácticas
└── 📋 [[Docker - Cheatsheet de comandos]]
```

## Ruta de estudio (siguiendo el curso)

| # | Episodio del curso | Nota |
|---|---|---|
| 1 | Introducción, Docker vs VMs, OCI, casos de uso | [[Docker]] |
| 2 | Instalación de Docker Desktop y Engine | [[Docker#Docker Engine vs Docker Desktop]] |
| 3 | Conceptos básicos: contenedor, imagen, Dockerfile, Docker Hub | [[Imagen]] · [[Contenedor]] · [[Registro de imágenes]] |
| 4 | Ejecución de contenedores: run, logs, puertos, procesos | [[Contenedor]] |
| 5 | Gestión de contenedores: exec, conflictos, rm, prune | [[Contenedor#Ciclo de vida]] |
| 6 | Imágenes: pull, capas, rmi, search, prune | [[Imagen]] |
| 7 | Dockerfile y docker build, optimizar la caché | [[Dockerfile]] |
| 8 | ENTRYPOINT vs CMD, ARG y variables de entorno | [[Dockerfile#ENTRYPOINT vs CMD]] |
| 9 | Gestión de imágenes: commit, save, load, tag, push | [[Registro de imágenes]] |
| 10 | Volúmenes, montaje de archivos y backups | [[Volúmenes]] |
| 11 | Redes y segregación | [[Redes en Docker]] |
| 12 | Docker Compose | [[Docker Compose]] |
| 13 | Docker Swarm: servicios, balanceo, updates y rollbacks | [[Docker Swarm]] |
| 14 | Tu primera app en un contenedor | [[Docker Compose#Dockerizar una aplicación (ejemplo completo)]] |
| 15 | Puesta en producción, seguridad y buenas prácticas | [[Docker en producción]] |
| 16 | Límites de CPU y memoria | [[Límites de recursos]] |
| 17 | Multi-arquitectura con buildx | [[Buildx y multi-arquitectura]] |
| + | Multi-stage y Distroless | [[Multi-stage y Distroless]] |

## Conceptos clave en una frase

- **Imagen** = plantilla inmutable por capas. **Contenedor** = proceso aislado creado a partir de una imagen. **Dockerfile** = receta para construir la imagen. **Registro** = almacén de imágenes.
- Un contenedor **comparte el kernel** del host (namespaces + cgroups); una VM trae su propio SO.
- Los contenedores son **efímeros**: los datos que importan van en **volúmenes**.
- La configuración que cambia entre entornos va en **variables de entorno**, nunca en la imagen.
- **Compose** = varios contenedores en un servidor; **Swarm** / **Kubernetes** = varios servidores.

## 🎯 Preguntas típicas de entrevista

- *"¿Diferencia entre imagen y contenedor?"* → La imagen es la plantilla inmutable; el contenedor, una instancia en ejecución (un proceso) con una capa de escritura propia.
- *"¿Contenedor vs máquina virtual?"* → La VM virtualiza hardware y tiene su propio kernel; el contenedor virtualiza el SO y comparte el kernel. Más ligero y rápido, aislamiento menor.
- *"¿ENTRYPOINT vs CMD?"* → ENTRYPOINT fija el ejecutable; CMD da argumentos/comando por defecto que se sobrescriben en `docker run`.
- *"¿Cómo reduces el tamaño de una imagen?"* → Multi-stage, imagen base mínima (slim/alpine/distroless), `.dockerignore`, limpiar cachés de paquetes, combinar `RUN`.
- *"¿Cómo aceleras los builds?"* → Ordenar capas de menos a más cambiantes (dependencias antes que código) para aprovechar la caché.
- *"¿Cómo persistes datos?"* → Volúmenes con nombre (o bind mounts en desarrollo).
- *"¿Cómo se comunican dos contenedores?"* → En la misma red definida por el usuario, por nombre de contenedor/servicio gracias al DNS interno.
- *"¿Por qué no usar `latest`?"* → No es "la última" garantizada y cambia con el tiempo: builds no reproducibles.
- *"¿ARG vs ENV?"* → ARG solo existe en build; ENV también en ejecución. Ninguno es seguro para secretos.
- *"¿Compose vs Swarm vs Kubernetes?"* → Un host vs clúster sencillo vs clúster con ecosistema completo.

## Relacionado

- [[Container Runtime]] · [[Workloads]] · [[Helm]] (Kubernetes)
- [[04 - Contenedores - ACI, AKS y Container Apps]] (AZ-900)
- [[13 - Azure Container Registry]] · [[14 - Azure Container Instances]] · [[15 - Azure Container Apps]] · [[16 - Comparación de servicios de contenedores]] (AZ-104)
- [[🏗️ Diseño de Microservicios]]

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
