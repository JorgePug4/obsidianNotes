---
tags: [az-900, azure, redes, vnet]
modulo: Redes
peso_examen: Alto
---

# Azure Virtual Network (VNet)

## Concepto

Una **Virtual Network (VNet)** es tu red privada dentro de Azure. Es el equivalente en la nube a la red física de tu oficina: un espacio aislado donde tus máquinas virtuales y otros recursos se comunican de forma segura.

**Problema que resuelve:** necesitas que tus recursos hablen entre sí sin exponerlos a Internet, y que puedas controlar quién entra y quién sale.

**Para qué se utiliza:**
- Alojar máquinas virtuales y servicios PaaS en una red privada.
- Segmentar la red en **subredes** (por ejemplo: web, aplicación, base de datos).
- Conectar la nube con tu red local.
- Aplicar seguridad y enrutamiento personalizado.

## Características principales

- **Aislamiento y segmentación:** cada VNet está aislada de las demás. Defines un **espacio de direcciones IP privado** y lo divides en subredes. Ver [[02 - Espacio de direcciones y subredes]].
- **Comunicación con Internet:** por defecto los recursos pueden **salir** a Internet. Para que algo **entre** desde Internet necesitas una **IP pública** o un balanceador.
- **Comunicación entre recursos de Azure:** VMs en la misma VNet se comunican sin configuración extra. Servicios PaaS (Storage, SQL) se integran mediante **service endpoints** o **private endpoints** (solo a nivel de concepto en AZ-900).
- **Comunicación con on-premises:** tres opciones:
  - **Point-to-site (P2S) VPN:** un equipo individual se conecta a la VNet.
  - **Site-to-site (S2S) VPN:** toda tu oficina se conecta por Internet cifrado. Ver [[05 - Azure VPN Gateway]].
  - **ExpressRoute:** conexión privada dedicada. Ver [[08 - Azure ExpressRoute]].
- **Enrutamiento:** Azure enruta el tráfico entre subredes de forma automática. Puedes crear **tablas de rutas** personalizadas (UDR) y usar **BGP** para intercambiar rutas con on-premises.
- **Filtrado de tráfico:** con **NSG** ([[03 - Grupo de seguridad de red (NSG)]]) o con **Azure Firewall**.
- **Peering de redes virtuales:** conecta dos VNets para que se comuniquen como si fueran una sola, usando la red troncal privada de Microsoft (nunca Internet). Puede ser **regional** (misma región) o **global** (regiones distintas).
- **Alcance:** una VNet pertenece a **una sola región** y a **una sola suscripción**. Puede abarcar varias zonas de disponibilidad dentro de esa región.

> [!warning] Confusiones frecuentes
> - **VNet vs. subred:** la VNet es el contenedor; la subred es una división dentro de ella. Un recurso se conecta a una subred, no "a la VNet" directamente.
> - **Peering vs. VPN Gateway:** peering conecta **VNet con VNet**. VPN Gateway conecta **VNet con on-premises** (o VNet con VNet cuando se necesita cifrado, pero eso es AZ-104).
> - **Peering es no transitivo:** si A está emparejada con B y B con C, A no ve a C automáticamente.

## Casos de uso

- Una empresa despliega un servidor web, un servidor de aplicación y una base de datos en tres subredes distintas de la misma VNet, con NSG que solo permiten el flujo web → app → BD.
- Un equipo de desarrollo tiene su VNet y el de producción tiene otra; las conectan con **peering** para compartir un servidor de licencias.
- Una empresa con oficina en Madrid extiende su red a Azure mediante **VPN site-to-site** para que los empleados accedan a las VMs como si estuvieran en la oficina.

## Comparaciones

### Formas de conectar una VNet con otras redes

| Escenario | Servicio | Va por Internet | Palabra clave |
|---|---|---|---|
| VNet ↔ VNet | **Peering** | No (backbone Microsoft) | "conectar dos redes virtuales" |
| Un portátil ↔ VNet | **VPN Point-to-Site** | Sí, cifrado | "usuario remoto", "dispositivo individual" |
| Oficina ↔ VNet | **VPN Site-to-Site** | Sí, cifrado | "red local", "oficina", "IPsec" |
| Oficina ↔ VNet, sin Internet | **ExpressRoute** | No (línea privada) | "dedicada", "privada", "mayor ancho de banda" |

## Conceptos que debo memorizar

> [!important]
> - Una VNet vive en **una región** y **una suscripción**.
> - Se compone de un **espacio de direcciones privado** dividido en **subredes**.
> - **Peering** = conectar dos VNets por la red privada de Microsoft, sin Internet. Existe **regional** y **global**.
> - Salida a Internet: **por defecto sí**. Entrada: solo con **IP pública** o balanceador.
> - Recursos en la misma VNet se comunican **sin configuración adicional**.
> - Azure DNS resuelve nombres de dominio; se puede usar para dominios públicos y para resolución privada dentro de la VNet.

## Tips para AZ-900

> [!tip] Palabras clave que apuntan a VNet
> "red privada en Azure", "aislar recursos", "segmentar en subredes", "comunicación entre máquinas virtuales".

> [!tip] Preguntas trampa habituales
> - "¿Puede una VNet abarcar dos regiones?" → **No**. Si necesitas dos regiones, creas dos VNets y las conectas con **peering global**.
> - "¿Necesito peering para que dos VMs de la misma VNet se comuniquen?" → **No**. Ya se comunican.
> - "¿El tráfico de peering atraviesa Internet?" → **No**. Usa el backbone de Microsoft.
> - "¿Puedo tener VNets con el mismo rango de IPs?" → Sí, pero **no podrás emparejarlas** si los rangos se solapan.

> [!tip] Azure DNS (aparece en el objetivo oficial junto a VNet)
> Servicio de hospedaje de dominios DNS en Azure. Recuerda solo esto: resuelve nombres a IPs, se administra con las mismas herramientas de Azure (portal, CLI, RBAC) y **no permite comprar dominios** (eso lo haces en un registrador externo y luego apuntas a Azure DNS).

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu empresa tiene una VNet en West Europe y otra en East US. Necesitas que las máquinas virtuales de ambas se comuniquen de forma privada sin pasar por Internet. ¿Qué debes configurar?

- A) Un Azure Load Balancer
- B) Peering global de redes virtuales
- C) Una IP pública en cada máquina virtual
- D) Azure Content Delivery Network

**Respuesta correcta: B.** El peering global conecta VNets de distintas regiones a través del backbone privado de Microsoft.
- A es incorrecta: Load Balancer distribuye tráfico, no conecta redes.
- C es incorrecta: las IPs públicas exponen las VMs a Internet, justo lo contrario de lo pedido.
- D es incorrecta: CDN sirve contenido en caché, no conecta redes.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre una Azure Virtual Network es correcta?

- A) Una VNet puede abarcar varias regiones de Azure.
- B) Los recursos de una misma VNet requieren peering para comunicarse.
- C) Una VNet se puede dividir en varias subredes.
- D) Una VNet solo puede contener una máquina virtual.

**Respuesta correcta: C.** Dividir la VNet en subredes es una de sus funciones básicas.
- A es incorrecta: una VNet pertenece a una única región.
- B es incorrecta: el peering es para conectar VNets distintas.
- D es incorrecta: una VNet aloja tantos recursos como permita su espacio de direcciones.

**Pregunta 3.** Un empleado que trabaja desde casa necesita conectarse de forma segura a las máquinas virtuales de la VNet corporativa desde su portátil. ¿Qué tipo de conexión es la más adecuada?

- A) VPN Site-to-Site
- B) ExpressRoute
- C) VPN Point-to-Site
- D) Peering de redes virtuales

**Respuesta correcta: C.** Point-to-Site conecta un dispositivo individual con la VNet.
- A es incorrecta: Site-to-Site conecta una red completa (oficina), no un único equipo.
- B es incorrecta: ExpressRoute es una línea dedicada para empresas, excesiva para un portátil.
- D es incorrecta: el peering conecta VNets entre sí.

## 🧠 Resumen para el examen

1. La VNet es la **red privada** de Azure; base de toda la conectividad.
2. Se divide en **subredes**; los recursos se conectan a una subred.
3. Una VNet = **una región + una suscripción**.
4. **Peering** conecta VNets (regional o global) sin pasar por Internet.
5. **P2S** = un dispositivo; **S2S** = una oficina; **ExpressRoute** = línea privada.
6. Recursos de la misma VNet se comunican **sin configurar nada**.
7. Salida a Internet por defecto sí; entrada solo con **IP pública**.
8. **Azure DNS** hospeda zonas DNS pero no vende dominios.
