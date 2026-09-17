---
tags: [az-104, azure, identidad, entra-id, usuarios]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Usuarios de Microsoft Entra ID

## ¿Qué es?

Un **usuario** es la identidad de una persona (o de una cuenta de servicio "humana") en el tenant. Es el objeto al que se asignan licencias, pertenencia a grupos, roles de directorio y roles RBAC.

## ¿Para qué sirve?

- Dar acceso a Azure, Microsoft 365 y aplicaciones.
- Recibir licencias (M365, Entra ID P1/P2).
- Ser miembro de grupos y destinatario de directivas (acceso condicional, SSPR).

## Conceptos clave

- **UPN (User Principal Name)**: `nombre@dominio` con el que inicia sesión. El sufijo debe ser un dominio verificado del tenant (o el `onmicrosoft.com`).
- **Tipo de usuario**: **Member** (miembro, pertenece a la organización) o **Guest** (invitado, externo). Se puede cambiar de uno a otro.
- **Origen de identidad (source)**: *Microsoft Entra ID* (nube), *Windows Server AD* (sincronizado por Entra Connect, la mayoría de atributos solo se editan on-premises), *External Microsoft Entra ID* / *Microsoft account* (invitados).
- **Propiedades**: nombre para mostrar, nombre de usuario, tipo, cuenta habilitada, **ubicación de uso (usage location)**, puesto, departamento, administrador (manager), atributos de contacto, autenticación (métodos, MFA).
- **Operaciones masivas**: crear, invitar y eliminar usuarios en bloque mediante **plantilla CSV** descargada del portal; también descarga masiva.
- **Eliminación**: al borrar un usuario queda **30 días en "usuarios eliminados"** y se puede **restaurar** con todas sus propiedades. Tras 30 días se elimina permanentemente.

## Cómo funciona

1. Un administrador con rol suficiente crea el usuario (portal, CLI, PowerShell, Graph, CSV).
2. Se le asigna una contraseña inicial (autogenerada o manual), normalmente con "cambiar al iniciar sesión".
3. Opcionalmente se establece la **ubicación de uso**, obligatoria para asignar licencias.
4. Se añade a grupos, se le asignan licencias y roles.

Comandos que debes reconocer:

```bash
# Azure CLI
az ad user create --display-name "Ana López" --user-principal-name ana@contoso.com --password "P@ssw0rd!2026" --force-change-password-next-sign-in true
az ad user list --output table
az ad user delete --id ana@contoso.com
```

```powershell
# Microsoft Graph PowerShell (sustituye al módulo AzureAD, retirado)
Connect-MgGraph -Scopes "User.ReadWrite.All"
$pwd = @{ Password = "P@ssw0rd!2026"; ForceChangePasswordNextSignIn = $true }
New-MgUser -DisplayName "Ana López" -UserPrincipalName ana@contoso.com -MailNickname ana -AccountEnabled -PasswordProfile $pwd -UsageLocation ES
Get-MgUser -All | Select-Object DisplayName, UserPrincipalName
```

## Componentes / propiedades importantes

| Propiedad | Por qué importa en el examen |
|---|---|
| **Usage location** | Sin ella, la asignación de licencia falla |
| **Account enabled** | Bloquear inicio de sesión sin borrar la cuenta |
| **User type** (Member/Guest) | Cambia permisos por defecto y facturación de licencias |
| **Source** | Si es sincronizado, se edita en AD local |
| **Manager** | Usado en flujos de aprobación y grupos dinámicos |
| **Department / Job title** | Atributos típicos en reglas de **grupos dinámicos** |
| **Authentication methods** | Requisito para SSPR y MFA |

## Configuración relevante para el examen

- **Crear en bloque**: Usuarios → Operaciones masivas → Crear en bloque → descargar plantilla CSV → rellenar (nombre, UPN, contraseña inicial, bloquear inicio de sesión) → subir. Se puede monitorizar en "Resultados de operaciones masivas".
- **Invitar en bloque**: mismo flujo con CSV de correos y mensaje de invitación.
- **Restaurar usuario**: Usuarios → Usuarios eliminados → Restaurar (dentro de 30 días).
- **Cambiar UPN**: posible en usuarios de nube; en sincronizados se cambia en AD DS.
- **Rol mínimo para crear/editar/borrar usuarios**: **User Administrator** (no puede tocar usuarios con roles de administrador; para eso Global Administrator / Privileged Authentication Administrator).

## Ejemplo

RR. HH. entrega una hoja con 250 nuevos empleados. Un User Administrator descarga la plantilla CSV, la rellena con UPN `@contoso.com` (dominio ya verificado), sube el archivo, espera el resultado y luego asigna licencias por grupo. No hace falta PowerShell ni Global Administrator.

## Comparaciones

| Tipo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Usuario miembro (nube)** | Empleado sin AD local | Se administra todo en Entra | Organizaciones cloud-only |
| **Usuario sincronizado** | Empleado con cuenta en AD DS | Una sola identidad híbrida | Organizaciones con AD local ([[17 - Identidad híbrida - Microsoft Entra Connect]]) |
| **Usuario invitado (B2B)** | Colaborador externo | Usa sus propias credenciales, sin licencia propia para lo básico | Partners, consultores ([[05 - Usuarios externos e invitados (B2B)]]) |
| **Entidad de servicio / identidad administrada** | Aplicaciones y automatización | Sin contraseñas humanas | Scripts, apps, VMs ([[18 - Identidades administradas y entidades de servicio]]) |

## 💻 Laboratorio: usuarios en Entra ID

1. Crear un usuario `lab.user1@<tenant>.onmicrosoft.com` desde el portal con contraseña autogenerada y ubicación de uso.
2. Descargar la plantilla CSV de creación masiva, añadir 3 usuarios y subirla. Revisar "Resultados de operaciones masivas".
3. Eliminar uno de ellos y restaurarlo desde "Usuarios eliminados".
4. Cambiar el tipo de usuario de uno de ellos a Guest y observar qué cambia.
5. Con Cloud Shell: `az ad user list --query "[].userPrincipalName" -o tsv`.

## AZ-104 Exam Tips

- ⭐ El UPN necesita un **dominio verificado**.
- 🧠 Usuario eliminado: **30 días** recuperable.
- 🧠 **Usage location** es obligatoria antes de asignar licencias.
- 🧠 Rol mínimo para crear usuarios: **User Administrator**.
- 💻 Creación e invitación masiva con **CSV** desde el portal.
- ⚠️ Usuario **sincronizado**: sus atributos se modifican en el AD local, no en Entra (el portal los muestra en gris).
- ⚠️ Un usuario **Guest** puede convertirse en Member sin recrearlo.

## Errores comunes

- Intentar asignar una licencia antes de definir la ubicación de uso.
- Editar en Entra un usuario sincronizado y no entender por qué "vuelve" el valor antiguo (lo sobrescribe la siguiente sincronización).
- Borrar y recrear un usuario en vez de restaurarlo (se pierden pertenencias y licencias).

## Preguntas que podrían aparecer

**1.** Un administrador borró por error a un usuario hace 10 días. El usuario tenía licencias y pertenecía a 12 grupos. ¿Qué debes hacer con el menor esfuerzo?
- A) Crear un usuario nuevo con el mismo UPN · B) Restaurarlo desde Usuarios eliminados · C) Abrir un ticket de soporte · D) Reimportar desde CSV

<details><summary>Respuesta</summary>

**B.** Dentro de los 30 días se restaura con todas sus propiedades, grupos y licencias. A crea una identidad nueva sin nada de lo anterior.
</details>

**2.** Necesitas asignar licencias de Microsoft 365 a 40 usuarios nuevos y la operación falla para todos. Los usuarios existen y hay licencias disponibles. ¿Cuál es la causa más probable?
- A) Falta el rol License Administrator · B) Los usuarios no tienen ubicación de uso · C) Los usuarios son invitados · D) El dominio no está verificado

<details><summary>Respuesta</summary>

**B.** La asignación de licencias requiere la propiedad *usage location*. Si faltara el rol (A) la operación no se habría podido iniciar; C y D no impiden la asignación.
</details>

## Relacionado

- [[03 - Grupos de Microsoft Entra ID]]
- [[04 - Licencias en Microsoft Entra ID]]
- [[05 - Usuarios externos e invitados (B2B)]]
- [[06 - Self-Service Password Reset (SSPR)]]
- [[01 - Microsoft Entra ID para administradores]]
- [[00 - Índice - Identidades y gobernanza]]
