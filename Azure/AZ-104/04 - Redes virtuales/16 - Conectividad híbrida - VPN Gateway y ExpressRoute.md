---
tags: [az-104, azure, redes, vpn, expressroute, hibrido]
modulo: Redes virtuales
peso_examen: Medio (contexto)
---

# Conectividad híbrida: VPN Gateway y ExpressRoute ➕

> [!info] La conectividad híbrida no es un objetivo literal del temario vigente de AZ-104, pero aparece constantemente en escenarios (rutas, DNS, private endpoints desde on-premises, Bastion, peering con gateway transit). Fundamentos en [[05 - Azure VPN Gateway]] y [[08 - Azure ExpressRoute]] (AZ-900).

## ¿Qué es?

- **VPN Gateway**: puerta de enlace que crea túneles **IPsec/IKE cifrados por Internet** entre una VNet y una red local, otra VNet o clientes individuales.
- **ExpressRoute**: **circuito privado dedicado** entre la red local y Azure a través de un proveedor de conectividad; **no pasa por Internet**.

## Conceptos clave

### VPN Gateway
- Se despliega en la subred **`GatewaySubnet`** (mínimo /27 recomendado) 🧠; **una sola por VNet**.
- **Tipos de conexión** 🧠:
  - **Site-to-Site (S2S)**: red local ↔ VNet, requiere un **dispositivo VPN** con IP pública y un **local network gateway** que describe la red local.
  - **Point-to-Site (P2S)**: un **equipo individual** ↔ VNet (OpenVPN, SSTP o IKEv2), con autenticación por **certificado**, **Entra ID** o RADIUS.
  - **VNet-to-VNet**: dos VNets mediante gateways (alternativa al peering cuando se requiere cifrado).
- **SKUs**: Basic (legado), **VpnGw1-5** y variantes **AZ** (zona-redundantes). Determinan ancho de banda, túneles y soporte de BGP.
- **Enrutamiento**: **basado en rutas (route-based)** es el estándar; policy-based es legado y muy limitado.
- **BGP**: intercambio dinámico de rutas; necesario para escenarios activo-activo y ExpressRoute.
- **Activo-activo** y **zona-redundante** para alta disponibilidad (SLA 99,95 %).
- Tarda **20-45 minutos** en desplegarse.

### ExpressRoute
- **Circuito** contratado a un proveedor, con ancho de banda (50 Mbps a 100 Gbps) y **SKU** (Local, Standard, **Premium**).
- **Peerings**: **privado** (a VNets) y **Microsoft** (a servicios PaaS/M365). El peering público está retirado.
- **ExpressRoute Gateway** en la VNet (también en `GatewaySubnet`).
- **No cifra por defecto** ⚠️ (se puede añadir IPsec sobre ExpressRoute).
- **Global Reach**: conectar sedes entre sí a través del backbone de Microsoft.
- **FastPath**: saltarse el gateway para mejorar rendimiento.
- SLA 99,95 % con configuración redundante; despliegue de semanas.

### Coexistencia y elección
- Se pueden combinar: ExpressRoute como principal y **VPN S2S como respaldo**.
- Con **peering + gateway transit**, los spokes usan el gateway del hub ([[03 - Peering de redes virtuales]]).
- **Azure Virtual WAN** ➕ centraliza VPN, ExpressRoute y conectividad entre VNets.

## Cómo funciona

```bash
# VPN Site-to-Site
az network vnet subnet create -g rg-net --vnet-name vnet-hub -n GatewaySubnet --address-prefixes 10.0.255.0/27
az network public-ip create -g rg-net -n pip-vpngw --sku Standard --allocation-method Static
az network vnet-gateway create -g rg-net -n vpngw-hub --vnet vnet-hub --public-ip-address pip-vpngw \
  --gateway-type Vpn --vpn-type RouteBased --sku VpnGw2AZ --no-wait
az network local-gateway create -g rg-net -n lng-sede --gateway-ip-address 203.0.113.1 --local-address-prefixes 192.168.0.0/16
az network vpn-connection create -g rg-net -n conn-sede --vnet-gateway1 vpngw-hub --local-gateway2 lng-sede --shared-key '<clave>'
az network vpn-connection show -g rg-net -n conn-sede --query connectionStatus
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Conectar la oficina con Azure de forma económica | **VPN Site-to-Site** |
| Un empleado desde casa necesita acceder a la VNet | **VPN Point-to-Site** |
| El tráfico no debe pasar por Internet | **ExpressRoute** |
| Ancho de banda alto y latencia predecible | **ExpressRoute** |
| Respaldo de ExpressRoute | **VPN S2S** como failover |
| Los spokes deben usar el gateway del hub | Peering con **gateway transit** / **use remote gateways** |
| Resolver nombres privados desde on-premises | DNS Private Resolver o reenviador ([[12 - Azure Private DNS y resolución de nombres]]) |
| Acceder a un private endpoint desde la sede | Private endpoint + DNS + VPN/ER |
| La subred del gateway | **GatewaySubnet**, /27 |

## Comparaciones

| | **VPN Gateway (S2S)** | **ExpressRoute** |
|---|---|---|
| Medio | Internet (cifrado IPsec) | **Circuito privado** del proveedor |
| Cifrado | **Sí**, por defecto | **No** por defecto |
| Ancho de banda | Hasta ~10 Gbps (SKU altos) | 50 Mbps – 100 Gbps |
| Latencia | Variable (Internet) | Predecible |
| Tiempo de despliegue | Minutos | Semanas |
| Coste | Bajo | Alto |
| SLA | 99,95 % (activo-activo/zonal) | 99,95 % |
| Cuándo | Sucursales, respaldo, presupuesto ajustado | Producción crítica, grandes volúmenes, cumplimiento |

| Tipo | Conecta | Cliente |
|---|---|---|
| **Site-to-Site** | Red ↔ VNet | Dispositivo VPN |
| **Point-to-Site** | Equipo ↔ VNet | Cliente VPN (certificado/Entra ID) |
| **VNet-to-VNet** | VNet ↔ VNet | Gateways |
| **Peering** | VNet ↔ VNet | Sin gateway |

## AZ-104 Exam Tips

- 🔥 🧠 Subred **GatewaySubnet** obligatoria; **un gateway por VNet**.
- 🔥 🧠 **S2S = red**, **P2S = equipo**, **VNet-to-VNet = redes virtuales**, **peering = sin gateway**.
- 🧠 ExpressRoute **no cifra** por defecto y **no pasa por Internet**.
- 🧠 **Route-based** es el tipo de VPN estándar.
- 🧠 Gateway transit permite a los spokes usar el gateway del hub.
- 📌 VPN (barata, rápida de desplegar) vs ExpressRoute (cara, semanas, alto rendimiento).
- ⚠️ El despliegue de un gateway tarda decenas de minutos: relevante en preguntas de tiempo.

## Preguntas que podrían aparecer

**1.** Un requisito exige que el tráfico entre la sede central y Azure nunca atraviese Internet público. ¿Qué implementas?
- A) VPN Site-to-Site · B) VPN Point-to-Site · C) ExpressRoute · D) Peering global

<details><summary>Respuesta</summary>

**C.** Solo ExpressRoute ofrece conectividad privada sin pasar por Internet.
</details>

**2.** Los spokes de una topología hub-and-spoke deben usar el VPN Gateway desplegado en el hub. ¿Qué configuras en los peerings?
- A) Allow forwarded traffic en ambos · B) Gateway transit en el hub y use remote gateways en los spokes · C) HA Ports · D) Un gateway en cada spoke

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[03 - Peering de redes virtuales]]
- [[04 - Rutas definidas por el usuario (UDR) y NVA]]
- [[12 - Azure Private DNS y resolución de nombres]]
- [[05 - Azure VPN Gateway]] (AZ-900)
- [[08 - Azure ExpressRoute]] (AZ-900)
- [[00 - Índice - Redes virtuales]]
