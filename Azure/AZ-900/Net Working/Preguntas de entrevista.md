**1. ¿Cuál es la diferencia principal entre Load Balancer y Application Gateway?**

> Load Balancer opera en capa 4 (TCP/UDP) y no entiende el contenido de la petición; solo distribuye por IP/puerto. Application Gateway opera en capa 7, entiende HTTP/HTTPS, y puede enrutar según la URL, hacer SSL termination y aplicar reglas de WAF.

**2. ¿Cuándo elegirías VPN Gateway sobre ExpressRoute, y viceversa?**

> VPN Gateway cuando necesito conectividad rápida de implementar, con cifrado nativo, para cargas moderadas o como backup. ExpressRoute cuando necesito una conexión privada, con mayor SLA y throughput consistente, típicamente para cargas de producción críticas o de gran volumen, aceptando mayor costo y tiempo de implementación.

**3. ¿Cómo mejorarías el rendimiento de un sitio web con usuarios en distintos continentes?**

> Usaría Azure CDN para cachear contenido estático (imágenes, CSS, JS) en edge nodes cercanos a cada usuario, reduciendo la latencia. Si además necesito enrutamiento inteligente global o failover entre regiones, combinaría con Azure Front Door.

**4. ¿Qué pasa si necesito cifrado sobre una conexión ExpressRoute?**

> Por defecto ExpressRoute no cifra el tráfico al no pasar por internet público. Si el compliance de la empresa exige cifrado punto a punto, se implementa IPsec sobre ExpressRoute como capa adicional.

**5. Tienes una app con backend en dos App Services distintos según la ruta de la URL (`/api` y `/web`). ¿Qué servicio usarías para enrutar el tráfico?**

> Azure Application Gateway, usando path-based routing, ya que Load Balancer no puede inspeccionar ni enrutar por la ruta de la URL al ser capa 4.

**6. ¿Qué arquitectura de alta disponibilidad usarías para la conectividad on-premises de una empresa crítica?**

> ExpressRoute como conexión primaria por su SLA y consistencia, con un VPN Gateway configurado como ruta de respaldo (failover automático) en caso de que el circuito de ExpressRoute falle.