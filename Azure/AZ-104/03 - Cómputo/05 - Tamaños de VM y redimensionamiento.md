---
tags: [az-104, azure, computo, vm, tamaños, sku]
modulo: Cómputo
peso_examen: Alto
---

# Tamaños de VM y redimensionamiento

## ¿Qué es?

El **tamaño (SKU)** de una VM define vCPUs, memoria, disco temporal, número máximo de discos de datos y NICs, IOPS/throughput de disco y ancho de banda de red. Se agrupa en **series (familias)** optimizadas para distintos usos. Se puede **cambiar** después de crear la VM (escalado vertical).

## ¿Para qué sirve?

- Ajustar rendimiento y coste a la carga.
- Cumplir requisitos: discos Premium, muchas NICs, GPU, memoria alta.

## Conceptos clave

### Familias 🧠

| Serie | Tipo | Uso | Ejemplos |
|---|---|---|---|
| **B** | Burstable (créditos de CPU) | Dev/test, cargas con picos, servidores pequeños | B1s, B2s, B2ms |
| **D / Dv5 / Dsv5 / Dasv5** | Uso general | Web, apps, bases de datos pequeñas | D2s_v5, D4as_v5 |
| **E** | Memoria optimizada | Bases de datos, caché en memoria | E4s_v5 |
| **F** | Cómputo optimizado (CPU alta) | Batch, gaming servers, análisis | F4s_v2 |
| **M** | Memoria muy alta | SAP HANA, DB grandes | M128s |
| **L** | Almacenamiento optimizado (NVMe local) | Big data, NoSQL | L8s_v3 |
| **N (NC, ND, NV)** | GPU | ML, render, visualización | NC6s_v3 |
| **H / HB / HC** | HPC | Simulaciones | HB120rs_v3 |
| **A (Av2)** | Básico/económico | Test ligeros | A2_v2 |
| **DC / EC** | Confidencial | Datos sensibles | DC2s_v3 |

### Letras del nombre 🧠
- **s** = soporta **discos Premium** (SSD) → `Standard_D2s_v5`. Sin "s" → no soporta Premium.
- **a** = procesador AMD; **p** = Arm (Ampere); **l** = memoria baja; **m** = memoria alta; **d** = **disco temporal local** presente; **i** = aislado; **t** = tiny memory.
- **_v5** = versión de la familia.

### Redimensionar (resize)
- Se hace desde **Tamaño** en el portal, `az vm resize`, `Update-AzVM`.
- Si el nuevo tamaño está disponible en el **clúster de hardware** actual, la VM **se reinicia** y cambia. Si no está disponible, hay que **desasignar** (deallocate) primero: la VM se mueve a otro clúster.
- El **disco temporal se pierde** y la **IP pública dinámica** cambia (usar estática).
- Restricciones: no cambiar a un tamaño sin "s" si hay discos Premium; no reducir por debajo del número de discos de datos/NICs en uso; cambiar entre familias con distinta generación puede no ser posible sin desasignar; en un **availability set** todas las VMs deben moverse si el tamaño no está en el clúster (opción "desasignar todas").
- Las **cuotas** de vCPU por familia y región aplican al nuevo tamaño.
- **Azure Advisor** recomienda tamaños por infrautilización.
- **Constrained vCPU** ➕ (por ejemplo `E8-4s_v3`): menos vCPUs para reducir licencias por core manteniendo memoria.

## Cómo funciona

```bash
az vm list-sizes --location westeurope -o table
az vm list-vm-resize-options --resource-group rg-web --name vm-web01 -o table   # tamaños disponibles sin desasignar
az vm resize --resource-group rg-web --name vm-web01 --size Standard_D4s_v5
# Si falla por no disponible en el clúster:
az vm deallocate --resource-group rg-web --name vm-web01
az vm resize --resource-group rg-web --name vm-web01 --size Standard_E4s_v5
az vm start --resource-group rg-web --name vm-web01
az vm list-usage --location westeurope -o table   # cuotas
```

```powershell
Get-AzVMSize -Location westeurope
Get-AzVMSize -ResourceGroupName rg-web -VMName vm-web01   # disponibles para esa VM
$vm = Get-AzVM -ResourceGroupName rg-web -Name vm-web01
$vm.HardwareProfile.VmSize = "Standard_D4s_v5"
Update-AzVM -ResourceGroupName rg-web -VM $vm
```

## Configuración relevante para el examen

| Escenario | Respuesta |
|---|---|
| Añadir un disco Premium SSD a una VM `Standard_D2_v3` falla | El tamaño no tiene "s" → redimensionar a `D2s_v3` |
| Necesito 8 discos de datos y el tamaño solo admite 4 | Redimensionar a un tamaño con más discos |
| El resize no muestra el tamaño deseado | Desasignar primero (cambia de clúster) |
| Servidor con picos ocasionales y presupuesto bajo | Serie **B** |
| SQL Server con mucha memoria | Serie **E** o **M** |
| Render 3D / ML | Serie **N** |
| Reducir coste tras ver CPU al 5 % | Redimensionar a menor (Advisor) |
| Tras redimensionar, la app perdió archivos en D: | Disco temporal; restaurar y no usar D: |

## Ejemplo

Una VM `Standard_D8s_v5` de un servidor web tiene CPU media del 6 %. Advisor sugiere `D2s_v5`. El administrador comprueba con `az vm list-vm-resize-options` que está disponible, programa una ventana (reinicio), avisa de que el disco temporal se borrará y ejecuta el resize. Ahorro ~75 %.

## Comparaciones

| Escalado | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Vertical (resize)** | Más/menos recursos en la misma VM | Simple, sin cambiar arquitectura | Cargas que no escalan horizontalmente |
| **Horizontal (VMSS / más VMs + LB)** | Más instancias | Sin límite de tamaño, HA | Web, workers sin estado |
| **Burstable (B)** | Créditos | Barato | Cargas intermitentes |
| **Reserved / Savings plan** | Compromiso | Descuento | Tamaño estable a largo plazo |

## AZ-104 Exam Tips

- 🔥 🧠 **"s" en el nombre = discos Premium.**
- 🔥 🧠 Resize = **reinicio**; si el tamaño no está en el clúster → **desasignar** primero; **se pierde el disco temporal**.
- 🧠 Familias: **B** burst, **D** general, **E** memoria, **F** CPU, **M** memoria extrema, **L** almacenamiento, **N** GPU, **H** HPC.
- 🧠 Redimensionar en availability set puede exigir desasignar **todas** las VMs del conjunto.
- 💻 `az vm resize`, `az vm list-vm-resize-options`, `Update-AzVM`.
- ⚠️ Cuotas de vCPU por familia: un resize puede fallar por cuota.

## Errores comunes

- Elegir un tamaño sin "s" y luego intentar discos Premium.
- No prever la pérdida del disco temporal.
- Ignorar que el cambio de tamaño reinicia la VM.

## Preguntas que podrían aparecer

**1.** Intentas redimensionar una VM de `Standard_D2s_v3` a `Standard_E4s_v5` y el tamaño no aparece en la lista. ¿Qué haces?
- A) Crear una VM nueva · B) Desasignar la VM y volver a intentar el cambio de tamaño · C) Cambiar la región · D) Añadir cuota

<details><summary>Respuesta</summary>

**B.** Los tamaños de otra familia/hardware requieren desasignar para mover la VM a un clúster compatible.
</details>

**2.** Una VM `Standard_D4_v3` necesita un disco de datos Premium SSD. ¿Qué debes hacer?
- A) Nada, es compatible · B) Redimensionar a `Standard_D4s_v3` · C) Cambiar a Ultra Disk · D) Habilitar encryption at host

<details><summary>Respuesta</summary>

**B.** Solo los tamaños con "s" soportan almacenamiento Premium.
</details>

## Relacionado

- [[04 - Máquinas virtuales - creación y configuración]]
- [[06 - Discos administrados]]
- [[09 - Alta disponibilidad - Availability Sets y Availability Zones]]
- [[16 - Administración de costes (presupuestos, alertas y Advisor)]]
- [[00 - Índice - Cómputo]]
