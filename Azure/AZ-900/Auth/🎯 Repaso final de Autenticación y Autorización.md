---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# 🎯 Repaso final de Autenticación y Autorización

## Los conceptos más importantes

- **Autenticación** (quién eres) ocurre antes que **autorización** (qué puedes hacer). MFA, passwordless y SSO son autenticación; RBAC es autorización.
- **Microsoft Entra ID** es el servicio de identidad en la nube (Azure AD renombrado). Gestiona usuarios, grupos, dispositivos y apps. Un tenant por suscripción.
- **Entra Connect** sincroniza AD local → Entra ID (identidad híbrida).
- **Entra Domain Services** entrega Kerberos, LDAP y GPO como servicio administrado para apps legacy.
- **Entra External ID** permite invitar socios (B2B, guest) y atender clientes (antes B2C).
- **Confianza cero**: verificar explícitamente, privilegio mínimo, asumir la brecha.
- **MFA** exige dos factores de categorías distintas (sabes, tienes, eres).
- **Acceso condicional** aplica reglas "si… entonces…" con señales; requiere P1.
- **Passwordless**: Windows Hello for Business, Authenticator, FIDO2.
- **SSO**: una autenticación, muchas aplicaciones.

## Tabla de servicios y su propósito

| Servicio / concepto | Propósito en una frase | Palabra clave de examen |
|---|---|---|
| **Microsoft Entra ID** | Identidad y acceso en la nube para Azure, M365 y SaaS | *cloud identity, tenant, directory* |
| **Microsoft Entra Connect** | Sincronizar AD local hacia Entra ID | *hybrid identity, synchronize on-premises* |
| **Microsoft Entra Domain Services** | Dominio administrado con Kerberos, LDAP, GPO | *domain join, group policy, legacy apps, no domain controllers* |
| **Microsoft Entra External ID (B2B)** | Invitar usuarios de otras organizaciones como guest | *partner, vendor, guest, invite* |
| **Microsoft Entra External ID (clientes / B2C)** | Identidad de consumidores con cuentas sociales | *customers, social accounts, public app* |
| **MFA** | Dos o más factores de categorías distintas | *something you know/have/are* |
| **Valores predeterminados de seguridad** | MFA para todos, gratis, sin personalizar | *no additional licenses, all users* |
| **Acceso condicional** | Directivas "si… entonces…" basadas en señales | *based on location/device, require MFA when, P1* |
| **Passwordless** | Autenticación sin contraseña (Hello, Authenticator, FIDO2) | *eliminate passwords, biometric, FIDO2* |
| **SSO** | Un inicio de sesión para múltiples aplicaciones | *sign in once, multiple apps* |
| **Confianza cero** | Modelo: nunca confíes, siempre verifica | *verify explicitly, least privilege, assume breach* |
| **Azure RBAC** | Autorización: qué puede hacer cada identidad sobre recursos | *role, permission, scope* |

## Las diferencias que más fácilmente puedo confundir

> [!warning] Pares de confusión clásicos
> | Confusión | Cómo distinguirlos |
> |---|---|
> | **Entra ID vs AD DS local** | Entra ID: nube, OAuth/SAML, sin GPO. AD DS: local, Kerberos/LDAP, con GPO. |
> | **Entra ID vs Entra Domain Services** | Entra ID: identidad moderna. Domain Services: AD clásico administrado (Kerberos, GPO, unión a dominio). |
> | **Entra Connect vs Entra Domain Services** | Connect: sincroniza local → nube. Domain Services: crea dominio en la nube. |
> | **Entra Domain Services vs AD DS en VM** | Domain Services: Microsoft administra (PaaS). VM: tú administras (IaaS), control total. |
> | **MFA vs Acceso condicional** | MFA: método. Acceso condicional: motor que decide cuándo exigirlo. |
> | **MFA vs Passwordless** | MFA: contraseña + factor. Passwordless: sin contraseña, dispositivo + biometría/PIN. |
> | **Valores predeterminados vs Acceso condicional** | Predeterminados: gratis, fijos, todos. Acceso condicional: P1, personalizable. |
> | **Acceso condicional vs RBAC** | Acceso condicional: si puedes entrar y cómo. RBAC: qué puedes hacer dentro. |
> | **B2B vs B2C** | B2B: socios como guest en tu tenant. B2C: consumidores en tenant separado. |
> | **SSO vs Entra Connect** | SSO: una vez para todas las apps. Connect: mismas credenciales local y nube. |
> | **Autenticación vs Autorización** | ¿Quién eres? vs ¿Qué puedes hacer? |
> | **2FA vs MFA** | 2FA = exactamente 2 factores; MFA = 2 o más. En la práctica, sinónimos. |
> | **PIN de Windows Hello vs contraseña** | PIN ligado al dispositivo, no viaja por red. Contraseña se envía al servidor. |

## 10 tips de examen

> [!tip]
> 1. Ante nombres antiguos (Azure AD, Azure AD DS, B2B/B2C), traduce mentalmente a Entra ID, Entra Domain Services, External ID.
> 2. "Kerberos", "LDAP", "GPO", "unir a dominio" → **Entra Domain Services**. Nunca Entra ID.
> 3. "Sincronizar usuarios locales" → **Entra Connect**. "Sin administrar controladores" → **Domain Services**.
> 4. "Socio/proveedor con su propia cuenta" → **B2B guest**. "Clientes con Google/Facebook" → **External ID para clientes**.
> 5. Para saber si algo es MFA, comprueba que los factores sean de **categorías distintas**.
> 6. "MFA para todos sin licencias" → **valores predeterminados de seguridad**. "MFA solo cuando…" → **Acceso condicional (P1)**.
> 7. SMS y llamada de voz son MFA pero **no** passwordless. Windows Hello, Authenticator (inicio por teléfono) y FIDO2 sí.
> 8. Un principio de Confianza cero siempre será: verificar explícitamente, privilegio mínimo o asumir la brecha. Descarta cualquier opción que "confíe en la red interna".
> 9. "Inicia sesión bien pero no puede hacer X" → problema de **autorización (RBAC)**, no de autenticación.
> 10. SSO **mejora** la seguridad (menos contraseñas, baja centralizada) si la identidad única lleva MFA. No caigas en el distractor de "una credencial es más riesgo".

## 10 preguntas de repaso tipo AZ-900

**1.** Una organización con Active Directory local quiere que sus usuarios usen las mismas credenciales en Microsoft 365. ¿Qué debe implementar?
- A) Microsoft Entra Domain Services
- B) Microsoft Entra Connect
- C) Microsoft Entra External ID
- D) Acceso condicional

**Respuesta: B.** Entra Connect sincroniza las identidades locales con Entra ID. A crea un dominio administrado; C es para externos; D aplica directivas de acceso.

**2.** ¿Cuál de los siguientes describe la autenticación?
- A) Conceder permiso de lectura sobre una cuenta de almacenamiento
- B) Verificar la identidad de un usuario mediante contraseña y MFA
- C) Asignar el rol Propietario a un grupo
- D) Definir el ámbito de una asignación de rol

**Respuesta: B.** Verificar identidad es autenticación. A, C y D son autorización (RBAC).

**3.** Una empresa quiere bloquear el acceso al portal de Azure desde países en los que no opera. ¿Qué debe configurar?
- A) Valores predeterminados de seguridad
- B) Microsoft Entra Connect
- C) Acceso condicional
- D) Windows Hello for Business

**Respuesta: C.** Bloquear por ubicación es una directiva de Acceso condicional. A no permite condiciones; B sincroniza identidades; D es un método passwordless.

**4.** ¿Qué principio de Confianza cero recomienda otorgar solo el acceso necesario durante el tiempo necesario?
- A) Verificar explícitamente
- B) Asumir la brecha
- C) Privilegio mínimo
- D) Defensa en profundidad

**Respuesta: C.** Privilegio mínimo limita el acceso a lo estrictamente necesario. A trata de verificar cada solicitud; B de limitar el daño; D es otro modelo de seguridad, no un principio de Confianza cero.

**5.** ¿Cuál de las siguientes opciones es un ejemplo de "algo que eres"?
- A) Contraseña
- B) Llave FIDO2
- C) Reconocimiento facial
- D) Código por SMS

**Respuesta: C.** La biometría es inherencia. A es conocimiento; B y D son posesión.

**6.** Una aplicación heredada en una VM de Azure requiere LDAP y autenticación NTLM. La empresa no quiere administrar controladores de dominio. ¿Qué servicio debe usar?
- A) Microsoft Entra ID
- B) Microsoft Entra Domain Services
- C) Azure Key Vault
- D) Microsoft Entra External ID

**Respuesta: B.** Domain Services ofrece LDAP y NTLM administrados. A no soporta esos protocolos; C guarda secretos; D es para usuarios externos.

**7.** ¿Cuál de los siguientes métodos es passwordless?
- A) Contraseña + código SMS
- B) Contraseña + llamada de voz
- C) Microsoft Authenticator con inicio de sesión por teléfono
- D) Contraseña + pregunta de seguridad

**Respuesta: C.** El inicio de sesión por teléfono reemplaza la contraseña. A, B y D siguen usando contraseña.

**8.** ¿Qué tipo de usuario se crea en tu tenant cuando invitas a un empleado de otra empresa mediante B2B?
- A) Member
- B) Guest
- C) Administrador global
- D) Entidad de servicio

**Respuesta: B.** Los invitados B2B tienen tipo Guest. A es para empleados propios; C es un rol, no un tipo; D es una identidad de aplicación.

**9.** ¿Cuál de las siguientes afirmaciones sobre Microsoft Entra ID es correcta?
- A) Requiere controladores de dominio administrados por el cliente
- B) Soporta directivas de grupo (GPO)
- C) Proporciona SSO para aplicaciones SaaS de terceros
- D) Solo está disponible con Microsoft Entra ID P2

**Respuesta: C.** SSO a SaaS es una capacidad central de Entra ID. A y B describen AD DS o Domain Services; D es falsa, existe edición Free.

**10.** Una empresa quiere aplicar MFA a todos los usuarios de inmediato, sin comprar licencias adicionales ni definir condiciones. ¿Qué debe habilitar?
- A) Acceso condicional
- B) Valores predeterminados de seguridad
- C) Microsoft Entra Domain Services
- D) Colaboración B2B

**Respuesta: B.** Los valores predeterminados de seguridad son gratuitos y aplican MFA a todos. A requiere P1; C y D no tienen relación con MFA.

---

> [!tip] Cómo usar estas notas
> Repasa primero la tabla de servicios y la de confusiones del repaso final. Después, responde las 10 preguntas sin mirar. Cualquier fallo te dice a qué sección volver.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
