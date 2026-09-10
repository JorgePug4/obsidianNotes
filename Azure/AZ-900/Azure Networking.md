# 1. Azure Load Balancer

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

---

# 2. VPN Gateway

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

---

# 3. Azure Application Gateway

## ¿Qué es?

Un **balanceador de carga de capa 7 (HTTP/HTTPS)**, entiende el contenido de la petición web (URL, headers, cookies) y enruta según eso.

## Conceptos fundamentales

- **Listener**: escucha en un puerto/protocolo específico (HTTP/HTTPS).
- **Routing Rule**: decide a qué backend mandar según la regla (path-based o básica).
- **Backend Pool**: destino final (VMs, VMSS, App Service, IPs).
- **WAF (Web Application Firewall)**: SKU que agrega protección contra OWASP Top 10 (SQL injection, XSS, etc.).
- **SSL Termination**: descifra HTTPS en el Gateway, así las instancias backend no cargan con ese trabajo.
- **Cookie-based session affinity**: mantiene a un cliente conectado a la misma instancia backend.

## ¿Cómo funciona?

1. El cliente hace una petición HTTPS.
2. El listener la recibe y hace **SSL termination** (opcional).
3. La routing rule inspecciona la URL (ej. `/imagenes/*` → backend pool A, `/api/*` → backend pool B).
4. Envía la petición al backend correspondiente.

## Puntos importantes

- Capa 7 = **entiende contenido web**, a diferencia de Load Balancer (capa 4).
- **Path-based routing**: enruta según la ruta de la URL, algo que un Load Balancer normal no puede hacer.
- Con **WAF SKU**, protege contra ataques comunes sin tocar el código de la app.
- Autoescala según el SKU (v2).

## Ejemplo práctico

```
/tienda/*  → Backend Pool "eCommerce"
/blog/*    → Backend Pool "CMS"
```

Un solo Application Gateway enruta a dos aplicaciones distintas según la URL, algo imposible con Load Balancer básico.

## Comparación clave (cae seguido en exámenes/entrevistas)

||Load Balancer|Application Gateway|
|---|---|---|
|Capa OSI|4 (TCP/UDP)|7 (HTTP/HTTPS)|
|Enruta por URL|No|Sí|
|SSL Termination|No|Sí|
|WAF|No|Sí (SKU WAF)|
|Alcance|Regional|Regional|

## Ventajas / Desventajas

|Ventajas|Desventajas|
|---|---|
|Enrutamiento inteligente por contenido|Más caro que Load Balancer|
|WAF integrado|Solo protocolos web (HTTP/HTTPS/WebSocket)|
|SSL offloading|Mayor latencia que capa 4|

## Errores comunes

- Usarlo pensando que sirve para tráfico no-web (TCP genérico) — para eso es Load Balancer.
- Confundir Application Gateway (regional) con **Front Door** (global, capa 7 también, pero para múltiples regiones).
- No habilitar WAF pensando que el Gateway ya protege contra ataques por defecto.

## Buenas prácticas

- Usa **WAF SKU** si expones aplicaciones públicas críticas.
- Combina con **Front Door** cuando necesites failover/enrutamiento global entre regiones.
- Usa path-based routing para arquitecturas de microservicios detrás de un solo dominio.

---

# 4. Azure Content Delivery Network (CDN)

## ¿Qué es?

Red de servidores distribuidos globalmente (**edge nodes / PoPs**) que **cachean contenido estático** cerca del usuario final, reduciendo latencia.

## Conceptos fundamentales

- **Edge node / PoP (Point of Presence)**: servidor físico distribuido geográficamente que sirve el contenido cacheado.
- **Origin**: la fuente real del contenido (Storage Account, App Service, Web App).
- **Caching rules**: definen cuánto tiempo se guarda el contenido en el edge (TTL).
- **Purge**: fuerza a limpiar la caché antes de que expire el TTL.

## ¿Cómo funciona?

1. El usuario pide un archivo (ej. una imagen).
2. Si el edge node más cercano ya lo tiene cacheado → lo entrega directo (**cache hit**), rápido.
3. Si no lo tiene (**cache miss**) → lo pide al origen, lo entrega al usuario, y lo guarda en caché para la próxima vez.

## Puntos importantes

- Ideal para **contenido estático**: imágenes, CSS, JS, videos, descargas.
- **No sirve para contenido dinámico personalizado** por usuario (aunque hay configuraciones avanzadas para ciertos casos).
- Reduce carga en el origen y mejora la experiencia para usuarios geográficamente distantes.
- Puede combinarse con HTTPS personalizado y compresión automática.

## Ejemplo práctico

Una tienda online con clientes en Europa y Asia, pero su Storage Account está en East US. Sin CDN, un usuario en Tokio tiene alta latencia al cargar imágenes. Con CDN, la imagen se cachea en un PoP cercano a Tokio y carga mucho más rápido.

## Ventajas / Desventajas

|Ventajas|Desventajas|
|---|---|
|Reduce latencia globalmente|No apto para contenido altamente dinámico|
|Reduce carga en el origen|Requiere gestión de invalidación de caché (purge)|
|Mejora experiencia de usuario|Costo adicional por transferencia de datos|

## Errores comunes

- Usar CDN para contenido que cambia constantemente sin configurar bien el TTL → usuarios ven contenido desactualizado.
- Olvidar hacer **purge** después de actualizar un archivo crítico en el origen.
- Pensar que CDN sirve como balanceador de carga — no lo es, es una capa de caching/distribución de contenido.

## Buenas prácticas

- Configura TTLs adecuados según qué tan frecuente cambia el contenido.
- Usa versión en el nombre del archivo (`logo.v2.png`) en vez de depender solo de purge.
- Habilita compresión y HTTPS en el perfil del CDN.

---

# 5. Azure ExpressRoute

## ¿Qué es?

Una conexión **privada y dedicada** entre tu red on-premises y Azure, que **no pasa por internet público**.

## Conceptos fundamentales

- **Peering Location**: el punto físico de conexión (proveedor de conectividad) donde se establece el enlace.
- **Private Peering**: para conectar tu red interna con recursos en VNets de Azure.
- **Microsoft Peering**: para acceder a servicios públicos de Microsoft (Microsoft 365, Azure PaaS públicos) sin pasar por internet.
- **Circuit**: el circuito lógico que representa la conexión contratada.
- **ExpressRoute Gateway**: el recurso dentro de la VNet que termina la conexión (similar rol a VPN Gateway pero para ExpressRoute).
- **FastPath**: opción que evita el salto por el Gateway para reducir latencia.
- **Global Reach**: conecta dos sitios on-premises entre sí _a través_ de la red de Microsoft usando sus circuitos ExpressRoute.

## ¿Cómo funciona?

1. Contratas conectividad con un proveedor certificado (colocation, punto a punto, o Ethernet).
2. Se crea un **circuito ExpressRoute** en Azure.
3. Configuras peering (Private y/o Microsoft) según qué necesites alcanzar.
4. Despliegas un ExpressRoute Gateway en tu VNet para terminar la conexión.
5. El tráfico viaja por la red privada del proveedor, sin tocar internet público.

## Puntos importantes

- **No usa internet público** → mayor seguridad, menor latencia, mayor consistencia.
- SLA más alto que VPN Gateway (hasta 99.95%+).
- No está cifrado por defecto (a diferencia de VPN) — si necesitas cifrado sobre ExpressRoute, se combina con **IPsec sobre ExpressRoute**.
- Es el estándar para cargas críticas empresariales de alto volumen.

## Ejemplo práctico

Un banco necesita mover grandes volúmenes de datos entre su datacenter y Azure de forma consistente y con SLA garantizado, sin exponerse a la variabilidad de internet → **ExpressRoute** con Private Peering.

## Comparación clave: VPN Gateway vs ExpressRoute

||VPN Gateway|ExpressRoute|
|---|---|---|
|Medio|Internet público|Red privada dedicada|
|Cifrado|Sí, nativo (IPsec)|No por defecto|
|Latencia/Consistencia|Variable|Predecible|
|Costo|Menor|Mayor|
|Implementación|Rápida (minutos-horas)|Lenta (semanas, depende del proveedor)|
|Uso típico|Backup, cargas moderadas, PoC|Producción crítica, alto volumen|

**Buena práctica combinada**: usar VPN Gateway como **failover** de ExpressRoute.

## Ventajas / Desventajas

|Ventajas|Desventajas|
|---|---|
|Latencia baja y predecible|Costo alto|
|No pasa por internet|Tiempo de aprovisionamiento largo|
|SLA alto|Sin cifrado nativo (requiere config extra)|

## Errores comunes

- Asumir que ExpressRoute está cifrado por defecto — no lo está.
- No configurar un VPN Gateway de respaldo, dejando un único punto de falla.
- Confundir Private Peering (acceso a tus VNets) con Microsoft Peering (acceso a servicios públicos de Microsoft).

## Buenas prácticas

- Combina ExpressRoute con **VPN Gateway como failover** automático.
- Usa **FastPath** para cargas sensibles a latencia.
- Si el compliance exige cifrado, implementa **IPsec sobre ExpressRoute**.

---

# Resumen para mi libreta

- **Load Balancer**: capa 4, dentro de una región, reparte tráfico TCP/UDP entre VMs.
- **VPN Gateway**: túnel cifrado sobre internet, conecta on-prem↔Azure o VNet↔VNet. Requiere `GatewaySubnet`.
- **Application Gateway**: capa 7, entiende HTTP, enruta por URL, hace SSL termination, opción WAF.
- **CDN**: cachea contenido estático en edge nodes cercanos al usuario, reduce latencia global.
- **ExpressRoute**: conexión privada dedicada, no pasa por internet, mayor SLA, sin cifrado nativo.
- Combo típico en producción: **ExpressRoute (primario) + VPN Gateway (failover)**.
- Combo típico web: **Application Gateway (WAF, capa 7 regional) + Front Door/CDN (global)**.

---

# Preguntas de entrevista

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

---

# 🧠 Lo que debo recordar

1. Load Balancer = capa 4, regional, sin inteligencia de contenido.
2. Application Gateway = capa 7, entiende HTTP, path-based routing, SSL termination, WAF opcional.
3. VPN Gateway = túnel cifrado sobre internet, requiere `GatewaySubnet`, ideal para backup o cargas moderadas.
4. ExpressRoute = conexión privada dedicada, sin cifrado nativo, mayor SLA, para cargas críticas.
5. CDN = cachea contenido estático en edge nodes, reduce latencia para usuarios distantes.
6. El combo de alta disponibilidad para on-premises: **ExpressRoute + VPN Gateway de respaldo**.
7. Application Gateway y Load Balancer son ambos regionales; para alcance global se usa Front Door o Traffic Manager.
8. WAF solo existe en Application Gateway (y Front Door), no en Load Balancer.
9. ExpressRoute tiene dos tipos de peering: Private (VNets) y Microsoft (servicios públicos de MS).