## ¿Qué es?

Distribuye tráfico entrante entre varias VMs **dentro de una región**, operando en **capa 4** (TCP/UDP). No entiende HTTP, solo IP y puerto.

## Conceptos fundamentales

- **Frontend IP**: la IP (pública o privada) que recibe el tráfico.
- **Backend Pool**: grupo de VMs/instancias que reciben el tráfico repartido.
- **Health Probe**: verifica que una instancia esté sana antes de enviarle tráfico.
- **Load Balancing Rule**: define qué puerto frontend mapea a qué puerto backend.
- **SKU**: **Basic** (retirado, ya no se debe usar) vs **Standard** (el actual, con SLA 99.99%).

## ¿Cómo funciona?

1. El cliente envía tráfico a la Frontend IP.
2. El Load Balancer consulta el Health Probe de cada instancia del backend.
3. Reparte el tráfico según un algoritmo de hash de 5-tuplas (IP origen, puerto origen, IP destino, puerto destino, protocolo).
4. La conexión persiste con la misma instancia mientras dure la sesión (a menos que cambie la tupla).

## Puntos importantes

- **Público** (internet → VMs) vs **Interno/Privado** (dentro de la VNet, ej. entre capas app-db).
- Standard SKU es **secure by default**: necesitas NSG explícito, si no, nada entra.
- No hace SSL termination ni enrutamiento por URL — para eso está Application Gateway.
- Permite **outbound rules** para dar salida a internet a VMs sin IP pública (soluciona SNAT exhaustion parcialmente, aunque NAT Gateway es mejor para esto).

## Ejemplo práctico

```
Frontend: 20.50.10.5:443
Backend Pool: VM-Web1 (10.0.1.4), VM-Web2 (10.0.1.5)
Health Probe: HTTP /health cada 5s
```

Si VM-Web1 falla el probe, el Load Balancer deja de enviarle tráfico automáticamente.

## Ventajas / Desventajas

|Ventajas|Desventajas|
|---|---|
|Muy rápido (capa 4, sin inspección)|No entiende HTTP/HTTPS|
|Económico|No hace SSL offloading|
|Alta disponibilidad dentro de una región|No cruza regiones (para eso: Traffic Manager/Front Door)|

## Errores comunes

- Confundirlo con Application Gateway (capa 7) — LB es capa 4, ciego al contenido.
- Olvidar el NSG en Standard SKU y pensar que "no funciona" cuando en realidad está bloqueado por defecto.
- No configurar Health Probes correctamente, causando que el tráfico siga yendo a instancias caídas.

## Buenas prácticas

- Usa **Standard SKU** siempre (Basic está en retiro).
- Combina con **Availability Zones** para alta disponibilidad zonal.
- Usa Load Balancer **interno** para tráfico entre capas (app→db) sin exponerlo a internet.
