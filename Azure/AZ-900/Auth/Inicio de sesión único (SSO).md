---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Inicio de sesión único (SSO)

## Concepto

**SSO (Single Sign-On)** permite que un usuario se autentique **una sola vez** y acceda a **múltiples aplicaciones y servicios** sin volver a introducir credenciales.

Problema que resuelve: sin SSO, cada aplicación tiene su propio usuario y contraseña. El usuario acaba con decenas de contraseñas (las reutiliza o las anota), y TI pierde el control de accesos cuando alguien deja la empresa.

Para qué se usa: simplificar la experiencia, reducir contraseñas y **centralizar** la administración de identidades en [[Microsoft Entra ID]].

## Características principales

- Se autentica una vez contra el **proveedor de identidad** (Entra ID) y este emite **tokens** que las aplicaciones aceptan.
- Funciona con Microsoft 365, Azure, Dynamics y **miles de apps SaaS de terceros** (Salesforce, ServiceNow, Google Workspace, etc.) a través de la galería de aplicaciones de Entra.
- Protocolos: **SAML, OpenID Connect, OAuth 2.0** (solo reconocerlos).
- Beneficios de seguridad: **una sola identidad** que proteger con MFA y Acceso condicional; **un solo lugar** para dar de baja al usuario cuando sale de la empresa.
- SSO **no reduce** la seguridad si la identidad única está bien protegida; la **concentra**.
- Disponible en la edición **Free** de Entra ID (con límite de apps por usuario en Free; ilimitado en P1/P2, detalle no crítico para AZ-900).

> [!warning] Confusión frecuente
> - **SSO ≠ misma contraseña en todas partes**. SSO significa autenticarse **una sola vez**; la app no ve tu contraseña, recibe un token.
> - **SSO ≠ MFA**. SSO reduce el número de inicios de sesión; MFA refuerza cada inicio. Se combinan: un inicio de sesión fuerte (MFA) que da acceso a todo (SSO).
> - **SSO ≠ Entra Connect**. Connect permite que las credenciales locales funcionen en la nube (identidad híbrida); SSO es la experiencia de "una vez y listo" para las apps.

## Casos de uso

- El usuario entra a Outlook por la mañana y luego abre Teams, SharePoint, Salesforce y Azure sin volver a escribir su contraseña.
- Un empleado renuncia: TI deshabilita su cuenta en Entra ID y pierde acceso a todas las apps al instante.
- Una empresa reduce las llamadas al soporte por contraseñas olvidadas al pasar de 15 contraseñas a una.

## Comparaciones

| | Sin SSO | Con SSO |
|---|---|---|
| Credenciales | Una por aplicación | Una única identidad |
| Inicios de sesión | Uno por app | Uno para todas |
| Baja de usuario | Hay que revisar cada app | Se deshabilita en Entra ID |
| Riesgo | Contraseñas reutilizadas y débiles | Una identidad fuerte protegida con MFA |
| Experiencia | Fricción constante | Fluida |

## Conceptos que debo memorizar

> [!important]
> - SSO = **una autenticación, muchas aplicaciones**.
> - Lo proporciona **Microsoft Entra ID** como proveedor de identidad.
> - Beneficios: **menos contraseñas, mejor experiencia, administración centralizada, baja inmediata**.
> - Funciona con **M365, Azure y apps SaaS de terceros**.
> - La seguridad depende de proteger **esa única identidad** (MFA).

## Tips para AZ-900

> [!tip]
> - Palabras clave: *sign in once*, *access multiple applications*, *reduce the number of passwords*, *third-party SaaS apps with corporate credentials*.
> - Trampa: "SSO disminuye la seguridad porque una sola credencial da acceso a todo". El examen espera que reconozcas que SSO **mejora** la seguridad al reducir contraseñas y centralizar el control, siempre que la identidad se proteja con MFA.
> - Trampa: confundir SSO con la sincronización de contraseñas de Entra Connect. Son cosas distintas que suelen coexistir.
> - Relación: SSO + MFA + Acceso condicional = combinación típica en Confianza cero.

## Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa quiere que sus empleados accedan a Microsoft 365, Salesforce y su app interna después de iniciar sesión una sola vez. ¿Qué capacidad de Microsoft Entra ID cumple este requisito?
- A) Autenticación multifactor
- B) Inicio de sesión único (SSO)
- C) Microsoft Entra Domain Services
- D) Acceso condicional

**Respuesta: B.** SSO permite un inicio de sesión para múltiples apps. A refuerza el inicio de sesión pero no elimina los repetidos. C es un dominio administrado. D decide bajo qué condiciones se accede, no elimina inicios de sesión.

**Pregunta 2.** ¿Cuál es un beneficio de seguridad del inicio de sesión único?
- A) Cada aplicación almacena la contraseña del usuario para mayor redundancia
- B) Al deshabilitar la cuenta en Entra ID, el usuario pierde acceso a todas las aplicaciones conectadas
- C) Elimina la necesidad de MFA
- D) Permite que los usuarios compartan credenciales de forma segura

**Respuesta: B.** La administración centralizada es la ventaja de seguridad de SSO. A es lo contrario de SSO. C es falsa: MFA sigue siendo recomendable. D nunca es una práctica segura.

## 🧠 Resumen para el examen

1. SSO = iniciar sesión una vez, acceder a muchas apps.
2. Entra ID actúa como proveedor de identidad y emite tokens.
3. Funciona con M365, Azure y miles de apps SaaS.
4. Beneficios: menos contraseñas, mejor experiencia, control centralizado.
5. Baja de usuario en un solo lugar.
6. Protege la identidad única con MFA.
7. SSO ≠ MFA ≠ Entra Connect; se complementan.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
