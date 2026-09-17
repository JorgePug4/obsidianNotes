---
tags: [az-104, azure, computo, vm, discos, managed-disks]
modulo: Cómputo
peso_examen: Alto
---

# Discos administrados (Managed Disks)

## ¿Qué es?

Los **discos administrados** son volúmenes de bloque persistentes que Azure gestiona por ti (sin cuentas de almacenamiento que administrar). Una VM tiene un **disco de SO**, opcionalmente un **disco temporal** (no administrado, local) y **discos de datos**. Fundamentos en [[Azure Disk Storage]] (AZ-900).

## ¿Para qué sirve?

- Almacenar el SO y los datos de la VM con durabilidad (LRS 3 copias; ZRS disponible en algunos tipos).
- Adjuntar, ampliar, copiar (snapshot), compartir entre VMs y cifrar.

## Conceptos clave

### Tipos de disco 🧠

| Tipo | Medio | IOPS/throughput | Uso | Notas |
|---|---|---|---|---|
| **Standard HDD** | HDD | Bajo | Dev/test, backup, acceso poco frecuente | Más barato |
| **Standard SSD** | SSD | Medio | Web ligero, dev/test, cargas no críticas | Latencia consistente |
| **Premium SSD** | SSD | Alto, según tamaño (P1-P80) | Producción | Requiere tamaño de VM con "s"; **SLA 99,9 % para VM única** |
| **Premium SSD v2** | SSD | IOPS y throughput **configurables independientemente** del tamaño | Producción exigente | **Solo disco de datos** (no SO); sin snapshots incrementales? Soporta snapshots incrementales ahora; no soporta ADE ni ciertas características; sin caché de host |
| **Ultra Disk** | SSD NVMe | Muy alto, ajustable en caliente | SAP HANA, DB top | **Solo disco de datos**; requiere zona; sin snapshots; no compatible con ADE ni con backup estándar |

- **Tamaños**: hasta **32 TiB** por disco (Premium/Standard); Ultra hasta 64 TiB. Disco de SO hasta 4 TiB (Windows) — verificar límites actuales.
- **Redundancia**: **LRS** por defecto; **ZRS** disponible para Premium SSD y Standard SSD en regiones con zonas (permite adjuntar a VMs de otra zona tras fallo).
- **Caché de host**: **None / ReadOnly / ReadWrite**. SO: ReadWrite por defecto; datos: ReadOnly recomendado para lectura intensiva; None para logs/escritura intensiva. Cambiar la caché requiere que la VM esté desasignada (o reinicio) en algunos casos.
- **Cambiar el tipo de disco** (Standard ↔ Premium): la VM debe estar **desasignada** 🧠. Premium SSD v2 y Ultra no se convierten.
- **Ampliar un disco**: se puede aumentar (no reducir) el tamaño; para discos de datos sin desasignar en muchos casos (expansión en línea); disco de SO requiere desasignar. Luego **extender la partición** en el SO.
- **Adjuntar/desconectar**: discos de datos se adjuntan en caliente; número máximo según tamaño de VM; se identifican por **LUN**.
- **Snapshot**: copia completa o **incremental** (recomendada, más barata) de solo lectura de un disco en un momento; sirve para crear discos nuevos o backups puntuales.
- **Imagen**: disco de SO generalizado (Sysprep / waagent -deprovision) para crear VMs; mejor en **Azure Compute Gallery** ([[12 - Imágenes y Azure Compute Gallery]]).
- **Discos compartidos (shared disks)** ➕: Premium/Ultra adjuntados a varias VMs (SCSI PR) para clústeres.
- **Bursting**: Premium SSD ≤ P20 tiene bursting bajo demanda/crédito.
- **Private Link para discos** ➕: exportar/importar VHD por red privada.
- **Discos no administrados** (VHD en cuenta de almacenamiento): legado; migrar con `ConvertTo-AzVMManagedDisk`.
- **Cifrado**: SSE con PMK por defecto; CMK; encryption at host; ADE; ver [[07 - Azure Disk Encryption y cifrado de discos]].
- **Eliminar con la VM**: opción por disco (SO/datos) para borrar el disco al borrar la VM.
- **Cambio de rendimiento de nivel** (Premium): se puede subir el tier de rendimiento (P30 → P40) sin cambiar el tamaño, temporalmente.

## Cómo funciona

```bash
# Crear y adjuntar un disco de datos
az disk create --resource-group rg-web --name disk-data01 --size-gb 256 --sku Premium_LRS --zone 1
az vm disk attach --resource-group rg-web --vm-name vm-web01 --name disk-data01 --caching ReadOnly --lun 0
# Crear disco vacío al vuelo
az vm disk attach --resource-group rg-web --vm-name vm-web01 --name disk-data02 --new --size-gb 128 --sku StandardSSD_LRS
# Ampliar
az disk update --resource-group rg-web --name disk-data01 --size-gb 512
# Cambiar tipo (VM desasignada)
az vm deallocate -g rg-web -n vm-web01
az disk update --resource-group rg-web --name disk-data01 --sku StandardSSD_LRS
# Snapshot incremental
az snapshot create --resource-group rg-web --name snap-data01 --source disk-data01 --incremental true
# Disco desde snapshot
az disk create --resource-group rg-web --name disk-data01-restored --source snap-data01
# Desconectar
az vm disk detach --resource-group rg-web --vm-name vm-web01 --name disk-data02
```

```powershell
$cfg = New-AzDiskConfig -Location westeurope -CreateOption Empty -DiskSizeGB 256 -SkuName Premium_LRS -Zone 1
$disk = New-AzDisk -ResourceGroupName rg-web -DiskName disk-data01 -Disk $cfg
$vm = Get-AzVM -ResourceGroupName rg-web -Name vm-web01
$vm = Add-AzVMDataDisk -VM $vm -Name disk-data01 -CreateOption Attach -ManagedDiskId $disk.Id -Lun 0 -Caching ReadOnly
Update-AzVM -ResourceGroupName rg-web -VM $vm
$snapCfg = New-AzSnapshotConfig -SourceUri $disk.Id -Location westeurope -CreateOption Copy -Incremental
New-AzSnapshot -ResourceGroupName rg-web -SnapshotName snap-data01 -Snapshot $snapCfg
```

Dentro del SO: Windows → Administración de discos → inicializar (GPT), nuevo volumen; Linux → `lsblk`, `parted`, `mkfs.ext4`, `/etc/fstab` con UUID.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Disco de datos de máximo rendimiento configurable, sin depender del tamaño | **Premium SSD v2** (solo datos) o **Ultra** |
| Cambiar disco de Standard HDD a Premium SSD | Desasignar la VM → cambiar SKU (VM debe ser tamaño "s") |
| Ampliar un disco de datos de 128 a 512 GB | `az disk update --size-gb 512` → extender volumen en el SO |
| Copia puntual de un disco antes de un cambio | **Snapshot** (incremental) |
| Crear 20 VMs idénticas a una configurada | Generalizar → **imagen** en Compute Gallery |
| Base de datos con escrituras intensivas | Caché **None** en el disco de datos |
| Disco compartido entre dos VMs de un clúster de failover | **Shared disk** Premium/Ultra |
| VM de producción con SLA de VM única | Todos los discos **Premium SSD** (o Ultra) → 99,9 % |
| Migrar discos no administrados | `ConvertTo-AzVMManagedDisk` (desasignar) |
| La VM está en zona 2; el disco debe estar en… | La **misma zona** (o ZRS) |

## Ejemplo

Una VM SQL Server necesita: disco de SO Premium (caché RW), disco de datos de 1 TiB con caché **ReadOnly** y disco de logs de 256 GB con caché **None**. Se crean como Premium SSD en la misma zona que la VM y se adjuntan en LUN 0 y 1. Antes de aplicar un parche mayor, se toma un snapshot incremental de cada disco.

## Comparaciones

| Tipo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Standard HDD** | Coste mínimo | Barato | Backups, dev, acceso raro |
| **Standard SSD** | Entrada SSD | Latencia consistente, barato | Web, dev/test |
| **Premium SSD** | Producción | Alto rendimiento, SLA VM única | La mayoría de producción |
| **Premium SSD v2** | Producción flexible | IOPS/MBps independientes, barato por GiB | Datos exigentes, solo disco de datos |
| **Ultra** | Extremo | Hasta 160 000 IOPS por disco, ajustable | SAP HANA, DB top |

## AZ-104 Exam Tips

- 🔥 🧠 Cambiar tipo de disco (HDD ↔ SSD ↔ Premium) → **VM desasignada**.
- 🔥 🧠 **Premium SSD v2 y Ultra: solo discos de datos**, requieren zona, sin ADE.
- 🧠 Ampliar sí, **reducir no**; después extender en el SO.
- 🧠 Caché: SO **ReadWrite**, datos lectura **ReadOnly**, escritura intensiva **None**.
- 🧠 Snapshot **incremental** = más barato; disco de SO ≠ imagen (imagen = generalizada).
- 💻 `az disk create/update`, `az vm disk attach/detach`, `az snapshot create`.
- 📌 Disco temporal (local, se pierde) vs disco de datos (administrado, persistente).
- ⚠️ La VM y sus discos deben estar en la **misma región** (y zona si es zonal).

## Errores comunes

- Intentar usar Ultra/Premium v2 como disco de SO.
- Ampliar el disco y olvidar extender el volumen en el SO.
- Cambiar el SKU con la VM en ejecución.

## Preguntas que podrían aparecer

**1.** Necesitas convertir el disco de SO de una VM de Standard HDD a Premium SSD. ¿Qué debes hacer primero?
- A) Crear un snapshot · B) Desasignar la VM · C) Cambiar la región · D) Eliminar el disco temporal

<details><summary>Respuesta</summary>

**B.** El cambio de tipo de disco requiere la VM desasignada (y un tamaño que soporte Premium).
</details>

**2.** Amplías un disco de datos de 100 GB a 500 GB desde el portal, pero en Windows el volumen sigue mostrando 100 GB. ¿Qué falta?
- A) Reiniciar Azure · B) Extender el volumen desde Administración de discos · C) Cambiar la caché · D) Crear un snapshot

<details><summary>Respuesta</summary>

**B.** Azure amplía el disco; el sistema operativo debe extender la partición/volumen.
</details>

**3.** Una base de datos necesita 80 000 IOPS en un único disco de datos con posibilidad de ajustar el rendimiento sin cambiar de tamaño. ¿Qué tipo de disco eliges?
- A) Premium SSD P80 · B) Ultra Disk · C) Standard SSD · D) Disco temporal

<details><summary>Respuesta</summary>

**B.** Ultra Disk permite configurar IOPS y throughput de forma independiente y ajustarlos en caliente. Premium SSD v2 también podría, pero Ultra alcanza el rango más alto.
</details>

## Relacionado

- [[04 - Máquinas virtuales - creación y configuración]]
- [[05 - Tamaños de VM y redimensionamiento]]
- [[07 - Azure Disk Encryption y cifrado de discos]]
- [[12 - Imágenes y Azure Compute Gallery]]
- [[Azure Disk Storage]] (AZ-900)
- [[00 - Índice - Cómputo]]
