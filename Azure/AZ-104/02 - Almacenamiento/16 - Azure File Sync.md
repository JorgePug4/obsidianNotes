---
tags: [az-104, azure, almacenamiento, files, file-sync, hibrido]
modulo: Almacenamiento
peso_examen: Medio
---

# Azure File Sync ➕

> [!info] No aparece literalmente en el temario vigente, pero forma parte de la ruta oficial de aprendizaje de almacenamiento y es un distractor habitual en preguntas de Azure Files y de migración.

## ¿Qué es?

**Azure File Sync** centraliza los recursos compartidos de la organización en **Azure Files** manteniendo **servidores Windows locales** como **cachés** con la flexibilidad, rendimiento y compatibilidad de un servidor de archivos tradicional (SMB, NFS, FTPS a nivel local).

## ¿Para qué sirve?

- Sucursales con acceso rápido local a los mismos archivos.
- Reducir el almacenamiento local con **niveles en la nube (cloud tiering)**.
- Migrar servidores de archivos gradualmente y tener backup/DR centralizado.

## Conceptos clave (componentes) 🧠

| Componente | Qué es |
|---|---|
| **Storage Sync Service** | Recurso de Azure de nivel superior; agrupa servidores registrados. Los servidores solo se pueden registrar en un Storage Sync Service |
| **Sync group** | Define la topología de sincronización para un conjunto de archivos; contiene **un** cloud endpoint y **uno o más** server endpoints |
| **Cloud endpoint** | Un recurso compartido de Azure Files (uno por sync group) |
| **Server endpoint** | Una ruta (volumen o carpeta) en un servidor registrado. Un servidor puede tener varios server endpoints en distintos sync groups |
| **Registered server** | Servidor Windows con el **agente de Azure File Sync** instalado y registrado (Windows Server 2016 o posterior) |
| **Cloud tiering** | Opcional por server endpoint: mantiene localmente solo los archivos más usados y deja **punteros** (reparse points) de los demás; **política de espacio libre del volumen** (%) y **política de fecha** (no accedidos en N días) |

- Los archivos se sincronizan por **cambios**; los conflictos se resuelven conservando ambos (renombrado con sufijo de servidor).
- **Sincronización inicial**: se recomienda sembrar el cloud endpoint (copia inicial) y luego añadir los servidores; o bien usar **Azure Data Box** para sembrar.
- **Backup**: hacer copia de seguridad del **cloud endpoint** (Azure Backup para Files), no de los servidores con cloud tiering (los punteros no contienen datos).
- Las **ACL NTFS** se sincronizan y se aplican en el recurso compartido (con AD DS auth pueden usarse también en Azure directamente).
- **Requisitos del servidor**: Windows Server 2016+, PowerShell 5.1, TLS 1.2, acceso saliente HTTPS a Azure (no requiere 445 desde el servidor? Sí: el agente usa HTTPS para la sincronización; **445 solo si los usuarios quieren montar directamente el recurso de Azure**).
- El agente se actualiza automáticamente (Microsoft Update) si se habilita.
- **Desconexión**: los usuarios siguen accediendo a los archivos ya en caché; los archivos en niveles no están disponibles hasta reconectar.

## Cómo funciona

```
Azure Files (cloud endpoint) ◄──sync──► Storage Sync Service
                                             │
                    ┌────────────────────────┼─────────────────────────┐
              Servidor Madrid           Servidor Lima            Servidor Tokio
             (server endpoint)         (server endpoint)        (server endpoint)
               D:\Datos  caché             E:\Datos                 D:\Compartido
               cloud tiering 20 % libre
```

Pasos de despliegue 🧠 (orden que pregunta el examen):
1. Crear la cuenta de almacenamiento y el **recurso compartido de Azure Files**.
2. Crear el **Storage Sync Service** (misma región que la cuenta, recomendado).
3. Instalar el **agente** en el servidor Windows y **registrar** el servidor.
4. Crear un **sync group** con el cloud endpoint (el recurso compartido).
5. Añadir el **server endpoint** (ruta local; opcional cloud tiering).
6. Esperar la sincronización inicial.

```powershell
# En el servidor: registrar
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.psd1"
Register-AzStorageSyncServer -ResourceGroupName rg -StorageSyncServiceName sss-contoso
# En Azure
New-AzStorageSyncService -ResourceGroupName rg -Name sss-contoso -Location westeurope
New-AzStorageSyncGroup -ResourceGroupName rg -StorageSyncServiceName sss-contoso -Name sg-datos
New-AzStorageSyncCloudEndpoint -ResourceGroupName rg -StorageSyncServiceName sss-contoso -SyncGroupName sg-datos -Name ce1 -StorageAccountResourceId <id> -AzureFileShareName share1
New-AzStorageSyncServerEndpoint -ResourceGroupName rg -StorageSyncServiceName sss-contoso -SyncGroupName sg-datos -Name se-madrid -ServerResourceId <registeredServerId> -ServerLocalPath "D:\Datos" -CloudTiering -VolumeFreeSpacePercent 20 -TierFilesOlderThanDays 30
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Sucursales con acceso local rápido a los mismos archivos | File Sync con un server endpoint por sucursal en el mismo sync group |
| El disco local del servidor está lleno | **Cloud tiering** con política de espacio libre |
| Liberar archivos no usados en 60 días | Política de fecha (tier files older than 60 days) |
| Dos recursos compartidos distintos en un servidor | Dos sync groups, cada uno con su cloud endpoint y su server endpoint |
| Copia de seguridad de los archivos | Azure Backup del **recurso compartido** (cloud endpoint) |
| Puerto 445 bloqueado en la sucursal | File Sync (el agente usa HTTPS) sirve los archivos localmente por SMB |
| Servidor Windows Server 2012 R2 | **No soportado** por el agente actual; actualizar |

## Ejemplo

Contoso tiene 3 oficinas con servidores de archivos de 2 TB cada uno y quiere una única copia en Azure con backup central, dejando en cada oficina solo el 30 % más usado. Crea `share-datos` en Azure Files, un Storage Sync Service, registra los 3 servidores, crea un sync group con el cloud endpoint y tres server endpoints con cloud tiering (30 % de espacio libre). Azure Backup protege el recurso compartido.

## Comparaciones

| Herramienta | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Azure File Sync** | Sincronización continua servidor ↔ Azure Files | Caché local, niveles, multi-sitio | Servidores de archivos, sucursales |
| **Azure Files directo (SMB)** | Montar desde Azure/on-premises | Sin servidor local | Cuando 445 está abierto o hay VPN |
| **AzCopy / Robocopy** | Copia puntual | Rápido para migrar | Migración inicial, sin sincronización continua |
| **Azure Data Box** | Transferencia física | Sin red | Sembrar TB iniciales |
| **DFS-R** ➕ | Replicación entre servidores Windows | Nativo Windows | Puede coexistir con File Sync en migraciones |

## AZ-104 Exam Tips

- 🔥 🧠 Componentes: **Storage Sync Service → Sync group → 1 cloud endpoint + N server endpoints**; **registered server** con el **agente**.
- 🧠 Orden: cuenta/recurso compartido → Storage Sync Service → agente y registro → sync group → server endpoint.
- 🧠 **Cloud tiering**: espacio libre del volumen (%) y días sin acceso.
- 📌 File Sync (sincronización continua + caché) vs AzCopy (copia) vs Data Box (físico).
- ⚠️ Un recurso compartido = **un** cloud endpoint = **un** sync group.
- ⚠️ Backup del cloud endpoint, no de los servidores con tiering.
- ⚠️ El agente requiere **Windows Server 2016+**.

## Errores comunes

- Añadir el mismo recurso compartido a dos sync groups.
- Hacer backup del servidor local con tiering y obtener solo punteros.
- Registrar un servidor en dos Storage Sync Services.

## Preguntas que podrían aparecer

**1.** Tienes un Storage Sync Service y necesitas sincronizar la carpeta `D:\Ventas` de un servidor con el recurso compartido `ventas` de Azure Files, manteniendo localmente solo los archivos usados en los últimos 30 días. ¿Qué configuras?
- A) Un server endpoint con cloud tiering y política de fecha de 30 días · B) Un cloud endpoint con tiering · C) AzCopy sync programado · D) Un segundo Storage Sync Service

<details><summary>Respuesta</summary>

**A.** Cloud tiering se configura en el server endpoint, con política de espacio libre y/o de fecha.
</details>

**2.** ¿Cuántos cloud endpoints puede tener un sync group?
- A) 1 · B) 2 · C) 10 · D) Ilimitados

<details><summary>Respuesta</summary>

**A.** Un sync group tiene exactamente un cloud endpoint (un recurso compartido de Azure Files) y uno o más server endpoints.
</details>

## Relacionado

- [[13 - Azure Files]]
- [[15 - Instantáneas y eliminación temporal en Azure Files]]
- [[08 - Azure Storage Explorer y AzCopy]]
- [[Movimiento y migración de datos]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
