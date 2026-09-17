---
tags: [az-104, azure, computo, vm, maquinas-virtuales]
modulo: Cómputo
peso_examen: Muy alto
---

# Máquinas virtuales · creación y configuración

## ¿Qué es?

Una **máquina virtual (VM)** de Azure es cómputo IaaS: tú eliges SO, tamaño, discos y red, y Azure gestiona el hardware. Crear una VM implica crear (o reutilizar) varios recursos: **disco de SO**, **NIC**, **IP pública** (opcional), **NSG** (opcional), **VNet/subred**, y opcionalmente discos de datos, extensiones, availability set/zone.

## ¿Para qué sirve?

- Migraciones "lift and shift", aplicaciones que necesitan control del SO, servidores de dominio, bases de datos autogestionadas, jump boxes.

## Conceptos clave

### Pestañas del asistente de creación 🧠

| Pestaña | Decisiones clave |
|---|---|
| **Básico** | Suscripción, RG, nombre, **región**, **opciones de disponibilidad** (ninguna / availability zone / availability set / VMSS), **tipo de seguridad** (Standard / **Trusted launch** por defecto / Confidential), **imagen** (Marketplace, imagen compartida, Compute Gallery, imagen propia), arquitectura (x64/Arm64), **tamaño**, **cuenta de administrador** (usuario+contraseña o clave SSH), **puertos de entrada públicos**, licencia (**Azure Hybrid Benefit**) |
| **Discos** | Tipo de disco de SO (Standard HDD/SSD, Premium SSD, Premium SSD v2, Ultra), cifrado (SSE PMK / CMK / **encryption at host**), discos de datos, "eliminar con la VM" |
| **Redes** | VNet, subred, IP pública (SKU Standard), NSG de NIC (ninguno/básico/avanzado), **eliminar IP y NIC con la VM**, balanceador de carga |
| **Administración** | Defender, identidad administrada, **inicio de sesión con Entra ID**, **apagado automático**, **copia de seguridad** (Recovery Services vault y política), actualizaciones (Update Manager), **diagnóstico de arranque** (boot diagnostics) |
| **Supervisión** | Alertas, diagnósticos de SO invitado, Azure Monitor Agent |
| **Opciones avanzadas** | **Extensiones**, aplicaciones de VM, **datos personalizados / cloud-init**, **user data**, host dedicado, grupo de selección de ubicación (proximity placement group), **generación de VM (Gen1/Gen2)** |
| **Etiquetas** | Tags |

### Otros conceptos
- **Nombre de la VM**: Windows máx. **15** caracteres, Linux **64** 🧠. Nombre del recurso de Azure puede ser distinto del nombre del equipo.
- **Usuario administrador**: nombres reservados (`admin`, `administrator`, `root`…); contraseña 12-123 caracteres con complejidad.
- **Disco temporal** (`D:` en Windows, `/dev/sdb` o `/mnt` en Linux): **se pierde** al desasignar o redimensionar; no guardar datos. Algunas series (v5 con "d" en el nombre lo tienen; sin "d" no).
- **Estados de energía** 🧠: Running, **Stopped** (apagado desde el SO: **se sigue facturando el cómputo**), **Stopped (deallocated)** (desde el portal/CLI: **no se factura cómputo**, sí los discos; libera la IP pública dinámica y la CPU).
- **IP pública**: SKU **Standard** (estática, zona-redundante) es la actual; Basic retirada. Si se quiere conservar la IP tras desasignar → estática.
- **Puertos de entrada**: abrir RDP 3389/SSH 22 a Internet solo en laboratorio; en producción → **Azure Bastion**, JIT (Defender) o VPN.
- **Trusted launch**: Secure Boot + vTPM; requiere Gen2; algunas características (por ejemplo, backup con política Standard, ADE… ) tienen requisitos específicos.
- **Boot diagnostics**: captura de pantalla y log serie para diagnosticar arranques fallidos; usa cuenta administrada por Azure o propia.
- **Serial console**: acceso a la consola serie desde el portal (requiere boot diagnostics).
- **Redeploy**: mueve la VM a otro host de Azure (soluciona problemas de conectividad); **Reapply**: reaplica el estado.
- **Reset password / Run Command**: desde el portal para recuperar acceso (extensión VMAccess).
- **Azure Hybrid Benefit**: usar licencias Windows Server/SQL propias con Software Assurance para no pagar la licencia en Azure.
- **Spot VM**: hasta 90 % más barata, puede ser desalojada; para cargas interrumpibles.
- **Reserved instances**: 1/3 años.
- **Cuotas**: vCPUs por región y por familia; solicitar aumento en Suscripción → Uso + cuotas.

## Cómo funciona

```bash
az vm create --resource-group rg-web --name vm-web01 --image Win2022Datacenter --size Standard_D2s_v5 \
  --admin-username azureadmin --admin-password 'P@ssw0rd!2026abc' --vnet-name vnet-web --subnet web \
  --public-ip-sku Standard --nsg-rule RDP --zone 1 --license-type Windows_Server
az vm create --resource-group rg-web --name vm-lnx01 --image Ubuntu2204 --size Standard_B2s --generate-ssh-keys --public-ip-address ""
az vm list -d -o table                       # -d muestra estado y IP
az vm deallocate --resource-group rg-web --name vm-web01
az vm start --resource-group rg-web --name vm-web01
az vm redeploy --resource-group rg-web --name vm-web01
az vm boot-diagnostics enable --resource-group rg-web --name vm-web01
az vm run-command invoke --resource-group rg-web --name vm-web01 --command-id RunPowerShellScript --scripts "Get-Service"
az vm open-port --resource-group rg-web --name vm-web01 --port 80 --priority 900
```

```powershell
New-AzVM -ResourceGroupName rg-web -Name vm-web01 -Location westeurope -Image Win2022Datacenter -Size Standard_D2s_v5 `
  -VirtualNetworkName vnet-web -SubnetName web -PublicIpAddressName pip-web01 -OpenPorts 3389 -Credential (Get-Credential) -Zone 1
Get-AzVM -Status | Select-Object Name, PowerState
Stop-AzVM -ResourceGroupName rg-web -Name vm-web01 -Force        # desasigna
Stop-AzVM -ResourceGroupName rg-web -Name vm-web01 -StayProvisioned   # apaga sin desasignar (se factura)
Start-AzVM -ResourceGroupName rg-web -Name vm-web01
Set-AzVMBootDiagnostic -VM $vm -Enable
Invoke-AzVMRunCommand -ResourceGroupName rg-web -VMName vm-web01 -CommandId RunPowerShellScript -ScriptPath .\script.ps1
```

## Componentes de una VM

| Recurso | Obligatorio | Nota |
|---|---|---|
| VM (Microsoft.Compute/virtualMachines) | Sí | Tamaño, imagen, SO |
| Disco de SO (managed disk) | Sí | Tipo y cifrado; ver [[06 - Discos administrados]] |
| NIC | Sí (al menos 1) | IP privada dinámica/estática, NSG, ASG, IP forwarding, DNS |
| VNet/subred | Sí | Deben existir o crearse |
| IP pública | No | Standard, estática; DNS label opcional |
| NSG | No | En NIC y/o subred |
| Discos de datos | No | Hasta N según tamaño |
| Extensiones | No | Custom Script, DSC, AMA, antimalware… |
| Availability set / zone | No | Se decide **al crear**; no se cambia después |
| Identidad administrada | No | System/user-assigned |

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| Dejar de pagar cómputo sin borrar la VM | **Deallocate** (Stop desde el portal/CLI) |
| Conservar la IP pública al desasignar | IP pública **estática** (Standard siempre lo es) |
| Conectarse sin exponer RDP/SSH a Internet | **Azure Bastion** |
| Recuperar acceso a una VM con contraseña olvidada | **Restablecer contraseña** (extensión VMAccess) en el portal |
| La VM no arranca | **Boot diagnostics** (captura) + **Serial console**; **Redeploy** si es problema de host |
| Ejecutar un script post-creación | **Custom Script Extension** o `cloud-init` / datos personalizados |
| Reducir coste con licencias propias | **Azure Hybrid Benefit** |
| Windows: nombre de equipo | ≤ 15 caracteres |
| Necesito disco temporal grande / no quiero disco temporal | Serie con "d" (por ejemplo D2ds_v5) / sin "d" |
| Error "Operation could not be completed as it results in exceeding approved quota" | Solicitar aumento de cuota de vCPU en la región |
| Iniciar sesión en Windows/Linux con credenciales de Entra ID | Extensión **Microsoft Entra login** + rol *Virtual Machine Administrator/User Login* |

## Ejemplo

Contoso necesita una VM Windows Server 2022 para una app interna: sin IP pública, acceso por Bastion, disco de SO Premium SSD, disco de datos de 256 GB, en la zona 2 de West Europe, apagado automático a las 19:00, backup diario y licencias con Hybrid Benefit. Todo se configura desde el asistente (Básico: zona 2, Hybrid Benefit; Discos: Premium + disco de datos; Redes: sin IP pública; Administración: apagado automático y backup).

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **VM individual** | Control total | Flexibilidad | Cargas tradicionales |
| **VM en availability set** | HA en un centro de datos | SLA 99,95 % | Cuando la región no tiene zonas o la app no las soporta |
| **VM en availability zone** | HA entre zonas | SLA 99,99 % | Producción en regiones con zonas |
| **VMSS** | Muchas VMs iguales, autoscale | Escalado automático | Web/worker escalables |
| **Spot VM** | Muy barato | Hasta 90 % descuento | Batch, test |
| **Dedicated host** | Hardware dedicado | Aislamiento, cumplimiento | Regulación, licencias por host |
| **App Service / Container Apps** | PaaS | Sin gestionar SO | Cuando no hace falta el SO |

## 💻 Laboratorio: crear y administrar una VM

1. Crear `vm-lab01` (Windows Server 2022, B2s, zona 1, sin IP pública, NSG básico) con disco de datos de 32 GB.
2. Desplegar Azure Bastion (Developer o Basic) en la VNet y conectarse.
3. Inicializar el disco de datos en el SO. Reiniciar la VM y confirmar que el disco temporal `D:` sigue vacío.
4. Desasignar la VM (`az vm deallocate`) y comprobar en Cost analysis que solo se factura el disco.
5. Habilitar boot diagnostics y ver la captura de pantalla.
6. Ejecutar `az vm run-command invoke` para obtener `hostname`.

## AZ-104 Exam Tips

- 🔥 🧠 **Stopped** (desde el SO) sigue facturando; **Stopped (deallocated)** no factura cómputo.
- 🔥 🧠 El **disco temporal** se pierde al desasignar/redimensionar.
- 🧠 Nombre Windows ≤ **15** caracteres; Linux ≤ 64.
- 🧠 Availability set/zone se elige **al crear**.
- 🧠 Boot diagnostics + Serial console para arranques fallidos; **Redeploy** para problemas de host.
- 💻 `az vm create`, `New-AzVM`, deallocate/start, run-command, open-port, reset password.
- 📌 IP pública **Standard** = estática, requiere NSG (cerrada por defecto).
- ⚠️ Abrir 3389/22 a Internet es la respuesta incorrecta si el enunciado habla de seguridad → Bastion.

## Errores comunes

- Guardar datos en D:.
- Apagar desde dentro de Windows creyendo que deja de facturarse.
- Crear la VM sin availability set y querer añadirla después (no se puede; hay que recrear).

## Preguntas que podrían aparecer

**1.** Un administrador apaga una VM desde el menú Inicio de Windows. Al día siguiente sigue apareciendo coste de cómputo. ¿Por qué?
- A) Los discos se facturan · B) La VM está en estado Stopped pero no desasignada; el hardware sigue reservado · C) Es un error de facturación · D) Hybrid Benefit está desactivado

<details><summary>Respuesta</summary>

**B.** Solo Stop desde el portal/CLI (deallocate) libera el cómputo y detiene su facturación.
</details>

**2.** Necesitas que una VM Linux ejecute un script de configuración automáticamente al primer arranque. ¿Qué opción del asistente usas?
- A) Etiquetas · B) Datos personalizados (cloud-init) o extensión Custom Script · C) Diagnóstico de arranque · D) Apagado automático

<details><summary>Respuesta</summary>

**B.** cloud-init (datos personalizados) o la Custom Script Extension ejecutan scripts al aprovisionar.
</details>

**3.** Una VM Windows dejó de responder por RDP y sospechas de un problema en el host físico. ¿Qué acción pruebas primero para moverla a otro host sin perder datos?
- A) Eliminar y recrear · B) Redeploy · C) Cambiar el tamaño · D) Restaurar desde backup

<details><summary>Respuesta</summary>

**B.** Redeploy apaga la VM y la mueve a un nuevo host de Azure conservando discos y configuración.
</details>

## Relacionado

- [[05 - Tamaños de VM y redimensionamiento]]
- [[06 - Discos administrados]]
- [[07 - Azure Disk Encryption y cifrado de discos]]
- [[09 - Alta disponibilidad - Availability Sets y Availability Zones]]
- [[11 - Extensiones de VM y automatización]]
- [[08 - Azure Bastion]]
- [[00 - Índice - Cómputo]]
