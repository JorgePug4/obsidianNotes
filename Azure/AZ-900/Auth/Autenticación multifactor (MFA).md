---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Autenticación multifactor (MFA)

## Concepto

**MFA** es un método de autenticación que exige **dos o más factores de distinta categoría** para verificar la identidad. Una contraseña sola ya no basta.

Problema que resuelve: las contraseñas se filtran, se adivinan y se reutilizan. Si un atacante consigue tu contraseña, MFA lo detiene porque no tiene el segundo factor.

Para qué se usa: proteger cuentas de Azure, Microsoft 365 y aplicaciones conectadas a [[Microsoft Entra ID]].

## Características principales

Las **tres categorías de factores** (memorizar):

| Categoría | Descripción | Ejemplos |
|---|---|---|
| **Algo que sabes** | Conocimiento | Contraseña, PIN, respuesta de seguridad |
| **Algo que tienes** | Posesión | Teléfono con Microsoft Authenticator, llave FIDO2, tarjeta inteligente, código SMS |
| **Algo que eres** | Inherencia (biometría) | Huella dactilar, reconocimiento facial, iris |

Puntos clave:
- Los factores deben ser de **categorías distintas**. Dos contraseñas no son MFA.
- Métodos en Entra ID: **Microsoft Authenticator** (notificación push, código TOTP), **SMS**, **llamada de voz**, **llaves de seguridad FIDO2**, **Windows Hello for Business**, **tokens OATH**.
- Se puede activar por usuario, con **valores predeterminados de seguridad** (security defaults, gratis) o mediante [[Acceso condicional]] (P1/P2), que permite exigir MFA solo bajo ciertas condiciones.
- Microsoft exige MFA obligatoria para iniciar sesión en el **portal de Azure** y en las herramientas de administración desde 2024-2025.
- MFA es parte de "verificar explícitamente" en [[Confianza cero (Zero Trust)]].

> [!warning] Confusión frecuente
> - **2FA** es un caso particular de MFA (exactamente dos factores). En el examen se usan como sinónimos prácticos.
> - **MFA no es lo mismo que passwordless**. MFA añade factores a la contraseña; passwordless la elimina, aunque puede seguir siendo multifactor (dispositivo + biometría).
> - **SMS es MFA válido pero el menos seguro**. Microsoft recomienda Authenticator o FIDO2.

## Casos de uso

- Administrador global de Azure: contraseña + notificación push en el Authenticator.
- Empleado que entra a Outlook desde un país nuevo: Acceso condicional le pide MFA solo esa vez.
- Cajero automático: tarjeta (algo que tienes) + PIN (algo que sabes) es MFA de la vida real.

## Comparaciones

| Método | Categoría | Seguridad | Notas |
|---|---|---|---|
| Contraseña | Sabes | Baja sola | Base sobre la que se añade MFA |
| SMS / llamada | Tienes | Media | Vulnerable a SIM swapping; aún aceptado |
| Microsoft Authenticator | Tienes | Alta | Push, código TOTP, coincidencia de número |
| Llave FIDO2 | Tienes | Muy alta | Resistente a phishing; también sirve passwordless |
| Windows Hello for Business | Tienes + eres | Muy alta | Dispositivo + biometría/PIN; passwordless |

## Conceptos que debo memorizar

> [!important]
> - MFA = **dos o más factores de categorías diferentes**.
> - Categorías: **sabes · tienes · eres**.
> - Herramienta principal: **Microsoft Authenticator**.
> - Los **valores predeterminados de seguridad** activan MFA de forma gratuita para todos.
> - **Acceso condicional** permite exigir MFA según condiciones (requiere P1/P2).
> - Protege contra **contraseñas comprometidas**.

## Tips para AZ-900

> [!tip]
> - Si la pregunta muestra dos métodos y pregunta si es MFA, comprueba que sean **categorías distintas**: contraseña + PIN → NO (ambos "sabes"); contraseña + huella → SÍ.
> - Palabras clave: *second form of verification*, *something you know/have/are*, *reduce risk of compromised passwords*.
> - Trampa: "MFA requiere licencia P1" es falsa. La edición Free incluye MFA vía valores predeterminados de seguridad. Lo que requiere P1 es el Acceso condicional granular.
> - Trampa: "Passwordless es lo mismo que MFA" es falsa.
> - Si la pregunta dice "sin comprar licencias adicionales, activar MFA para todos" → **valores predeterminados de seguridad**.

## Ejemplo de pregunta de examen

**Pregunta 1.** ¿Cuál de las siguientes combinaciones constituye autenticación multifactor?
- A) Contraseña y PIN
- B) Contraseña y notificación en Microsoft Authenticator
- C) Dos preguntas de seguridad
- D) Nombre de usuario y contraseña

**Respuesta: B.** Contraseña (algo que sabes) + teléfono (algo que tienes) son dos categorías. A y C usan dos factores de la misma categoría (conocimiento). D es un solo factor.

**Pregunta 2.** Una empresa con la edición Free de Microsoft Entra ID quiere exigir MFA a todos sus usuarios sin comprar licencias. ¿Qué debe hacer?
- A) Configurar directivas de Acceso condicional
- B) Habilitar los valores predeterminados de seguridad
- C) Implementar Microsoft Entra Domain Services
- D) Comprar Entra ID P2

**Respuesta: B.** Los valores predeterminados de seguridad son gratuitos y fuerzan MFA para todos. A requiere P1/P2. C no tiene relación con MFA. D contradice el requisito de no comprar licencias.

**Pregunta 3.** ¿A qué categoría de factor pertenece una llave de seguridad FIDO2?
- A) Algo que sabes
- B) Algo que tienes
- C) Algo que eres
- D) Algo que haces

**Respuesta: B.** Una llave física es un factor de posesión. A es conocimiento (contraseña), C es biometría, D no es una categoría estándar en AZ-900.

## 🧠 Resumen para el examen

1. MFA = 2+ factores de categorías distintas.
2. Categorías: algo que sabes, tienes, eres.
3. Microsoft Authenticator es la app recomendada.
4. MFA gratis para todos con valores predeterminados de seguridad.
5. MFA selectiva por condiciones → Acceso condicional (P1/P2).
6. FIDO2 y Windows Hello son los métodos más seguros.
7. MFA ≠ passwordless, aunque se complementan.
8. Mitiga el riesgo de contraseñas robadas o filtradas.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
