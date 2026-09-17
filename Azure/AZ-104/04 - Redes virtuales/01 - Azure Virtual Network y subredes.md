---
tags: [az-104, azure, redes, vnet, subredes, cidr]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Azure Virtual Network y subredes

## ¿Qué es?

Una **red virtual (VNet)** es la red privada de tu organización en Azure: un espacio de direcciones IP privado, dividido en **subredes**, dentro de **una región** y **una suscripción**. Todos los recursos de IaaS (VMs, balanceadores, gateways) y muchos de PaaS se conectan a ella. Fundamentos en [[01 - Azure Virtual Network]] (AZ-900).

## ¿Para qué sirve?

- Aislar y segmentar cargas de trabajo.
- Comunicar recursos entre sí de forma privada.
- Conectar con on-premises y con servicios PaaS de forma privada.

## Conceptos clave

- **Espacio de direcciones**: uno o varios bloques **CIDR** (se pueden añadir y quitar después, si no hay solapamientos con peerings o rutas). Se recomienda usar rangos privados **RFC 1918**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- **Subred**: subdivisión del espacio; los recursos se despliegan **en subredes**. Tamaño mínimo **/29** y máximo **/2** 🧠 (recomendado /27 o mayor para servicios con requisitos).
- **IPs reservadas por subred: 5** 🧠. En `10.0.1.0/24`:
  - `10.0.1.0` dirección de red
  - `10.0.1.1` **gateway predeterminado**
  - `10.0.1.2` y `10.0.1.3` DNS de Azure
  - `10.0.1.255` broadcast
  - → **251 IPs utilizables** en un /24.
- **Fórmula útil**: IPs utilizables = 2^(32−prefijo) − 5.

| Prefijo | Total | Utilizables |
|---|---|---|
| /29 | 8 | 3 |
| /28 | 16 | 11 |
| /27 | 32 | 27 |
| /26 | 64 | 59 |
| /24 | 256 | 251 |
| /16 | 65 536 | 65 531 |

- **Subredes con nombre reservado** 🧠: `GatewaySubnet` (VPN/ExpressRoute Gateway, mínimo /27 recomendado), `AzureBastionSubnet` (mínimo **/26**), `AzureFirewallSubnet` (/26), `AzureFirewallManagementSubnet` (/26), `RouteServerSubnet` (/27).
- **Delegación de subred**: ceder la subred a un servicio PaaS (`Microsoft.Web/serverFarms`, `Microsoft.ContainerInstance/containerGroups`, `Microsoft.Sql/managedInstances`…).
- **Directivas de red de la subred**: habilitar/deshabilitar directivas de red para **private endpoints** (NSG y UDR sobre private endpoints).
- **NAT Gateway**: salida a Internet con IP pública fija por subred (recomendado frente al SNAT por defecto).
- **Salida a Internet por defecto**: toda VM tiene salida a Internet mediante SNAT implícito ➕ (Microsoft lo está retirando para despliegues nuevos a partir de septiembre de 2025: habrá que usar NAT Gateway, IP pública o Load Balancer).
- **Límites**: por defecto hasta **1000 VNets por suscripción/región** y **3000 subredes por VNet**; 65 536 IPs privadas por VNet.
- Una VNet **no puede abarcar varias regiones ni suscripciones**; para unirlas → **peering** ([[03 - Peering de redes virtuales]]).
- **Cambiar el espacio de direcciones**: se puede ampliar/reducir si no afecta a subredes existentes; ampliar una subred requiere que no haya recursos que lo impidan (hoy se puede modificar el prefijo de una subred con recursos en muchos casos, pero sigue siendo delicado).

## Cómo funciona

```bash
az network vnet create -g rg-net -n vnet-hub --address-prefixes 10.0.0.0/16 --subnet-name snet-web --subnet-prefixes 10.0.1.0/24
az network vnet subnet create -g rg-net --vnet-name vnet-hub -n snet-data --address-prefixes 10.0.2.0/24
az network vnet subnet create -g rg-net --vnet-name vnet-hub -n AzureBastionSubnet --address-prefixes 10.0.250.0/26
az network vnet subnet create -g rg-net --vnet-name vnet-hub -n GatewaySubnet --address-prefixes 10.0.255.0/27
az network vnet update -g rg-net -n vnet-hub --address-prefixes 10.0.0.0/16 10.1.0.0/16   # añadir espacio
az network vnet subnet update -g rg-net --vnet-name vnet-hub -n snet-app --delegations Microsoft.Web/serverFarms
az network vnet list-available-ips ...    # (según versión)
az network vnet subnet list -g rg-net --vnet-name vnet-hub -o table
```

```powershell
$web = New-AzVirtualNetworkSubnetConfig -Name snet-web -AddressPrefix 10.0.1.0/24
New-AzVirtualNetwork -ResourceGroupName rg-net -Name vnet-hub -Location westeurope -AddressPrefix 10.0.0.0/16 -Subnet $web
$vnet = Get-AzVirtualNetwork -Name vnet-hub -ResourceGroupName rg-net
Add-AzVirtualNetworkSubnetConfig -Name snet-data -AddressPrefix 10.0.2.0/24 -VirtualNetwork $vnet
$vnet | Set-AzVirtualNetwork
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Desplegar Azure Bastion | Subred llamada **AzureBastionSubnet**, mínimo **/26** |
| Desplegar VPN Gateway | Subred **GatewaySubnet**, /27 o mayor |
| Integrar App Service (salida) | Subred **delegada** a `Microsoft.Web/serverFarms` |
| Cuántas VMs caben en 10.1.4.0/26 | 64 − 5 = **59** |
| Dos VNets con rangos solapados | **No se pueden emparejar**: rediseñar direccionamiento |
| Recursos en dos regiones | Dos VNets + **peering global** |
| Salida a Internet con IP fija | **NAT Gateway** en la subred |
| Aislar tráfico entre subredes | **NSG** por subred ([[05 - Network Security Group (NSG)]]) |

## Ejemplo

Contoso diseña `vnet-prod` con `10.10.0.0/16` en West Europe: `snet-web 10.10.1.0/24` (251 IPs), `snet-app 10.10.2.0/24`, `snet-data 10.10.3.0/24`, `AzureBastionSubnet 10.10.250.0/26` y `GatewaySubnet 10.10.255.0/27`. El direccionamiento no solapa con la red on-premises (`192.168.0.0/16`) para poder conectar por VPN.

## Comparaciones

| Concepto | Ámbito | Nota |
|---|---|---|
| **VNet** | Región + suscripción | Contenedor de subredes |
| **Subred** | Dentro de la VNet | Donde viven las NICs; 5 IPs reservadas |
| **Peering** | VNet ↔ VNet | Regional o global, no transitivo |
| **VPN/ExpressRoute** | VNet ↔ on-premises | Ver [[16 - Conectividad híbrida - VPN Gateway y ExpressRoute]] |

## 💻 Laboratorio: diseño de VNet

1. Crear `vnet-lab` con `10.20.0.0/16` y las subredes `web` (/24), `app` (/24) y `data` (/26).
2. Añadir `AzureBastionSubnet` (/26) y comprobar que Azure exige ese nombre exacto.
3. Desplegar una VM en `web` y comprobar su IP privada (la 4ª disponible: `.4`).
4. Intentar crear una subred solapada y observar el error.
5. Añadir un segundo espacio de direcciones `10.21.0.0/16` a la VNet.

## AZ-104 Exam Tips

- ⭐ Una VNet = **una región, una suscripción**.
- 🔥 🧠 **5 IPs reservadas por subred**; utilizables = 2^(32−prefijo) − 5.
- 🔥 🧠 Nombres obligatorios: **GatewaySubnet**, **AzureBastionSubnet** (/26 mínimo), **AzureFirewallSubnet** (/26).
- 🧠 Subred mínima **/29**.
- 🧠 Los rangos de VNets que se emparejan **no pueden solaparse**.
- 💻 `az network vnet create/subnet create`, delegaciones, añadir espacios de direcciones.
- ⚠️ La primera IP asignable de una subred es la **.4**.

## Errores comunes

- Calcular 256 IPs útiles en un /24 (son 251).
- Nombrar mal la subred de Bastion o del gateway.
- Diseñar rangos que solapan con on-premises y luego no poder conectar.

## Preguntas que podrían aparecer

**1.** ¿Cuántas direcciones IP se pueden asignar a máquinas virtuales en la subred `10.0.5.0/27`?
- A) 32 · B) 30 · C) 27 · D) 29

<details><summary>Respuesta</summary>

**C.** 32 direcciones totales menos las 5 reservadas por Azure = 27.
</details>

**2.** Vas a desplegar Azure Bastion. ¿Qué requisito debe cumplir la subred?
- A) Llamarse BastionSubnet y ser /29 · B) Llamarse AzureBastionSubnet y tener al menos /26 · C) Estar delegada a Microsoft.Network · D) Tener un NSG sin reglas

<details><summary>Respuesta</summary>

**B.** El nombre es obligatorio y el tamaño mínimo recomendado actual es /26.
</details>

**3.** Dos redes virtuales usan `10.0.0.0/16`. Necesitas comunicarlas. ¿Qué haces?
- A) Peering global · B) VPN VNet-to-VNet · C) Cambiar el direccionamiento de una de ellas · D) Azure Firewall

<details><summary>Respuesta</summary>

**C.** Los espacios de direcciones no pueden solaparse en un peering ni en una conexión VNet a VNet.
</details>

## Relacionado

- [[02 - Direcciones IP públicas y privadas]]
- [[03 - Peering de redes virtuales]]
- [[05 - Network Security Group (NSG)]]
- [[08 - Azure Bastion]]
- [[01 - Azure Virtual Network]] (AZ-900)
- [[00 - Índice - Redes virtuales]]
