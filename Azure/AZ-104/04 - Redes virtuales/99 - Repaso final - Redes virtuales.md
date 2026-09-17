---
tags: [az-104, azure, redes, repaso]
modulo: Redes virtuales
---

# 🎯 Repaso final · Redes virtuales (15-20 %)

## 1. Los conceptos más importantes

1. **VNet**: una región y una suscripción; **5 IPs reservadas** por subred; subredes con nombre obligatorio (GatewaySubnet, **AzureBastionSubnet /26**, AzureFirewallSubnet). ([[01 - Azure Virtual Network y subredes]])
2. **IPs**: Standard = estática, cerrada por defecto, zona-redundante; IP privada estática **en Azure**; NAT Gateway para salida fija. ([[02 - Direcciones IP públicas y privadas]])
3. **Peering**: ambos lados, **no transitivo**, sin solapamientos, gateway transit + use remote gateways. ([[03 - Peering de redes virtuales]])
4. **UDR**: prioridad **UDR > BGP > sistema**; next hop Virtual appliance requiere **IP forwarding**; None descarta. ([[04 - Rutas definidas por el usuario (UDR) y NVA]])
5. **NSG**: prioridad 100-4096, menor gana, primera coincidencia; subred y/o NIC; **ambos deben permitir**; stateful; etiquetas de servicio. ([[05 - Network Security Group (NSG)]])
6. **ASG**: se asigna a la **configuración IP de la NIC**; se usa dentro de reglas NSG; microsegmentación. ([[06 - Application Security Group (ASG)]])
7. **Reglas efectivas**: entrada subred→NIC, salida NIC→subred; **IP flow verify** dice qué regla bloquea. ([[07 - Reglas de seguridad efectivas]])
8. **Bastion**: AzureBastionSubnet /26; RDP/SSH por 443 desde el portal; Standard = cliente nativo y shareable links; Premium = grabación. ([[08 - Azure Bastion]])
9. **Service endpoint**: en la subred, el servicio mantiene IP pública, gratis, **no desde on-premises**. ([[09 - Service Endpoints]])
10. **Private endpoint**: IP privada, un endpoint **por subrecurso**, **DNS `privatelink.*` imprescindible**, funciona desde on-premises. ([[10 - Private Endpoint y Private Link]])
11. **Azure DNS público**: hospeda, no registra; **delegación NS**; registros **alias** para el apex. ([[11 - Azure DNS (zonas públicas)]])
12. **DNS privado**: el DNS de Azure **no cruza VNets**; zona privada vinculada; **un solo vínculo con autorregistro**; Private Resolver para híbrido. ([[12 - Azure Private DNS y resolución de nombres]])
13. **Load Balancer**: capa 4; frontend + backend pool + sonda + regla; sondas desde **168.63.129.16**; Standard cerrado por defecto. ([[13 - Azure Load Balancer]])
14. **Troubleshooting de balanceo**: sonda HTTP exige **200 OK**; SNAT agotado → NAT Gateway; idle timeout 4 min. ([[14 - Solución de problemas de balanceo de carga]])
15. **Diagnóstico de red**: DNS → rutas → NSG → app → SO; IP flow verify, Next hop, Connection troubleshoot/Monitor, **VNet flow logs**. ([[15 - Solución de problemas de conectividad de red]])
16. **Híbrido**: S2S red, P2S equipo, ExpressRoute privado sin cifrar. ([[16 - Conectividad híbrida - VPN Gateway y ExpressRoute]])
17. **Balanceadores**: LB capa 4 regional, App Gateway capa 7 regional con WAF, Front Door capa 7 global, Traffic Manager DNS. ([[17 - Comparación de balanceadores (Load Balancer, Application Gateway, Front Door, Traffic Manager)]])

## 2. Tabla de decisión rápida

| Si el enunciado dice… | Respuesta |
|---|---|
| "¿cuántas IPs útiles?" | 2^(32−prefijo) − 5 |
| "conectar dos VNets" | Peering (crear ambos lados) |
| "spoke A con spoke B" | No es transitivo: firewall + UDR o peering directo |
| "todo el tráfico por el firewall" | UDR 0.0.0.0/0 → IP privada del firewall + IP forwarding |
| "permitir/denegar por puerto" | NSG |
| "agrupar VMs por rol en las reglas" | ASG |
| "qué regla bloquea" | IP flow verify |
| "por dónde sale el tráfico" | Next hop / rutas efectivas |
| "RDP sin IP pública" | Azure Bastion |
| "grabar sesiones administrativas" | Bastion Premium |
| "cliente RDP nativo con Bastion" | Bastion Standard |
| "solo desde esta subred, gratis" | Service endpoint + firewall del recurso |
| "IP privada para el PaaS / desde on-premises" | Private endpoint + zona privatelink |
| "el nombre resuelve a IP pública" | Falta zona DNS privada vinculada |
| "VMs de VNets distintas no se resuelven" | Zona DNS privada vinculada a ambas |
| "apex a Front Door" | Registro alias |
| "balancear UDP" | Load Balancer |
| "enrutar por URL / WAF" | Application Gateway |
| "web global con caché" | Front Door |
| "elegir región por DNS" | Traffic Manager |
| "backend no saludable" | NSG bloquea AzureLoadBalancer o la sonda no devuelve 200 |
| "misma VM para el mismo cliente" | Distribución Client IP |
| "RDP a VMs concretas por puertos distintos" | Reglas NAT de entrada |

## 3. Números que debo memorizar

| Dato | Valor |
|---|---|
| IPs reservadas por subred | 5 |
| Subred mínima / máxima | /29 … /2 |
| AzureBastionSubnet | /26 mínimo |
| GatewaySubnet | /27 recomendado |
| AzureFirewallSubnet | /26 |
| Prioridad de reglas NSG | 100-4096 |
| Reglas predeterminadas NSG | 65000, 65001, 65500 |
| IP de sondas y servicios de plataforma | 168.63.129.16 |
| Backend pool Standard LB | 1000 instancias |
| Idle timeout LB | 4 min (4-30) |
| Intervalo de sonda por defecto | 5 s |
| SLA Standard LB / VPN Gateway | 99,99 % / 99,95 % |
| Retirada de NSG flow logs | 30/09/2027 |
| Retirada de Basic LB / Basic IP | 30/09/2025 |
| Vínculos con autorregistro por VNet | 1 |

## 4. Diferencias que más fácil puedo confundir

| Pareja | Diferencia |
|---|---|
| NSG vs UDR | Filtra vs enruta |
| NSG vs Azure Firewall | Capa 3-4 por subred/NIC vs capa 3-7 centralizado con FQDN |
| NSG vs ASG | Reglas vs agrupación de NICs usada en las reglas |
| NSG de subred vs de NIC | Ambos se evalúan; gana el más restrictivo |
| Service endpoint vs private endpoint | IP pública/gratis/solo Azure vs IP privada/de pago/on-premises |
| Zona DNS pública vs privada | Internet con delegación NS vs VNets vinculadas |
| Autorregistro vs resolución | Crea registros A vs solo resuelve |
| Peering vs VPN VNet-to-VNet | Sin gateway vs con gateway y cifrado |
| Gateway transit vs use remote gateways | Quien ofrece vs quien consume |
| S2S vs P2S | Red vs equipo |
| VPN vs ExpressRoute | Internet cifrado vs privado sin cifrar |
| LB vs App Gateway | Capa 4 vs capa 7 |
| App Gateway vs Front Door | Regional vs global |
| Front Door vs Traffic Manager | Proxy vs DNS |
| Sonda TCP vs HTTP | Puerto abierto vs respuesta 200 |
| IP Basic vs Standard | Retirada vs estática/cerrada/zonal |
| Bastion Developer/Basic/Standard/Premium | Compartido / dedicado / cliente nativo / grabación |

## 5. Checklist de dominio

- [ ] Sé calcular IPs útiles y diseñar subredes con los nombres reservados.
- [ ] Sé crear IPs públicas Standard y configurar salida con NAT Gateway.
- [ ] Sé configurar peering en ambos lados con gateway transit.
- [ ] Sé crear UDRs y explicar la prioridad de rutas.
- [ ] Sé escribir reglas NSG con prioridades y etiquetas de servicio.
- [ ] Sé usar ASGs para microsegmentación.
- [ ] Sé interpretar reglas efectivas y usar IP flow verify.
- [ ] Sé desplegar Bastion con su subred y elegir el SKU.
- [ ] Sé configurar service endpoints y el firewall del recurso.
- [ ] Sé crear private endpoints con su zona DNS privada.
- [ ] Sé crear zonas DNS públicas, delegarlas y usar alias.
- [ ] Sé crear zonas privadas con autorregistro y vínculos.
- [ ] Sé configurar un Load Balancer público e interno con sondas y reglas.
- [ ] Sé diagnosticar sondas, SNAT y timeouts.
- [ ] Sé qué herramienta de Network Watcher usar en cada síntoma.
- [ ] Sé elegir entre LB, App Gateway, Front Door y Traffic Manager.

## 6. Preguntas de repaso

**1.** ¿Cuántas direcciones puede usar una VM en la subred 10.1.2.0/28?
- A) 16 · B) 14 · C) 11 · D) 13

<details><summary>Respuesta</summary>**C.** 16 − 5 = 11.</details>

---

**2.** VNet1 ↔ VNet2 y VNet2 ↔ VNet3 están emparejadas. ¿VNet1 llega a VNet3?
- A) Sí · B) No, salvo peering directo o NVA con UDR · C) Solo con gateway transit · D) Solo en la misma región

<details><summary>Respuesta</summary>**B.**</details>

---

**3.** El NSG de la subred permite 443 pero el de la NIC no tiene reglas personalizadas. ¿Pasa el tráfico?
- A) Sí · B) No, DenyAllInBound del NSG de la NIC lo bloquea · C) Depende de la prioridad · D) Solo desde la VNet

<details><summary>Respuesta</summary>**B.**</details>

---

**4.** ¿Qué next hop descarta el tráfico en una tabla de rutas?
- A) Internet · B) Virtual network · C) None · D) Virtual appliance

<details><summary>Respuesta</summary>**C.**</details>

---

**5.** Necesitas acceso privado a Azure SQL desde la red local por VPN. ¿Qué implementas?
- A) Service endpoint · B) Private endpoint con zona privatelink y DNS · C) Regla de firewall por IP · D) Peering

<details><summary>Respuesta</summary>**B.**</details>

---

**6.** Las VMs de dos VNets emparejadas no resuelven sus nombres. ¿Qué falta?
- A) Peering · B) Zona DNS privada vinculada a ambas · C) NSG · D) UDR

<details><summary>Respuesta</summary>**B.**</details>

---

**7.** El backend del balanceador aparece no saludable. La app responde en localhost. ¿Qué revisas?
- A) El SKU · B) Que el NSG permita AzureLoadBalancer y que la sonda reciba 200 OK · C) El idle timeout · D) Las zonas

<details><summary>Respuesta</summary>**B.**</details>

---

**8.** ¿Qué SKU de Bastion permite usar el cliente RDP nativo del equipo?
- A) Developer · B) Basic · C) Standard · D) Ninguno

<details><summary>Respuesta</summary>**C.**</details>

---

**9.** Quieres que el dominio raíz apunte a Azure Front Door. ¿Qué registro creas en Azure DNS?
- A) CNAME · B) Alias · C) TXT · D) NS

<details><summary>Respuesta</summary>**B.**</details>

---

**10.** Una aplicación web necesita WAF y enrutamiento por ruta en una sola región. ¿Qué servicio?
- A) Load Balancer · B) Application Gateway · C) Traffic Manager · D) NAT Gateway

<details><summary>Respuesta</summary>**B.**</details>

---

**11.** ¿Qué herramienta indica qué regla de NSG bloquea una conexión concreta?
- A) Next hop · B) IP flow verify · C) Topology · D) Effective routes

<details><summary>Respuesta</summary>**B.**</details>

---

**12.** Un service endpoint está habilitado en la subred pero el acceso desde la sede local por VPN falla. ¿Por qué?
- A) Falta el firewall · B) Los service endpoints no funcionan desde on-premises · C) Falta una UDR · D) Falta DNS

<details><summary>Respuesta</summary>**B.**</details>

> [!tip] Última pasada
> **5 IPs reservadas**, **peering no transitivo**, **UDR > BGP > sistema**, **ambos NSG deben permitir**, **service vs private endpoint**, **zona privada vinculada**, **sondas desde 168.63.129.16**, **capa 4 vs capa 7 vs global vs DNS**.

Volver: [[00 - Índice - Redes virtuales]] · [[00 - AZ-104 Índice general (MOC)]]
