---
tags: [az-104, azure, redes, nsg, seguridad]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Network Security Group (NSG)

## ¿Qué es?

Un **grupo de seguridad de red (NSG)** es un firewall de **capa 3-4** con reglas **Allow/Deny** por origen, destino, puerto y protocolo, que se asocia a **subredes** y/o a **interfaces de red (NIC)**. Fundamentos en [[03 - Grupo de seguridad de red (NSG)]] (AZ-900); aquí se profundiza en reglas, prioridades, etiquetas y evaluación.

## ¿Para qué sirve?

- Permitir solo los puertos necesarios (443 a web, 1433 a base de datos).
- Aislar subredes entre sí.
- Bloquear el acceso a Internet de una subred.

## Conceptos clave

### Anatomía de una regla 🧠
`Prioridad (100-4096) · Origen · Puerto de origen · Destino · Puerto de destino · Protocolo (TCP/UDP/ICMP/Any) · Acción (Allow/Deny) · Dirección (Inbound/Outbound)`

- **Menor número de prioridad = se evalúa antes**; la **primera coincidencia gana** y se detiene la evaluación.
- **Stateful**: si se permite una conexión entrante, la respuesta sale sin regla explícita.
- **Origen/destino** pueden ser: IP/CIDR, **etiqueta de servicio**, **grupo de seguridad de aplicación (ASG)** o `Any`.

### Reglas predeterminadas (no se borran, se sobrescriben con prioridad menor) 🧠

**Entrada:**
| Prioridad | Nombre | Origen | Destino | Acción |
|---|---|---|---|---|
| 65000 | AllowVNetInBound | VirtualNetwork | VirtualNetwork | Allow |
| 65001 | AllowAzureLoadBalancerInBound | AzureLoadBalancer | Any | Allow |
| 65500 | DenyAllInBound | Any | Any | **Deny** |

**Salida:**
| Prioridad | Nombre | Origen | Destino | Acción |
|---|---|---|---|---|
| 65000 | AllowVnetOutBound | VirtualNetwork | VirtualNetwork | Allow |
| 65001 | AllowInternetOutBound | Any | Internet | Allow |
| 65500 | DenyAllOutBound | Any | Any | **Deny** |

### Etiquetas de servicio (service tags) 🧠
Grupos de prefijos IP gestionados por Microsoft que se actualizan solos: `Internet`, `VirtualNetwork`, `AzureLoadBalancer`, `AzureCloud`, `Storage`, `Sql`, `AzureActiveDirectory`, `AzureMonitor`, `AzureBastionSubnet`? (no), `GatewayManager`, `AzureFrontDoor.Backend`, etc. Se pueden usar con sufijo de región (`Storage.WestEurope`).

### Asociación y evaluación 🧠
- Un NSG se asocia a **subred** y/o **NIC**; **nunca a la VNet completa** ni a una VM directamente.
- **Entrada**: se evalúa primero el NSG de la **subred**, luego el de la **NIC**.
- **Salida**: primero el de la **NIC**, luego el de la **subred**.
- Ambos deben **permitir** para que el tráfico pase: gana el **más restrictivo**.
- El tráfico **dentro de la misma subred** solo se filtra por los NSG de NIC (el de subred no se aplica entre recursos de esa misma subred si el tráfico no la abandona... en la práctica el NSG de subred **sí** evalúa el tráfico intra-subred en Azure; el examen espera que ambos NSGs se evalúen).
- **Puertos y direcciones especiales**: `168.63.129.16` (sondas de estado, DHCP, DNS de Azure) está permitido por `AzureLoadBalancer`/plataforma; bloquearlo rompe sondas y agente.
- **Límites**: 1000 reglas por NSG (por defecto 200 ajustable), 5000 NSGs por suscripción.
- **Registro de flujos (flow logs)**: NSG flow logs **se retiran el 30/09/2027** → migrar a **VNet flow logs** ([[15 - Solución de problemas de conectividad de red]]).
- **Reglas aumentadas (augmented rules)**: varias IPs, rangos de puertos y ASGs en una sola regla.

## Cómo funciona

```bash
az network nsg create -g rg-net -n nsg-web
az network nsg rule create -g rg-net --nsg-name nsg-web -n allow-https \
  --priority 100 --direction Inbound --access Allow --protocol Tcp \
  --source-address-prefixes Internet --destination-port-ranges 443
az network nsg rule create -g rg-net --nsg-name nsg-web -n allow-rdp-oficina \
  --priority 110 --direction Inbound --access Allow --protocol Tcp \
  --source-address-prefixes 203.0.113.10/32 --destination-port-ranges 3389
az network nsg rule create -g rg-net --nsg-name nsg-web -n deny-internet-out \
  --priority 200 --direction Outbound --access Deny --protocol '*' \
  --source-address-prefixes '*' --destination-address-prefixes Internet --destination-port-ranges '*'
az network nsg rule create -g rg-net --nsg-name nsg-web -n allow-storage-out \
  --priority 150 --direction Outbound --access Allow --protocol Tcp \
  --destination-address-prefixes Storage.WestEurope --destination-port-ranges 443
# Asociar
az network vnet subnet update -g rg-net --vnet-name vnet-hub -n snet-web --network-security-group nsg-web
az network nic update -g rg-net -n nic-web01 --network-security-group nsg-web
az network nsg rule list -g rg-net --nsg-name nsg-web -o table
```

```powershell
$rule = New-AzNetworkSecurityRuleConfig -Name allow-https -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
  -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 443
New-AzNetworkSecurityGroup -ResourceGroupName rg-net -Location westeurope -Name nsg-web -SecurityRules $rule
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name snet-web -AddressPrefix 10.0.1.0/24 -NetworkSecurityGroup $nsg | Set-AzVirtualNetwork
```

## Configuración relevante para el examen

| Escenario | Regla |
|---|---|
| Solo HTTPS desde Internet | Inbound Allow TCP 443 desde `Internet`, prioridad baja (100) |
| RDP solo desde la oficina | Inbound Allow TCP 3389 desde la IP pública de la oficina |
| Bloquear toda salida a Internet pero permitir Storage | Outbound Deny a `Internet` (prio 200) + Allow a `Storage` (prio 150) |
| La web no debe hablar con la base de datos salvo 1433 | En `snet-data`: Allow TCP 1433 desde `snet-web`, Deny del resto de VNet |
| Las sondas del balanceador no llegan | Permitir origen `AzureLoadBalancer` |
| No usar IPs en las reglas, sino roles | **ASG** ([[06 - Application Security Group (ASG)]]) |
| Filtrar por FQDN o inspeccionar amenazas | **Azure Firewall**, no NSG |
| Proteger una app web de inyección SQL | **WAF** en Application Gateway/Front Door |

## Ejemplo

`snet-web` tiene `nsg-web`: Allow 443 desde Internet (100), Allow 80 desde Internet (110), Deny Any (4000, opcional porque ya existe la predeterminada). `snet-data` tiene `nsg-data`: Allow TCP 1433 desde el ASG `asg-web` (100) y Deny desde `VirtualNetwork` (200). Resultado: la capa web solo expone HTTP/HTTPS y la base de datos solo acepta SQL desde los servidores web.

## Comparaciones

| Servicio | Capa | Ámbito | Filtra por | Cuándo utilizarlo |
|---|---|---|---|---|
| **NSG** | 3-4 | Subred / NIC | IP, puerto, protocolo, etiquetas, ASG | Segmentación básica, siempre |
| **Azure Firewall** | 3-7 | VNet / hub | FQDN, categorías, inteligencia de amenazas | Perímetro centralizado |
| **WAF (App Gateway / Front Door)** | 7 | Aplicación web | OWASP, reglas personalizadas | Proteger apps web |
| **ASG** | — | Agrupación lógica | Se usa **dentro** del NSG | Evitar IPs en las reglas |
| **Restricciones de acceso de App Service** | 7 | App | IP, etiquetas | PaaS web |

## 💻 Laboratorio: NSG

1. Crear `nsg-lab` y asociarlo a `snet-web`.
2. Añadir Allow 443 desde Internet (prio 100) y comprobar el acceso; añadir Deny 443 con prio 90 y comprobar que ahora falla.
3. Crear un NSG en la NIC de la VM que deniegue 443 y verificar que **gana el más restrictivo**.
4. Ver **Reglas de seguridad efectivas** en la NIC ([[07 - Reglas de seguridad efectivas]]).
5. Usar **Verificación de flujo de IP** de Network Watcher para saber qué regla bloquea.

## AZ-104 Exam Tips

- ⭐ Prioridad **100-4096**, **menor número gana**, **primera coincidencia** detiene la evaluación.
- 🔥 🧠 Se asocia a **subred y/o NIC**, nunca a la VNet.
- 🔥 🧠 Entrada: **subred → NIC**; salida: **NIC → subred**; **ambos deben permitir**.
- 🧠 Reglas predeterminadas: permiten VNet y balanceador, **deniegan el resto de entrada**; permiten salida a Internet.
- 🧠 **Etiquetas de servicio** (Internet, VirtualNetwork, AzureLoadBalancer, Storage, Sql…) se actualizan solas.
- 💻 `az network nsg rule create` con prioridad, dirección, protocolo y etiquetas; asociar a subred/NIC.
- 📌 NSG filtra, **UDR enruta**.
- ⚠️ NSG flow logs se retiran en 2027 → **VNet flow logs**.

## Errores comunes

- Pensar que 4000 tiene más prioridad que 100.
- Poner la regla solo en la NIC cuando el enunciado pide proteger toda la subred.
- Bloquear `AzureLoadBalancer` y romper las sondas de estado.

## Preguntas que podrían aparecer

**1.** Un NSG de subred permite el puerto 80 y el NSG de la NIC lo deniega. ¿Qué ocurre con el tráfico entrante al puerto 80?
- A) Se permite · B) Se deniega · C) Depende de la prioridad global · D) Error de configuración

<details><summary>Respuesta</summary>

**B.** Ambos NSG deben permitir el tráfico; gana el más restrictivo.
</details>

**2.** Necesitas permitir el acceso saliente de una subred solo a Azure Storage de tu región, bloqueando el resto de Internet. ¿Qué configuras?
- A) Una UDR con next hop None · B) Regla Allow a la etiqueta `Storage.WestEurope` con prioridad menor y regla Deny a `Internet` con prioridad mayor · C) Un private endpoint · D) Azure Firewall obligatoriamente

<details><summary>Respuesta</summary>

**B.** Las etiquetas de servicio permiten expresar el destino y el orden de prioridad decide.
</details>

**3.** ¿A qué elementos se puede asociar un grupo de seguridad de red?
- A) Máquina virtual y VNet · B) Subred e interfaz de red · C) Solo subred · D) Grupo de recursos

<details><summary>Respuesta</summary>

**B.** Subred y/o NIC.
</details>

## Relacionado

- [[06 - Application Security Group (ASG)]]
- [[07 - Reglas de seguridad efectivas]]
- [[04 - Rutas definidas por el usuario (UDR) y NVA]]
- [[15 - Solución de problemas de conectividad de red]]
- [[03 - Grupo de seguridad de red (NSG)]] (AZ-900)
- [[00 - Índice - Redes virtuales]]
