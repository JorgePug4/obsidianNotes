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