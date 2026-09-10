## ¿Qué es?

Un gateway que crea un **túnel cifrado (IPsec/IKE) sobre internet público** entre tu VNet y otra red (on-premises, otra VNet, o un usuario remoto).

## Conceptos fundamentales

- **GatewaySubnet**: subred obligatoria y dedicada (nombre exacto), mínimo `/29`, recomendado `/27`.
- **Site-to-Site (S2S)**: conecta tu red local con Azure vía un dispositivo VPN local.
- **Point-to-Site (P2S)**: conecta un dispositivo individual (laptop) directo a la VNet.
- **VNet-to-VNet**: conecta dos VNets usando VPN Gateway en lugar de Peering (más lento, pero útil entre distintos modelos de despliegue o cuando peering no es viable).
- **SKU**: define throughput y número de túneles (Basic, VpnGw1-5, etc.).
- **Active-Active vs Active-Standby**: define si ambas instancias del gateway están activas simultáneamente.

## ¿Cómo funciona?

1. Creas la `GatewaySubnet` dentro de tu VNet.
2. Despliegas el VPN Gateway (tarda ~30-45 min en aprovisionar).
3. Configuras un **Local Network Gateway** representando el rango de IP de la red remota.
4. Estableces la conexión con una **clave compartida (PSK)**.
5. El tráfico viaja cifrado sobre internet.

## Puntos importantes

- El tráfico **pasa por internet**, aunque cifrado (a diferencia de ExpressRoute).
- Latencia y ancho de banda **no garantizados** por SLA de red dedicada.
- Puede combinarse con **BGP** para intercambio dinámico de rutas.

## Ejemplo práctico

Una oficina quiere que sus 50 empleados accedan a recursos en Azure sin exponerlos a internet → **Site-to-Site VPN** entre el router de la oficina y el VPN Gateway de Azure.

Un desarrollador remoto necesita conectarse puntualmente desde su laptop → **Point-to-Site VPN**.

## Ventajas / Desventajas

|Ventajas|Desventajas|
|---|---|
|Rápido de implementar, económico|Depende de la calidad de internet|
|Cifrado end-to-end|Menor throughput que ExpressRoute|
|Bueno para conexiones ocasionales/backup|No apto para cargas críticas de gran volumen|

## Errores comunes

- Olvidar que `GatewaySubnet` **debe** llamarse exactamente así.
- Sub-dimensionar la subred del gateway (usar `/29` cuando se planea escalar, en vez de `/27`).
- Confundir VPN Gateway con **Azure Bastion** (Bastion es para RDP/SSH seguro a VMs, no para conectar redes).

## Buenas prácticas

- Usa `/27` para `GatewaySubnet` si prevés crecer o usar Active-Active.
- Usa VPN Gateway como **respaldo** de ExpressRoute (failover automático).
- Habilita BGP si la topología de red es compleja o cambia con frecuencia.