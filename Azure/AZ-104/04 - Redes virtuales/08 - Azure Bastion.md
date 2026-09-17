---
tags: [az-104, azure, redes, bastion, rdp, ssh, seguridad]
modulo: Redes virtuales
peso_examen: Alto
---

# Azure Bastion

## ¿Qué es?

**Azure Bastion** es un servicio PaaS que ofrece conectividad **RDP y SSH** a las máquinas virtuales **directamente desde el portal de Azure por TLS (443)**, sin necesidad de **IP pública en las VMs**, sin abrir los puertos 3389/22 a Internet y sin jump box propio.

## ¿Para qué sirve?

- Administrar VMs de forma segura desde el navegador.
- Eliminar IPs públicas de las VMs (reducir superficie de ataque).
- Evitar VPN para tareas administrativas puntuales.

## Conceptos clave

- **Subred obligatoria** 🧠: debe llamarse **`AzureBastionSubnet`** y tener al menos **/26** (antes /27; /26 es el requisito actual recomendado y exigido para SKUs con escalado). **No** puede tener UDR que rompa su salida ni, en general, NSG restrictivo mal configurado.
- Se despliega **por red virtual**; las VMs de VNets **emparejadas** pueden usarlo (SKU Basic o superior).
- **IP pública Standard** asociada al recurso Bastion (salvo el SKU **Premium en modo privado**).
- **SKUs** 🧠:

| SKU | Características | Escalado | Uso |
|---|---|---|---|
| **Developer** | Gratuito/coste mínimo, **recurso compartido**, 1 VM a la vez, **sin peering**, sin subred dedicada | No | Dev/test |
| **Basic** | Despliegue dedicado, RDP/SSH desde el portal, peering | Fijo (2 instancias) | Uso estándar |
| **Standard** | Basic + **cliente nativo**, **vínculos compartibles**, **conexión por IP**, **puertos personalizados**, transferencia de archivos, **escalado hasta 50 instancias** | Sí | Producción |
| **Premium** | Standard + **grabación de sesión** y **despliegue solo privado** (sin IP pública) | Sí | Cumplimiento y auditoría |

- **Cliente nativo** (Standard+): `az network bastion ssh/rdp --name ... --target-resource-id ...` desde tu equipo.
- **Vínculo compartible (shareable link)** (Standard+): URL temporal para que alguien sin acceso al portal se conecte.
- **Conexión por IP** (Standard+): conectarse a una IP privada (incluye on-premises alcanzable por VPN/ER).
- **Instancias**: cada instancia admite un número limitado de sesiones concurrentes (aprox. 20 RDP / 40 SSH por instancia según tamaño de VM).
- **NSG en AzureBastionSubnet** (opcional, pero si se pone debe permitir) 🧠:
  - Entrada: **443 desde Internet** (o desde el origen permitido), **443 desde GatewayManager**, **8080/5701 desde VirtualNetwork** (plano de datos), **443 desde AzureLoadBalancer**.
  - Salida: **3389/22 hacia VirtualNetwork**, **443 hacia AzureCloud**, 8080/5701 hacia VirtualNetwork.
- El NSG de las **VMs destino** debe permitir 3389/22 **desde el rango de AzureBastionSubnet** (o desde `VirtualNetwork`).
- **Roles necesarios**: Reader en la VM, en la NIC y en el recurso Bastion.

## Cómo funciona

```
Navegador (HTTPS 443) ──► Azure Bastion (IP pública Standard, AzureBastionSubnet)
                                   │ RDP 3389 / SSH 22 por IP privada
                                   ▼
                            VM sin IP pública
```

```bash
az network vnet subnet create -g rg-net --vnet-name vnet-hub -n AzureBastionSubnet --address-prefixes 10.0.250.0/26
az network public-ip create -g rg-net -n pip-bastion --sku Standard --allocation-method Static
az network bastion create -g rg-net -n bastion-hub --public-ip-address pip-bastion --vnet-name vnet-hub --location westeurope --sku Standard
# Cliente nativo (SKU Standard+)
az network bastion ssh -g rg-net -n bastion-hub --target-resource-id $(az vm show -g rg-net -n vm-lnx01 --query id -o tsv) --auth-type ssh-key --username azureadmin --ssh-key ~/.ssh/id_rsa
az network bastion rdp -g rg-net -n bastion-hub --target-resource-id $(az vm show -g rg-net -n vm-web01 --query id -o tsv)
```

Portal: **Bastiones** → Crear (VNet, subred, IP pública, SKU) o desde la VM → **Conectar → Bastion**.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Administrar VMs sin IP pública ni abrir 3389 | **Azure Bastion** |
| Conectarse con el cliente RDP/SSH del equipo | SKU **Standard** (cliente nativo) |
| Grabar las sesiones administrativas | SKU **Premium** |
| Sin ninguna IP pública en el propio Bastion | SKU **Premium** (despliegue privado) |
| Bastion para VMs de otra VNet | **Peering** + SKU Basic o superior |
| Compartir acceso puntual con un proveedor | **Shareable link** (Standard) |
| Muchas sesiones simultáneas | Aumentar **instancias** (Standard+) |
| El despliegue falla por la subred | Nombre **AzureBastionSubnet** y tamaño **/26** |
| Alternativas más baratas | **JIT VM Access** (Defender for Cloud), VPN P2S, jump box propio |

## Ejemplo

Contoso elimina las IPs públicas de sus 40 VMs. Despliega **Bastion Standard** en el hub con `AzureBastionSubnet` 10.0.250.0/26 y 4 instancias. Los administradores se conectan desde el portal o con el cliente nativo. Los NSGs de las VMs permiten 3389/22 solo desde el rango de la subred de Bastion. Para auditoría de proveedores externos, se plantea subir a **Premium** por la grabación de sesiones.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Azure Bastion** | RDP/SSH desde el navegador | Sin IP pública ni puertos abiertos, sin agente | Administración estándar |
| **IP pública + NSG** | Acceso directo | Simple, gratis | Solo laboratorio |
| **JIT VM Access** (Defender) | Abrir el puerto temporalmente | Ventana limitada, auditada | Complemento con IP pública |
| **VPN P2S / S2S** | Túnel a la VNet | Acceso a todo, no solo RDP | Administración amplia desde casa/oficina |
| **Jump box propio** | VM de salto | Control total | Requisitos muy específicos; mantenimiento propio |

## 💻 Laboratorio: Bastion

1. Crear `AzureBastionSubnet` (/26) en `vnet-lab` y desplegar Bastion (Basic o Developer).
2. Crear una VM **sin IP pública** y conectarse desde el portal con Bastion.
3. Revisar el NSG de la VM: permitir 3389/22 solo desde el rango de AzureBastionSubnet.
4. Con SKU Standard, probar `az network bastion rdp` desde el cliente nativo y un shareable link.
5. Comprobar que el puerto 3389 no está expuesto a Internet con un escaneo desde fuera.

## AZ-104 Exam Tips

- 🔥 🧠 Subred **AzureBastionSubnet**, mínimo **/26**, nombre exacto.
- 🔥 🧠 Conexión por **HTTPS 443** desde el portal; las VMs **no necesitan IP pública**.
- 🧠 **Cliente nativo, shareable links, conexión por IP y puertos personalizados = Standard**; **grabación de sesión y despliegue privado = Premium**; **Developer** es compartido, 1 VM, sin peering.
- 🧠 Bastion sirve a VMs de VNets **emparejadas**.
- 💻 Crear Bastion, conectar desde el portal, `az network bastion ssh/rdp`.
- ⚠️ Si pones NSG en AzureBastionSubnet debe permitir 443 entrante (Internet y GatewayManager) y 3389/22 saliente a VirtualNetwork.

## Errores comunes

- Nombrar la subred `BastionSubnet` o crearla /27 cuando se requiere /26.
- Bloquear el tráfico de gestión con un NSG demasiado estricto en la subred de Bastion.
- Elegir Developer y luego necesitar peering o varias sesiones.

## Preguntas que podrían aparecer

**1.** ¿Qué requisito debe cumplir la subred donde se despliega Azure Bastion?
- A) Llamarse BastionSubnet · B) Llamarse AzureBastionSubnet y tener al menos /26 · C) Estar delegada a Microsoft.Network/bastionHosts · D) No tener NSG

<details><summary>Respuesta</summary>

**B.** El nombre es obligatorio y el prefijo mínimo actual es /26.
</details>

**2.** Necesitas grabar las sesiones RDP de los administradores por requisitos de auditoría. ¿Qué SKU de Bastion eliges?
- A) Developer · B) Basic · C) Standard · D) Premium

<details><summary>Respuesta</summary>

**D.** La grabación de sesiones es una característica del SKU Premium.
</details>

**3.** Los usuarios quieren conectarse a las VMs con su cliente SSH local a través de Bastion. ¿Qué SKU es el mínimo?
- A) Developer · B) Basic · C) Standard · D) Premium

<details><summary>Respuesta</summary>

**C.** El soporte de cliente nativo aparece en Standard.
</details>

## Relacionado

- [[01 - Azure Virtual Network y subredes]]
- [[05 - Network Security Group (NSG)]]
- [[04 - Máquinas virtuales - creación y configuración]]
- [[16 - Conectividad híbrida - VPN Gateway y ExpressRoute]]
- [[00 - Índice - Redes virtuales]]
