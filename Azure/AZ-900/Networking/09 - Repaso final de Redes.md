---
tags: [az-900, azure, redes, repaso]
modulo: Redes
---

# 🎯 Repaso final de Redes (AZ-900)

## Conceptos más importantes

- **Virtual Network (VNet):** red privada en Azure, en una región y una suscripción, dividida en **subredes**. Ver [[01 - Azure Virtual Network]].
- **Espacio de direcciones:** rango CIDR privado; menor sufijo = más IPs; Azure reserva 5 por subred. Ver [[02 - Espacio de direcciones y subredes]].
- **Peering:** conecta VNets (regional o global) por el backbone de Microsoft, sin Internet. No es transitivo. Los rangos no pueden solaparse.
- **NSG:** firewall básico Allow/Deny por IP, puerto y protocolo; se asocia a subred o NIC; menor prioridad numérica gana. Ver [[03 - Grupo de seguridad de red (NSG)]].
- **Load Balancer:** balanceo capa 4 (TCP/UDP), regional, público o interno. Ver [[04 - Azure Load Balancer]].
- **VPN Gateway:** túnel cifrado por Internet (IPsec). S2S, P2S, VNet-to-VNet. Vive en GatewaySubnet. Ver [[05 - Azure VPN Gateway]].
- **Application Gateway:** balanceo capa 7 (HTTP/HTTPS), enrutamiento por URL, terminación SSL, WAF. Regional. Ver [[06 - Azure Application Gateway]].
- **CDN:** caché global de contenido estático en puntos de presencia. Ver [[07 - Azure Content Delivery Network]].
- **ExpressRoute:** conexión privada dedicada sin Internet, alto ancho de banda, vía proveedor. Ver [[08 - Azure ExpressRoute]].
- **Azure DNS:** hospeda zonas DNS y resuelve nombres; no vende dominios.

## Tabla de servicios y propósito

| Servicio | Propósito en una frase | Alcance | Capa | Palabra clave |
|---|---|---|---|---|
| **Virtual Network** | Red privada donde viven los recursos | Regional | 3 | "red privada", "subred" |
| **Subred** | División de la VNet para agrupar recursos | Dentro de la VNet | 3 | "segmentar" |
| **Peering** | Conectar dos VNets sin Internet | Regional / Global | 3 | "conectar VNets" |
| **NSG** | Permitir o denegar tráfico por IP y puerto | Subred / NIC | 3-4 | "filtrar", "reglas" |
| **Azure Firewall** | Firewall administrado centralizado con inteligencia de amenazas | VNet / hub | 3-7 | "FQDN", "amenazas" |
| **Load Balancer** | Repartir tráfico TCP/UDP entre VMs | Regional | 4 | "TCP/UDP", "capa 4" |
| **Application Gateway** | Repartir tráfico web por URL, SSL, WAF | Regional | 7 | "HTTP", "URL", "WAF" |
| **Front Door** | Balanceo web global con aceleración y WAF | Global | 7 | "global", "aplicación web" |
| **Traffic Manager** | Dirigir usuarios por DNS a la región adecuada | Global | DNS | "DNS", "varias regiones" |
| **CDN** | Cachear contenido estático cerca del usuario | Global | 7 | "caché", "estático" |
| **VPN Gateway** | Túnel cifrado por Internet con on-premises | Regional | 3 | "cifrado", "Internet", "IPsec" |
| **ExpressRoute** | Línea privada dedicada con on-premises | Regional / geopolítico | 3 | "privada", "sin Internet" |
| **Azure DNS** | Resolver nombres de dominio | Global | 7 | "DNS", "zona" |

## Diferencias que más fácil puedo confundir

> [!warning] Las 8 parejas críticas

| Pareja | La diferencia en una línea |
|---|---|
| **VPN Gateway vs. ExpressRoute** | VPN va por Internet cifrado y es barata; ExpressRoute es línea privada, cara y de alto rendimiento. |
| **Site-to-Site vs. Point-to-Site** | S2S conecta una red (oficina, dispositivo VPN); P2S conecta un equipo (cliente VPN). |
| **Peering vs. VPN Gateway** | Peering conecta VNet con VNet; VPN Gateway conecta VNet con on-premises. |
| **Load Balancer vs. Application Gateway** | Capa 4 TCP/UDP vs. capa 7 HTTP con URL, SSL y WAF. |
| **Application Gateway vs. Front Door** | Regional vs. global; funciones similares. |
| **NSG vs. Azure Firewall** | Reglas básicas por IP/puerto en subred o NIC vs. firewall administrado centralizado con FQDN y amenazas. |
| **NSG vs. WAF** | NSG filtra red (puertos); WAF protege aplicaciones web (inyección SQL, XSS). |
| **CDN vs. Load Balancer** | CDN cachea contenido en POPs globales; Load Balancer reparte tráfico entre tus VMs. |

> [!important] Regla mental de conectividad híbrida
> "¿Puede pasar por Internet?" → Sí = **VPN**. No = **ExpressRoute**.
> "¿Es un equipo o una red?" → Equipo = **P2S**. Red = **S2S**.
> "¿Es VNet con VNet?" → **Peering**.

> [!important] Regla mental de balanceo
> "¿Es HTTP y necesita URL/SSL/WAF?" → Sí = **Application Gateway** (regional) o **Front Door** (global). No = **Load Balancer**.
> "¿Es solo entregar contenido estático rápido?" → **CDN**.

## 10 tips de examen

> [!tip]
> 1. Una VNet vive en **una región y una suscripción**. Para dos regiones: dos VNets + **peering global**.
> 2. Recursos en la misma VNet se comunican **sin configurar nada**; el peering es solo entre VNets distintas.
> 3. En CIDR, **/16 tiene más IPs que /24**. Azure reserva **5 IPs** por subred.
> 4. El NSG se asocia a **subred o NIC**, nunca a la VNet. **Menor número de prioridad gana**.
> 5. "TCP/UDP" o "capa 4" = **Load Balancer**. "HTTP", "URL", "WAF" o "capa 7" = **Application Gateway**.
> 6. "Cifrado por Internet" = **VPN Gateway**. "Privado, sin Internet, dedicado" = **ExpressRoute**.
> 7. **VPN** es la opción **económica y rápida de desplegar**; **ExpressRoute** tarda semanas y necesita proveedor.
> 8. ExpressRoute **no cifra por defecto**; privado y cifrado son cosas distintas.
> 9. Un solo **VPN Gateway por VNet**, siempre en la subred **GatewaySubnet**.
> 10. "Usuarios en todo el mundo + contenido estático + latencia" = **CDN**. "Aplicación web global con WAF" = **Front Door**.

## 10 preguntas de repaso tipo AZ-900

**1.** Necesitas conectar dos redes virtuales en distintas regiones de Azure sin que el tráfico atraviese Internet. ¿Qué usas?
- A) VPN Site-to-Site
- B) Peering global de VNets
- C) Azure CDN
- D) Azure Firewall

**Respuesta: B.** El peering global conecta VNets de distintas regiones por el backbone de Microsoft. A conecta con on-premises; C y D no conectan redes.

---

**2.** Tu empresa exige que los datos entre su centro de datos y Azure nunca viajen por Internet público. ¿Qué servicio cumple el requisito?
- A) VPN Point-to-Site
- B) VPN Site-to-Site
- C) ExpressRoute
- D) Peering

**Respuesta: C.** ExpressRoute es la única conexión con on-premises que evita Internet. A y B usan Internet cifrado; D conecta VNets.

---

**3.** ¿Cuál es la forma más económica de conectar una oficina pequeña con una VNet de Azure de forma segura?
- A) ExpressRoute Direct
- B) VPN Site-to-Site
- C) ExpressRoute con Global Reach
- D) Azure Front Door

**Respuesta: B.** S2S cifra por Internet y no requiere proveedor ni circuitos. A y C son costosos; D no conecta redes.

---

**4.** Un NSG tiene la regla X (prioridad 150, Deny puerto 22) y la regla Y (prioridad 100, Allow puerto 22). ¿Se permite el tráfico SSH?
- A) Sí, porque la regla Y tiene mayor prioridad.
- B) No, porque las reglas Deny siempre ganan.
- C) Sí, porque el puerto 22 está permitido por defecto.
- D) No, porque la regla X se evalúa primero.

**Respuesta: A.** 100 es menor que 150, así que Y se evalúa antes y permite. B y D invierten la lógica de prioridad; C es falsa, el tráfico desde Internet se deniega por defecto.

---

**5.** Necesitas que las peticiones a `/api` vayan a un grupo de servidores y las de `/web` a otro, todas por HTTPS. ¿Qué servicio utilizas?
- A) Azure Load Balancer
- B) Azure Application Gateway
- C) Network Security Group
- D) Azure ExpressRoute

**Respuesta: B.** El enrutamiento por ruta URL es capa 7. A no lee URLs; C filtra puertos; D conecta redes.

---

**6.** Tienes tres VMs que reciben tráfico UDP de un juego en línea. Necesitas repartir las conexiones y detectar VMs caídas. ¿Qué servicio es el adecuado?
- A) Azure Application Gateway
- B) Azure Front Door
- C) Azure Load Balancer
- D) Azure CDN

**Respuesta: C.** UDP es capa 4; solo Load Balancer lo soporta. A y B son solo HTTP; D cachea contenido.

---

**7.** ¿A cuál de estos elementos NO se puede asociar un Network Security Group?
- A) Una subred
- B) Una interfaz de red
- C) Una Virtual Network completa
- D) Ninguna de las anteriores; se puede asociar a todas

**Respuesta: C.** El NSG solo se asocia a subredes y NICs.

---

**8.** Los usuarios de Japón tardan mucho en descargar los vídeos de tu web alojada en Europa. ¿Qué servicio reduce la latencia sin mover la aplicación?
- A) Azure ExpressRoute
- B) Azure Content Delivery Network
- C) Azure Load Balancer interno
- D) Peering global

**Respuesta: B.** La CDN cachea los vídeos en POPs cercanos a Japón. A conecta redes corporativas; C reparte tráfico dentro de una VNet; D conecta VNets.

---

**9.** ¿Cuál de estas afirmaciones sobre ExpressRoute es FALSA?
- A) Utiliza un proveedor de conectividad.
- B) El tráfico viaja cifrado por defecto mediante IPsec.
- C) Ofrece mayor ancho de banda que una VPN.
- D) El tráfico no atraviesa Internet público.

**Respuesta: B.** ExpressRoute es privado pero no cifra por defecto. A, C y D son verdaderas.

---

**10.** Un empleado necesita acceder a una VM de la VNet desde su portátil en casa. ¿Qué tipo de conexión configuras?
- A) VPN Site-to-Site
- B) VPN Point-to-Site
- C) ExpressRoute
- D) Peering regional

**Respuesta: B.** P2S conecta un dispositivo individual mediante cliente VPN. A requiere dispositivo VPN en una red local; C es para empresas con proveedor; D conecta VNets.

---

> [!tip] Última pasada antes del examen
> Si solo puedes repasar tres cosas: **VPN vs. ExpressRoute**, **Load Balancer vs. Application Gateway**, y **qué es una VNet con sus subredes y peering**. Esas tres cubren la mayoría de preguntas de redes del AZ-900.
