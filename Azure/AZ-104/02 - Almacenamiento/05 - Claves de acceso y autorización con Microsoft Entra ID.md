---
tags: [az-104, azure, almacenamiento, seguridad, claves, rbac]
modulo: Almacenamiento
peso_examen: Alto
---

# Claves de acceso y autorización con Microsoft Entra ID

## ¿Qué es?

Cada cuenta tiene **dos claves de acceso** (key1 y key2) de 512 bits que otorgan **control total** sobre los datos de todos los servicios de la cuenta (equivalen a "root"). La alternativa recomendada es **autorizar con Microsoft Entra ID** mediante **roles RBAC de plano de datos**.

## ¿Para qué sirve?

- Claves: acceso administrativo, herramientas antiguas, firmar SAS.
- Entra ID: acceso de usuarios y aplicaciones **sin secretos**, con auditoría y privilegio mínimo.

## Conceptos clave

- **Dos claves** para permitir **rotación sin interrupción**: las apps usan key1 → cambias las apps a key2 → regeneras key1 → (siguiente ciclo al revés).
- **Regenerar una clave invalida todas las SAS firmadas con ella** y rompe cualquier cliente que la use.
- **Key Vault** puede gestionar y rotar las claves automáticamente (Storage account keys managed by Key Vault) ➕.
- **Directiva de expiración de claves** (key expiration policy): recordatorio/alerta cuando una clave lleva más de N días sin rotar. Azure Policy integrada para auditar.
- **Deshabilitar la autorización por clave** (`Allow storage account key access = Disabled`): obliga a usar Entra ID; rompe account/service SAS y herramientas que usen clave.
- **Roles de datos** 🧠 (plano de datos, distintos de Contributor/Owner):

| Servicio | Roles |
|---|---|
| Blob | Storage Blob Data **Owner** / **Contributor** / **Reader**; Storage Blob **Delegator** (para crear user delegation SAS) |
| Queue | Storage Queue Data Contributor / Reader / Message Processor / Message Sender |
| Table | Storage Table Data Contributor / Reader |
| Files (SMB con identidad) | Storage File Data SMB Share **Reader** / **Contributor** / **Elevated Contributor** |
| Files (REST/portal con Entra) | Storage File Data Privileged Contributor / Reader ➕ |

- **Owner / Contributor / Storage Account Contributor** son de **plano de control**: pueden **listar las claves** (`listKeys`) y por tanto acceder a los datos *a través de la clave*, pero **no** tienen permiso directo de datos con Entra ID. El **Reader** no puede listar claves.
- Para que el **portal** muestre blobs con Entra ID hay que cambiar el "método de autenticación" a *cuenta de usuario de Microsoft Entra* y tener un rol de datos; por defecto el portal usa la clave.
- **Predeterminado a Entra ID en el portal** (`defaultToOAuthAuthentication`) ➕.
- Files con **SMB** usa identidad de dominio (AD DS / Entra DS / Entra Kerberos), ver [[14 - Acceso basado en identidad para Azure Files]].

## Cómo funciona

```
Petición con clave → firma HMAC con la clave → acceso total
Petición con SAS   → firma verificada → permisos de la SAS
Petición con token de Entra ID → RBAC de datos en el ámbito (cuenta / contenedor) → permitido o 403
```

```bash
az storage account keys list --account-name st001 --resource-group rg -o table
az storage account keys renew --account-name st001 --resource-group rg --key key1
az storage account update --name st001 --resource-group rg --allow-shared-key-access false
az role assignment create --assignee ana@contoso.com --role "Storage Blob Data Contributor" --scope /subscriptions/<id>/resourceGroups/rg/providers/Microsoft.Storage/storageAccounts/st001/blobServices/default/containers/docs
az storage blob list --account-name st001 --container-name docs --auth-mode login
```

```powershell
Get-AzStorageAccountKey -ResourceGroupName rg -Name st001
New-AzStorageAccountKey -ResourceGroupName rg -Name st001 -KeyName key2
Set-AzStorageAccount -ResourceGroupName rg -Name st001 -AllowSharedKeyAccess $false
New-AzRoleAssignment -SignInName ana@contoso.com -RoleDefinitionName "Storage Blob Data Reader" -Scope "/subscriptions/<id>/resourceGroups/rg/providers/Microsoft.Storage/storageAccounts/st001"
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Rotar claves sin interrumpir la app | Cambiar la app a key2, regenerar key1, luego alternar |
| Usuario con Contributor no ve blobs en el portal con "Entra ID" | Asignarle **Storage Blob Data Reader/Contributor** |
| Prohibir claves y SAS de cuenta | Deshabilitar *Allow storage account key access* |
| Identidad administrada de VM debe escribir blobs | Rol **Storage Blob Data Contributor** en la cuenta/contenedor |
| Usuario debe generar user delegation SAS | Rol que incluya `generateUserDelegationKey` (Blob Data Contributor/Owner o Blob Delegator) |
| Auditar claves antiguas | Key expiration policy + Azure Policy |

## Ejemplo

Una aplicación en App Service escribía en Blob con `key1` guardada en la configuración. Auditoría exige eliminar secretos. Solución: habilitar la **identidad administrada** del App Service, asignarle **Storage Blob Data Contributor** en el contenedor, cambiar el SDK a `DefaultAzureCredential` y, al final, **deshabilitar el acceso por clave** en la cuenta.

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Claves de cuenta** | Control total | Compatible con todo | Solo administración y herramientas; rotar |
| **SAS** | Acceso delegado temporal | Granular y temporal | Clientes sin identidad de Entra |
| **Entra ID + RBAC de datos** | Usuarios, apps, identidades administradas | Sin secretos, auditoría, privilegio mínimo | Opción recomendada |
| **Acceso anónimo** | Contenedor público de lectura | Sin autenticación | Contenido web público (deshabilitado por defecto en cuentas nuevas) |

## AZ-104 Exam Tips

- 🔥 📌 **Roles de datos** (Blob Data Reader/Contributor/Owner) ≠ **roles de control** (Contributor, Storage Account Contributor). Contributor no lee blobs con Entra ID, pero puede listar claves.
- 🔥 🧠 Regenerar una clave **invalida las SAS** firmadas con ella; **dos claves** para rotar sin cortes.
- 🧠 Reader (RBAC) **no puede listar claves**.
- 🧠 Deshabilitar acceso por clave → rompe account/service SAS y clientes con clave.
- 💻 `az storage account keys renew`, `--auth-mode login`, asignar roles de datos.
- ⚠️ El portal usa la clave por defecto; para probar RBAC de datos cambiar el método de autenticación.

## Errores comunes

- Asignar Owner a una identidad administrada para que lea blobs (excesivo y además no basta para el plano de datos).
- Regenerar ambas claves a la vez en producción.
- Confundir Storage Account Contributor (control) con Storage Blob Data Contributor (datos).

## Preguntas que podrían aparecer

**1.** Un usuario con el rol Reader en la cuenta de almacenamiento intenta ver los blobs en el portal y recibe un error de autorización. ¿Cuál es la solución que respeta el privilegio mínimo?
- A) Asignarle Contributor · B) Asignarle Storage Blob Data Reader y usar autenticación de Entra ID en el portal · C) Darle la clave de acceso · D) Asignarle Owner

<details><summary>Respuesta</summary>

**B.** Reader no puede listar claves ni tiene permisos de datos. El rol de datos de lectura es el mínimo.
</details>

**2.** Necesitas rotar las claves de una cuenta sin que las aplicaciones dejen de funcionar. Las aplicaciones usan key1. ¿Cuál es el orden correcto?
- A) Regenerar key1, actualizar apps con la nueva key1 · B) Actualizar apps a key2, regenerar key1, actualizar apps a key1, regenerar key2 · C) Regenerar ambas · D) Deshabilitar el acceso por clave

<details><summary>Respuesta</summary>

**B.** Alternar entre claves permite regenerar la que no está en uso sin interrupción.
</details>

## Relacionado

- [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]]
- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[18 - Identidades administradas y entidades de servicio]]
- [[14 - Acceso basado en identidad para Azure Files]]
- [[00 - Índice - Almacenamiento]]
