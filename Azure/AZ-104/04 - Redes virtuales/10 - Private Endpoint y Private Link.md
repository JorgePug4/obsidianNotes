---
tags: [az-104, azure, redes, private-endpoint, private-link, dns]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Private Endpoint y Private Link

## ¿Qué es?

Un **punto de conexión privado (private endpoint)** es una **interfaz de red con una IP privada de tu VNet** que representa a un servicio PaaS (o a un servicio propio publicado con **Private Link Service**). El tráfico viaja por la red privada de Microsoft y el servicio deja de necesitar exposición pública.

**Azure Private Link** es la tecnología que hace posible esa conexión privada.

## ¿Para qué sirve?

- Acceder a Storage, SQL, Key Vault, App Service, ACR, Cosmos DB… con **IP privada**.
- Permitir el acceso **desde on-premises** por VPN/ExpressRoute.
- Eliminar por completo la exposición pública del recurso.

## Conceptos clave 🧠

- **Subrecurso (group ID)**: se elige qué parte del servicio se expone: `blob`, `file`, `queue`, `table`, `dfs`, `web` (Storage); `sqlServer` (Azure SQL); `vault` (Key Vault); `sites` (App Service); `registry` (ACR). **Un private endpoint por subrecurso**.
- **Aprobación**: la conexión puede ser automática (si tienes permisos sobre el recurso) o requerir **aprobación manual** del propietario (conexión entre suscripciones/tenants).
- **DNS es la parte crítica** ⚠️: el FQDN público (`st001.blob.core.windows.net`) debe resolver a la **IP privada**. Azure lo consigue con:
  - Una **zona DNS privada** con el nombre `privatelink.<servicio>` (por ejemplo `privatelink.blob.core.windows.net`), **vinculada a la VNet**.
  - Un **registro A** con el nombre del recurso apuntando a la IP privada (lo crea el portal si eliges "Integrar con la zona DNS privada").
  - El FQDN público pasa a ser un **CNAME** hacia `...privatelink...`, que resuelve a la IP privada.
  - Desde **on-premises** hace falta un **reenviador DNS** hacia el DNS de Azure (168.63.129.16) o **Azure DNS Private Resolver** ([[12 - Azure Private DNS y resolución de nombres]]).
- **NSG y UDR sobre private endpoints**: soportados si se habilitan las **directivas de red** de la subred (`privateEndpointNetworkPolicies`). Históricamente estaban deshabilitadas por defecto.
- El private endpoint es **unidireccional** (entrada al servicio). Para la salida del servicio hacia tu VNet existe la **integración de VNet** (App Service) o los **private link services**.
- **Coste**: se paga por endpoint/hora y por datos procesados.
- Funciona con **VNets emparejadas** y **on-premises** (con DNS correcto).
- Al crear un private endpoint conviene **deshabilitar el acceso público** del recurso para que no queden puertas abiertas.
- **Private Link Service** ➕: publicar tu propio servicio (detrás de un Standard Load Balancer) para que otros lo consuman con private endpoints.

## Cómo funciona

```
VM 10.0.1.4 ──► DNS: st001.blob.core.windows.net
                  └► CNAME st001.privatelink.blob.core.windows.net
                       └► zona privada vinculada a la VNet → A 10.0.2.5 (IP del private endpoint)
                            └► tráfico privado ──► Storage (acceso público deshabilitado)
```

```bash
# Crear el private endpoint para blob
az network private-endpoint create -g rg-net -n pe-st001-blob \
  --vnet-name vnet-hub --subnet snet-pe \
  --private-connection-resource-id $(az storage account show -g rg-data -n st001 --query id -o tsv) \
  --group-id blob --connection-name conn-st001
# Zona DNS privada y vinculación
az network private-dns zone create -g rg-net -n privatelink.blob.core.windows.net
az network private-dns link vnet create -g rg-net -z privatelink.blob.core.windows.net -n link-hub --virtual-network vnet-hub --registration-enabled false
# Grupo de zonas DNS del endpoint (crea el registro A automáticamente)
az network private-endpoint dns-zone-group create -g rg-net --endpoint-name pe-st001-blob -n default \
  --private-dns-zone privatelink.blob.core.windows.net --zone-name blob
# Deshabilitar acceso público del recurso
az storage account update -g rg-data -n st001 --public-network-access Disabled
# Comprobar
az network private-endpoint show -g rg-net -n pe-st001-blob --query "customDnsConfigs"
nslookup st001.blob.core.windows.net    # desde una VM de la VNet → IP privada
```

Portal: recurso → **Redes → Conexiones de punto de conexión privado → + Punto de conexión privado**, o buscar "Private Link Center".

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Acceso privado desde on-premises por VPN | **Private endpoint** + reenviador DNS o **DNS Private Resolver** |
| El nombre sigue resolviendo a la IP pública desde la VM | Falta la **zona DNS privada** vinculada a la VNet o el registro A |
| Necesito blob y file de la misma cuenta | **Dos private endpoints** (un subrecurso cada uno) |
| El recurso está en otra suscripción | Conexión con **aprobación manual** |
| Quiero NSG sobre el private endpoint | Habilitar las **directivas de red** de la subred |
| Que el recurso no sea accesible desde Internet | **Public network access = Disabled** |
| Publicar mi propio servicio para clientes | **Private Link Service** detrás de un Standard LB |
| Alternativa gratuita solo para VMs de Azure | **Service endpoint** |

## Ejemplo

Contoso debe eliminar toda exposición pública de su Azure SQL. Crea un **private endpoint** (`sqlServer`) en `snet-pe`, la zona `privatelink.database.windows.net` vinculada al hub y a los spokes, y deshabilita el acceso público del servidor. En on-premises configura el DNS corporativo para reenviar `database.windows.net` hacia una máquina de Azure con reenviador DNS (o despliega un **Private Resolver**), de modo que las aplicaciones locales resuelvan la IP privada.

## Comparaciones

| Opción | IP privada | On-premises | Granularidad | Coste | Cuándo |
|---|---|---|---|---|---|
| **Private endpoint** | **Sí** | **Sí** | Por recurso/subrecurso | De pago | Máxima seguridad, híbrido |
| **Service endpoint** | No | No | Por subred/servicio | Gratis | Rápido, solo desde Azure |
| **Firewall de IP del recurso** | No | Sí (IP pública) | Por IP | Gratis | Orígenes con IP fija |
| **Private Link Service** | Sí (para clientes) | Sí | Tu servicio | De pago | Publicar servicios propios |

## 💻 Laboratorio: private endpoint para Storage

1. Crear la subred `snet-pe` en `vnet-lab`.
2. Crear un private endpoint para el subrecurso **blob** de tu cuenta, integrando la zona DNS privada.
3. Desde una VM de la VNet: `nslookup <cuenta>.blob.core.windows.net` → debe devolver una IP privada.
4. Deshabilitar el acceso público de la cuenta y comprobar que desde Cloud Shell falla y desde la VM funciona.
5. Ver la zona `privatelink.blob.core.windows.net` y su registro A.

## AZ-104 Exam Tips

- 🔥 🧠 **IP privada en tu VNet**, un endpoint **por subrecurso** (blob, file, sqlServer, vault, sites…).
- 🔥 🧠 **El DNS es imprescindible**: zona `privatelink.*` vinculada a la VNet; desde on-premises, **reenviador DNS** o **Private Resolver**.
- 🧠 Funciona **desde on-premises**; el service endpoint no.
- 🧠 Tras crearlo, **deshabilitar el acceso público** del recurso.
- 🧠 Conexiones entre suscripciones requieren **aprobación**.
- 💻 `az network private-endpoint create`, zona DNS privada, `dns-zone-group create`.
- 📌 Private endpoint = **entrada privada al servicio**; VNet integration de App Service = **salida**.

## Errores comunes

- Crear el endpoint y olvidar el DNS (la app sigue yendo a la IP pública).
- Crear un solo endpoint y esperar que cubra blob y file.
- Dejar el acceso público habilitado y pensar que el recurso ya es privado.

## Preguntas que podrían aparecer

**1.** Tras crear un private endpoint para una cuenta de almacenamiento, las VMs siguen conectándose por la IP pública. ¿Qué falta?
- A) Un NSG · B) Vincular la zona DNS privada `privatelink.blob.core.windows.net` a la red virtual y crear el registro A · C) Un service endpoint · D) Una UDR

<details><summary>Respuesta</summary>

**B.** Sin la resolución DNS privada, el FQDN sigue resolviendo a la dirección pública.
</details>

**2.** Necesitas acceso privado a los subrecursos **blob** y **file** de la misma cuenta de almacenamiento. ¿Cuántos private endpoints creas?
- A) Uno · B) Dos, uno por subrecurso · C) Uno por VNet · D) Ninguno, basta con el firewall

<details><summary>Respuesta</summary>

**B.** Cada subrecurso (group ID) requiere su propio private endpoint.
</details>

**3.** ¿Cuál es la principal ventaja del private endpoint frente al service endpoint?
- A) Es gratis · B) Asigna una IP privada al servicio y permite el acceso desde redes locales · C) Se configura en la subred · D) No requiere DNS

<details><summary>Respuesta</summary>

**B.** El private endpoint da IP privada y funciona desde on-premises.
</details>

## Relacionado

- [[09 - Service Endpoints]]
- [[12 - Azure Private DNS y resolución de nombres]]
- [[03 - Firewalls y redes virtuales de Azure Storage]]
- [[21 - App Service - redes]]
- [[Service Endpoints y Private Endpoints]] (AZ-900)
- [[00 - Índice - Redes virtuales]]
