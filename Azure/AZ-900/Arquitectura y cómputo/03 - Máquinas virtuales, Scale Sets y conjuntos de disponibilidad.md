---
tags: [az-900, azure, computo, maquinas-virtuales, vmss, availability-set, avd]
modulo: Arquitectura y cómputo
peso_examen: Alto
---

# Máquinas virtuales, Scale Sets y conjuntos de disponibilidad

## Concepto

**Azure Virtual Machines** es el servicio **IaaS** de cómputo: un servidor virtual (Windows o Linux) que tú controlas por completo, desde el sistema operativo hacia arriba. Sobre él se construyen dos opciones de escalado y disponibilidad: los **Virtual Machine Scale Sets** (muchas VMs idénticas que crecen y decrecen) y los **conjuntos de disponibilidad** (VMs repartidas para sobrevivir a fallos de hardware). **Azure Virtual Desktop** es el servicio de escritorios virtuales en la nube.

**Problema que resuelve:** necesitas un servidor "de verdad" (control del SO, software específico, migración tal cual) pero sin comprar hardware ni esperar semanas.

**Para qué se utiliza:**
- Migrar servidores existentes a Azure sin reescribir (*lift-and-shift*).
- Entornos de desarrollo y prueba que se crean y destruyen a demanda.
- Aplicaciones que requieren un SO o software concretos.
- Extender el centro de datos en picos de demanda.

## Características principales

### Máquina virtual (VM)
- Eliges **imagen** (Windows Server, Ubuntu, imágenes de Marketplace), **tamaño** (familia y número de vCPU/RAM) y **región**.
- Tú administras: **sistema operativo, parches, antivirus, aplicaciones y datos**. Microsoft administra el host físico y el hipervisor.
- Se paga **por segundo mientras está encendida** (más el disco, que se paga siempre que exista). Una VM **detenida y desasignada** (deallocated) no cobra cómputo.
- Familias de tamaño: uso general (B, D), optimizadas para cómputo (F), memoria (E, M), almacenamiento (L), GPU (N), alto rendimiento (H).

#### Recursos que necesita una VM
Al crear una VM, Azure crea (o reutiliza) estos recursos asociados:

| Recurso | Para qué |
|---|---|
| **Disco de SO** ([[Azure Disk Storage]]) | Sistema operativo |
| **Discos de datos** (opcionales) | Datos de la aplicación |
| **Red virtual y subred** ([[01 - Azure Virtual Network]]) | Dónde vive la VM |
| **Interfaz de red (NIC)** | Conecta la VM a la subred |
| **IP pública** (opcional) | Acceso desde Internet |
| **Grupo de seguridad de red** ([[03 - Grupo de seguridad de red (NSG)]]) | Filtrar tráfico |
| **Grupo de recursos** | Contenedor lógico |

### Virtual Machine Scale Sets (VMSS)
- **Grupo de VMs idénticas** creadas a partir de la misma imagen y configuración.
- **Autoescalado**: añade o quita instancias según métricas (CPU, memoria, cola) o calendario. Es la herramienta de **escalado horizontal** y **elasticidad**.
- Se reparte automáticamente entre **zonas** o **dominios de error**, y suele ir detrás de un [[04 - Azure Load Balancer|Load Balancer]].
- Puede llegar a **1.000 instancias** (600 con imágenes personalizadas).
- Tú pagas solo las instancias que existen en cada momento.

### Conjuntos de disponibilidad (Availability Sets)
- Agrupación lógica de VMs **dentro de un mismo centro de datos** para que no fallen todas a la vez.
- Azure las reparte en:
  - **Dominios de error (fault domains)**: racks con alimentación y switch de red distintos. Hasta 3.
  - **Dominios de actualización (update domains)**: grupos que Azure reinicia por turnos durante mantenimientos planificados. Hasta 20 (5 por defecto).
- Protegen contra **fallo de hardware** (rack) y **mantenimiento planificado**. **No** protegen contra la caída del datacenter completo (para eso, zonas).
- Se necesitan **al menos dos VMs** para que el SLA del 99,95 % aplique.
- Sin coste adicional; solo pagas las VMs.

### Azure Virtual Desktop (AVD)
- **Escritorios y aplicaciones Windows virtualizados** en Azure, accesibles desde cualquier dispositivo (Windows, Mac, iOS, Android, navegador).
- Permite **Windows 10/11 multisesión**: varios usuarios en la misma VM, lo que abarata el coste.
- Ventajas: los datos se quedan en Azure (no en el portátil), integración con Microsoft 365 y Entra ID, MFA y acceso condicional.
- Caso típico: teletrabajo seguro, contratistas, puestos con hardware antiguo.

### Escalado de una VM

| Acción | Tipo | Requiere reinicio |
|---|---|---|
| Cambiar el tamaño de la VM (más vCPU/RAM) | **Vertical** (scale up) | Sí |
| Añadir instancias con un Scale Set | **Horizontal** (scale out) | No |

## Casos de uso

- Servidor de aplicaciones heredado que solo funciona en Windows Server 2012: **VM** con esa imagen.
- Web que recibe diez veces más tráfico los fines de semana: **VMSS** con autoescalado por CPU.
- Dos servidores de base de datos que no pueden caer a la vez por mantenimiento: **conjunto de disponibilidad**.
- Empleados que trabajan desde casa con portátiles personales: **Azure Virtual Desktop**.

## Comparaciones

| Necesidad | Solución | No confundir con |
|---|---|---|
| Control total del SO | VM | App Service (no da acceso al SO) |
| Crear VMs automáticamente según la carga | VMSS | Load Balancer (solo reparte tráfico entre las que existen) |
| Proteger contra fallo de rack o mantenimiento | Conjunto de disponibilidad | Zonas de disponibilidad (protegen contra fallo de datacenter) |
| Proteger contra fallo de datacenter | Zonas de disponibilidad | Conjunto de disponibilidad |
| Escritorio Windows para usuarios remotos | Azure Virtual Desktop | VM con RDP (una VM por usuario, sin multisesión) |
| Dejar de pagar cómputo sin borrar la VM | Detener y **desasignar** | Apagar desde el SO (sigue cobrando) |

## Conceptos que debo memorizar

> [!important]
> - **VM = IaaS**. Tú gestionas SO, parches y software. Pago por segundo encendida; el disco se paga siempre.
> - **VMSS** = VMs idénticas con **autoescalado**; herramienta de escalado **horizontal**.
> - **Conjunto de disponibilidad** = **dominios de error** (hardware) + **dominios de actualización** (mantenimiento); dentro de un datacenter; mínimo 2 VMs; gratuito.
> - **Zonas** protegen contra fallo de datacenter; **conjuntos** contra fallo de rack. Zonas > conjuntos.
> - **AVD** = escritorios Windows en la nube, multisesión, desde cualquier dispositivo.
> - Recursos de una VM: disco de SO, NIC, VNet/subred, IP pública (opcional), NSG, grupo de recursos.

## Tips para AZ-900

> [!tip]
> - "Automáticamente según la demanda" + "máquinas virtuales" → **VMSS**.
> - "Fallo de hardware" o "mantenimiento planificado" → **conjunto de disponibilidad**. "Fallo de un centro de datos" → **zonas**.
> - "Escritorio Windows desde cualquier dispositivo" o "multisesión" → **Azure Virtual Desktop**.
> - "Control total", "instalar software", "elegir versión del SO" → **VM**.
> - Trampa: "Un Load Balancer crea VMs según la carga". **No**; eso es VMSS.
> - Trampa: "Un conjunto de disponibilidad reparte VMs entre regiones". **No**; todo ocurre dentro de un datacenter.
> - Trampa: "Apagar la VM desde Windows deja de cobrar". **No**; hay que detenerla y desasignarla desde Azure.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tienes dos máquinas virtuales que ejecutan la misma aplicación. Necesitas asegurarte de que Azure no reinicie ambas a la vez durante un mantenimiento planificado. ¿Qué debes usar?

- A) Un Virtual Machine Scale Set
- B) Un conjunto de disponibilidad
- C) Azure Virtual Desktop
- D) Un Load Balancer

**Respuesta: B.** Los dominios de actualización del conjunto de disponibilidad garantizan que el mantenimiento se aplique por turnos.
- A) VMSS escala instancias; puede usar dominios, pero la pregunta describe justo el propósito del conjunto de disponibilidad.
- C) Es un servicio de escritorios, no de disponibilidad.
- D) Reparte tráfico; no controla el mantenimiento.

**Pregunta 2.** Una aplicación web en máquinas virtuales debe añadir instancias automáticamente cuando la CPU media supere el 75 %. ¿Qué servicio debes utilizar?

- A) Azure Virtual Machines con un conjunto de disponibilidad
- B) Virtual Machine Scale Sets
- C) Azure Virtual Desktop
- D) Azure Container Instances

**Respuesta: B.** El autoescalado por métricas es la función principal de los Scale Sets.
- A) Un conjunto de disponibilidad no añade VMs.
- C) AVD es para escritorios de usuario.
- D) Ejecuta contenedores, no VMs, y no autoescala por sí solo.

## 🧠 Resumen para el examen

1. VM = IaaS: control total del SO; pago por segundo encendida.
2. Una VM necesita disco de SO, NIC, VNet/subred, y opcionalmente IP pública y NSG.
3. VMSS = VMs idénticas con autoescalado = escalado horizontal.
4. Conjunto de disponibilidad = dominios de error + actualización; protege dentro del datacenter; mínimo 2 VMs.
5. Zonas protegen contra fallo de datacenter; conjuntos solo contra rack/mantenimiento.
6. Azure Virtual Desktop = escritorios Windows multisesión en la nube.

---
