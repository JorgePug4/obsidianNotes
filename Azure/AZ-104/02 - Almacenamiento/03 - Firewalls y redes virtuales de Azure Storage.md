---
tags: [az-104, azure, almacenamiento, seguridad, firewall, redes]
modulo: Almacenamiento
peso_examen: Muy alto
---

# Firewalls y redes virtuales de Azure Storage

## ¿Qué es?

El **firewall de la cuenta de almacenamiento** controla **desde qué redes** se puede llegar a la cuenta: desde cualquier sitio (público), solo desde redes virtuales y rangos IP concretos, o solo por endpoints privados. Es la **primera capa** de seguridad, antes de la autenticación.

## ¿Para qué sirve?

- Impedir que la cuenta sea accesible desde Internet aunque alguien tenga la clave.
- Permitir solo las subredes de las aplicaciones (service endpoints) o la oficina (IP pública).
- Exponer la cuenta con una IP privada de la VNet (private endpoint).

## Conceptos clave

- **Acceso de red público** (Public network access) 🧠:
  - **Enabled from all networks**: por defecto; cualquier origen (sigue exigiendo autenticación).
  - **Enabled from selected virtual networks and IP addresses**: modo firewall; lista de **subredes** (con service endpoint `Microsoft.Storage`) y **rangos IP públicos** (CIDR, no rangos privados RFC 1918).
  - **Disabled**: solo **private endpoints**.
- **Excepciones**: "Allow Azure services on the trusted services list to access this storage account" (Backup, Site Recovery, Monitor, Event Grid, Defender…), "Allow read access to storage logging/metrics from any network".
- **Service endpoint**: se habilita en la **subred** (Microsoft.Storage); el tráfico va por la red troncal y la cuenta ve la IP privada de la subred. Ver [[09 - Service Endpoints]]. Para VNets en otra región: **cross-region service endpoints** (`Microsoft.Storage.Global`).
- **Private endpoint**: NIC con IP privada en tu VNet que apunta al subrecurso (blob, file, queue, table, dfs, web). Requiere DNS (`privatelink.blob.core.windows.net`). Funciona desde on-premises vía VPN/ExpressRoute. Ver [[10 - Private Endpoint y Private Link]].
- **Instancias de recursos**: permitir el acceso a la cuenta a recursos concretos (por ejemplo, un espacio de trabajo de Synapse) por identidad administrada.
- **Enrutamiento de red**: Microsoft network routing (por defecto) vs Internet routing.
- ⚠️ El firewall **no** afecta a operaciones de plano de control (ARM), pero **sí** al portal: si tu IP no está permitida, el portal no muestra blobs. Cloud Shell tampoco (su IP es de Azure y cambia); hay que permitir la IP o usar una VM en una subred permitida.

## Cómo funciona

```
Petición → ¿Acceso público habilitado?
   ├─ All networks → pasa a autenticación
   ├─ Selected → ¿origen en subred permitida (service endpoint) o IP permitida o servicio de confianza? → sí: autenticación / no: 403 AuthorizationFailure
   └─ Disabled → solo vía private endpoint (IP privada) → autenticación
```

```bash
# Modo firewall: denegar por defecto
az storage account update --name st001 --resource-group rg --default-action Deny
# Permitir subred (requiere service endpoint en la subred)
az network vnet subnet update --vnet-name vnet1 --name app --resource-group rg --service-endpoints Microsoft.Storage
az storage account network-rule add --account-name st001 --resource-group rg --vnet-name vnet1 --subnet app
# Permitir IP pública de la oficina
az storage account network-rule add --account-name st001 --resource-group rg --ip-address 203.0.113.0/24
# Servicios de confianza
az storage account update --name st001 --resource-group rg --bypass AzureServices
# Deshabilitar acceso público (solo private endpoints)
az storage account update --name st001 --resource-group rg --public-network-access Disabled
```

```powershell
Update-AzStorageAccountNetworkRuleSet -ResourceGroupName rg -Name st001 -DefaultAction Deny -Bypass AzureServices
Add-AzStorageAccountNetworkRule -ResourceGroupName rg -Name st001 -VirtualNetworkResourceId $subnet.Id
Add-AzStorageAccountNetworkRule -ResourceGroupName rg -Name st001 -IPAddressOrRange "203.0.113.0/24"
```

## Componentes

| Componente | Dónde | Detalle |
|---|---|---|
| Acceso de red público | Cuenta → Redes → Firewalls y redes virtuales | 3 modos |
| Redes virtuales permitidas | Misma hoja | Subred con service endpoint; se puede añadir el endpoint desde ahí |
| Rangos IP | Misma hoja | Solo IPs públicas; máximo 400 reglas (200 IP + 200 VNet) 🧠 |
| Excepciones | Misma hoja | Servicios de confianza; logging/metrics |
| Private endpoints | Cuenta → Redes → Conexiones de punto de conexión privado | Uno por subrecurso |
| Instancias de recursos | Misma hoja | Acceso por identidad de recurso |

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Solo la subred "app" de VNet1 puede acceder | Firewall Selected + service endpoint Microsoft.Storage en la subred + regla de VNet |
| La oficina (IP pública fija) debe acceder por Internet | Regla de IP con el rango público |
| Acceso desde on-premises por VPN con IP privada | **Private endpoint** (service endpoint no sirve desde on-premises) |
| Azure Backup debe seguir funcionando con el firewall activo | Excepción "servicios de confianza" |
| Cloud Shell no puede listar blobs tras activar el firewall | Permitir tu IP pública o usar una VM en subred permitida; Cloud Shell no es servicio de confianza |
| La cuenta nunca debe ser accesible desde Internet | Public network access = Disabled + private endpoints |
| VNet en otra región | Cross-region service endpoint o private endpoint |

## Ejemplo

Contoso tiene una cuenta con datos de RR. HH. Requisitos: acceso desde las VMs de la subred `hr-app` (misma región), desde el ERP on-premises a través de ExpressRoute, y nunca desde Internet. Solución: **private endpoint** para el subrecurso `blob` (cubre VMs y on-premises con DNS privado) y **public network access = Disabled**. Si solo fueran las VMs de Azure, bastaría con firewall + service endpoint (gratuito).

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Firewall por IP** | Orígenes con IP pública fija | Simple | Oficinas, servicios externos con IP fija |
| **Service endpoint** | Subredes de Azure | Gratis, ruta óptima, fácil | Solo desde VNets de Azure, sin necesidad de IP privada |
| **Private endpoint** | IP privada en la VNet | Funciona desde on-premises, sin exposición pública | Máxima seguridad, híbrido, requisitos de cumplimiento |
| **Servicios de confianza** | Servicios de Azure | Sin abrir IPs | Backup, Monitor, Site Recovery, Defender |

## 💻 Laboratorio: restringir una cuenta a una subred

1. Crear VNet `vnet-lab` con subred `app`; habilitar el service endpoint **Microsoft.Storage** en `app`.
2. En la cuenta → Redes → "Habilitado desde redes virtuales y direcciones IP seleccionadas" → agregar `vnet-lab/app`.
3. Desde el portal (tu IP no permitida) intentar abrir un contenedor: error de autorización. Añadir tu IP pública y comprobar que ya funciona.
4. Crear una VM en `app` y, con `az storage blob list --auth-mode login`, listar blobs.
5. Marcar la excepción de servicios de confianza y explicar para qué sirve.

## AZ-104 Exam Tips

- 🔥 📌 **Service endpoint** = subred de Azure, IP pública del servicio, gratis. **Private endpoint** = IP privada, on-premises, DNS privado, de pago.
- 🔥 ⚠️ Al activar el firewall, **el portal y Cloud Shell dejan de ver los datos** si tu IP no está permitida.
- 🧠 Rangos IP: solo **públicos**; máximo 200 reglas IP + 200 de VNet.
- 🧠 "Servicios de confianza" es la excepción para Backup/Monitor/ASR.
- 💻 `--default-action Deny`, `network-rule add`, `--public-network-access Disabled`.
- ⚠️ El firewall no impide la administración ARM (crear/borrar la cuenta), solo el acceso a datos.

## Errores comunes

- Añadir la subred al firewall sin habilitar el service endpoint en la subred (el portal lo ofrece hacer automáticamente).
- Intentar poner una IP privada (10.x) en las reglas IP.
- Desactivar el acceso público sin haber creado antes el private endpoint y el DNS.

## Preguntas que podrían aparecer

**1.** Configuras el firewall de una cuenta de almacenamiento para permitir solo la subred `Subnet1` de `VNet1`. Una VM en `Subnet1` sigue sin acceder. ¿Qué falta?
- A) Un private endpoint · B) Habilitar el service endpoint Microsoft.Storage en Subnet1 · C) Un NSG · D) Peering

<details><summary>Respuesta</summary>

**B.** Sin el service endpoint en la subred, el tráfico llega con IP pública y el firewall lo rechaza.
</details>

**2.** Los servidores on-premises conectados por VPN Site-to-Site deben acceder a una cuenta de almacenamiento cuya red pública está deshabilitada. ¿Qué configuras?
- A) Service endpoint en la GatewaySubnet · B) Regla de IP con el rango privado on-premises · C) Private endpoint y resolución DNS de la zona privatelink · D) Excepción de servicios de confianza

<details><summary>Respuesta</summary>

**C.** Solo el private endpoint es alcanzable desde on-premises; el service endpoint no aplica a tráfico de fuera de Azure y las reglas IP no aceptan rangos privados.
</details>

## Relacionado

- [[09 - Service Endpoints]]
- [[10 - Private Endpoint y Private Link]]
- [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]]
- [[05 - Claves de acceso y autorización con Microsoft Entra ID]]
- [[Service Endpoints y Private Endpoints]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
