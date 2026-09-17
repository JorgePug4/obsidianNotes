---
tags: [az-104, azure, computo, vm, mover, resource-mover]
modulo: Cómputo
peso_examen: Medio-Alto
---

# Mover una VM (grupo de recursos, suscripción o región)

## ¿Qué es?

Tres operaciones distintas que el examen mezcla a propósito:

1. **Mover a otro grupo de recursos** (misma suscripción) → operación ARM *move*.
2. **Mover a otra suscripción** (mismo tenant) → operación ARM *move*.
3. **Mover a otra región** → **NO** es un move: se usa **Azure Resource Mover**, **Azure Site Recovery** o se recrea desde snapshot/imagen.

Reglas generales de movimiento en [[14 - Grupos de recursos y movimiento de recursos]].

## Conceptos clave

### Mover a otro RG o suscripción
- Hay que mover **la VM y todas sus dependencias juntas** 🧠: discos administrados, NIC, IP pública, NSG (si solo lo usa esa VM), availability set, y en algunos casos la VNet.
- Requisitos: **mismo tenant**, proveedor de recursos registrado en el destino, cuota suficiente, sin locks que lo impidan.
- Durante el movimiento, **origen y destino quedan bloqueados** para escritura; la VM **sigue funcionando**.
- **No se conservan** las asignaciones RBAC hechas directamente sobre los recursos movidos.
- Restricciones específicas 🧠:
  - VMs con **Azure Disk Encryption** tienen restricciones para moverse entre suscripciones (el Key Vault debe estar en la misma suscripción/región).
  - VMs en un **availability set**: hay que mover **todo el conjunto** con todas sus VMs.
  - VMs protegidas por **Azure Backup**: hay que detener la protección (conservando datos) o eliminar el punto de restauración según el escenario; el vault no se mueve con la VM.
  - Discos con **CMK**: la VM debe estar **desasignada**.
  - VMs con **reservas** o **licencias de Marketplace** pueden requerir pasos extra.
- **Validar antes**: `az resource invoke-action --action validateMoveResources`.

### Mover a otra región
- **Azure Resource Mover**: servicio que orquesta el movimiento entre regiones (VMs, discos, NICs, VNets, NSGs, IPs, availability sets, SQL…). Flujo: **agregar recursos → resolver dependencias → preparar → iniciar movimiento → confirmar (commit) → descartar origen**.
- Internamente usa **Azure Site Recovery** para replicar los discos.
- Alternativa manual: snapshot del disco → crear disco en la otra región → crear VM.
- Cambia: IPs, nombres de recursos (se pueden mantener si se elimina el origen), zona de disponibilidad.
- **No** se conservan: IP pública (se asigna una nueva), extensiones que dependan de recursos regionales.

## Cómo funciona

```bash
# Validar y mover a otro RG (misma suscripción)
az resource invoke-action --action validateMoveResources \
  --ids "/subscriptions/<sub>/resourceGroups/rg-old" \
  --request-body '{"resources":["<vmId>","<diskId>","<nicId>","<pipId>"],"targetResourceGroup":"/subscriptions/<sub>/resourceGroups/rg-new"}'
az resource move --destination-group rg-new --ids <vmId> <diskId> <nicId> <pipId>
# Mover a otra suscripción
az resource move --destination-group rg-new --destination-subscription-id <subIdDestino> --ids <vmId> <diskId> <nicId>
```

```powershell
Move-AzResource -DestinationResourceGroupName rg-new -ResourceId $vm.Id
Move-AzResource -DestinationSubscriptionId <subId> -DestinationResourceGroupName rg-new -ResourceId $vm.Id
```

Portal: VM → **Información general → Mover** → "Mover a otro grupo de recursos" / "Mover a otra suscripción" / "Mover a otra región" (abre Resource Mover).

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Reorganizar recursos por aplicación | Mover a otro RG con todas las dependencias |
| Cambiar la facturación de un entorno | Mover a otra suscripción (mismo tenant) |
| Requisito de residencia de datos: llevar la VM a otra región | **Azure Resource Mover** (o ASR) |
| La suscripción destino está en otro tenant | Transferir primero la suscripción al mismo tenant |
| Mover una VM con ADE a otra suscripción | Revisar Key Vault; suele requerir deshabilitar el cifrado o mover el Key Vault |
| Mover una VM de un availability set | Mover el **conjunto completo** |
| La VM tiene backup activo | Detener protección (conservar datos) antes de mover entre suscripciones |
| Tras mover, un usuario perdió acceso | Reasignar los roles que estaban en el propio recurso |

## Ejemplo

Contoso debe llevar `vm-app01` de East US a West Europe por normativa GDPR. Como un *move* de ARM no cambia la región, usan **Azure Resource Mover**: añaden la VM (Resource Mover detecta disco, NIC, VNet y NSG como dependencias), eligen el RG y la VNet de destino, ejecutan **Preparar** (inicia la replicación con ASR), **Iniciar movimiento** (crea los recursos en destino), validan la app y **confirman**, eliminando después los recursos de origen. La IP pública es nueva, así que actualizan el DNS.

## Comparaciones

| Operación | Cambia región | Downtime | Herramienta | Cuándo |
|---|---|---|---|---|
| **Mover a otro RG** | No | Ninguno | ARM move | Reorganización |
| **Mover a otra suscripción** | No | Ninguno | ARM move | Facturación/propiedad |
| **Mover a otra región** | Sí | Corto (conmutación) | **Resource Mover** / ASR | Residencia, cercanía, consolidación |
| **Recrear desde snapshot/imagen** | Sí | Sí | Snapshot + crear VM | Casos simples o no soportados |
| **Azure Migrate** | Sí (desde on-premises) | Variable | Azure Migrate | Migración desde fuera de Azure |

## AZ-104 Exam Tips

- 🔥 🧠 Mover **no cambia la región**. Para cambiar de región → **Azure Resource Mover** (usa Site Recovery por debajo).
- 🔥 🧠 Hay que mover la VM **con todas sus dependencias**; mismo **tenant** para cambiar de suscripción.
- 🧠 Durante el movimiento, los RGs de origen y destino se **bloquean**; la VM sigue en marcha.
- 🧠 Las asignaciones RBAC **en el recurso** no viajan.
- 🧠 Availability set → mover el conjunto entero; ADE y backup imponen pasos previos.
- 💻 `az resource move`, `Move-AzResource`, validación previa, portal → Mover.
- ⚠️ Al mover de región, la **IP pública cambia**.

## Errores comunes

- Intentar mover solo la VM y dejar atrás discos o NIC.
- Buscar la opción "cambiar región" en el move de ARM.
- Olvidar detener la protección de backup antes de un movimiento entre suscripciones.

## Preguntas que podrían aparecer

**1.** Necesitas trasladar una VM de la región North Europe a France Central conservando su configuración. ¿Qué servicio usas?
- A) `az resource move` · B) Azure Resource Mover · C) Peering de VNets · D) Azure Migrate

<details><summary>Respuesta</summary>

**B.** Resource Mover orquesta el movimiento entre regiones con sus dependencias. `az resource move` solo cambia de RG o suscripción. Azure Migrate es para cargas externas a Azure.
</details>

**2.** Mueves una VM a otro grupo de recursos y después un usuario que administraba esa VM ya no tiene permisos. ¿Por qué?
- A) La VM cambió de región · B) Las asignaciones de rol con ámbito en el recurso no se conservan al mover · C) El lock lo impide · D) Hay que reiniciar la VM

<details><summary>Respuesta</summary>

**B.** Las asignaciones RBAC definidas sobre el propio recurso deben recrearse tras el movimiento; las del RG destino sí se heredan.
</details>

## Relacionado

- [[14 - Grupos de recursos y movimiento de recursos]]
- [[04 - Máquinas virtuales - creación y configuración]]
- [[11 - Azure Site Recovery]]
- [[10 - Interpretar asignaciones de acceso]]
- [[00 - Índice - Cómputo]]
