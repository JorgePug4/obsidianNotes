---
tags: [az-104, azure, identidad, entra-id, b2b, invitados]
modulo: Identidades y gobernanza
peso_examen: Medio
---

# Usuarios externos e invitados (Microsoft Entra B2B)

## ¿Qué es?

**Colaboración B2B** (business-to-business) de Microsoft Entra External ID permite invitar a personas de **otras organizaciones** (u otro tenant, cuenta Microsoft, Google, correo con código) para que accedan a tus recursos usando **sus propias credenciales**. Se crean en tu tenant como usuarios de tipo **Guest**.

En AZ-900 se ve como concepto ([[Acceso de usuarios externos (Microsoft Entra External ID)]]). En AZ-104 debes saber invitar, configurar restricciones y administrar el ciclo de vida.

## ¿Para qué sirve?

- Dar acceso a consultores, proveedores o clientes a una suscripción, un grupo de recursos, Teams o SharePoint.
- No gestionar contraseñas de externos: se autentican en su propio proveedor.
- Aplicar a los invitados MFA y acceso condicional de tu tenant.

## Conceptos clave

- **Invitación**: se envía un correo con enlace de canje (redemption). Hasta que el invitado lo acepta, el estado es "PendingAcceptance". También se puede dar acceso directo sin correo (canje al primer inicio de sesión).
- **Usuario invitado (Guest)**: `userType = Guest`. UPN con formato `nombre_dominio.com#EXT#@tenant.onmicrosoft.com`.
- **Origen**: External Microsoft Entra ID, Microsoft account, Google, correo con código de acceso único (OTP), SAML/WS-Fed federado.
- **Configuración de colaboración externa** (External collaboration settings): quién puede invitar (administradores y Guest Inviter / todos los miembros / invitados también / nadie), **restricciones de dominio** (lista de permitidos o bloqueados), permisos de invitado (limitados por defecto, se pueden restringir más).
- **Cross-tenant access settings**: confianza de MFA y de dispositivo con tenants concretos, y accesos entrantes/salientes por tenant.
- **Guest Inviter**: rol de Entra que solo puede invitar.
- **Licencias**: los invitados no consumen licencia para funciones básicas; las funciones premium se facturan por MAU (los primeros 50 000 al mes son gratuitos).
- **Revisiones de acceso** (P2 / Governance): revisar periódicamente qué invitados siguen necesitando acceso.

## Cómo funciona

```
1. Administrador (o miembro autorizado) → Usuarios → Nuevo usuario → Invitar usuario externo
2. Entra crea el objeto Guest y envía el correo (opcional)
3. El invitado canjea la invitación con su identidad (Entra, MSA, Google, OTP)
4. Se le asignan grupos, roles RBAC, aplicaciones como a cualquier usuario
5. Acceso condicional y MFA del tenant anfitrión se le aplican
```

```bash
az ad user invite ...  # no existe en CLI; se usa Graph
```

```powershell
Connect-MgGraph -Scopes "User.Invite.All"
New-MgInvitation -InvitedUserEmailAddress "pepe@partner.com" -InviteRedirectUrl "https://portal.azure.com" -SendInvitationMessage
```

## Componentes / configuración relevante para el examen

| Configuración | Dónde | Para qué |
|---|---|---|
| Quién puede invitar | Identidades externas → Configuración de colaboración externa | Restringir invitaciones a administradores |
| Restricciones de colaboración | Misma pantalla | Permitir solo dominios concretos (allow list) o bloquear (deny list). No ambos a la vez |
| Permisos de invitado | Misma pantalla | Limitar lo que ven del directorio |
| Cross-tenant access | Identidades externas → Configuración de acceso entre inquilinos | Confiar MFA/dispositivo del tenant del invitado |
| Invitación masiva | Usuarios → Operaciones masivas → Invitar en bloque (CSV) | Muchos invitados |
| Reenviar invitación | Usuario → Reenviar invitación | Enlace caducado o perdido |
| Convertir Guest → Member | Usuario → propiedades → tipo | Cuando pasa a ser empleado |

- **Rol mínimo para invitar**: **Guest Inviter** (si la configuración lo permite, también miembros normales).
- Una invitación se puede canjear durante **90 días**.

## Ejemplo

Un consultor externo debe gestionar las VMs de un grupo de recursos durante tres meses. Se le invita como Guest, se añade al grupo "Consultores" que tiene Virtual Machine Contributor sobre el RG, y se configura una revisión de acceso trimestral. Al no renovarse, se le quita el acceso sin que nadie tenga que acordarse.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **B2B (Guest)** | Colaborar con otras organizaciones | Sin gestionar sus credenciales; MFA propio | Partners, proveedores, consultores |
| **Crear un usuario Member** | Persona externa tratada como interna | Control total | Contratistas de larga duración, necesitan licencias completas |
| **External ID para clientes (CIAM)** | Identidades de consumidores en apps propias | Registro autoservicio, branding | Aplicaciones para clientes finales (fuera del alcance de AZ-104) |
| **Cross-tenant sync** ➕ | Sincronizar usuarios entre tenants de la misma empresa | Automático | Fusiones, multi-tenant corporativo |

## AZ-104 Exam Tips

- ⭐ Invitado = **usa sus propias credenciales**, aparece como `userType = Guest`.
- 🧠 Rol mínimo para invitar: **Guest Inviter**.
- 🧠 Restricciones de dominio: lista de **permitidos o bloqueados**, no ambas.
- 🧠 Invitación válida **90 días**; se puede reenviar.
- 💻 Invitar individual (portal), en bloque (CSV) y con `New-MgInvitation`.
- 📌 Guest vs Member: se puede cambiar el tipo sin recrear el usuario.
- ⚠️ MFA: el invitado usa la MFA de **tu** tenant salvo que configures confianza cross-tenant.

## Errores comunes

- Pensar que un invitado necesita licencia de Entra P1 para funciones básicas.
- Bloquear todas las invitaciones a nivel de tenant y luego no entender por qué un Guest Inviter falla.
- Olvidar asignar el rol RBAC tras invitar: la invitación solo crea la identidad, no da acceso a recursos.

## Preguntas que podrían aparecer

**1.** Debes permitir que solo los usuarios del dominio `fabrikam.com` puedan ser invitados como externos a tu tenant. ¿Qué configuras?
- A) Un grupo dinámico · B) Restricciones de colaboración con lista de dominios permitidos · C) Acceso condicional · D) Cross-tenant sync

<details><summary>Respuesta</summary>

**B.** Las restricciones de colaboración externa permiten definir una allow list de dominios. Las demás opciones no controlan quién puede ser invitado.
</details>

**2.** Un usuario del departamento de compras necesita invitar a proveedores sin tener más permisos en el directorio. ¿Qué rol le asignas?
- A) User Administrator · B) Guest Inviter · C) Global Reader · D) Groups Administrator

<details><summary>Respuesta</summary>

**B.** Guest Inviter es el rol mínimo para invitar. A excede permisos; C y D no permiten invitar.
</details>

## Relacionado

- [[02 - Usuarios de Microsoft Entra ID]]
- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[Acceso de usuarios externos (Microsoft Entra External ID)]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
