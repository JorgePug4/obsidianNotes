---
tags:
  - docker
  - networking
aliases:
  - Redes en Docker
  - Docker network
  - docker network
  - Bridge
  - Red bridge
  - Driver de red
  - Overlay
  - Macvlan
---

# Redes en Docker

- [[#Redes y drivers]]
- [[#Gestión de redes]]
- [[#Resolución por nombre (DNS interno)]]

---

## Redes y drivers

### Qué es
Docker crea **redes virtuales** para conectar contenedores entre sí y con el exterior. Cada red usa un **driver** que decide cómo se comporta:

| Driver | Comportamiento | Uso típico |
|---|---|---|
| **`bridge`** (defecto) | Red virtual privada dentro del host. Los contenedores de la misma red se ven entre sí; hacia fuera solo con puertos publicados | El 90 % de los casos |
| **`host`** | El contenedor **comparte la red del host**, sin aislamiento: sus puertos son directamente los del host | Rendimiento de red máximo, herramientas de red |
| **`overlay`** | Red que **abarca varios hosts** | [[Docker Swarm]] y cargas distribuidas |
| **`macvlan`** | Asigna al contenedor una **dirección MAC** propia: aparece como un dispositivo más en la red física | Integración con redes heredadas |
| **`none`** | Sin red | Aislamiento total |

Docker crea por defecto tres redes que **no se pueden borrar**: `bridge`, `host` y `none`. Si no indicas red, el contenedor se conecta a la `bridge` por defecto.

### Para qué sirve
Un servidor potente suele alojar muchos servicios que no tienen nada que ver entre sí. Las redes permiten:
- **Conectar** lo que debe hablar (WordPress con su base de datos).
- **Segregar** por seguridad lo que no (dos webs de clientes distintos no deben verse entre ellas).

> [!warning] Cuidado con `host`
> El contenedor queda fuera del aislamiento de red: cualquier puerto que abra queda expuesto en el host, posiblemente de forma **pública** y sin control.

> [!tip] Una red por aplicación
> En un servidor compartido, crea una red para cada aplicación y conecta solo sus contenedores. Es la misma idea que las *NetworkPolicies* en Kubernetes o los NSG en Azure.

### Ejemplo
```bash
docker network ls
# NETWORK ID     NAME      DRIVER    SCOPE
# 1a2b3c...      bridge    bridge    local
# 4d5e6f...      host      host      local
# 7a8b9c...      none      null      local

docker network inspect bridge    # subred, gateway, contenedores conectados
```

### Relacionado
- [[Contenedor#Publicación de puertos]]
- [[Networking|Networking en Kubernetes]]

---

## Gestión de redes

### Qué es
Comandos `docker network ...` para crear, conectar, desconectar y borrar redes, **incluso en caliente** con los contenedores en marcha.

### Para qué sirve
Construir la topología de red de tus aplicaciones. Un contenedor puede estar conectado a **una o varias redes** a la vez (cada una le da una IP en su rango).

> [!warning] La red por defecto sigue conectada
> Si lanzas un contenedor sin `--network` y **después** lo conectas a otra red, sigue conectado también a la `bridge` por defecto. Si buscas segregar, desconéctalo de ella con `docker network disconnect bridge <contenedor>`, o mejor, indica la red desde el `docker run`.

> [!warning] Protección al borrar
> No se puede borrar una red con contenedores conectados (*active endpoints*). `docker network prune` borra las redes **personalizadas** sin uso; las de Docker (`bridge`, `host`, `none`) nunca se borran.

### Ejemplo
```bash
# Crear una red (bridge por defecto)
docker network create red-web
docker network create --driver bridge --subnet 172.30.0.0/16 red-backend

# Lanzar un contenedor directamente en esa red
docker run -d --name web --network red-web nginx

# Un Ubuntu en la red por defecto NO llega a "web"
docker run -it --name cliente ubuntu bash
#   apt update && apt install -y curl && curl web   -> falla

# Conectarlo en caliente a red-web (desde otra terminal)
docker network connect red-web cliente
#   curl web   -> devuelve la página de nginx

# Desconectar
docker network disconnect red-web cliente

# Ver a qué redes está conectado un contenedor y con qué IP
docker inspect cliente --format '{{json .NetworkSettings.Networks}}'

# Borrar
docker rm -f web cliente
docker network rm red-web
docker network prune
```

### Relacionado
- [[Docker Compose#Volúmenes y redes en Compose]]

---

## Resolución por nombre (DNS interno)

### Qué es
En las **redes definidas por el usuario** (las que creas tú y las que crea [[Docker Compose]]), Docker ofrece un **DNS interno**: cada contenedor es accesible **por su nombre** (y en Compose, **por el nombre del servicio**).

### Para qué sirve
Configurar conexiones entre contenedores **sin IPs**, que cambian en cada recreación. Tu API se conecta a `postgres:5432` o `redis:6379` en lugar de a `172.18.0.3`.

> [!warning] La `bridge` por defecto no tiene DNS
> En la red `bridge` por defecto los contenedores solo se ven **por IP** (el antiguo `--link` está obsoleto). Es otro motivo para crear siempre tus propias redes.

> [!important] `localhost` dentro de un contenedor es el propio contenedor
> Si tu app en un contenedor apunta a `localhost:5432`, busca PostgreSQL **dentro de sí misma**. Usa el nombre del servicio/contenedor. Para llegar a un servicio del **host** desde un contenedor, en Docker Desktop existe `host.docker.internal`.

### Ejemplo
```bash
docker network create app-net
docker run -d --name db --network app-net -e POSTGRES_PASSWORD=secreto postgres:16
docker run --rm --network app-net postgres:16 pg_isready -h db -p 5432
# db:5432 - accepting connections
```

### Relacionado
- [[Docker Compose]]
- [[Service Discovery]]
