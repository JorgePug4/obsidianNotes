---
tags: [az-104, azure, redes, peering, vnet]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Peering de redes virtuales

## ¿Qué es?

El **emparejamiento (peering)** conecta dos redes virtuales para que sus recursos se comuniquen con **IPs privadas**, como si estuvieran en la misma red, usando el **backbone de Microsoft** (sin Internet, sin gateway, sin cifrado adicional).

- **Peering regional**: VNets en la **misma región**.
- **Peering global**: VNets en **regiones distintas** (incluso en otras suscripciones o tenants).

## ¿Para qué sirve?

- Arquitecturas **hub-and-spoke**: servicios compartidos (firewall, DNS, Bastion, gateway) en el hub y cargas de trabajo en los spokes.
- Comunicar entornos o equipos con redes separadas.

## Conceptos clave 🧠

- **Bidireccional pero con dos objetos**: hay que crear el peering **en ambas VNets** (el portal lo hace en un paso si tienes permisos en las dos). Si solo existe un lado, el estado queda **Initiated**; cuando existen ambos, **Connected**.
- **No transitivo** ⚠️: si A↔B y B↔C, **A no habla con C**. Para lograrlo: peering directo A↔C, o una **NVA/Azure Firewall** en B con **UDR** y "permitir tránsito por puerta de enlace"/"reenvío de tráfico".
- **Rangos sin solapamiento**: obligatorio.
- **Opciones de configuración** por lado:

| Opción | Qué hace |
|---|---|
| **Permitir el acceso a la red virtual** (Allow virtual network access) | Habilita la comunicación (etiqueta `VirtualNetwork` incluye el peer) |
| **Permitir el tráfico reenviado** (Allow forwarded traffic) | Acepta tráfico que **no** se originó en la VNet emparejada (viene de una NVA) |
| **Permitir la puerta de enlace** (Allow gateway transit) | El lado que **tiene** el gateway lo ofrece |
| **Usar puerta de enlace remota** (Use remote gateway) | El lado que **no** tiene gateway lo usa; **no puede tener su propio gateway** |

- **Latencia y ancho de banda**: los limita la VM, no el peering. Sin restricción de ancho de banda del propio peering.
- **Coste**: se paga el **tráfico entrante y saliente** (más caro en global/entre regiones).
- **Peering entre suscripciones o tenants**: posible; requiere permisos (`Network Contributor`) en ambas o usar el `peer resource ID` con autorización.
- **Peering y DNS**: no comparte resolución automáticamente; las zonas **privadas de DNS** deben vincularse a cada VNet ([[12 - Azure Private DNS y resolución de nombres]]).
- **NSG**: la etiqueta de servicio `VirtualNetwork` incluye el espacio de direcciones de las VNets emparejadas, así que **el tráfico entre peers está permitido por defecto** por las reglas predeterminadas.
- **Azure Virtual WAN** ➕ / **AVNM (Azure Virtual Network Manager)** ➕: gestionan topologías de malla y hub-and-spoke a escala, con conectividad transitiva.

## Cómo funciona

```
        ┌──────── hub (10.0.0.0/16) ────────┐
        │  Firewall · Bastion · Gateway     │
        └───▲───────────────────────▲───────┘
   peering  │                       │  peering
        ┌───┴────┐              ┌───┴────┐
        │spoke-A │              │spoke-B │     A y B NO se ven entre sí
        │10.1/16 │              │10.2/16 │     salvo firewall + UDR en el hub
        └────────┘              └────────┘
```

```bash
# Crear ambos lados
az network vnet peering create -g rg-net -n hub-to-spokeA --vnet-name vnet-hub \
  --remote-vnet $(az network vnet show -g rg-net -n vnet-spokeA --query id -o tsv) \
  --allow-vnet-access --allow-forwarded-traffic --allow-gateway-transit
az network vnet peering create -g rg-net -n spokeA-to-hub --vnet-name vnet-spokeA \
  --remote-vnet $(az network vnet show -g rg-net -n vnet-hub --query id -o tsv) \
  --allow-vnet-access --allow-forwarded-traffic --use-remote-gateways
az network vnet peering list -g rg-net --vnet-name vnet-hub -o table
az network vnet peering show -g rg-net --vnet-name vnet-hub -n hub-to-spokeA --query peeringState
az network vnet peering delete -g rg-net --vnet-name vnet-hub -n hub-to-spokeA
```

```powershell
Add-AzVirtualNetworkPeering -Name hub-to-spokeA -VirtualNetwork $hub -RemoteVirtualNetworkId $spokeA.Id -AllowGatewayTransit
Add-AzVirtualNetworkPeering -Name spokeA-to-hub -VirtualNetwork $spokeA -RemoteVirtualNetworkId $hub.Id -UseRemoteGateways
Get-AzVirtualNetworkPeering -VirtualNetworkName vnet-hub -ResourceGroupName rg-net | Select-Object Name, PeeringState
```

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| Los spokes deben usar el VPN Gateway del hub | Hub: **Allow gateway transit**; Spoke: **Use remote gateways** |
| El spoke ya tiene su propio gateway | **No** puede usar "use remote gateways" |
| Spoke A debe hablar con Spoke B | Peering directo **o** NVA/Firewall en el hub + **UDR** + allow forwarded traffic |
| El peering aparece como *Initiated* | Falta crear el **otro lado** |
| El peering aparece como *Disconnected* | Se borró un lado: eliminar y recrear ambos |
| VNets en regiones distintas | **Peering global** (mismas capacidades, con alguna limitación histórica ya resuelta) |
| Tráfico entre VNets debe pasar por el firewall | **UDR** con next hop = IP privada del firewall + allow forwarded traffic |
| Nombres privados compartidos entre VNets | Vincular la **zona DNS privada** a ambas VNets |

## Ejemplo

Contoso monta hub-and-spoke: `vnet-hub` (VPN Gateway, Azure Firewall, Bastion) y dos spokes. En cada peering hub→spoke marca *Allow gateway transit*; en spoke→hub marca *Use remote gateways*. Para que los spokes se comuniquen entre sí, añade en cada spoke una **UDR** con `0.0.0.0/0` y el rango del otro spoke apuntando a la IP privada del Azure Firewall, y activa *Allow forwarded traffic* en los peerings.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Peering regional** | VNets de la misma región | Baja latencia, sin gateway | Hub-and-spoke local |
| **Peering global** | VNets de distintas regiones | Backbone de Microsoft | Multi-región |
| **VPN VNet-to-VNet** | VNets conectadas por gateway | Cifrado IPsec, soporta transitividad limitada | Requisito de cifrado o escenarios legacy |
| **Azure Virtual WAN** | Muchas VNets y sucursales | Transitividad gestionada | Redes grandes |
| **Virtual Network Manager** | Topologías a escala | Malla, grupos de red, reglas de seguridad | Muchas suscripciones |

## 💻 Laboratorio: hub-and-spoke

1. Crear `vnet-hub` (10.0.0.0/16) y `vnet-spoke1` (10.1.0.0/16) con una VM en cada una.
2. Crear el peering en ambos sentidos y comprobar el estado **Connected**.
3. Hacer ping/SSH entre las VMs por IP privada (ajustando el NSG para ICMP si hace falta).
4. Añadir `vnet-spoke2` emparejada solo con el hub y comprobar que **no** hay conectividad spoke1↔spoke2.
5. Consultar las **rutas efectivas** de la NIC y ver las rutas de tipo `VNetPeering`.

## AZ-104 Exam Tips

- ⭐ El peering **no es transitivo**.
- 🔥 🧠 Hay que crearlo **en ambos lados**; estados *Initiated* / *Connected* / *Disconnected*.
- 🔥 🧠 **Allow gateway transit** (quien tiene el gateway) + **Use remote gateways** (quien lo usa, y no puede tener el suyo).
- 🧠 Los rangos **no pueden solaparse**.
- 🧠 El tráfico entre VNets emparejadas está permitido por la regla NSG predeterminada (`VirtualNetwork`).
- 💻 `az network vnet peering create/list/show`, ver rutas efectivas.
- 📌 Peering (VNet↔VNet) vs VPN Gateway (VNet↔on-premises).

## Errores comunes

- Crear solo un lado y no entender el estado *Initiated*.
- Esperar tránsito automático entre spokes.
- Marcar *Use remote gateways* en una VNet que ya tiene su propio gateway.

## Preguntas que podrían aparecer

**1.** VNet1 está emparejada con VNet2 y VNet2 con VNet3. ¿Pueden comunicarse VNet1 y VNet3?
- A) Sí, el peering es transitivo · B) No, salvo que se cree un peering directo o se enrute por una NVA · C) Solo si están en la misma región · D) Solo con gateway transit

<details><summary>Respuesta</summary>

**B.** El peering no es transitivo.
</details>

**2.** Un spoke debe usar el VPN Gateway que está en el hub. ¿Qué opciones configuras?
- A) Gateway transit en el spoke y remote gateways en el hub · B) Gateway transit en el hub y use remote gateways en el spoke · C) Forwarded traffic en ambos · D) Un segundo gateway en el spoke

<details><summary>Respuesta</summary>

**B.** El lado que posee el gateway lo comparte; el lado sin gateway lo consume.
</details>

**3.** El estado de un emparejamiento es *Initiated*. ¿Qué significa?
- A) Está funcionando · B) Falta crear el emparejamiento en la otra red virtual · C) Los rangos se solapan · D) Falta un NSG

<details><summary>Respuesta</summary>

**B.** Hasta que existen ambos objetos de peering el estado no pasa a Connected.
</details>

## Relacionado

- [[01 - Azure Virtual Network y subredes]]
- [[04 - Rutas definidas por el usuario (UDR) y NVA]]
- [[12 - Azure Private DNS y resolución de nombres]]
- [[16 - Conectividad híbrida - VPN Gateway y ExpressRoute]]
- [[00 - Índice - Redes virtuales]]
