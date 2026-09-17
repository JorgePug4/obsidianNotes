---
tags: [az-104, azure, almacenamiento, files, identidad, smb, kerberos]
modulo: Almacenamiento
peso_examen: Alto
---

# Acceso basado en identidad para Azure Files

## ¿Qué es?

La **autenticación basada en identidad** permite que los usuarios accedan a un recurso compartido **SMB** de Azure Files con **sus credenciales de dominio** (Kerberos), en lugar de la clave de la cuenta, y que se apliquen **permisos por usuario/grupo** en dos niveles: **nivel de recurso compartido (RBAC)** y **nivel de archivo/carpeta (ACL NTFS)**.

## ¿Para qué sirve?

- Migrar servidores de archivos conservando los permisos NTFS.
- Que cada usuario vea solo lo que le corresponde.
- Perfiles FSLogix en Azure Virtual Desktop.

## Conceptos clave

- **Tres orígenes de identidad** 🧠 (solo **uno** por cuenta de almacenamiento):

| Origen | Cómo | Requisitos | Cuándo |
|---|---|---|---|
| **AD DS local (on-premises)** | La cuenta de almacenamiento se **une al dominio** (objeto de equipo o cuenta de servicio con `AzFilesHybrid`); los clientes deben ver un **controlador de dominio** | Identidades **sincronizadas con Entra ID** (Entra Connect / Cloud Sync) para asignar RBAC granular | Empresas con AD DS |
| **Microsoft Entra Domain Services** | Habilitar en la cuenta; los clientes unidos a Entra DS | Tenant con Entra DS desplegado | Sin AD local; VMs unidas a Entra DS |
| **Microsoft Entra Kerberos (identidades híbridas / cloud-only)** | Entra ID emite los tickets Kerberos | Clientes **Windows unidos a Entra o híbridos**; identidades híbridas o cloud-only (cloud-only requiere que el usuario tenga... soporte reciente) | Sin línea de visión a un DC; AVD |

- **Permisos de nivel de recurso compartido** (RBAC de Azure, "quién puede entrar"):
  - **Storage File Data SMB Share Reader**: lectura.
  - **Storage File Data SMB Share Contributor**: lectura, escritura, borrado.
  - **Storage File Data SMB Share Elevated Contributor**: además **modificar ACLs NTFS**.
  - **Permiso predeterminado de nivel de recurso compartido** (default share-level permission) 🧠: aplica a **todos los usuarios autenticados** sin necesidad de asignaciones individuales (útil cuando las identidades no están sincronizadas).
- **Permisos de nivel de directorio/archivo** (ACL NTFS, "qué puede hacer dentro"): se configuran con **icacls** o el Explorador de Windows tras montar el recurso con **clave de la cuenta** (superusuario) o con un usuario Elevated Contributor. El permiso efectivo es la **intersección** de RBAC y NTFS.
- Con identidad, el cliente monta con `net use Z: \\<cuenta>.file.core.windows.net\<share>` **sin credenciales** (Kerberos).
- **Entra ID para REST/portal** ➕: roles *Storage File Data Privileged Contributor/Reader* permiten operar por REST con Entra ID, saltando ACLs.
- NFS usa **UID/GID** y root squash, no Kerberos ni RBAC de recurso compartido.

## Cómo funciona

```
Usuario (dominio) ──► DC / Entra DS / Entra ID ──► ticket Kerberos para cifs/<cuenta>.file.core.windows.net
        │
        ▼
Azure Files verifica ticket → RBAC nivel share (Reader/Contributor/Elevated) → ACL NTFS → acceso
```

```powershell
# AD DS: unir la cuenta al dominio (módulo AzFilesHybrid, desde un equipo unido al dominio)
Import-Module .\AzFilesHybrid.psd1
Join-AzStorageAccount -ResourceGroupName rg -StorageAccountName st001 -DomainAccountType "ComputerAccount" -OrganizationalUnitDistinguishedName "OU=AzureFiles,DC=contoso,DC=com"
# Entra Kerberos
Set-AzStorageAccount -ResourceGroupName rg -StorageAccountName st001 -EnableAzureActiveDirectoryKerberosForFile $true -ActiveDirectoryDomainName contoso.com -ActiveDirectoryDomainGuid <guid>
# RBAC nivel share
New-AzRoleAssignment -SignInName ana@contoso.com -RoleDefinitionName "Storage File Data SMB Share Contributor" -Scope "/subscriptions/<id>/resourceGroups/rg/providers/Microsoft.Storage/storageAccounts/st001/fileServices/default/fileshares/share1"
# Permiso predeterminado
Set-AzStorageAccount -ResourceGroupName rg -StorageAccountName st001 -DefaultSharePermission StorageFileDataSmbShareReader
# ACL NTFS (montado como superusuario con clave)
icacls Z:\ /grant "contoso\ventas:(OI)(CI)(M)"
```

```bash
az storage account update --name st001 --resource-group rg --enable-files-aadds true      # Entra DS
az storage account update --name st001 --resource-group rg --default-share-permission StorageFileDataSmbShareContributor
```

Portal: cuenta → Recursos compartidos de archivos → **Acceso basado en identidad** → elegir origen → configurar permiso predeterminado.

## Configuración relevante para el examen

| Escenario | Origen / configuración |
|---|---|
| Usuarios de AD DS local, VMs unidas al dominio con acceso al DC | **AD DS** + identidades sincronizadas + RBAC share + ACL NTFS |
| Usuarios híbridos en portátiles unidos a Entra que trabajan remotos sin VPN al DC | **Entra Kerberos** |
| VMs unidas a Entra Domain Services | **Entra DS** |
| Dar acceso a todos los usuarios autenticados sin asignar roles uno a uno | **Permiso predeterminado de nivel de recurso compartido** |
| Un usuario debe poder cambiar permisos NTFS | **Elevated Contributor** |
| Configurar ACLs NTFS iniciales | Montar con la **clave de la cuenta** (superusuario) e `icacls` |
| Un usuario con Contributor a nivel share no puede abrir una carpeta | Revisar **ACL NTFS** (intersección) |

## Ejemplo

Contoso migra `\\fs01\proyectos` a Azure Files. Sus usuarios están en AD DS y sincronizados con Entra Connect; las VMs de Azure están unidas al dominio por VPN. Se habilita **AD DS** en la cuenta, se asigna *SMB Share Contributor* al grupo "Proyectos" y se copian los datos con Robocopy `/COPYALL` (conservando ACLs). Los usuarios montan la unidad sin introducir credenciales.

## Comparaciones

| Método de acceso | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Clave de la cuenta** | Montaje simple | Sin infraestructura | Apps, contenedores, configuración inicial de ACLs |
| **AD DS** | Empresas con dominio | ACL NTFS completo, sin cambiar clientes | On-premises + Azure con DC accesible |
| **Entra DS** | Sin AD local | Dominio administrado | VMs en Azure unidas a Entra DS |
| **Entra Kerberos** | Híbrido/cloud moderno | Sin línea de visión al DC | AVD, trabajo remoto |
| **NFS (UID/GID)** | Linux | Semántica POSIX | Cargas Linux |

## AZ-104 Exam Tips

- 🔥 🧠 Tres orígenes: **AD DS, Entra Domain Services, Entra Kerberos**; **solo uno por cuenta**.
- 🔥 📌 Dos niveles: **RBAC de recurso compartido** (Reader / Contributor / Elevated Contributor) + **ACL NTFS**; el efectivo es la **intersección**.
- 🧠 **Permiso predeterminado** de nivel share para todos los autenticados.
- 🧠 AD DS requiere identidades **sincronizadas** a Entra para RBAC granular.
- 💻 Habilitar el origen de identidad, asignar roles SMB Share, montar con clave para `icacls`.
- ⚠️ Con clave de cuenta eres **superusuario**: no hay permisos por usuario.
- ⚠️ La autenticación por identidad es solo **SMB**; NFS no la usa.

## Errores comunes

- Asignar solo RBAC y olvidar las ACL NTFS (o viceversa).
- Intentar usar AD DS sin sincronizar los usuarios a Entra ID.
- Habilitar dos orígenes en la misma cuenta (no es posible).

## Preguntas que podrían aparecer

**1.** Los usuarios de Contoso tienen cuentas en AD DS sincronizadas con Entra ID y acceden desde portátiles unidos a Entra desde casa, sin VPN. Necesitan montar un recurso compartido SMB con sus credenciales. ¿Qué origen de identidad configuras?
- A) AD DS local · B) Microsoft Entra Domain Services · C) Microsoft Entra Kerberos · D) Clave de la cuenta

<details><summary>Respuesta</summary>

**C.** Entra Kerberos no requiere línea de visión al controlador de dominio y funciona con identidades híbridas desde dispositivos unidos a Entra.
</details>

**2.** Un usuario tiene el rol Storage File Data SMB Share Reader y necesita modificar los permisos NTFS de una carpeta. ¿Qué rol necesita?
- A) Contributor · B) Storage File Data SMB Share Elevated Contributor · C) Owner · D) Storage Account Contributor

<details><summary>Respuesta</summary>

**B.** Elevated Contributor añade la capacidad de cambiar ACLs NTFS.
</details>

## Relacionado

- [[13 - Azure Files]]
- [[05 - Claves de acceso y autorización con Microsoft Entra ID]]
- [[17 - Identidad híbrida - Microsoft Entra Connect]]
- [[Microsoft Entra Domain Services]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
