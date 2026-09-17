---
tags: [az-104, azure, redes, service-endpoints, paas]
modulo: Redes virtuales
peso_examen: Alto
---

# Service Endpoints (puntos de conexión de servicio)

## ¿Qué es?

Un **punto de conexión de servicio de red virtual** extiende la identidad de la VNet hasta un servicio PaaS de Azure (Storage, SQL, Key Vault…): el tráfico deja de salir por Internet y viaja por el **backbone de Microsoft**, y el servicio puede **restringir el acceso a esa subred** mediante su firewall.

## ¿Para qué sirve?

- Que una cuenta de almacenamiento o una base de datos solo acepte tráfico de subredes concretas.
- Mejorar el enrutamiento (ruta óptima) sin coste adicional.

## Conceptos clave 🧠

- Se habilita **a nivel de subred** y por **servicio** (`Microsoft.Storage`, `Microsoft.Sql`, `Microsoft.KeyVault`, `Microsoft.ServiceBus`, `Microsoft.EventHub`, `Microsoft.AzureCosmosDB`, `Microsoft.Web`, `Microsoft.ContainerRegistry`, `Microsoft.AzureActiveDirectory`…).
- **El servicio PaaS conserva su IP pública**: no se crea ninguna IP privada.
- El recurso PaaS ve la **IP privada** de la VM como origen y aplica su regla de firewall de red virtual.
- **Gratuito**.
- **No funciona desde on-premises** (VPN/ExpressRoute) ⚠️: el origen debe estar en la VNet.
- **Regional por defecto**; existen **service endpoints entre regiones** (`Microsoft.Storage.Global`) para llegar a cuentas de otra región.
- Añade una **ruta del sistema** más específica hacia los prefijos del servicio (visible en rutas efectivas como `VirtualNetworkServiceEndpoint`).
- **Service Endpoint Policies** ➕: restringir a qué **cuentas de almacenamiento concretas** puede salir el tráfico de la subred (evita exfiltración a cuentas ajenas).
- Combinar con el **firewall del recurso**: habilitar el endpoint en la subred **y** añadir la regla de red virtual en el servicio.

## Cómo funciona

```
VM (10.0.1.4) ──ruta de service endpoint──► backbone de Microsoft ──► Storage (IP pública)
                                            Storage ve el origen 10.0.1.4 y su firewall permite snet-app
```

```bash
# 1) Habilitar el endpoint en la subred
az network vnet subnet update -g rg-net --vnet-name vnet-hub -n snet-app --service-endpoints Microsoft.Storage Microsoft.Sql
# 2) Restringir el servicio a esa subred
az storage account update -g rg-data -n st001 --default-action Deny
az storage account network-rule add -g rg-data --account-name st001 --vnet-name vnet-hub --subnet snet-app
# SQL
az sql server vnet-rule create -g rg-data -s sql-contoso -n allow-app --subnet $(az network vnet subnet show -g rg-net --vnet-name vnet-hub -n snet-app --query id -o tsv)
# Comprobar
az network vnet subnet show -g rg-net --vnet-name vnet-hub -n snet-app --query serviceEndpoints
az network nic show-effective-route-table -g rg-net -n nic-app01 -o table   # ver rutas VirtualNetworkServiceEndpoint
```

```powershell
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name snet-app -AddressPrefix 10.0.1.0/24 -ServiceEndpoint Microsoft.Storage | Set-AzVirtualNetwork
Add-AzStorageAccountNetworkRule -ResourceGroupName rg-data -Name st001 -VirtualNetworkResourceId $subnet.Id
```

Portal: VNet → Subredes → subred → **Puntos de conexión de servicio** → elegir servicios. Luego en el recurso PaaS → **Redes** → añadir la red virtual.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Solo las VMs de `snet-app` deben acceder a la cuenta de almacenamiento | Service endpoint en la subred + regla de red virtual en la cuenta + `--default-action Deny` |
| Acceso desde on-premises por VPN | **No sirve el service endpoint** → **private endpoint** |
| El servicio debe tener una IP privada | **Private endpoint** |
| Evitar que la subred escriba en cuentas de almacenamiento ajenas | **Service endpoint policies** |
| VNet y cuenta de almacenamiento en regiones distintas | Service endpoint **entre regiones** o private endpoint |
| Coste mínimo y configuración rápida | **Service endpoint** (gratis) |

## Ejemplo

Una aplicación en `snet-app` usa Azure SQL y una cuenta de almacenamiento. Se habilitan `Microsoft.Sql` y `Microsoft.Storage` en la subred, se pone el firewall de ambos recursos en **Deny** por defecto y se añaden reglas de red virtual para `snet-app`. Ni siquiera con la clave de la cuenta se puede acceder desde fuera de esa subred.

## Comparaciones

| | **Service endpoint** | **Private endpoint** |
|---|---|---|
| IP del servicio | **Pública** (no cambia) | **Privada** en tu VNet |
| Ámbito | **Toda la subred** hacia todo el servicio (o cuentas concretas con policies) | **Un recurso concreto** (por subrecurso) |
| Desde on-premises | **No** | **Sí** (VPN/ExpressRoute) |
| DNS | Sin cambios | Requiere zona `privatelink.*` |
| Coste | **Gratis** | De pago (endpoint + tráfico) |
| Exfiltración de datos | Posible sin policies | Limitada al recurso |
| Configuración | Muy simple | Más piezas (NIC, DNS, aprobación) |
| Recomendación de Microsoft | Válido | **Preferido** para nuevos diseños |

## 💻 Laboratorio: service endpoint con Storage

1. Crear `st<iniciales>se` y subir un blob.
2. Habilitar `Microsoft.Storage` en `snet-app`.
3. En la cuenta → Redes → "Habilitado desde redes virtuales y direcciones IP seleccionadas" → añadir `snet-app`.
4. Desde una VM de `snet-app`, listar blobs (funciona). Desde Cloud Shell o tu equipo, comprobar el error 403.
5. Ver en las rutas efectivas la entrada `VirtualNetworkServiceEndpoint`.

## AZ-104 Exam Tips

- ⭐ Se habilita **en la subred**; el servicio **mantiene su IP pública**.
- 🔥 🧠 **No funciona desde on-premises** → si el enunciado menciona VPN/ExpressRoute, la respuesta es **private endpoint**.
- 🧠 Hay que configurar **las dos partes**: endpoint en la subred **y** regla de red virtual en el recurso.
- 🧠 Es **gratuito**; el private endpoint no.
- 💻 `az network vnet subnet update --service-endpoints`, `az storage account network-rule add`.
- ⚠️ Sin `--default-action Deny` en el recurso, el endpoint no restringe nada.

## Errores comunes

- Habilitar el endpoint y olvidar el firewall del recurso.
- Esperar acceso desde la red local.
- Confundirlo con private endpoint en preguntas de "IP privada".

## Preguntas que podrían aparecer

**1.** Habilitas un service endpoint de Microsoft.Storage en una subred pero cualquiera con la clave sigue accediendo a la cuenta desde Internet. ¿Qué falta?
- A) Un private endpoint · B) Poner el firewall de la cuenta en Deny y añadir la regla de red virtual · C) Una UDR · D) Un NSG

<details><summary>Respuesta</summary>

**B.** El service endpoint habilita la ruta; la restricción la impone el firewall del servicio.
</details>

**2.** Servidores on-premises conectados por ExpressRoute deben acceder de forma privada a una cuenta de almacenamiento. ¿Qué implementas?
- A) Service endpoint en la GatewaySubnet · B) Private endpoint · C) Regla de IP privada en el firewall · D) Service endpoint entre regiones

<details><summary>Respuesta</summary>

**B.** Solo el private endpoint es alcanzable desde fuera de Azure.
</details>

## Relacionado

- [[10 - Private Endpoint y Private Link]]
- [[03 - Firewalls y redes virtuales de Azure Storage]]
- [[04 - Rutas definidas por el usuario (UDR) y NVA]]
- [[Service Endpoints y Private Endpoints]] (AZ-900)
- [[00 - Índice - Redes virtuales]]
