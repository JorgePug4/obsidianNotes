---
tags: [az-104, azure, redes, asg, nsg, seguridad]
modulo: Redes virtuales
peso_examen: Alto
---

# Application Security Group (ASG)

## ¿Qué es?

Un **grupo de seguridad de aplicación (ASG)** es una **etiqueta lógica** que agrupa **interfaces de red** por función (web, app, base de datos). En las reglas de NSG se usa el ASG como **origen o destino** en lugar de direcciones IP.

## ¿Para qué sirve?

- Escribir reglas legibles: "permitir 1433 desde `asg-web` hacia `asg-db`".
- Evitar mantener listas de IPs cuando se añaden o quitan VMs.
- Aplicar microsegmentación dentro de una misma subred.

## Conceptos clave 🧠

- El ASG se asigna a la **configuración IP de una NIC** (no a la VM ni a la subred). Una NIC puede pertenecer a **varios** ASGs.
- Las reglas NSG que usan ASG deben cumplir: todas las NICs de los ASGs implicados deben estar en la **misma red virtual** que el NSG (si origen y destino son ASGs, ambos deben estar en la misma VNet).
- El ASG **no filtra por sí solo**: es solo una agrupación usada por el NSG.
- Se pueden mezclar ASGs con IPs y etiquetas de servicio en distintas reglas, pero **no** combinar ASG con prefijos de dirección en el mismo campo de una regla.
- Límites: 3000 ASGs por suscripción; 100 IP configurations por ASG (valores orientativos, ampliables).
- Permite **microsegmentación**: dos VMs en la misma subred con reglas distintas según su ASG.

## Cómo funciona

```
NIC vm-web01 ─┐
NIC vm-web02 ─┼──► ASG "asg-web"
NIC vm-db01  ────► ASG "asg-db"

Regla NSG: Allow TCP 1433  origen: asg-web  destino: asg-db  prioridad 100
Regla NSG: Deny  Any       origen: *        destino: asg-db  prioridad 200
```

```bash
az network asg create -g rg-net -n asg-web
az network asg create -g rg-net -n asg-db
# Asignar la NIC al ASG
az network nic ip-config update -g rg-net --nic-name nic-web01 -n ipconfig1 --application-security-groups asg-web
az network nic ip-config update -g rg-net --nic-name nic-db01 -n ipconfig1 --application-security-groups asg-db
# Regla con ASGs
az network nsg rule create -g rg-net --nsg-name nsg-app -n allow-web-to-db \
  --priority 100 --direction Inbound --access Allow --protocol Tcp \
  --source-asgs asg-web --destination-asgs asg-db --destination-port-ranges 1433
az network nsg rule create -g rg-net --nsg-name nsg-app -n deny-others-to-db \
  --priority 200 --direction Inbound --access Deny --protocol '*' \
  --source-address-prefixes '*' --destination-asgs asg-db --destination-port-ranges '*'
```

```powershell
$asgWeb = New-AzApplicationSecurityGroup -ResourceGroupName rg-net -Name asg-web -Location westeurope
$nic = Get-AzNetworkInterface -ResourceGroupName rg-net -Name nic-web01
$nic.IpConfigurations[0].ApplicationSecurityGroups = $asgWeb
$nic | Set-AzNetworkInterface
$rule = New-AzNetworkSecurityRuleConfig -Name allow-web-to-db -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
  -SourceApplicationSecurityGroup $asgWeb -DestinationApplicationSecurityGroup $asgDb -SourcePortRange * -DestinationPortRange 1433
```

Portal: **Grupos de seguridad de aplicaciones** → Crear. VM → Redes → **Grupos de seguridad de aplicaciones** → Configurar. NSG → Reglas → origen/destino = *Application security group*.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Las VMs web cambian de IP con frecuencia | Agrupar sus NICs en `asg-web` y usar el ASG en las reglas |
| Dos roles distintos en la **misma subred** con reglas diferentes | ASG por rol (microsegmentación) |
| Añadir una VM nueva al conjunto de servidores web | Asignar su NIC a `asg-web`: las reglas se aplican solas |
| Regla que usa ASG en origen y destino | Ambos ASGs deben estar en la **misma VNet** que el NSG |
| Combinar ASG con un CIDR en el mismo campo | **No se puede**: crear reglas separadas |

## Ejemplo

Una aplicación de tres capas comparte la subred `snet-app`. Se crean `asg-web`, `asg-api` y `asg-db`. El NSG de la subred define: Allow 443 desde Internet a `asg-web`; Allow 8080 desde `asg-web` a `asg-api`; Allow 1433 desde `asg-api` a `asg-db`; Deny Any hacia `asg-db`. Al escalar la capa web, basta con asignar el ASG a las nuevas NICs.

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **ASG** | Agrupar NICs por rol | Reglas legibles, sin IPs, microsegmentación | Aplicaciones con varios roles |
| **Prefijos IP en reglas** | Direcciones concretas | Simple | Orígenes externos fijos |
| **Etiquetas de servicio** | Servicios de Azure e Internet | Se actualizan solas | Storage, Sql, AzureMonitor |
| **Subredes separadas** | Segmentación por red | NSG por subred | Cuando el diseño lo permite |

## AZ-104 Exam Tips

- ⭐ El ASG se asigna a la **configuración IP de la NIC**, no a la VM ni a la subred.
- 🔥 🧠 El ASG **no filtra**: se usa dentro de reglas de **NSG** como origen o destino.
- 🧠 Origen y destino ASG deben estar en la **misma VNet**.
- 🧠 Una NIC puede estar en varios ASGs; permite **microsegmentación dentro de una subred**.
- 💻 `az network asg create`, asignar en `nic ip-config update`, reglas con `--source-asgs/--destination-asgs`.
- ⚠️ No se pueden mezclar ASG y prefijos IP en el mismo campo de una regla.

## Errores comunes

- Intentar asociar un ASG a una subred.
- Usar ASGs de VNets distintas en la misma regla.
- Creer que el ASG sustituye al NSG.

## Preguntas que podrían aparecer

**1.** Necesitas permitir el tráfico SQL solo desde los servidores web, que se crean y destruyen con frecuencia y comparten subred con la base de datos. ¿Qué implementas?
- A) Reglas NSG por IP · B) Grupos de seguridad de aplicación como origen y destino en las reglas del NSG · C) Subredes separadas obligatoriamente · D) Azure Firewall

<details><summary>Respuesta</summary>

**B.** Los ASG permiten microsegmentación dentro de la misma subred sin depender de IPs.
</details>

**2.** ¿A qué elemento se asocia un grupo de seguridad de aplicación?
- A) A la subred · B) A la máquina virtual · C) A la configuración IP de una interfaz de red · D) Al grupo de recursos

<details><summary>Respuesta</summary>

**C.** El ASG se asigna a la configuración IP de la NIC.
</details>

## Relacionado

- [[05 - Network Security Group (NSG)]]
- [[07 - Reglas de seguridad efectivas]]
- [[01 - Azure Virtual Network y subredes]]
- [[00 - Índice - Redes virtuales]]
