---
tags: [az-104, azure, redes, udr, rutas, nva]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Rutas definidas por el usuario (UDR) y NVA

## ¿Qué es?

Azure crea **rutas del sistema** automáticamente para que todo funcione sin configurar nada. Una **ruta definida por el usuario (UDR)** dentro de una **tabla de rutas (route table)** permite **sobrescribir** ese comportamiento y dirigir el tráfico, por ejemplo, a través de un **dispositivo virtual de red (NVA)** o de **Azure Firewall**.

## ¿Para qué sirve?

- Forzar que todo el tráfico pase por un firewall (inspección).
- Publicar rutas hacia on-premises a través de una NVA.
- Aislar subredes (ruta a `None` = descartar tráfico).

## Conceptos clave

### Rutas del sistema (por defecto) 🧠

| Prefijo | Next hop |
|---|---|
| Rango de la VNet | **Virtual network** |
| `0.0.0.0/0` | **Internet** |
| `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `100.64.0.0/10` | **None** (descartado, salvo que sea de la VNet) |
| Rangos de VNets emparejadas | **VNet peering** |
| Rangos aprendidos por BGP (VPN/ER) | **Virtual network gateway** |
| Rangos de service endpoints | **VirtualNetworkServiceEndpoint** |

### Tipos de next hop en una UDR 🧠
- **Virtual appliance**: IP privada de la NVA/Firewall (requiere **IP forwarding** habilitado en la NIC de la NVA).
- **Virtual network gateway**: VPN o ExpressRoute.
- **Virtual network**: dentro de la VNet.
- **Internet**.
- **None**: descartar el tráfico (agujero negro).

### Reglas de selección 🧠
Azure elige la ruta por este orden de prioridad:
1. **Ruta definida por el usuario (UDR)**
2. **Ruta BGP** (aprendida por VPN/ExpressRoute)
3. **Ruta del sistema**

Dentro del mismo tipo, gana el **prefijo más específico** (longest prefix match). Ejemplo: una UDR `10.0.1.0/24` vence a una BGP `10.0.0.0/16`.

### Otros conceptos
- La tabla de rutas se **asocia a subredes** (una tabla por subred; una tabla puede asociarse a varias subredes).
- **Propagación de rutas de puerta de enlace** (BGP route propagation): se puede **deshabilitar** en la tabla para que las rutas aprendidas del gateway no se apliquen (típico en el hub cuando todo debe ir al firewall).
- **IP forwarding**: obligatorio en la NIC de la NVA para que reenvíe tráfico que no va dirigido a ella.
- **Rutas efectivas**: vista combinada por NIC (`az network nic show-effective-route-table`).
- **Service endpoints** añaden rutas más específicas hacia el servicio.
- **Azure Route Server** ➕: intercambia rutas BGP entre NVAs y la red de Azure sin UDRs manuales.

## Cómo funciona

```bash
az network route-table create -g rg-net -n rt-spoke --disable-bgp-route-propagation false
az network route-table route create -g rg-net --route-table-name rt-spoke -n default-to-fw \
  --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.4.4
az network route-table route create -g rg-net --route-table-name rt-spoke -n to-spoke2 \
  --address-prefix 10.2.0.0/16 --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.4.4
az network route-table route create -g rg-net --route-table-name rt-spoke -n block-legacy \
  --address-prefix 192.168.100.0/24 --next-hop-type None
az network vnet subnet update -g rg-net --vnet-name vnet-spoke1 -n snet-app --route-table rt-spoke
# IP forwarding en la NVA
az network nic update -g rg-net -n nic-nva --ip-forwarding true
# Ver rutas efectivas
az network nic show-effective-route-table -g rg-net -n nic-app01 -o table
```

```powershell
$rt = New-AzRouteTable -ResourceGroupName rg-net -Name rt-spoke -Location westeurope
Add-AzRouteConfig -Name default-to-fw -AddressPrefix 0.0.0.0/0 -NextHopType VirtualAppliance -NextHopIpAddress 10.0.4.4 -RouteTable $rt | Set-AzRouteTable
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name snet-app -AddressPrefix 10.1.1.0/24 -RouteTable $rt | Set-AzVirtualNetwork
Get-AzEffectiveRouteTable -ResourceGroupName rg-net -NetworkInterfaceName nic-app01 | Format-Table
```

## Configuración relevante para el examen

| Escenario | UDR |
|---|---|
| Todo el tráfico a Internet debe pasar por el firewall | `0.0.0.0/0` → **Virtual appliance** (IP privada del firewall) |
| Spoke1 debe llegar a Spoke2 por el firewall del hub | En spoke1: rango de spoke2 → firewall (y viceversa) + *allow forwarded traffic* en los peerings |
| Bloquear el acceso de una subred a un rango concreto | Prefijo → **None** |
| La NVA no reenvía tráfico | Falta **IP forwarding** en su NIC (y en el SO) |
| Que las rutas de la VPN no lleguen a una subred | **Deshabilitar la propagación de rutas BGP** en la tabla |
| ¿Qué ruta se aplica? | UDR > BGP > sistema; a igualdad, prefijo más específico |
| Diagnóstico de enrutamiento | **Rutas efectivas** de la NIC y **Next hop** de Network Watcher |

> [!warning] Cuidado con `0.0.0.0/0` → NVA
> Si aplicas esa ruta a la **subred del propio firewall** o a la **GatewaySubnet** sin cuidado, se crean bucles. Tampoco se debe aplicar a **AzureBastionSubnet** (Bastion necesita salida directa a Internet).

## Ejemplo

Seguridad exige que todo el tráfico de los spokes hacia Internet y entre spokes sea inspeccionado. Se despliega Azure Firewall en `AzureFirewallSubnet` (10.0.4.0/26, IP privada 10.0.4.4). En cada subred de los spokes se asocia una tabla de rutas con `0.0.0.0/0` → 10.0.4.4 y rutas específicas hacia el otro spoke. En los peerings se marca *Allow forwarded traffic*. Las rutas efectivas de una NIC muestran la UDR sustituyendo a la ruta del sistema.

## Comparaciones

| Mecanismo | Qué controla | Nota |
|---|---|---|
| **UDR / tabla de rutas** | **Por dónde va** el tráfico | Sobrescribe rutas del sistema |
| **NSG** | **Si se permite** el tráfico | Filtra, no enruta |
| **Azure Firewall / NVA** | Inspección y filtrado avanzado | Necesita UDR para recibir el tráfico |
| **Peering** | Conectividad entre VNets | Crea rutas automáticas |
| **BGP (VPN/ER)** | Rutas dinámicas con on-premises | Menor prioridad que UDR |
| **Route Server** | Intercambio BGP con NVAs | Evita UDRs manuales |

## 💻 Laboratorio: forzar tráfico por una NVA

1. Crear `vnet-lab` con `snet-app` (10.30.1.0/24) y `snet-nva` (10.30.2.0/24).
2. Desplegar una VM Linux en `snet-nva` con **IP forwarding** habilitado en la NIC y habilitar `net.ipv4.ip_forward=1` en el SO.
3. Crear una tabla de rutas con `0.0.0.0/0` → IP privada de la NVA y asociarla a `snet-app`.
4. Desde una VM en `snet-app`, comprobar con `az network nic show-effective-route-table` que la ruta activa es la UDR.
5. Ejecutar **Next hop** en Network Watcher hacia 8.8.8.8 y verificar que responde `VirtualAppliance`.

## AZ-104 Exam Tips

- ⭐ Orden de prioridad: **UDR > BGP > sistema**; a igualdad, **prefijo más específico**.
- 🔥 🧠 Next hop **Virtual appliance** exige **IP forwarding** en la NIC de la NVA.
- 🔥 🧠 Next hop **None** descarta el tráfico.
- 🧠 La tabla se asocia a **subredes**; se puede **deshabilitar la propagación BGP**.
- 🧠 Rutas del sistema: VNet, Internet (`0.0.0.0/0`), rangos privados a None, peering, gateway.
- 💻 `az network route-table route create`, asociar a subred, `show-effective-route-table`.
- ⚠️ No aplicar `0.0.0.0/0` → NVA a AzureBastionSubnet ni crear bucles en la subred del firewall.

## Errores comunes

- Crear la UDR y olvidar asociarla a la subred.
- No habilitar IP forwarding (el tráfico llega a la NVA y se descarta).
- Confundir UDR (enrutamiento) con NSG (filtrado).

## Preguntas que podrían aparecer

**1.** Una subred tiene una ruta del sistema `0.0.0.0/0 → Internet`, una ruta BGP `0.0.0.0/0 → Virtual network gateway` y una UDR `0.0.0.0/0 → Virtual appliance`. ¿Cuál se aplica?
- A) La del sistema · B) La BGP · C) La UDR · D) Ninguna, hay conflicto

<details><summary>Respuesta</summary>

**C.** Las rutas definidas por el usuario tienen la máxima prioridad.
</details>

**2.** Configuras una UDR que apunta a una máquina virtual NVA, pero el tráfico no llega a su destino. ¿Qué falta?
- A) Un NSG permisivo en la subred · B) Habilitar el reenvío de IP en la NIC de la NVA y en su sistema operativo · C) Un peering · D) Una IP pública

<details><summary>Respuesta</summary>

**B.** Sin IP forwarding, la NVA descarta el tráfico que no está dirigido a su propia IP.
</details>

**3.** Quieres impedir que una subred acceda a un rango de direcciones concreto mediante enrutamiento. ¿Qué tipo de next hop usas?
- A) Internet · B) Virtual network · C) None · D) Virtual network gateway

<details><summary>Respuesta</summary>

**C.** El next hop None descarta el tráfico destinado a ese prefijo.
</details>

## Relacionado

- [[03 - Peering de redes virtuales]]
- [[05 - Network Security Group (NSG)]]
- [[15 - Solución de problemas de conectividad de red]]
- [[16 - Conectividad híbrida - VPN Gateway y ExpressRoute]]
- [[00 - Índice - Redes virtuales]]
