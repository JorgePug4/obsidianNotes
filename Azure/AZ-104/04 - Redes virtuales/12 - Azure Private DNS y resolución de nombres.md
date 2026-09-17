---
tags: [az-104, azure, redes, dns, private-dns, resolucion-nombres]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Azure Private DNS y resolución de nombres

## ¿Qué es?

**Azure Private DNS** hospeda **zonas DNS privadas** que solo resuelven desde las **redes virtuales vinculadas**, sin exponer nada a Internet. Es la pieza que hace funcionar los **private endpoints** y los nombres internos personalizados.

Además, toda VNet dispone del **DNS proporcionado por Azure** (`168.63.129.16`) que resuelve los nombres de las VMs de la misma VNet automáticamente.

## Conceptos clave

### Opciones de resolución de nombres en una VNet 🧠

| Opción | Qué resuelve | Limitaciones |
|---|---|---|
| **DNS de Azure (por defecto)** | Nombres de VMs **dentro de la misma VNet** y nombres públicos | **No resuelve entre VNets** ni nombres on-premises |
| **Zona DNS privada** | Nombres personalizados en una o varias VNets vinculadas | Requiere vincular cada VNet |
| **Servidor DNS propio** (custom DNS en la VNet) | Lo que configures (AD DS, BIND) | Hay que mantenerlo; se configura a nivel de VNet o NIC |
| **Azure DNS Private Resolver** ➕ | Puente entre on-premises y zonas privadas de Azure | Servicio gestionado, sustituye a las VMs reenviadoras |

### Zonas privadas
- El **nombre de la zona** puede ser cualquiera (`contoso.internal`) o uno de `privatelink.*` para private endpoints.
- **Vínculo de red virtual (virtual network link)**: obliga a asociar cada VNet que debe resolver la zona. Se pueden vincular **VNets de otras suscripciones/regiones**.
- **Autorregistro (auto-registration)** 🧠: si se habilita en el vínculo, Azure crea y mantiene un **registro A** por cada VM de esa VNet (y lo borra al eliminarla). ⚠️ Una VNet solo puede tener **un vínculo con autorregistro habilitado**; puede tener muchos vínculos de resolución.
- Límites: 1000 vínculos por zona; 100 vínculos con autorregistro por zona (valores orientativos).
- Las zonas privadas **no necesitan delegación** y tienen prioridad sobre la resolución pública para esos nombres.
- El **peering no comparte DNS**: hay que vincular la zona a cada VNet.

### Private Resolver ➕
- **Endpoint de entrada (inbound)**: IP privada en una subred delegada a la que on-premises envía consultas → resuelve zonas privadas de Azure.
- **Endpoint de salida (outbound)** + **conjunto de reglas de reenvío**: Azure reenvía consultas de dominios concretos a los DNS on-premises.
- Sustituye a las VM reenviadoras tradicionales.

## Cómo funciona

```
Zona privada "privatelink.blob.core.windows.net"
   ├── vínculo a vnet-hub    (autorregistro: no)
   ├── vínculo a vnet-spoke1 (autorregistro: no)
   └── registro A st001 → 10.0.2.5

Zona privada "contoso.internal"
   └── vínculo a vnet-hub con AUTORREGISTRO → A de cada VM creado automáticamente
```

```bash
# Zona privada + vínculo con autorregistro
az network private-dns zone create -g rg-net -n contoso.internal
az network private-dns link vnet create -g rg-net -z contoso.internal -n link-hub --virtual-network vnet-hub --registration-enabled true
az network private-dns link vnet create -g rg-net -z contoso.internal -n link-spoke1 --virtual-network vnet-spoke1 --registration-enabled false
# Registro manual
az network private-dns record-set a add-record -g rg-net -z contoso.internal -n app -a 10.0.1.10
az network private-dns record-set list -g rg-net -z contoso.internal -o table
# DNS personalizado en la VNet
az network vnet update -g rg-net -n vnet-hub --dns-servers 10.0.1.4 10.0.1.5
# Private Resolver
az dns-resolver create -g rg-net -n resolver-hub --location westeurope --id-virtual-network $(az network vnet show -g rg-net -n vnet-hub --query id -o tsv)
az dns-resolver inbound-endpoint create -g rg-net --dns-resolver-name resolver-hub -n inbound --ip-configurations '[{"private-ip-allocation-method":"Dynamic","id":"<subnetId>"}]' --location westeurope
```

```powershell
New-AzPrivateDnsZone -ResourceGroupName rg-net -Name contoso.internal
New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName rg-net -ZoneName contoso.internal -Name link-hub -VirtualNetworkId $vnet.Id -EnableRegistration
New-AzPrivateDnsRecordSet -ResourceGroupName rg-net -ZoneName contoso.internal -Name app -RecordType A -Ttl 3600 `
  -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address 10.0.1.10)
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Las VMs de VNet1 no resuelven los nombres de VNet2 (emparejadas) | **Zona privada** vinculada a ambas (el DNS de Azure no cruza VNets) |
| Registro automático de cada VM nueva | Vínculo con **autorregistro** (uno por VNet) |
| Private endpoint resuelve a IP pública | Falta la zona `privatelink.*` vinculada a la VNet |
| On-premises debe resolver nombres privados de Azure | **Private Resolver (inbound)** o VM reenviadora + regla condicional en el DNS local |
| Azure debe resolver nombres de AD DS local | **Private Resolver (outbound)** con regla de reenvío, o DNS personalizado en la VNet apuntando al DC |
| Cambiar el DNS de la VNet a un servidor propio | VNet → **Servidores DNS** → Personalizado; **reiniciar las VMs** para que tomen el cambio |
| Nombres internos legibles para las apps | Zona privada `contoso.internal` con registros A |

## Ejemplo

Contoso tiene hub y dos spokes. Crea la zona `contoso.internal` y la vincula a las tres VNets, con **autorregistro solo en el hub**. Para los private endpoints crea `privatelink.blob.core.windows.net` y `privatelink.database.windows.net`, vinculadas a las tres VNets (sin autorregistro). Para que las aplicaciones on-premises resuelvan esos nombres, despliega un **Private Resolver** con endpoint de entrada en el hub y configura el DNS corporativo para reenviar esos dominios a su IP.

## Comparaciones

| Mecanismo | Resuelve | Mantenimiento | Cuándo utilizarlo |
|---|---|---|---|
| **DNS de Azure (por defecto)** | Misma VNet + Internet | Ninguno | Escenarios simples |
| **Zona DNS privada** | Nombres propios en varias VNets | Bajo | Estándar recomendado |
| **DNS personalizado (VM/AD DS)** | Lo que configures | Alto | Integración con AD DS local |
| **Private Resolver** | Puente bidireccional Azure ↔ on-premises | Bajo (gestionado) | Híbrido sin VMs de DNS |
| **Archivo hosts** | Local | Manual | Nunca en producción |

## 💻 Laboratorio: DNS privado

1. Crear la zona `lab.internal` y vincularla a `vnet-lab` con **autorregistro**.
2. Crear dos VMs y comprobar que aparecen registros A automáticamente.
3. Desde una VM, hacer `nslookup vm2.lab.internal`.
4. Crear `vnet-lab2` emparejada, vincularla a la zona **sin** autorregistro y comprobar la resolución cruzada.
5. Intentar crear un segundo vínculo con autorregistro en la misma VNet y ver el error.

## AZ-104 Exam Tips

- 🔥 🧠 El **DNS de Azure no resuelve entre VNets**, aunque estén emparejadas → **zona privada vinculada**.
- 🔥 🧠 **Una VNet solo admite un vínculo con autorregistro**.
- 🧠 Las zonas `privatelink.*` son imprescindibles para los **private endpoints**.
- 🧠 DNS de Azure = **168.63.129.16**; si se usa DNS personalizado, hay que **reiniciar las VMs** para aplicarlo.
- 🧠 **Private Resolver**: inbound (on-premises → Azure), outbound (Azure → on-premises).
- 💻 `az network private-dns zone create`, `link vnet create --registration-enabled`, registros A.
- 📌 Zona pública (Internet, delegación NS) vs zona privada (VNets vinculadas, sin delegación).

## Errores comunes

- Vincular la zona solo al hub y esperar que los spokes resuelvan.
- Intentar dos vínculos con autorregistro en la misma VNet.
- Cambiar el DNS de la VNet y no reiniciar las VMs.

## Preguntas que podrían aparecer

**1.** Dos redes virtuales están emparejadas, pero las VMs de una no resuelven los nombres de host de la otra. ¿Qué implementas?
- A) Un servidor DNS en cada VNet · B) Una zona DNS privada vinculada a ambas redes virtuales · C) Un private endpoint · D) Registros CNAME en Azure DNS público

<details><summary>Respuesta</summary>

**B.** La resolución predeterminada de Azure no cruza VNets; la zona privada vinculada sí.
</details>

**2.** ¿Cuántos vínculos de red virtual con autorregistro puede tener una misma red virtual?
- A) Uno · B) Dos · C) Diez · D) Ilimitados

<details><summary>Respuesta</summary>

**A.** Solo un vínculo con autorregistro por VNet, aunque puede tener muchos vínculos de resolución.
</details>

**3.** Las aplicaciones on-premises deben resolver el nombre del private endpoint de una cuenta de almacenamiento. ¿Qué solución gestionada usas?
- A) Azure DNS público · B) Azure DNS Private Resolver con endpoint de entrada y reenvío condicional desde el DNS local · C) Un registro alias · D) Service endpoint

<details><summary>Respuesta</summary>

**B.** El resolver de entrada permite que el DNS corporativo consulte las zonas privadas de Azure.
</details>

## Relacionado

- [[10 - Private Endpoint y Private Link]]
- [[11 - Azure DNS (zonas públicas)]]
- [[03 - Peering de redes virtuales]]
- [[15 - Solución de problemas de conectividad de red]]
- [[00 - Índice - Redes virtuales]]
