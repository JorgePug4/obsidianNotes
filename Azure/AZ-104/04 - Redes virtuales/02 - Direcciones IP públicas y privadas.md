---
tags: [az-104, azure, redes, ip, public-ip, nic]
modulo: Redes virtuales
peso_examen: Alto
---

# Direcciones IP públicas y privadas

## ¿Qué es?

Cada recurso conectado a una VNet tiene al menos una **IP privada** (de la subred) y, opcionalmente, una **IP pública** para ser accesible desde Internet o para salir a Internet con una identidad conocida.

## Conceptos clave

### IP privada
- Se asigna a la **configuración IP de una NIC** (o a un balanceador interno, un private endpoint, etc.).
- **Asignación dinámica** (por defecto, del DHCP de Azure; se conserva mientras la VM no se desasigne) o **estática** (se fija y nunca cambia).
- ⚠️ La IP estática se configura **en Azure**, no dentro del sistema operativo. Configurarla a mano en el SO rompe la conectividad.
- Una NIC puede tener **varias configuraciones IP** (IP secundarias), útiles para SSL por sitio o para NVAs.
- Una VM puede tener **varias NICs** (según el tamaño), y todas deben estar en la misma VNet (pueden estar en subredes distintas).

### IP pública 🧠

| Propiedad | Basic (**retirada el 30/09/2025**) | **Standard** |
|---|---|---|
| Asignación | Dinámica o estática | **Siempre estática** |
| Seguridad | Abierta por defecto | **Cerrada por defecto**: necesita NSG que permita |
| Zonas | No | **Zona-redundante o zonal** |
| SLA | No | 99,99 % |
| Uso | Legado | Todo lo nuevo |

- **SKU Standard** es la única opción para despliegues nuevos.
- **Nivel (tier)**: **Regional** o **Global** (para Front Door / Cross-region Load Balancer).
- **Etiqueta DNS**: `<label>.<region>.cloudapp.azure.com`.
- **Prefijo de IP pública (Public IP Prefix)** ➕: rango contiguo de IPs públicas reservadas.
- **IPv6**: soportado con configuración de doble pila.
- Una IP pública puede asociarse a: NIC de VM, **Load Balancer**, **Application Gateway**, **VPN/ExpressRoute Gateway**, **NAT Gateway**, **Azure Bastion**, **Azure Firewall**.
- Al **desasignar** una VM con IP pública **dinámica**, la IP se libera; con **estática** se conserva.

### Salida a Internet (SNAT)
- Sin IP pública ni NAT Gateway, las VMs salen con una IP pública compartida asignada por Azure (**default outbound access**), que Microsoft está retirando para despliegues nuevos.
- Opciones explícitas de salida 🧠: **NAT Gateway** (recomendado), **IP pública en la NIC**, **reglas de salida del Load Balancer estándar**.

## Cómo funciona

```bash
# IP pública estándar estática con etiqueta DNS
az network public-ip create -g rg-net -n pip-web01 --sku Standard --allocation-method Static --zone 1 2 3 --dns-name contoso-web01
# Asociar a una NIC
az network nic ip-config update -g rg-net --nic-name nic-web01 -n ipconfig1 --public-ip-address pip-web01
# IP privada estática
az network nic ip-config update -g rg-net --nic-name nic-web01 -n ipconfig1 --private-ip-address 10.0.1.10 --set privateIpAllocationMethod=Static
# IP secundaria
az network nic ip-config create -g rg-net --nic-name nic-web01 -n ipconfig2 --private-ip-address 10.0.1.11
# NAT Gateway para salida
az network nat gateway create -g rg-net -n nat-web --public-ip-addresses pip-nat --idle-timeout 10
az network vnet subnet update -g rg-net --vnet-name vnet-hub -n snet-web --nat-gateway nat-web
# Desasociar
az network nic ip-config update -g rg-net --nic-name nic-web01 -n ipconfig1 --remove publicIpAddress
```

```powershell
$pip = New-AzPublicIpAddress -ResourceGroupName rg-net -Name pip-web01 -Location westeurope -Sku Standard -AllocationMethod Static -Zone 1,2,3
$nic = Get-AzNetworkInterface -ResourceGroupName rg-net -Name nic-web01
$nic.IpConfigurations[0].PublicIpAddress = $pip
$nic | Set-AzNetworkInterface
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| La IP pública debe mantenerse tras apagar la VM | **Standard** (siempre estática) o Basic estática |
| Un servidor debe tener siempre la misma IP privada | IP privada **estática en Azure**, no en el SO |
| Varias VMs deben salir a Internet con una única IP conocida | **NAT Gateway** con IP pública estática |
| Alta disponibilidad de la IP pública entre zonas | IP **Standard zona-redundante** |
| Nombre DNS público sencillo para una demo | **Etiqueta DNS** (`<label>.<region>.cloudapp.azure.com`) |
| Tras crear la IP Standard, la VM no responde | El SKU Standard está **cerrado por defecto**: falta la regla NSG |
| Balanceador y VMs deben tener SKUs compatibles | IP pública y Load Balancer **ambos Standard** |

## Ejemplo

Un clúster de 6 VMs sin IP pública debe llamar a una API externa que solo acepta una IP de origen concreta. Se crea una **IP pública Standard estática**, un **NAT Gateway** asociado a la subred y todas las VMs salen con esa IP. Además, se elimina la dependencia del acceso saliente predeterminado, que está en retirada.

## Comparaciones

| Opción de salida | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **NAT Gateway** | Salida de una o varias subredes | IP fija, sin agotamiento de puertos SNAT, simple | Recomendado por defecto |
| **IP pública en la NIC** | Salida y entrada directa | Simple | VMs que deben ser accesibles |
| **Reglas de salida de Load Balancer** | Salida del backend pool | Control de puertos SNAT | Ya hay un balanceador estándar |
| **Acceso saliente predeterminado** | Implícito | Ninguna configuración | En retirada, no usar |

## AZ-104 Exam Tips

- 🔥 🧠 **IP pública Standard: siempre estática, cerrada por defecto (necesita NSG), zona-redundante.** Basic está retirada.
- 🔥 🧠 La IP privada estática se define **en Azure**; nunca dentro del sistema operativo.
- 🧠 IP dinámica se libera al **desasignar** la VM.
- 🧠 Para IP de salida fija → **NAT Gateway**.
- 💻 `az network public-ip create --sku Standard`, asociar/desasociar de la NIC, NAT Gateway en la subred.
- 📌 SKU de la IP y del Load Balancer deben **coincidir** (Standard con Standard).

## Errores comunes

- Configurar la IP estática dentro de Windows/Linux.
- Esperar que una IP pública Standard acepte tráfico sin reglas NSG.
- Mezclar IP Basic con Load Balancer Standard.

## Preguntas que podrían aparecer

**1.** Creas una VM con IP pública SKU Standard y no puedes conectarte por RDP aunque el puerto está escuchando. ¿Cuál es la causa más probable?
- A) La IP es dinámica · B) El SKU Standard está cerrado por defecto y falta una regla de entrada en el NSG · C) La VM está en otra región · D) Falta un NAT Gateway

<details><summary>Respuesta</summary>

**B.** Las IP públicas Standard requieren que el NSG permita explícitamente el tráfico entrante.
</details>

**2.** Varias VMs sin IP pública deben salir a Internet siempre con la misma dirección. ¿Qué implementas?
- A) Service endpoint · B) NAT Gateway con IP pública estática · C) Peering · D) Azure Bastion

<details><summary>Respuesta</summary>

**B.** El NAT Gateway proporciona una IP de salida fija y escalable para toda la subred.
</details>

## Relacionado

- [[01 - Azure Virtual Network y subredes]]
- [[05 - Network Security Group (NSG)]]
- [[13 - Azure Load Balancer]]
- [[08 - Azure Bastion]]
- [[00 - Índice - Redes virtuales]]
