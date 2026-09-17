---
tags: [az-104, azure, asr, site-recovery, dr, failover]
modulo: Monitorización y mantenimiento
peso_examen: Muy alto
---

# Azure Site Recovery (ASR)

## ¿Qué es?

**Azure Site Recovery** es el servicio de **recuperación ante desastres (DR)**: **replica continuamente** las máquinas virtuales a otra región (o desde on-premises a Azure) y permite hacer **conmutación por error (failover)** cuando la región principal no está disponible. Se configura dentro de un **Recovery Services vault**.

El temario lo cita explícitamente: "configurar Azure Site Recovery para recursos de Azure" y "realizar una conmutación por error a una región secundaria".

## ¿Para qué sirve?

- Continuidad de negocio: mantener las aplicaciones en marcha ante la caída de una región.
- Migraciones entre regiones con mínimo tiempo de inactividad.
- Cumplir objetivos de **RPO** (pérdida de datos) y **RTO** (tiempo de recuperación).

## Conceptos clave (Azure a Azure) 🧠

- **Replicación continua asíncrona** de los discos de la VM a la región de destino a través de una **cuenta de almacenamiento de caché** en la región de origen.
- **Puntos de recuperación**:
  - **Coherentes con el bloqueo (crash-consistent)**: se generan **cada 5 minutos** automáticamente.
  - **Coherentes con la aplicación (app-consistent)**: según la frecuencia que configures (por ejemplo, cada hora); **desactivados por defecto** en la política predeterminada.
- **Directiva de replicación**: **retención de puntos de recuperación** (por defecto **24 horas**, hasta 15 días) y **frecuencia de instantáneas coherentes con la aplicación**.
- **RPO** típico: minutos. **RTO**: minutos a horas según el plan.
- Al habilitar la replicación, ASR crea en la región de destino: **grupo de recursos**, **red virtual**, **cuentas de almacenamiento de caché**, **discos réplica** y, si se necesita, una **cuenta de Automation** para las actualizaciones de movilidad.
- **Asignación de red (network mapping)**: qué VNet/subred de destino corresponde a cada una de origen; se puede fijar la **IP privada** de destino.
- **Plan de recuperación (recovery plan)** 🧠: agrupa VMs y define el **orden de failover** (grupos 1, 2, 3) con **scripts de Automation** o **acciones manuales** entre pasos. Imprescindible para aplicaciones de varias capas.
- **Tipos de conmutación** 🧠:

| Tipo | Qué hace | Cuándo |
|---|---|---|
| **Test failover (prueba)** | Crea las VMs en una **red aislada**; **no afecta** a producción ni a la replicación | Simulacros, validación (recomendado cada 6 meses) |
| **Failover** | Conmuta a la región secundaria; puede elegirse el punto de recuperación (último procesado, más reciente, coherente con la aplicación) | Desastre real |
| **Planned failover** ➕ | Sin pérdida de datos, con la región de origen disponible | Mantenimientos planificados (no disponible en Azure→Azure en todos los casos) |
| **Commit** | Confirma el failover y elimina los puntos anteriores | Tras validar el failover |
| **Reprotect** | Invierte la replicación (destino → origen) | Antes de volver |
| **Failback** | Volver a la región original | Cuando se restablece |

- **Limpieza del test failover**: tras la prueba hay que ejecutar **"Limpiar la conmutación por error de prueba"** para eliminar los recursos temporales.
- **Requisitos y límites**: no todos los tamaños de VM ni todos los tipos de disco están soportados (Ultra Disk no está soportado; Premium SSD v2 tiene soporte limitado); la VM debe estar **en ejecución** para replicar; cuentas de caché en la región de origen.
- **Coste**: licencia por instancia protegida + almacenamiento y transferencia; las VMs de destino **solo se facturan al hacer failover** (los discos réplica sí se pagan).
- **Zonas**: ASR también soporta **replicación entre zonas** (zone-to-zone) dentro de una región.
- **On-premises → Azure**: VMware/Hyper-V con appliance o Azure Migrate; fuera del foco de AZ-104.

## Cómo funciona

```
Región primaria (West Europe)                 Región secundaria (North Europe)
  VM ──► cuenta de caché ──replicación asíncrona──► discos réplica
                                                     │
                        Failover (elige punto) ──► se crean las VMs en destino
                        Test failover ──────────► VMs en red aislada (sin impacto)
                        Reprotect / Failback ◄───
```

```bash
# Habilitar replicación (portal es lo habitual; CLI con az site-recovery)
az site-recovery vault create ...    # el vault es un Recovery Services vault
# Ver elementos replicados y su estado
az site-recovery replication-protected-item list --vault-name rsv-contoso -g rg-bkp -o table
# Test failover / failover con CLI de ASR
az site-recovery protected-item test-failover ...
```

```powershell
Set-AzRecoveryServicesAsrVaultContext -Vault $vault
$pi = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $container
Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $pi -Direction PrimaryToRecovery -AzureVMNetworkId $testVnetId
Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $pi
Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $pi -Direction PrimaryToRecovery
Start-AzRecoveryServicesAsrCommitFailoverJob -ReplicationProtectedItem $pi
Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $pi -Direction RecoveryToPrimary
```

Portal: **VM → Recuperación ante desastres** (flujo rápido) o **Recovery Services vault → Site Recovery → Replicar máquinas de Azure**; después **Elementos replicados**, **Planes de recuperación**, **Directivas de replicación**.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Probar el plan de DR sin afectar a producción | **Test failover** en una red aislada + **limpieza** |
| Conmutar una aplicación de 3 capas en orden | **Plan de recuperación** con grupos y scripts |
| Reducir la pérdida de datos a minutos | ASR (RPO de minutos), no Azure Backup |
| Volver a la región original tras el desastre | **Reprotect** y luego **failback** |
| Confirmar el failover y liberar puntos antiguos | **Commit** |
| Conservar 3 días de puntos de recuperación | Directiva de replicación con retención de 72 horas |
| Instantáneas coherentes con la aplicación cada hora | Configurar la frecuencia en la directiva (por defecto desactivadas) |
| Proteger frente a la caída de una **zona** | Replicación **zone-to-zone** |
| La VM usa Ultra Disk | **No soportado** por ASR |

## Ejemplo

Contoso protege 15 VMs de West Europe hacia North Europe. Crea una directiva con retención de 72 horas e instantáneas coherentes con la aplicación cada 4 horas. Define un **plan de recuperación**: grupo 1 base de datos, grupo 2 aplicación (con un script que cambia la cadena de conexión), grupo 3 web. Cada semestre ejecuta un **test failover** en una VNet aislada y luego la **limpieza**. Ante un incidente real, lanza el **failover**, valida, hace **commit** y, al restablecerse la región, **reprotect** y **failback**.

## Comparaciones

| | **Azure Backup** | **Azure Site Recovery** |
|---|---|---|
| Objetivo | Recuperar **datos** (puntos de restauración) | Mantener el **servicio** funcionando |
| RPO | Horas/días | **Minutos** |
| RTO | Horas | **Minutos** |
| Retención | Días a años | Horas a 15 días |
| Protege de | Borrado, corrupción, ransomware | Caída de región o zona |
| Coste | Almacenamiento de copias | Licencia + discos réplica |
| Complementarios | **Sí: se usan juntos** | |

| Opción de DR | Cuándo |
|---|---|
| **ASR** | VMs IaaS, DR gestionado |
| **Despliegue activo-activo multi-región** | Aplicaciones críticas con Front Door/Traffic Manager |
| **GZRS / ZRS** | Datos de almacenamiento |
| **Zonas de disponibilidad** | Fallo de un centro de datos |

## 💻 Laboratorio: DR con ASR

1. Crear una VM en West Europe y un Recovery Services vault en **North Europe** (el vault de ASR suele ir en la región de destino).
2. VM → **Recuperación ante desastres** → configurar región de destino, red y directiva; esperar a estado **Protegido**.
3. Revisar la directiva: retención 24 h y frecuencia de instantáneas coherentes con la aplicación.
4. Ejecutar un **test failover** con una VNet de prueba y comprobar la VM temporal; luego **limpiar**.
5. Crear un **plan de recuperación** con dos grupos y observar el orden.

## AZ-104 Exam Tips

- ⭐ ASR = **replicación continua + failover**; Backup = **puntos de restauración**.
- 🔥 🧠 Puntos **coherentes con el bloqueo cada 5 minutos**; los **coherentes con la aplicación** se configuran en la directiva (desactivados por defecto).
- 🔥 🧠 Retención de puntos por defecto **24 horas** (hasta 15 días).
- 🔥 🧠 **Test failover** no afecta a producción y requiere **limpieza** posterior.
- 🧠 Secuencia real: **failover → commit → reprotect → failback**.
- 🧠 **Plan de recuperación** para ordenar el failover de aplicaciones multicapa.
- 🧠 ASR vive en un **Recovery Services vault**.
- 💻 Habilitar replicación desde la VM, ejecutar test failover y limpieza.
- ⚠️ Ultra Disk no está soportado; la VM debe estar encendida para replicar.

## Errores comunes

- Confundir ASR con Backup en preguntas de "pérdida de datos mínima" (ASR) o "recuperar un archivo borrado" (Backup).
- Olvidar la limpieza tras el test failover (sigue costando).
- No hacer commit tras el failover y no poder reproteger.

## Preguntas que podrían aparecer

**1.** Quieres validar tu plan de recuperación ante desastres sin afectar a las máquinas virtuales de producción ni interrumpir la replicación. ¿Qué ejecutas?
- A) Failover · B) Test failover · C) Commit · D) Reprotect

<details><summary>Respuesta</summary>

**B.** El test failover crea las VMs en una red aislada y no altera la replicación.
</details>

**2.** ¿Con qué frecuencia genera Site Recovery puntos de recuperación coherentes con el bloqueo para VMs de Azure?
- A) Cada minuto · B) Cada 5 minutos · C) Cada hora · D) Cada 24 horas

<details><summary>Respuesta</summary>

**B.** Cada 5 minutos; los coherentes con la aplicación siguen la frecuencia configurada.
</details>

**3.** Tras un failover correcto a la región secundaria, ¿cuál es el siguiente paso antes de poder volver a la región original?
- A) Failback directo · B) Commit y luego reprotect · C) Eliminar el vault · D) Test failover

<details><summary>Respuesta</summary>

**B.** Se confirma el failover (commit) y se invierte la replicación (reprotect) antes del failback.
</details>

## Relacionado

- [[08 - Azure Backup - Recovery Services vault y Backup vault]]
- [[10 - Operaciones de copia de seguridad y restauración]]
- [[09 - Alta disponibilidad - Availability Sets y Availability Zones]]
- [[08 - Mover una VM (grupo de recursos, suscripción o región)]]
- [[00 - Índice - Monitorización y mantenimiento]]
