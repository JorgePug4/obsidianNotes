---
tags:
  - docker
  - operations
aliases:
  - Límites de recursos
  - Resource limits
  - Límites de CPU y memoria
  - Reservations
  - docker update
  - docker stats
  - cgroups
---

# Límites de recursos

### Qué es
Por defecto un contenedor **puede usar toda la CPU y la memoria del host**, y Docker reparte los recursos de forma equitativa entre los contenedores. Docker permite **limitar** (*limits*) y **reservar** (*reservations*) CPU y memoria por contenedor, usando los **cgroups** del kernel.

- **Límite**: el máximo que puede consumir. Si un contenedor supera su **límite de memoria**, el kernel lo mata (**OOM kill**, código de salida 137). Si alcanza su **límite de CPU**, se le **ralentiza** (*throttling*), no se le mata.
- **Reserva**: el mínimo que necesita para funcionar.

### Para qué sirve
Evitar que **un contenedor devore el servidor** y tumbe al resto. Caso real: probar en un servidor personal una carga de IA que consumía toda la CPU y RAM hizo caer el resto de webs alojadas en él.

- **Limitar** protege a los vecinos: un fallo (fuga de memoria, bucle infinito) se queda en su contenedor.
- **Reservar** garantiza que una aplicación exigente (por ejemplo, una JVM que necesita al menos 300-512 MB) tenga sus recursos mínimos.
- También se puede asignar **GPU** a contenedores (`--gpus`).

> [!info] Cómo actúan las reservas
> - En **Swarm**, el planificador solo coloca la tarea en un nodo que tenga **libres** los recursos reservados; si ninguno los tiene, la tarea queda pendiente.
> - Con `docker run --memory-reservation` es un **límite blando**: el contenedor puede superarlo, pero cuando el host está bajo presión de memoria Docker intenta devolverlo a esa cifra.

> [!tip] Mide antes de limitar
> Usa `docker stats` (o el dashboard de Docker Desktop) para ver el consumo real y fija límites con margen. Un límite de memoria demasiado bajo provoca reinicios constantes por OOM.

### Ejemplo

**Por CLI al crear el contenedor:**
```bash
# Como máximo 1 CPU y 512 MB de RAM
docker run -d --name web --cpus 1 --memory 512m nginx

# Probar el límite con una herramienta de estrés que intenta usar 6 CPUs
docker run -d --name stress --cpus 3 progrium/stress --cpu 6
docker stats stress          # no pasa del ~300 % (3 núcleos)

# Reserva blanda de memoria y límite de swap
docker run -d --memory 1g --memory-reservation 512m --memory-swap 1g mi-app
```

**Modificar un contenedor en marcha (`docker update`):**
```bash
docker update --cpus 6 stress      # subir el límite
docker update --cpus 1 stress      # bajarlo: en el dashboard pasa a ~100 %
docker update --memory 1g --memory-swap 1g web
docker update --restart unless-stopped web
```

**En Docker Compose (`deploy.resources`):**
```yaml
services:
  backend:
    build: .
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 1G
        reservations:
          cpus: "0.5"
          memory: 200M
```
```bash
docker compose up -d     # crea o actualiza los contenedores con los nuevos límites
```

**Monitorizar:**
```bash
docker stats                       # consumo en vivo de todos los contenedores
docker stats --no-stream           # una sola foto
docker inspect web --format '{{.HostConfig.NanoCpus}} {{.HostConfig.Memory}}'
```

### Relacionado
- [[Docker en producción]]
- [[Docker Swarm#Stacks]]
- [[Bulkhead]]
- [[Workloads#Pod|requests y limits en Kubernetes]]
