---
tags: [az-104, azure, computo, vm, alta-disponibilidad, sla]
modulo: Cómputo
peso_examen: Muy alto
---

# Alta disponibilidad: Availability Sets y Availability Zones

## ¿Qué es?

Dos mecanismos para que una aplicación sobreviva a fallos de hardware o mantenimiento:

- **Conjunto de disponibilidad (availability set)**: distribuye las VMs entre **dominios de error (fault domains, FD)** y **dominios de actualización (update domains, UD)** dentro de un mismo centro de datos.
- **Zonas de disponibilidad (availability zones)**: ubica las VMs en **centros de datos físicamente separados** dentro de la misma región.

## ¿Para qué sirve?

- Evitar que un fallo de rack, de alimentación o un mantenimiento planificado tumbe toda la aplicación.
- Alcanzar el **SLA** contractual de Azure.

## Conceptos clave

### Availability set
- **Fault domain (FD)**: grupo de VMs que comparten alimentación y red física (rack). Máximo **3** FD 🧠 (2 en algunas regiones).
- **Update domain (UD)**: grupo que se reinicia junto durante el mantenimiento planificado. Hasta **20** UD 🧠 (por defecto 5).
- Azure distribuye las VMs del conjunto entre FD y UD de forma automática.
- Se define **al crear la VM**; no se puede añadir una VM existente a un conjunto ni cambiarla de conjunto 🧠 (hay que recrearla conservando los discos).
- Todas las VMs de un conjunto están en la **misma región** y no pueden estar en zonas distintas.
- Es **gratuito** (solo pagas las VMs).
- Un conjunto solo puede contener VMs; para discos: los discos administrados también se alinean con los FD.

### Availability zones
- Cada región habilitada tiene **al menos 3 zonas**; cada zona es uno o varios centros de datos con alimentación, refrigeración y red independientes 🧠.
- Se elige la zona **al crear** la VM (zona 1, 2 o 3). Los números de zona son **lógicos y distintos por suscripción**.
- Recursos **zonales** (VM, disco, IP pública estándar) vs **zona-redundantes** (Standard Load Balancer, ZRS storage, Application Gateway v2).
- Para HA real: al menos **2 VMs en 2 zonas distintas** detrás de un **Standard Load Balancer**.
- Los **discos** deben estar en la misma zona que la VM (o ser ZRS).
- Mover una VM entre zonas: no es directo; se recrea desde snapshot o con Resource Mover.

### SLA 🧠

| Configuración | SLA de conectividad de VM |
|---|---|
| VM única con **todos los discos Premium SSD / Ultra** | **99,9 %** |
| VM única con Standard SSD | 99,5 % |
| VM única con Standard HDD | 95 % |
| **2+ VMs en un availability set** | **99,95 %** |
| **2+ VMs en 2+ availability zones** | **99,99 %** |

### Otros conceptos
- **Mantenimiento planificado**: Azure reinicia un UD cada vez, con ~30 minutos de recuperación entre UDs.
- **Eventos programados (scheduled events)** ➕: metadatos que avisan a la VM de un mantenimiento inminente.
- **Proximity placement group (PPG)**: agrupa recursos físicamente cerca para baja latencia (puede combinarse con availability set; con zonas limita la HA).
- **Dedicated host**: hardware físico dedicado, con sus propios dominios de error.

## Cómo funciona

```
Availability set "as-web" (3 FD, 5 UD)
  FD0/UD0: vm1      FD1/UD1: vm2      FD2/UD2: vm3
  Fallo de rack FD1 → vm2 cae, vm1 y vm3 siguen

Availability zones (región West Europe)
  Zona 1: vm1        Zona 2: vm2        Zona 3: vm3
  Fallo del centro de datos de la zona 2 → vm2 cae, vm1 y vm3 siguen
```

```bash
# Availability set
az vm availability-set create --resource-group rg-web --name as-web --platform-fault-domain-count 3 --platform-update-domain-count 5
az vm create --resource-group rg-web --name vm1 --availability-set as-web --image Win2022Datacenter --size Standard_D2s_v5 --admin-username azureadmin --admin-password '<pwd>'
# Zonas
az vm create --resource-group rg-web --name vm-z1 --zone 1 --image Ubuntu2204 --size Standard_D2s_v5 --generate-ssh-keys
az vm list --query "[].{name:name, zone:zones[0]}" -o table
# Comprobar zonas disponibles en la región
az vm list-skus --location westeurope --size Standard_D2s_v5 --query "[].locationInfo[].zones" -o json
```

```powershell
New-AzAvailabilitySet -ResourceGroupName rg-web -Name as-web -Location westeurope -PlatformFaultDomainCount 3 -PlatformUpdateDomainCount 5 -Sku Aligned
New-AzVM -ResourceGroupName rg-web -Name vm1 -AvailabilitySetName as-web -Image Win2022Datacenter -Credential (Get-Credential)
New-AzVM -ResourceGroupName rg-web -Name vm-z1 -Zone 1 -Image Ubuntu2204 -Credential (Get-Credential)
```

> [!info] Aligned vs Classic
> Los availability sets **Aligned** (managed) alinean los dominios de error con los discos administrados. Es el valor por defecto actual; "Classic" corresponde a discos no administrados (legado).

## Configuración relevante para el examen

| Requisito | Solución |
|---|---|
| SLA 99,99 % | 2+ VMs en 2+ **zonas** + Standard Load Balancer |
| SLA 99,95 % | 2+ VMs en un **availability set** |
| SLA 99,9 % con una sola VM | Todos los discos **Premium SSD** (o Ultra) |
| Región sin zonas de disponibilidad | **Availability set** |
| Añadir una VM existente a un conjunto | **No se puede**: recrear la VM (conservando discos) |
| Proteger frente a la caída de toda la región | **Azure Site Recovery** / despliegue multi-región ([[11 - Azure Site Recovery]]) |
| Baja latencia entre VMs de una app de 3 capas | **Proximity placement group** |
| Escalado automático + HA | **VMSS** con zonas ([[10 - Virtual Machine Scale Sets]]) |

## Ejemplo

Una aplicación web de tres capas necesita 99,99 % de disponibilidad. Se despliegan 2 VMs web en zonas 1 y 2 detrás de un **Standard Load Balancer** zona-redundante, 2 VMs de aplicación en zonas 1 y 2 con un Internal Load Balancer, y la base de datos en Azure SQL con configuración zona-redundante. Los discos de cada VM viven en su zona.

## Comparaciones

| Mecanismo | Protege frente a | SLA | Coste | Cuándo utilizarlo |
|---|---|---|---|---|
| **VM única con Premium SSD** | Fallo de disco | 99,9 % | Bajo | Entornos no críticos |
| **Availability set** | Fallo de rack y mantenimiento | 99,95 % | Gratis | Regiones sin zonas, apps legacy |
| **Availability zones** | Fallo de un centro de datos | 99,99 % | Gratis (paga tráfico entre zonas) | Producción crítica |
| **VMSS + zonas** | Igual + escalado | 99,99 % | Gratis | Cargas escalables |
| **Multi-región (ASR / Front Door)** | Caída de región | Según diseño | Alto | DR y continuidad |

## 💻 Laboratorio: HA con zonas

1. Crear `vm-z1` en zona 1 y `vm-z2` en zona 2 (Ubuntu, sin IP pública), instalando nginx con cloud-init.
2. Crear un **Standard Load Balancer** público zona-redundante con backend pool de ambas VMs, sonda HTTP y regla 80.
3. Probar el acceso por la IP del balanceador y apagar `vm-z1`: el tráfico debe seguir.
4. Crear un availability set y comprobar en el portal los FD/UD asignados a las VMs.
5. Intentar añadir una VM existente al availability set y confirmar que no es posible.

## AZ-104 Exam Tips

- ⭐ **FD = 3 máximo, UD = 20 máximo** (por defecto 5).
- 🔥 🧠 SLA: **99,9 % VM única Premium / 99,95 % availability set / 99,99 % zonas**.
- 🔥 🧠 El availability set y la zona se eligen **al crear la VM**; no se cambian después.
- 🧠 Una VM **no puede** estar a la vez en un availability set y en una zona.
- 🧠 Los discos deben estar en la **misma zona** que la VM.
- 💻 Crear availability set con FD/UD y crear VMs zonales.
- 📌 Availability set (dentro de un centro de datos) vs zonas (centros de datos separados) vs regiones (DR).
- ⚠️ Zonas protegen de fallo de zona, **no** de fallo de región: para eso ASR o despliegue multi-región.

## Errores comunes

- Creer que se puede mover una VM a un availability set después de crearla.
- Poner las dos VMs en la misma zona y pensar que hay HA.
- Confundir fault domain (hardware) con update domain (mantenimiento).

## Preguntas que podrían aparecer

**1.** ¿Cuál es el número máximo de dominios de error que puede tener un conjunto de disponibilidad?
- A) 2 · B) 3 · C) 5 · D) 20

<details><summary>Respuesta</summary>

**B.** Hasta 3 dominios de error (y hasta 20 dominios de actualización).
</details>

**2.** Una aplicación necesita un SLA del 99,99 % para sus máquinas virtuales. ¿Qué configuración cumple el requisito?
- A) Dos VMs en un availability set · B) Una VM con discos Ultra · C) Dos o más VMs distribuidas en al menos dos zonas de disponibilidad · D) Tres VMs en la misma zona

<details><summary>Respuesta</summary>

**C.** Solo la distribución en varias zonas alcanza el 99,99 %. El availability set llega al 99,95 %.
</details>

**3.** Tienes una VM en producción creada sin conjunto de disponibilidad y necesitas incluirla en uno. ¿Qué debes hacer?
- A) Editarlo desde la hoja Disponibilidad · B) Desasignar la VM y asignarla al conjunto · C) Eliminar la VM y recrearla asociándola al conjunto, reutilizando sus discos · D) Crear un peering

<details><summary>Respuesta</summary>

**C.** La pertenencia a un availability set solo se define en la creación; se recrea la VM conservando los discos administrados.
</details>

## Relacionado

- [[04 - Máquinas virtuales - creación y configuración]]
- [[10 - Virtual Machine Scale Sets]]
- [[13 - Azure Load Balancer]]
- [[11 - Azure Site Recovery]]
- [[00 - Índice - Cómputo]]
