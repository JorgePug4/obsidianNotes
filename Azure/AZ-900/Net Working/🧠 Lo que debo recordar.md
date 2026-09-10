1. Load Balancer = capa 4, regional, sin inteligencia de contenido.
2. Application Gateway = capa 7, entiende HTTP, path-based routing, SSL termination, WAF opcional.
3. VPN Gateway = túnel cifrado sobre internet, requiere `GatewaySubnet`, ideal para backup o cargas moderadas.
4. ExpressRoute = conexión privada dedicada, sin cifrado nativo, mayor SLA, para cargas críticas.
5. CDN = cachea contenido estático en edge nodes, reduce latencia para usuarios distantes.
6. El combo de alta disponibilidad para on-premises: **ExpressRoute + VPN Gateway de respaldo**.
7. Application Gateway y Load Balancer son ambos regionales; para alcance global se usa Front Door o Traffic Manager.
8. WAF solo existe en Application Gateway (y Front Door), no en Load Balancer.
9. ExpressRoute tiene dos tipos de peering: Private (VNets) y Microsoft (servicios públicos de MS).
10. Una VNet es regional y de una sola suscripción; las subredes viven dentro de ella.
11. Cada subred pierde 5 IPs por reservas de Azure (calcula siempre con eso en mente).
12. NSG es stateful y se evalúa por prioridad; el orden de evaluación cambia según sea entrante o saliente.
13. El Peering conecta VNets directamente pero **no es transitivo** — para eso se usa Hub and Spoke.
14. Precedencia de ruteo: UDR > BGP > rutas del sistema, y siempre gana el prefijo más específico.
15. Private Endpoint = IP privada real y accesible on-premises; Service Endpoint = optimización sobre IP pública.
16. La salida a internet ya no es automática por defecto: hay que configurarla explícitamente (idealmente con NAT Gateway).
17. `168.63.129.16` es la IP de infraestructura de Azure — jamás bloquearla.
18. IP Forwarding en la NIC es obligatorio para que una NVA pueda reenviar tráfico.
19. Network Watcher es la herramienta de diagnóstico oficial: úsala antes de adivinar.