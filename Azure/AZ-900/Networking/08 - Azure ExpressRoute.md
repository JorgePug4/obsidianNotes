---
tags: [az-900, azure, redes, expressroute, hibrido]
modulo: Redes
peso_examen: Alto
---

# Azure ExpressRoute

## Concepto

**Azure ExpressRoute** conecta tu red local con Azure mediante una **conexión privada y dedicada** que **no atraviesa Internet público**. La conexión la proporciona un **proveedor de conectividad** (operador de telecomunicaciones) o se hace en una instalación de intercambio (peering location).

**Problema que resuelve:** la VPN por Internet tiene latencia variable, ancho de banda limitado y depende de la calidad de Internet. ExpressRoute ofrece un enlace **predecible, rápido y aislado** para cargas críticas.

**Para qué se utiliza:**
- Migraciones masivas de datos a Azure.
- Aplicaciones que necesitan latencia baja y constante (bases de datos híbridas, VDI).
- Cumplimiento normativo que prohíbe que los datos viajen por Internet.
- Acceso a servicios de Microsoft (Microsoft 365, Dynamics 365) por el enlace privado.

## Características principales

- **No usa Internet:** el tráfico va por la red del proveedor hasta la red troncal de Microsoft.
- **Ancho de banda:** desde 50 Mbps hasta **10 Gbps** (y hasta **100 Gbps** con **ExpressRoute Direct**).
- **Baja latencia y alta fiabilidad:** SLA de conectividad del 99,95 % o superior según configuración.
- **Enrutamiento dinámico con BGP:** intercambia rutas entre tu red y Azure.
- **Redundancia integrada:** cada circuito incluye dos conexiones (primaria y secundaria) hacia dos routers de Microsoft (Microsoft Enterprise Edge, MSEE).
- **Alcance:** un circuito da acceso a todas las regiones de su **zona geopolítica**. Con **ExpressRoute Premium** se accede a regiones de todo el mundo.
- **ExpressRoute Global Reach:** conecta dos sedes on-premises entre sí a través de la red de Microsoft usando sus circuitos ExpressRoute.
- **Modelos de conectividad:**
  - **CloudExchange colocation:** tu equipo está en la misma instalación que un punto de intercambio.
  - **Point-to-point Ethernet:** enlace directo del proveedor.
  - **Any-to-any (IPVPN):** integración con tu WAN MPLS existente.
  - **ExpressRoute Direct:** conexión directa a la red de Microsoft a 10 o 100 Gbps.
- **El tráfico no va cifrado por defecto** porque es privado. Si se necesita cifrado se añade (MACsec en Direct, o IPsec sobre ExpressRoute). En AZ-900 basta con saber que "privado ≠ cifrado".
- Se despliega también en la **GatewaySubnet**, con un gateway de tipo **ExpressRoute** (distinto del gateway VPN).
- Se puede combinar con **VPN Site-to-Site** como enlace de respaldo.

> [!warning] Confusiones frecuentes
> - **ExpressRoute vs. VPN Gateway:** la diferencia estrella del examen. VPN = Internet + cifrado + barato + rápido de montar. ExpressRoute = privado + alto ancho de banda + caro + semanas de aprovisionamiento. Ver [[05 - Azure VPN Gateway]].
> - **"Privado" no significa "cifrado":** ExpressRoute aísla el tráfico de Internet, pero no lo cifra salvo configuración adicional.
> - **ExpressRoute vs. Peering:** peering conecta VNets. ExpressRoute conecta on-premises con Azure.
> - **ExpressRoute Direct vs. circuito estándar:** Direct es la versión de 100 Gbps sin proveedor intermedio, para grandes empresas. Solo reconócelo.

## Casos de uso

- Un banco migra 500 TB de datos a Azure y necesita hacerlo por una vía privada regulada: **ExpressRoute**.
- Un hospital ejecuta aplicaciones híbridas que consultan una base de datos en Azure miles de veces por segundo y no tolera la latencia variable de Internet.
- Una multinacional con sedes en Madrid y Ciudad de México quiere conectarlas a través de sus circuitos ExpressRoute: **Global Reach**.

## Comparaciones

| | **VPN Site-to-Site** | **ExpressRoute** |
|---|---|---|
| Medio | Internet público | Red privada del proveedor + backbone Microsoft |
| Cifrado | Sí, IPsec incluido | No por defecto |
| Ancho de banda | Hasta unos pocos Gbps | 50 Mbps a 10 Gbps (100 Gbps Direct) |
| Latencia | Variable | Baja y predecible |
| Coste | Bajo | Alto (circuito + proveedor + salida de datos) |
| Puesta en marcha | Minutos u horas | Semanas |
| Requiere proveedor externo | No | Sí (salvo Direct) |
| Cuándo usarlo | Pymes, respaldo, pruebas, bajo presupuesto | Cargas críticas, cumplimiento, gran volumen |

## Conceptos que debo memorizar

> [!important]
> - ExpressRoute = conexión **privada y dedicada**, **no pasa por Internet**.
> - Se contrata a través de un **proveedor de conectividad**.
> - Ancho de banda hasta **10 Gbps** (100 Gbps con **Direct**).
> - **Baja latencia**, **alta fiabilidad**, **BGP**.
> - Redundancia incluida: **conexión primaria y secundaria**.
> - **No cifrado por defecto**.
> - **Global Reach** conecta sedes on-premises entre sí.
> - Da acceso también a **Microsoft 365** y otros servicios de Microsoft.

## Tips para AZ-900

> [!tip] Palabras clave
> "privada", "dedicada", "no atraviesa Internet", "alto ancho de banda", "baja latencia", "proveedor de conectividad", "cumplimiento", "migración de grandes volúmenes".

> [!tip] Preguntas trampa
> - "La conexión más rápida de configurar" → **VPN**, no ExpressRoute.
> - "La conexión más barata" → **VPN**.
> - "El tráfico no debe salir a Internet" → **ExpressRoute**.
> - "¿ExpressRoute cifra el tráfico?" → **No por defecto**.
> - "¿ExpressRoute requiere IP pública en el dispositivo local?" → **No**; eso es la VPN.
> - "Conectar dos oficinas entre sí usando la red de Microsoft" → **Global Reach**.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu empresa necesita una conexión entre su centro de datos y Azure que ofrezca alto ancho de banda, latencia baja y constante, y que no utilice Internet público. ¿Qué servicio cumple los requisitos?

- A) Azure VPN Gateway (Site-to-Site)
- B) Azure ExpressRoute
- C) Peering de redes virtuales
- D) Azure Content Delivery Network

**Respuesta correcta: B.** Solo ExpressRoute ofrece un enlace privado dedicado fuera de Internet.
- A usa Internet, aunque cifrado.
- C conecta VNets, no centros de datos.
- D cachea contenido.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre ExpressRoute es correcta?

- A) El tráfico viaja cifrado por Internet mediante IPsec.
- B) Se puede desplegar en minutos sin intervención de terceros.
- C) Proporciona una conexión privada a través de un proveedor de conectividad.
- D) Solo puede conectar dos redes virtuales de Azure.

**Respuesta correcta: C.**
- A describe la VPN.
- B es falsa: requiere semanas y un proveedor.
- D describe el peering.

**Pregunta 3.** Una empresa ya utiliza ExpressRoute y quiere asegurar que si el circuito falla, la conectividad con Azure continúe con un coste moderado. ¿Qué debería añadir?

- A) Un segundo Application Gateway
- B) Una conexión VPN Site-to-Site como respaldo
- C) Un Azure Load Balancer interno
- D) Un perfil de Azure CDN

**Respuesta correcta: B.** VPN S2S puede coexistir con ExpressRoute y actuar como ruta de respaldo por Internet.
- A, C y D no proporcionan conectividad híbrida.

## 🧠 Resumen para el examen

1. ExpressRoute = conexión **privada y dedicada** con Azure, **sin Internet**.
2. Se contrata con un **proveedor de conectividad**; tarda **semanas**.
3. **Alto ancho de banda** (hasta 10 Gbps; 100 Gbps con Direct) y **baja latencia**.
4. **Más caro** que VPN.
5. **No cifra por defecto**: privado no es lo mismo que cifrado.
6. Redundancia integrada con **dos conexiones** por circuito.
7. Usa **BGP** para enrutamiento dinámico.
8. **Global Reach** conecta sedes entre sí.
9. Se puede combinar con **VPN** como respaldo.
10. Da acceso a **Microsoft 365** y otros servicios de Microsoft por el enlace privado.
