---
tags: [az-104, azure, computo, vm, imagenes, compute-gallery]
modulo: Cómputo
peso_examen: Medio
---

# Imágenes y Azure Compute Gallery ➕

> [!info] Complemento necesario para VMSS y despliegues repetibles. Aparece como opción en preguntas de "desplegar muchas VMs idénticas".

## ¿Qué es?

- **Imagen administrada**: recurso creado a partir de una VM **generalizada** que sirve de plantilla para crear nuevas VMs. Vive en una región y una suscripción.
- **Azure Compute Gallery** (antes Shared Image Gallery): servicio para **almacenar, versionar y distribuir** imágenes (y aplicaciones de VM) a escala, con replicación a varias regiones y compartición entre suscripciones o tenants.

## ¿Para qué sirve?

- Desplegar VMs y scale sets con la configuración y el software ya instalados.
- Mantener versiones de la imagen dorada (golden image) y su ciclo de vida.
- Reducir el tiempo de arranque y garantizar consistencia.

## Conceptos clave

- **Generalizar** 🧠: quitar identidad de la máquina antes de capturar.
  - Windows: `sysprep /generalize /oobe /shutdown`.
  - Linux: `sudo waagent -deprovision+user` y salir.
  - Tras generalizar, **la VM original no se puede volver a usar**.
- **Especializada (specialized)**: captura sin generalizar; conserva nombre de equipo, usuarios y estado; las VMs creadas son clones (no piden credenciales nuevas). Útil para clonar servidores concretos.
- **Jerarquía de Compute Gallery** 🧠: **Galería → Definición de imagen → Versión de imagen**.
  - *Definición*: publicador, oferta, SKU, tipo de SO, **estado (generalizada/especializada)**, generación (Gen1/Gen2), características (Trusted Launch, aceleración de red).
  - *Versión*: `1.0.0`, con **regiones de réplica**, número de **réplicas por región**, tipo de almacenamiento (Standard_LRS/ZRS/Premium) y opción de **excluir de la última versión**.
- **Replicación**: cada región de destino tiene N réplicas; más réplicas = más despliegues simultáneos.
- **Compartición**: RBAC (dentro del tenant), **comunidad** o **direct shared gallery** ➕.
- **VM Applications** ➕: paquetes de aplicación versionados que se despliegan en VMs sin recrear la imagen.
- **Azure Image Builder** ➕: construye imágenes automatizadas (basado en Packer) desde una plantilla.
- Alternativas: imagen del **Marketplace** + Custom Script Extension / cloud-init; **snapshot** de disco (no es imagen; sirve para clonar discos).

## Cómo funciona

```bash
# 1) Generalizar dentro de la VM (sysprep / waagent) y desasignar
az vm deallocate -g rg-img -n vm-base
az vm generalize -g rg-img -n vm-base

# 2a) Imagen administrada simple
az image create -g rg-img -n img-web --source vm-base

# 2b) Compute Gallery
az sig create -g rg-img --gallery-name galContoso
az sig image-definition create -g rg-img --gallery-name galContoso --gallery-image-definition web-win2022 \
  --publisher Contoso --offer WindowsServer --sku 2022-web --os-type Windows --os-state Generalized --hyper-v-generation V2
az sig image-version create -g rg-img --gallery-name galContoso --gallery-image-definition web-win2022 \
  --gallery-image-version 1.0.0 --virtual-machine /subscriptions/<sub>/resourceGroups/rg-img/providers/Microsoft.Compute/virtualMachines/vm-base \
  --target-regions westeurope=2 northeurope=1 --replica-count 2

# 3) Crear VMs desde la imagen
az vm create -g rg-web -n vm-web02 --image /subscriptions/<sub>/resourceGroups/rg-img/providers/Microsoft.Compute/galleries/galContoso/images/web-win2022/versions/1.0.0 \
  --admin-username azureadmin --admin-password '<pwd>'
```

```powershell
Stop-AzVM -ResourceGroupName rg-img -Name vm-base -Force
Set-AzVM -ResourceGroupName rg-img -Name vm-base -Generalized
$vm = Get-AzVM -ResourceGroupName rg-img -Name vm-base
$img = New-AzImageConfig -Location westeurope -SourceVirtualMachineId $vm.Id
New-AzImage -Image $img -ImageName img-web -ResourceGroupName rg-img
New-AzGallery -GalleryName galContoso -ResourceGroupName rg-img -Location westeurope
```

Portal: VM → **Capturar** → elegir "Compartir imagen en una galería" (crear galería/definición/versión) o "Solo imagen administrada", y marcar **"Se especializó/generalizó"** y si se elimina la VM tras la captura.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Desplegar 100 VMs idénticas con la app instalada en 3 regiones | **Compute Gallery** con versión replicada a las 3 regiones |
| Clonar un servidor concreto con su nombre y usuarios | Imagen **especializada** |
| Plantilla base para VMs nuevas que pedirán credenciales | Imagen **generalizada** |
| Actualizar la imagen sin romper despliegues existentes | Nueva **versión** (1.0.1) en la misma definición |
| Compartir la imagen con otra suscripción del tenant | RBAC sobre la galería |
| Muchos despliegues simultáneos lentos | Aumentar el número de **réplicas** |
| Actualizar la app sin recrear la imagen | **VM Applications** |
| Automatizar la creación de imágenes | **Azure Image Builder** |

## Ejemplo

El equipo mantiene una imagen dorada de Windows Server 2022 con IIS y agentes corporativos. Cada mes crean una VM desde la última versión, aplican parches, ejecutan `sysprep /generalize`, capturan una nueva **versión** (1.0.x) en la galería y la replican a West Europe y North Europe con 3 réplicas cada una. El VMSS de producción apunta a la definición sin fijar versión, por lo que las instancias nuevas usan siempre la última.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Imagen administrada** | Plantilla simple | Rápida de crear | Pocas VMs, una región |
| **Compute Gallery** | Imágenes a escala | Versiones, réplicas multi-región, compartición, ZRS | Producción, VMSS, varias regiones |
| **Marketplace + extensión/cloud-init** | Imagen estándar + script | Sin mantener imágenes | Configuración ligera |
| **Snapshot de disco** | Copia puntual de un disco | Backup rápido, clonar discos | Restaurar, no plantilla |
| **VM Applications** | Paquetes de app | Actualizar apps sin nueva imagen | Software que cambia a menudo |

## AZ-104 Exam Tips

- ⭐ **Generalizar antes de capturar**: `sysprep /generalize` (Windows) o `waagent -deprovision+user` (Linux).
- 🔥 🧠 Jerarquía: **Galería → Definición → Versión**; la definición fija generalizada/especializada y la generación.
- 🧠 Tras generalizar, la **VM original queda inutilizable**.
- 🧠 Más **réplicas** = más despliegues concurrentes; la replicación multi-región se define por versión.
- 📌 Imagen (plantilla para VMs nuevas) vs snapshot (copia de un disco).
- 💻 `az vm generalize`, `az image create`, `az sig image-version create`, capturar desde el portal.

## Errores comunes

- Capturar sin generalizar y obtener VMs con conflictos de SID/nombre.
- Esperar poder encender la VM original tras generalizarla.
- Usar una imagen de una región distinta sin haberla replicado.

## Preguntas que podrían aparecer

**1.** Debes crear una plantilla a partir de una VM Windows para desplegar nuevas VMs que soliciten sus propias credenciales. ¿Qué haces antes de capturar la imagen?
- A) Ejecutar `waagent -deprovision` · B) Ejecutar `sysprep /generalize /oobe /shutdown` y luego generalizar la VM en Azure · C) Crear un snapshot · D) Habilitar Trusted Launch

<details><summary>Respuesta</summary>

**B.** Sysprep elimina la identidad de la máquina y permite crear una imagen generalizada.
</details>

**2.** Un VMSS en tres regiones despliega instancias muy lentamente desde una imagen de galería. ¿Qué configuras?
- A) Más zonas · B) Más réplicas de la versión de imagen en cada región · C) Discos Ultra · D) Rolling upgrades

<details><summary>Respuesta</summary>

**B.** El número de réplicas determina cuántos despliegues concurrentes soporta la versión de imagen en cada región.
</details>

## Relacionado

- [[04 - Máquinas virtuales - creación y configuración]]
- [[06 - Discos administrados]]
- [[10 - Virtual Machine Scale Sets]]
- [[11 - Extensiones de VM y automatización]]
- [[00 - Índice - Cómputo]]
