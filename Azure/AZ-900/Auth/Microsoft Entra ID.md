---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Microsoft Entra ID

## Concepto

**Microsoft Entra ID** (antes Azure Active Directory) es el servicio de **gestión de identidades y acceso basado en la nube** de Microsoft. Es el directorio que almacena usuarios, grupos, dispositivos y aplicaciones, y el que autentica a esos usuarios cuando acceden a Azure, Microsoft 365, Dynamics 365 o aplicaciones SaaS de terceros.

Problema que resuelve: antes cada aplicación tenía su propia base de usuarios. Entra ID centraliza la identidad en un solo lugar accesible desde internet, sin necesidad de servidores propios.

Para qué se usa:
- Iniciar sesión en el portal de Azure y en Microsoft 365.
- Controlar quién accede a qué aplicaciones.
- Aplicar [[Autenticación multifactor (MFA)|MFA]], [[Acceso condicional]] y [[Inicio de sesión único (SSO)|SSO]].
- Detectar inicios de sesión sospechosos.

## Características principales

- **Tenant (inquilino)**: instancia dedicada de Entra ID que representa a una organización. Cada suscripción de Azure está asociada a **un** tenant; un tenant puede tener **varias** suscripciones.
- Identidades que gestiona: **usuarios**, **grupos**, **dispositivos**, **aplicaciones** y **entidades de servicio** (identidades para apps y automatizaciones).
- Es un servicio **PaaS / SaaS**: Microsoft lo opera; tú no administras servidores.
- Funciona con protocolos web modernos (**OAuth 2.0, OpenID Connect, SAML**), no con los protocolos del AD tradicional.
- Ofrece **SSO**, **MFA**, **Acceso condicional**, **B2B/External ID**, informes de inicio de sesión y autoservicio de restablecimiento de contraseña (SSPR).
- Tiene ediciones: **Free** (incluida con cualquier suscripción de Azure o Microsoft 365), **P1** y **P2** (con [[Acceso condicional]] completo, Identity Protection, PIM). Para AZ-900 basta saber que existen niveles de pago con más funciones.

> [!warning] Entra ID NO es Active Directory en la nube
> Active Directory Domain Services (AD DS) es el directorio **local** de Windows Server: usa **Kerberos, NTLM, LDAP** y **directivas de grupo (GPO)**, y requiere **controladores de dominio**. Entra ID es un servicio de identidad web: no tiene GPO, no tiene unidades organizativas, no se "une" un servidor a él con Kerberos. Son productos distintos con propósitos distintos. Esta diferencia aparece en el examen.

## Casos de uso

- Una empresa nueva, sin servidores, quiere que sus empleados entren a Microsoft 365 y Azure con una sola cuenta → Entra ID.
- Una empresa quiere que Salesforce y Dropbox usen las credenciales corporativas → Entra ID con SSO.
- El área de TI quiere ver desde dónde se inició sesión cada usuario → informes de Entra ID.

## Comparaciones

| | Active Directory DS (local) | Microsoft Entra ID (nube) | Microsoft Entra Domain Services |
|---|---|---|---|
| Dónde vive | Servidores propios (on-premises) | Nube de Microsoft | Nube (Azure), administrado por Microsoft |
| Tú administras servidores | Sí (controladores de dominio) | No | No |
| Protocolos | Kerberos, NTLM, LDAP | OAuth 2.0, OpenID Connect, SAML | Kerberos, NTLM, LDAP |
| Directivas de grupo (GPO) | Sí | No | Sí |
| Unir VMs a dominio | Sí | No (registro de dispositivo, no unión clásica) | Sí |
| Uso típico | Red corporativa tradicional | Apps web, M365, Azure, SaaS | Migrar apps legacy a Azure sin controladores de dominio |

## Conceptos que debo memorizar

> [!important]
> - **Entra ID = identidad en la nube** para Azure, Microsoft 365 y apps SaaS.
> - **Tenant** = la organización. Una suscripción → un tenant; un tenant → muchas suscripciones.
> - Gestiona **usuarios, grupos, dispositivos, aplicaciones**.
> - Ofrece **SSO, MFA, Acceso condicional, identidades externas, SSPR**.
> - **No** es un reemplazo del AD local ni usa Kerberos/GPO.
> - Edición **Free** incluida; P1/P2 añaden funciones avanzadas.

## Tips para AZ-900

> [!tip]
> - Si la pregunta dice "usuarios de Azure, Microsoft 365 y aplicaciones SaaS" → **Entra ID**.
> - Si dice "directivas de grupo, Kerberos, LDAP, unir VM a dominio, sin administrar controladores" → **[[Microsoft Entra Domain Services]]**.
> - Si dice "sincronizar usuarios locales con la nube" → **[[Microsoft Entra Connect]]**.
> - Trampa: "Azure Active Directory" y "Microsoft Entra ID" son **el mismo servicio**.
> - Trampa: Entra ID no es gratuito solo con Microsoft 365; la edición Free viene con **cualquier** suscripción de Azure o de un servicio Microsoft en la nube.
> - Palabras clave: *identity provider*, *cloud-based identity*, *tenant*, *directory*.

## Ejemplo de pregunta de examen

**Pregunta 1.** Una organización necesita un servicio de identidad en la nube para que sus empleados inicien sesión en Microsoft 365 y en varias aplicaciones SaaS con una sola cuenta. ¿Qué servicio debe usar?
- A) Active Directory Domain Services
- B) Microsoft Entra ID
- C) Microsoft Entra Domain Services
- D) Azure Key Vault

**Respuesta: B.** Entra ID es el proveedor de identidad en la nube para M365 y SaaS. A es el directorio local de Windows Server. C ofrece servicios de dominio administrados (Kerberos/LDAP) para VMs, no SSO a SaaS. D almacena secretos y claves, no identidades.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre Microsoft Entra ID es verdadera?
- A) Requiere que la organización administre controladores de dominio en Azure
- B) Soporta directivas de grupo (GPO) igual que Active Directory local
- C) Una suscripción de Azure se asocia a un único tenant de Entra ID
- D) Solo está disponible con licencias de pago P1 o P2

**Respuesta: C.** Cada suscripción confía en un solo tenant. A es falsa (Microsoft lo opera). B es falsa (no hay GPO en Entra ID). D es falsa (existe edición Free).

**Pregunta 3.** ¿Cuál de los siguientes NO es un tipo de identidad que administra Microsoft Entra ID?
- A) Usuarios
- B) Grupos
- C) Dispositivos
- D) Máquinas virtuales locales unidas con Kerberos

**Respuesta: D.** La unión clásica de dominio con Kerberos corresponde a AD DS o Entra Domain Services. A, B y C son objetos estándar de Entra ID.

## 🧠 Resumen para el examen

1. Entra ID = Azure AD renombrado. Identidad y acceso en la nube.
2. Es el directorio de Azure, Microsoft 365 y apps SaaS.
3. Tenant = organización. Suscripción → 1 tenant; tenant → N suscripciones.
4. Objetos: usuarios, grupos, dispositivos, aplicaciones.
5. Funciones clave: SSO, MFA, Acceso condicional, External ID, SSPR.
6. No es AD local: sin Kerberos, sin GPO, sin controladores de dominio.
7. Edición Free incluida; P1/P2 desbloquean funciones avanzadas.
8. Microsoft opera el servicio; tú no administras servidores.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
