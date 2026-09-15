---
tags: [az-900, azure, redes, vpn, hibrido]
modulo: Redes
peso_examen: Alto
---

# Azure VPN Gateway

## Concepto

**Azure VPN Gateway** es un tipo de puerta de enlace de red virtual que crea túneles **cifrados** a través de **Internet público** entre tu [[01 - Azure Virtual Network]] y tu red local, otro dispositivo o incluso otra VNet.

**Problema que resuelve:** tus servidores en la oficina y tus VMs en Azure necesitan hablar como si estuvieran en la misma red, pero entre medias está Internet, que no es seguro. La VPN cifra ese tráfico con **IPsec/IKE**.

**Para qué se utiliza:**
- Conectar la sede de la empresa con Azure (**Site-to-Site**).
- Permitir que empleados remotos accedan a la VNet desde su equipo (**Point-to-Site**).
- Conectar dos VNets con tráfico cifrado (**VNet-to-VNet**).

## Características principales

- Se despliega en una **subred especial** llamada **GatewaySubnet** dentro de la VNet.
- **Una VNet solo puede tener un VPN Gateway**, pero ese gateway puede tener varias conexiones.
- **Tipos de conexión:**
  - **Site-to-Site (S2S):** oficina ↔ Azure. Requiere un dispositivo VPN físico o virtual en la oficina con IP pública.
  - **Point-to-Site (P2S):** equipo individual ↔ Azure. Se instala un cliente VPN en el portátil. Ideal para teletrabajo.
  - **VNet-to-VNet:** VNet ↔ VNet con cifrado (alternativa al peering cuando se necesita cifrado o hay requisitos específicos).
- **Tipos de VPN:**
  - **Basada en rutas (route-based):** la más usada, soporta P2S, VNet-to-VNet, coexistencia con ExpressRoute.
  - **Basada en directivas (policy-based):** solo S2S, escenarios heredados. En AZ-900 basta con saber que route-based es la opción por defecto.
- **Alta disponibilidad:** por defecto **activo/pasivo** (una instancia activa, otra en espera). Se puede configurar **activo/activo** con dos IPs públicas. En on-premises se logra redundancia con **varios dispositivos VPN**.
- El tráfico va **cifrado por Internet**: la latencia y el ancho de banda dependen de tu conexión a Internet.
- Se cobra por **hora de gateway** y por **tráfico de salida**.

> [!warning] Confusiones frecuentes
> - **VPN Gateway vs. ExpressRoute:** los dos conectan on-premises con Azure. VPN va **por Internet cifrado**; ExpressRoute va por una **línea privada** que no toca Internet. Ver [[08 - Azure ExpressRoute]].
> - **VPN Gateway vs. Peering:** para conectar dos VNets, el peering es más simple y rápido. VPN VNet-to-VNet solo cuando necesitas cifrado explícito o límites que el peering no cubre.
> - **S2S vs. P2S:** S2S es **red con red** (necesita dispositivo VPN en la oficina). P2S es **un equipo con la red** (necesita cliente VPN en el equipo).
> - **GatewaySubnet:** es un nombre fijo; si le pones otro nombre el gateway no se despliega.

## Casos de uso

- Una pyme con 30 empleados en Barcelona quiere que su servidor de archivos on-premises y sus VMs en Azure se vean mutuamente: **Site-to-Site VPN**.
- Un consultor viaja y necesita entrar a las VMs de la VNet desde hoteles: **Point-to-Site VPN**.
- Una empresa tiene ExpressRoute como enlace principal y quiere un **respaldo** por si la línea dedicada falla: **VPN Site-to-Site en paralelo** (coexistencia).

## Comparaciones

| Tipo de conexión | Conecta | Qué necesitas en el otro extremo | Caso típico |
|---|---|---|---|
| **Site-to-Site** | Red local ↔ VNet | Dispositivo VPN con IP pública | Oficina completa |
| **Point-to-Site** | Un equipo ↔ VNet | Cliente VPN instalado | Teletrabajo, usuario remoto |
| **VNet-to-VNet** | VNet ↔ VNet | Otro VPN Gateway | Cifrado entre VNets (poco frecuente frente a peering) |

| | **VPN Gateway** | **ExpressRoute** |
|---|---|---|
| Medio | Internet público, cifrado | Línea privada dedicada |
| Ancho de banda | Limitado (hasta pocos Gbps según SKU) | Hasta 100 Gbps |
| Latencia | Variable | Baja y predecible |
| Coste | Bajo | Alto (requiere proveedor de conectividad) |
| Tiempo de puesta en marcha | Minutos/horas | Semanas |
| Cifrado | Sí, incluido | No incluido por defecto (tráfico privado) |

## Conceptos que debo memorizar

> [!important]
> - VPN Gateway = túnel **cifrado** por **Internet** con **IPsec/IKE**.
> - Vive en la subred **GatewaySubnet**.
> - **Un gateway por VNet**, varias conexiones por gateway.
> - **S2S** = oficina; **P2S** = un dispositivo; **VNet-to-VNet** = dos VNets.
> - **Route-based** es el tipo estándar.
> - Por defecto **activo/pasivo**; puede ser **activo/activo**.
> - Sirve como **respaldo de ExpressRoute**.

## Tips para AZ-900

> [!tip] Palabras clave
> "conexión cifrada", "a través de Internet", "IPsec", "sitio a sitio", "punto a sitio", "red híbrida de bajo coste".

> [!tip] Preguntas trampa
> - Si la pregunta dice **"el tráfico no debe atravesar Internet"** → la respuesta es **ExpressRoute**, no VPN.
> - Si dice **"la solución más económica para conectar la oficina"** → **VPN Site-to-Site**.
> - Si dice **"usuario individual / portátil / cliente"** → **Point-to-Site**.
> - "¿Cuántos VPN Gateway puede tener una VNet?" → **Uno**.
> - "¿Cómo aseguro que un fallo del dispositivo VPN local no corte la conexión?" → **Varios dispositivos VPN on-premises** (o gateway activo/activo en Azure).

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu empresa necesita conectar su centro de datos local con una VNet de Azure de forma segura y con el menor coste posible. El tráfico puede atravesar Internet siempre que vaya cifrado. ¿Qué servicio utilizas?

- A) Azure ExpressRoute
- B) Azure VPN Gateway con conexión Site-to-Site
- C) Peering de redes virtuales
- D) Azure Load Balancer

**Respuesta correcta: B.** S2S VPN cifra el tráfico por Internet y es la opción más económica.
- A es incorrecta: ExpressRoute es más caro y no usa Internet; el enunciado permite Internet.
- C es incorrecta: el peering conecta VNets, no redes locales.
- D es incorrecta: no conecta redes.

**Pregunta 2.** Un empleado necesita acceder desde su portátil personal a recursos de una VNet mientras viaja. ¿Qué tipo de conexión VPN es la adecuada?

- A) Site-to-Site
- B) VNet-to-VNet
- C) Point-to-Site
- D) ExpressRoute Direct

**Respuesta correcta: C.** P2S conecta un único dispositivo con la VNet mediante un cliente VPN.
- A requiere un dispositivo VPN en una red local, no aplica a un portátil.
- B conecta dos VNets.
- D es una modalidad de ExpressRoute para grandes empresas.

**Pregunta 3.** ¿En qué subred debe desplegarse un Azure VPN Gateway?

- A) En cualquier subred de la VNet
- B) En una subred llamada GatewaySubnet
- C) En la subred predeterminada (default)
- D) Fuera de la VNet, en un grupo de recursos independiente

**Respuesta correcta: B.** El gateway requiere una subred con ese nombre exacto.
- A y C son incorrectas: el nombre es obligatorio.
- D es incorrecta: el gateway forma parte de la VNet.

## 🧠 Resumen para el examen

1. VPN Gateway crea túneles **cifrados por Internet** (IPsec/IKE).
2. Se despliega en la subred **GatewaySubnet**.
3. **Un gateway por VNet**; varias conexiones por gateway.
4. **Site-to-Site** = oficina ↔ Azure (dispositivo VPN local).
5. **Point-to-Site** = un equipo ↔ Azure (cliente VPN).
6. **VNet-to-VNet** = cifrado entre VNets (peering es la opción habitual).
7. **Route-based** es el tipo estándar.
8. Por defecto **activo/pasivo**; puede ser **activo/activo**.
9. Es la opción **económica y rápida**; ExpressRoute es la **privada y de alto rendimiento**.
10. Puede usarse como **respaldo** de ExpressRoute.
