---
tags: [az-104, azure, identidad, entra-id]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Microsoft Entra ID para administradores

## ¿Qué es?

**Microsoft Entra ID** (antes Azure Active Directory) es el servicio de identidad y acceso en la nube de Microsoft. Cada organización tiene un **tenant** (inquilino, también llamado directorio): una instancia aislada que contiene usuarios, grupos, dispositivos, aplicaciones y la configuración de autenticación.

En AZ-900 bastaba saber qué es. En AZ-104 debes **administrarlo**: crear objetos, asignar licencias, invitar externos, configurar SSPR y entender qué rol de directorio hace falta para cada tarea. Si necesitas repasar los fundamentos, ver [[Microsoft Entra ID]] (AZ-900).

## ¿Para qué sirve?

- Autenticar a los usuarios que entran al portal de Azure, Microsoft 365 y aplicaciones SaaS.
- Ser la fuente de identidades sobre la que se asignan roles de [[08 - Azure RBAC - roles integrados y ámbitos|Azure RBAC]].
- Aplicar seguridad de identidad: MFA, acceso condicional, protección de contraseñas.

## Conceptos clave

- **Tenant / directorio**: la organización. Tiene un dominio inicial `<nombre>.onmicrosoft.com` y puede añadir **dominios personalizados** verificados (registro TXT o MX en el DNS público).
- **Suscripción ↔ tenant**: una suscripción de Azure **confía en un solo tenant**; un tenant puede tener muchas suscripciones. Una suscripción se puede **transferir a otro tenant** (se pierden las asignaciones RBAC).
- **Ediciones**: **Free** (incluida), **P1** (grupos dinámicos, acceso condicional, SSPR con writeback, licencias por grupo, AUs) y **P2** (P1 + Identity Protection, PIM, revisiones de acceso). Existe también **Microsoft Entra ID Governance** como complemento.
- **Roles de Microsoft Entra** (roles de directorio): Global Administrator, User Administrator, Groups Administrator, License Administrator, Password Administrator, Helpdesk Administrator, Application Administrator, Billing Administrator, etc. Gestionan el **directorio**, no los recursos de Azure.
- **Objetos**: usuarios (miembro/invitado), grupos, dispositivos, aplicaciones y entidades de servicio, identidades administradas.

## Cómo funciona

```
Tenant contoso.onmicrosoft.com  (+ dominio personalizado contoso.com)
 ├── Usuarios (miembros, invitados, sincronizados desde AD DS)
 ├── Grupos (seguridad, Microsoft 365; asignados o dinámicos)
 ├── Dispositivos (registrados, unidos a Entra, híbridos)
 ├── Aplicaciones / entidades de servicio / identidades administradas
 ├── Roles de Entra (directorio)
 └── Suscripciones de Azure que confían en este tenant
        └── RBAC (Owner, Contributor…) sobre recursos
```

Autenticación con protocolos web modernos (OAuth 2.0, OpenID Connect, SAML). No hay Kerberos, LDAP ni GPO: para eso existe [[Microsoft Entra Domain Services]] (AZ-900) o un AD DS local sincronizado con [[17 - Identidad híbrida - Microsoft Entra Connect]].

## Componentes que administra un AZ-104

| Componente | Dónde se administra | Nota |
|---|---|---|
| Usuarios | Entra admin center / portal → Microsoft Entra ID → Usuarios | [[02 - Usuarios de Microsoft Entra ID]] |
| Grupos | → Grupos | [[03 - Grupos de Microsoft Entra ID]] |
| Licencias | → Licencias, o en el centro de administración de Microsoft 365 | [[04 - Licencias en Microsoft Entra ID]] |
| Identidades externas | → Identidades externas | [[05 - Usuarios externos e invitados (B2B)]] |
| Restablecimiento de contraseña | → Protección → Restablecimiento de contraseña | [[06 - Self-Service Password Reset (SSPR)]] |
| Unidades administrativas, dispositivos | → Roles y administradores / Dispositivos | [[07 - Unidades administrativas y dispositivos]] |
| Dominios personalizados | → Nombres de dominio personalizados | Verificación por registro TXT/MX |

## Configuración relevante para el examen

- **Añadir dominio personalizado**: agregar dominio → copiar registro TXT → crearlo en el registrador DNS → verificar. Hasta que se verifica, no se puede usar como sufijo de UPN.
- **Rol mínimo para tareas de directorio** (principio de privilegio mínimo):

| Tarea | Rol de Entra mínimo |
|---|---|
| Crear/borrar usuarios, restablecer contraseñas de no administradores | **User Administrator** |
| Solo restablecer contraseñas | **Password Administrator** / Helpdesk Administrator |
| Crear y administrar grupos | **Groups Administrator** |
| Asignar licencias | **License Administrator** |
| Invitar usuarios externos | **Guest Inviter** (o User Administrator) |
| Todo, incluidos otros administradores | **Global Administrator** |

- **Global Administrator ≠ Owner de suscripción**. Un Global Administrator puede **elevar su acceso** ("Access management for Azure resources") para convertirse en User Access Administrator en el ámbito raíz `/` y así ver todas las suscripciones del tenant. Es una acción puntual y auditada.

## Ejemplo

Contoso compra una suscripción nueva. El Global Administrator del tenant no ve la suscripción en el portal. Causa: los roles de Entra no dan acceso a recursos. Solución: que el propietario de la suscripción le asigne Owner/Reader, o que el Global Administrator eleve su acceso al ámbito raíz.

## Comparaciones

| Servicio | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Microsoft Entra ID** | Identidad en la nube (usuarios, apps, SSO, MFA) | Sin servidores, protocolos web, integración con Azure y M365 | Siempre; es la base |
| **Microsoft Entra Domain Services** | Dominio administrado (Kerberos, LDAP, GPO) | Sin controladores de dominio propios | VMs o apps legacy que necesitan unión a dominio en Azure |
| **AD DS local (Windows Server)** | Directorio on-premises | Control total, GPO completo | Entornos locales; se sincroniza con Entra Connect |
| **Roles de Entra ID** | Administrar el directorio | Granularidad por tarea | Gestionar usuarios, grupos, licencias, dominios |
| **Azure RBAC** | Administrar recursos de Azure | Ámbitos jerárquicos | Dar acceso a VMs, redes, storage |

## AZ-104 Exam Tips

- ⭐ **Una suscripción → un tenant; un tenant → N suscripciones.**
- 🔥 📌 **Roles de Entra (directorio) vs roles de Azure RBAC (recursos)**: un Global Administrator no puede crear VMs sin un rol RBAC; un Owner de suscripción no puede crear usuarios sin un rol de Entra.
- 🧠 Ediciones: **P1** → grupos dinámicos, acceso condicional, licencias por grupo, SSPR con writeback, unidades administrativas. **P2** → PIM, Identity Protection, revisiones de acceso.
- 🧠 Verificar dominio personalizado = registro **TXT** (o MX) en el DNS público.
- 💻 Saber elevar acceso de Global Administrator al ámbito raíz y saber dónde se asigna cada rol de Entra.
- ⚠️ "Azure AD", "AAD" y "Microsoft Entra ID" son lo mismo. El examen usa el nombre nuevo.

## Errores comunes

- Dar Global Administrator para una tarea de helpdesk; el examen siempre quiere el **rol mínimo**.
- Creer que transferir una suscripción a otro tenant conserva los roles RBAC: **se borran** y hay que reasignarlos.
- Confundir "unir un dispositivo a Entra ID" con "unir una VM a un dominio" (esto último es Entra Domain Services o AD DS).

## Preguntas que podrían aparecer

**1.** Un usuario con el rol Global Administrator no puede ver una suscripción de Azure que pertenece a su tenant. ¿Cuál es la forma más rápida de que vea todas las suscripciones?
- A) Asignarle Owner en cada suscripción · B) Activar "Access management for Azure resources" y elevar su acceso · C) Crear un grupo de administración · D) Transferir la suscripción

<details><summary>Respuesta</summary>

**B.** La elevación de acceso convierte al Global Administrator en User Access Administrator en el ámbito raíz, desde donde ve y puede asignar roles en todas las suscripciones. A funciona pero requiere que alguien con Owner lo haga en cada una. C y D no resuelven el acceso.
</details>

**2.** Debes permitir que el equipo de soporte restablezca contraseñas de usuarios estándar, sin poder crear ni borrar usuarios. ¿Qué rol asignas?
- A) User Administrator · B) Global Administrator · C) Password Administrator · D) Security Administrator

<details><summary>Respuesta</summary>

**C.** Password Administrator (o Helpdesk Administrator) solo restablece contraseñas de usuarios no administradores. A también puede crear/borrar usuarios, excede el mínimo.
</details>

## Relacionado

- [[02 - Usuarios de Microsoft Entra ID]]
- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[17 - Identidad híbrida - Microsoft Entra Connect]]
- [[Microsoft Entra ID]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
