---
tags: [az-104, azure, almacenamiento, files, smb, nfs]
modulo: Almacenamiento
peso_examen: Alto
---

# Azure Files

## ¿Qué es?

**Azure Files** ofrece **recursos compartidos de archivos administrados** accesibles por **SMB** (Windows, Linux, macOS) y **NFS** (Linux), montables como unidad de red desde Azure y desde on-premises. Fundamentos en [[Azure Files]] (AZ-900). En AZ-104: crear y configurar el recurso compartido, niveles, montaje, puertos y protección.

## ¿Para qué sirve?

- Sustituir servidores de archivos (con [[16 - Azure File Sync]] como caché local).
- Compartir configuración o datos entre varias VMs / contenedores (ACI, Container Apps, AKS).
- Perfiles de usuario (FSLogix para Azure Virtual Desktop).
- Aplicaciones "lift and shift" que necesitan una ruta UNC.

## Conceptos clave

- **Cuenta**: **Standard GPv2** (HDD, pago por uso: transacciones + capacidad; niveles **Transaction optimized, Hot, Cool**) o **Premium FileStorage** (SSD, **aprovisionado**, SMB y **NFS 4.1**). Existe también el modelo **aprovisionado v2** en Standard ➕.
- **Protocolos** 🧠: **SMB 3.x** (puerto **445**; cifrado en tránsito; Windows, Linux, macOS) y **NFS 4.1** (solo Premium, solo desde VNet/privado, sin cifrado en tránsito → private/service endpoint obligatorio). Un recurso compartido es SMB **o** NFS, no ambos. También acceso **REST** y portal.
- **Tamaño**: hasta **100 TiB** por recurso compartido (Standard requiere "grandes recursos compartidos" en cuentas nuevas ya viene habilitado). Cuota por recurso compartido configurable.
- **Autenticación SMB**: **clave de la cuenta** (identidad de la cuenta de almacenamiento: `AZURE\<cuenta>`) o **identidad** (AD DS / Entra DS / Entra Kerberos) → [[14 - Acceso basado en identidad para Azure Files]].
- **Puerto 445**: muchos ISP lo bloquean; desde on-premises, usar VPN/ExpressRoute o File Sync. Comprobar con `Test-NetConnection -ComputerName <cuenta>.file.core.windows.net -Port 445`.
- **Montaje**: Windows `net use Z: \\<cuenta>.file.core.windows.net\<share> /user:AZURE\<cuenta> <clave>` (o el script del portal que persiste la credencial con `cmdkey`); Linux `mount -t cifs` con `vers=3.1.1` o `mount -t nfs` con `vers=4,minorversion=1,sec=sys`.
- **Snapshots** y **soft delete** del recurso compartido → [[15 - Instantáneas y eliminación temporal en Azure Files]].
- **Azure Backup** para Files (snapshots gestionados + vaulted backup).
- **Redundancia**: Standard LRS/ZRS/GRS/GZRS (sin RA); Premium LRS/ZRS.
- **Cifrado**: en reposo siempre; en tránsito con SMB 3.x cifrado (requerido si "Secure transfer required" está activo); NFS sin cifrado en tránsito.
- **Configuración de seguridad SMB**: versiones SMB permitidas, algoritmos de cifrado, Kerberos ticket encryption (AES-256).
- **Límites Standard**: 20 000 IOPS por recurso compartido (grandes), 100 TiB; Premium: IOPS según capacidad aprovisionada (baseline + burst).

## Cómo funciona

```bash
az storage share-rm create --storage-account st001 --resource-group rg --name share1 --quota 1024 --access-tier Hot
az storage share-rm list --storage-account st001 --resource-group rg -o table
az storage file upload --account-name st001 --share-name share1 --source ./config.json --account-key <key>
az storage directory create --account-name st001 --share-name share1 --name docs --account-key <key>
az storage share-rm update --storage-account st001 --resource-group rg --name share1 --quota 2048
```

```powershell
New-AzRmStorageShare -ResourceGroupName rg -StorageAccountName st001 -Name share1 -QuotaGiB 1024 -AccessTier Hot
# Montar en Windows (script del portal, simplificado)
cmd.exe /C "cmdkey /add:`"st001.file.core.windows.net`" /user:`"localhost\st001`" /pass:`"<clave>`""
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\st001.file.core.windows.net\share1" -Persist
```

```bash
# Linux SMB
sudo mount -t cifs //st001.file.core.windows.net/share1 /mnt/share1 -o username=st001,password=<clave>,serverino,nosharesock,actimeo=30,vers=3.1.1
# Linux NFS (Premium)
sudo mount -t nfs st001.file.core.windows.net:/st001/share1 /mnt/share1 -o vers=4,minorversion=1,sec=sys
```

## Componentes / configuración de un recurso compartido

| Configuración | Opciones | Nota |
|---|---|---|
| Nombre | 3-63 caracteres, minúsculas, números, guiones | |
| Protocolo | SMB / NFS | NFS solo Premium |
| Nivel (Standard) | Transaction optimized / Hot / Cool | Cambiable |
| Cuota / capacidad aprovisionada | GiB | Premium: define IOPS y rendimiento |
| Snapshots | Manuales / Backup | Hasta 200 por recurso compartido |
| Soft delete | 1-365 días | A nivel de cuenta (Files) |
| Root squash (NFS) | No root squash / Root squash / All squash | Seguridad NFS |
| Permisos de nivel de recurso compartido | Roles RBAC SMB (con identidad) | Ver nota 14 |

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Montar desde Windows on-premises falla; puerto 445 bloqueado por el ISP | VPN Site-to-Site / ExpressRoute / Point-to-Site, o Azure File Sync |
| Linux con NFS | Cuenta **Premium FileStorage**, recurso compartido NFS, acceso solo desde VNet (private/service endpoint) |
| Compartir entre varias VMs Windows con permisos NTFS de dominio | SMB + autenticación con AD DS / Entra DS / Entra Kerberos |
| Máximo tamaño de recurso compartido | 100 TiB (grandes recursos compartidos) |
| Minimizar coste con acceso poco frecuente | Nivel **Cool** (Standard) |
| Muchas transacciones pequeñas | Nivel **Transaction optimized** o Premium |
| Contenedor ACI que necesita volumen persistente | Azure Files SMB montado con clave |

## Ejemplo

Fabrikam retira un servidor de archivos Windows con 4 TB. Crea un recurso compartido SMB Standard Hot de 5 TiB, habilita autenticación con **AD DS** (sus usuarios están sincronizados con Entra Connect), asigna *Storage File Data SMB Share Contributor* al grupo de empleados, copia los datos con **Robocopy** conservando ACLs y despliega **Azure File Sync** en la sucursal para mantener una caché local con niveles en la nube.

## Comparaciones

| Servicio | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Azure Files SMB** | Unidad de red Windows/Linux | Identidad, ACLs NTFS, snapshots, File Sync | Servidores de archivos, perfiles |
| **Azure Files NFS** | Linux/POSIX | Semántica NFS, Premium | Cargas Linux, HPC |
| **Azure Blob Storage** | Objetos por HTTP | Barato, niveles, inmutabilidad | Apps web, backups, data lake |
| **Azure NetApp Files** ➕ | NAS de alto rendimiento | Latencia muy baja, SMB+NFS | SAP, HPC exigente |
| **Discos administrados** | Bloque para una VM | Máximo rendimiento por VM | Disco de SO/datos |

## 💻 Laboratorio: crear y montar un recurso compartido

1. En una cuenta GPv2 crear el recurso compartido `share1` (Hot, 5 GiB).
2. Crear la carpeta `docs` y subir un archivo desde el portal.
3. En una VM Windows de Azure, ejecutar el script de "Conectar" del portal (PowerShell) y verificar la unidad Z:.
4. Comprobar con `Test-NetConnection -Port 445` desde tu equipo local si el puerto está abierto.
5. Crear una instantánea del recurso compartido y restaurar un archivo eliminado desde "Versiones anteriores".

## AZ-104 Exam Tips

- 🔥 🧠 SMB usa el puerto **445**; si está bloqueado → VPN/ExpressRoute o File Sync.
- 🔥 📌 **NFS = solo Premium FileStorage**, solo desde red privada, sin cifrado en tránsito.
- 🧠 Máximo **100 TiB** por recurso compartido; hasta **200 snapshots**.
- 🧠 Niveles Standard: **Transaction optimized, Hot, Cool** (no Archive).
- 🧠 Files **no tiene RA-GRS**.
- 💻 Crear recurso compartido, montar en Windows/Linux, subir archivos, cambiar cuota.
- ⚠️ Autenticación con clave = identidad `AZURE\<cuenta>` con control total; para permisos por usuario hace falta identidad.

## Errores comunes

- Intentar crear NFS en una cuenta Standard.
- Olvidar que el firewall corporativo bloquea 445 y culpar a Azure.
- Confundir Azure Files con Azure File Sync (Files es el servicio; File Sync lo sincroniza con servidores).

## Preguntas que podrían aparecer

**1.** Los usuarios de una oficina no pueden montar un recurso compartido de Azure Files SMB desde Internet, pero desde una VM de Azure funciona. ¿Cuál es la causa más probable?
- A) La cuota es insuficiente · B) El ISP bloquea el puerto 445 · C) Falta soft delete · D) El recurso compartido es NFS

<details><summary>Respuesta</summary>

**B.** Muchos proveedores bloquean SMB (445) hacia Internet. Solución: VPN, ExpressRoute o File Sync.
</details>

**2.** Necesitas un recurso compartido NFS 4.1 para 20 VMs Linux. ¿Qué requisito debe cumplir la cuenta?
- A) GPv2 con GRS · B) Premium FileStorage · C) Blob Storage con espacio de nombres jerárquico · D) GPv1

<details><summary>Respuesta</summary>

**B.** NFS solo está disponible en cuentas Premium FileStorage.
</details>

## Relacionado

- [[14 - Acceso basado en identidad para Azure Files]]
- [[15 - Instantáneas y eliminación temporal en Azure Files]]
- [[16 - Azure File Sync]]
- [[03 - Firewalls y redes virtuales de Azure Storage]]
- [[Azure Files]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
