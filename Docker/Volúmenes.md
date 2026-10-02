---
tags:
  - docker
  - storage
aliases:
  - Volúmenes
  - Volumen Docker
  - Docker volume
  - docker volume
  - Bind mount
  - Persistencia de datos
  - docker cp
---

# Volúmenes

- [[#Volúmenes con nombre]]
- [[#Bind mount]]
- [[#Copiar archivos con docker cp]]
- [[#Copias de seguridad de volúmenes]]

---

## Volúmenes con nombre

### Qué es
Los contenedores son **efímeros**: lo que escriben en su propia capa se pierde al borrarlos. Un **volumen** es un **directorio gestionado por Docker, fuera del sistema de archivos del contenedor**, que se **monta** en una ruta del contenedor. No es un disco virtual: es un **punto de montaje** (en Linux vive en `/var/lib/docker/volumes/<nombre>/_data`).

### Para qué sirve
- **Persistir datos** aunque el contenedor se borre o se recree: bases de datos, ficheros subidos por usuarios, resultados de un script.
- **Compartir datos** entre varios contenedores que montan el mismo volumen.
- Desacoplar el ciclo de vida de los datos del ciclo de vida del contenedor (actualizar la imagen de PostgreSQL sin perder la base de datos).

> [!example] ¿Por qué hace falta?
> Si levantas `postgres` sin volumen, creas una base de datos y tablas, y luego borras el contenedor, **todo se pierde**. Con un volumen montado en la ruta de datos de PostgreSQL, el nuevo contenedor encuentra los datos intactos.

> [!warning] Escritura concurrente
> Un volumen puede montarse en varios contenedores a la vez, pero Docker no coordina la escritura: evita que dos contenedores escriban los mismos ficheros (dos bases de datos sobre el mismo volumen lo corromperían).

> [!warning] Protección al borrar
> `docker volume rm` falla si algún contenedor (incluso parado) usa el volumen. `docker volume prune` borra los volúmenes **sin uso**: comprueba antes que no contienen datos que necesites.

> [!tip] La ruta correcta de cada imagen
> Monta el volumen exactamente donde la aplicación guarda los datos; viene en la documentación de la imagen. Ejemplos: MySQL/MariaDB `/var/lib/mysql`, PostgreSQL hasta la 17 `/var/lib/postgresql/data` (desde la imagen 18 se recomienda montar `/var/lib/postgresql`), WordPress `/var/www/html`, MongoDB `/data/db`.

### Ejemplo
```bash
# Crear, listar e inspeccionar
docker volume create mi-volumen
docker volume ls
docker volume inspect mi-volumen       # Mountpoint, driver, fecha...

# Montar el volumen en /datos y escribir algo
docker run -it --name c1 -v mi-volumen:/datos ubuntu bash
#   root@c1:/# touch /datos/test.txt && exit
docker rm c1

# Otro contenedor ve el mismo fichero
docker run --rm -it -v mi-volumen:/datos ubuntu ls /datos     # test.txt

# Sintaxis larga equivalente (más explícita)
docker run -d --mount type=volume,source=mi-volumen,target=/datos nginx

# PostgreSQL con datos persistentes
docker run -d --name db -e POSTGRES_PASSWORD=secreto \
  -v pgdata:/var/lib/postgresql/data postgres:16

# Borrar
docker volume rm mi-volumen
docker volume prune
```

### Relacionado
- [[Docker Compose#Volúmenes y redes en Compose]]
- [[Volume|Volume (Kubernetes)]]
- [[PersistentVolume]]

---

## Bind mount

### Qué es
Un **bind mount** monta **una ruta concreta del host** (absoluta o relativa) dentro del contenedor. Se distingue de un volumen en que la parte izquierda de `-v` es una **ruta** en lugar de un nombre.

### Para qué sirve
- **Desarrollo**: editar el código en tu equipo con tu IDE y que el contenedor lo vea **en vivo**, usando las herramientas del contenedor sin "ensuciar" tu sistema con instalaciones (es la base de los **Dev Containers**).
- Inyectar ficheros de configuración (`nginx.conf`, certificados).
- Sacar resultados de un proceso directamente a una carpeta del host.

| | Volumen con nombre | Bind mount |
|---|---|---|
| Sintaxis | `-v mi-volumen:/datos` | `-v ./src:/app` o `-v /home/yo/src:/app` |
| Quién gestiona la ubicación | Docker | Tú |
| Portabilidad | Alta (no depende de rutas del host) | Depende de la estructura del host |
| Uso típico | Datos de producción, bases de datos | Código en desarrollo, configuración |
| Rendimiento en Docker Desktop (Mac/Windows) | Mejor | Más lento (cruza la frontera de la VM) |

> [!tip] Montajes de solo lectura
> Añade `:ro` para que el contenedor no pueda modificar los ficheros: `-v ./nginx.conf:/etc/nginx/nginx.conf:ro`.

### Ejemplo
```bash
# Montar la carpeta actual en /datos de un Ubuntu
docker run --rm -it -v "$(pwd)":/datos ubuntu bash

# Servir una web local con nginx y ver los cambios al recargar
docker run -d -p 8080:80 -v ./web:/usr/share/nginx/html:ro nginx

# PowerShell en Windows
docker run --rm -it -v "${PWD}:/datos" ubuntu bash
```

### Relacionado
- [[Dockerfile#docker build y la caché]]

---

## Copiar archivos con docker cp

### Qué es
`docker cp` copia ficheros o carpetas **entre el host y un contenedor**, en cualquier dirección, sin necesidad de volúmenes y **sin parar el contenedor**.

### Para qué sirve
Extraer en caliente un log, un fichero generado o un volcado, o meter un fichero puntual en un contenedor ya en marcha.

> [!tip] El orden importa
> Siempre `docker cp ORIGEN DESTINO`; el lado del contenedor se escribe `contenedor:/ruta`.

### Ejemplo
```bash
# Del contenedor al host (a la carpeta actual)
docker cp mi-contenedor:/datos/test.txt .

# Del host al contenedor
docker cp ./config.json mi-contenedor:/app/config.json
```

> [!info] Desde Docker Desktop
> En la ficha de un contenedor, la pestaña **Files** permite navegar por su sistema de archivos, ver qué ficheros se han **añadido o modificado** respecto a la imagen, editarlos, guardarlos e importarlos. La sección **Volumes** permite ver, clonar, vaciar y borrar volúmenes.

### Relacionado
- [[Contenedor#Modos de ejecución attached, detached e interactivo|docker exec]]

---

## Copias de seguridad de volúmenes

### Qué es
Técnicas para **exportar el contenido de un volumen** y poder restaurarlo en otro servidor.

### Para qué sirve
Backups de bases de datos y migraciones. Docker Desktop permite **exportar un volumen** a un `.tar.gz` local, a una **imagen local** o directamente a un **registro** (Docker Hub u otro): el backup se maneja como una imagen de contenedor más, con su versionado y distribución.

> [!warning] Consistencia de bases de datos
> Copiar los ficheros de una base de datos **en caliente** puede dar una copia inconsistente. Para datos críticos, para el contenedor antes de copiar o usa la herramienta nativa (`pg_dump`, `mysqldump`).

### Ejemplo

**Backup y restauración con la CLI (patrón clásico con un contenedor auxiliar):**
```bash
# Backup: monta el volumen y la carpeta actual, y empaqueta los datos
docker run --rm -v pgdata:/datos -v "$(pwd)":/backup alpine \
  tar czf /backup/pgdata-backup.tar.gz -C /datos .

# Restaurar en un volumen nuevo
docker volume create pgdata-restore
docker run --rm -v pgdata-restore:/datos -v "$(pwd)":/backup alpine \
  tar xzf /backup/pgdata-backup.tar.gz -C /datos

# Volcado lógico de PostgreSQL
docker exec db pg_dump -U postgres midb > midb.sql
```

### Relacionado
- [[Registro de imágenes#docker save y docker load]]
- [[Docker en producción]]
