---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Autenticación sin contraseña (Passwordless)

## Concepto

**Passwordless** es un método de autenticación que **elimina la contraseña** por completo. En su lugar, el usuario demuestra su identidad con **algo que tiene** (dispositivo registrado, llave física) combinado con **algo que es** (biometría) o **algo que sabe** (PIN local del dispositivo).

Problema que resuelve: las contraseñas son el eslabón más débil (phishing, reutilización, fuerza bruta, olvido). Sin contraseña no hay nada que robar ni adivinar.

Para qué se usa: dar la **mayor seguridad con la mejor experiencia de usuario**. Microsoft lo posiciona como el objetivo final de la estrategia de autenticación.

## Características principales

Los **tres métodos passwordless** que evalúa AZ-900:

| Método | Cómo funciona | Escenario |
|---|---|---|
| **Windows Hello for Business** | Biometría (rostro, huella) o PIN vinculado al dispositivo Windows. El PIN nunca sale del equipo. | Empleados con PC Windows corporativo |
| **Microsoft Authenticator** | El teléfono se convierte en credencial. El usuario recibe un número en pantalla, lo confirma en la app y valida con biometría/PIN del móvil. | Cualquier usuario con smartphone |
| **Llaves de seguridad FIDO2** | Llave física USB/NFC/Bluetooth basada en el estándar abierto FIDO2. Resistente a phishing. | Usuarios de alta seguridad, equipos compartidos, sin móvil corporativo |

Puntos clave:
- Passwordless es **multifactor por diseño**: dispositivo (tienes) + biometría o PIN (eres/sabes).
- El **PIN de Windows Hello no es una contraseña**: está ligado al hardware del equipo y no viaja por la red.
- **FIDO2** es un estándar abierto de la FIDO Alliance; no es exclusivo de Microsoft.
- Mejora la seguridad **y** la experiencia: menos fricción que MFA tradicional.
- La configuración del Authenticator (DEMO 106-107) consiste en registrar el dispositivo desde "Información de seguridad" del usuario y habilitar el inicio de sesión por teléfono.

> [!warning] Confusión frecuente
> - **Passwordless ≠ un solo factor**. Aunque no hay contraseña, sigue habiendo dos factores (dispositivo + biometría/PIN).
> - **PIN ≠ contraseña**. Una contraseña se envía al servidor y sirve en cualquier dispositivo; el PIN de Windows Hello solo funciona en ese equipo.
> - **MFA con Authenticator ≠ passwordless con Authenticator**. En MFA el Authenticator es el *segundo* factor tras la contraseña; en passwordless *reemplaza* la contraseña.

## Casos de uso

- Empleado desbloquea su laptop Windows con el rostro y entra a Microsoft 365 sin escribir nada → Windows Hello for Business.
- Trabajador de campo entra al portal de Azure desde una tablet: escribe su usuario, ve un número, lo confirma en el Authenticator con huella → passwordless con Authenticator.
- Administrador de seguridad usa una llave USB para iniciar sesión en cualquier equipo → FIDO2.

## Comparaciones

| | Contraseña + MFA | Passwordless |
|---|---|---|
| Contraseña | Sí, es el primer factor | No existe |
| Factores | Contraseña + segundo factor | Dispositivo + biometría/PIN |
| Riesgo de phishing | La contraseña puede robarse | Muy bajo (FIDO2 lo elimina) |
| Experiencia | Escribir contraseña + aprobar | Solo aprobar / biometría |
| Métodos Microsoft | Authenticator (2º factor), SMS, llamada | Windows Hello for Business, Authenticator, FIDO2 |

## Conceptos que debo memorizar

> [!important]
> - Tres métodos passwordless: **Windows Hello for Business · Microsoft Authenticator · Llaves FIDO2**.
> - Passwordless **elimina** la contraseña, no la complementa.
> - Sigue siendo **multifactor** (tienes + eres/sabes).
> - **FIDO2** = estándar abierto, resistente a phishing.
> - Es la opción **más segura y más cómoda**.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *without a password*, *eliminate passwords*, *biometric*, *FIDO2*, *Windows Hello*, *phone sign-in*.
> - Si la pregunta enumera métodos y pide identificar cuál es passwordless: SMS y llamada de voz **NO** lo son (son segundos factores). Windows Hello, Authenticator (inicio de sesión por teléfono) y FIDO2 **SÍ**.
> - Trampa: "Passwordless es menos seguro porque usa un solo factor". Falso.
> - Trampa: "Windows Hello for Business almacena la contraseña en el dispositivo". Falso: almacena una clave criptográfica; el PIN solo desbloquea esa clave localmente.
> - Diferencia evaluable: Windows Hello es para **dispositivos Windows**; FIDO2 y Authenticator funcionan en **cualquier plataforma**.

## Ejemplo de pregunta de examen

**Pregunta 1.** ¿Cuál de los siguientes es un método de autenticación sin contraseña compatible con Microsoft Entra ID?
- A) Código enviado por SMS
- B) Llamada de voz
- C) Llave de seguridad FIDO2
- D) Preguntas de seguridad

**Respuesta: C.** FIDO2 permite iniciar sesión sin contraseña. A y B son segundos factores que se añaden a una contraseña. D es un método de recuperación, no de inicio de sesión.

**Pregunta 2.** Una empresa quiere que sus empleados con equipos Windows inicien sesión con reconocimiento facial y eliminen las contraseñas. ¿Qué solución deben implementar?
- A) Windows Hello for Business
- B) Valores predeterminados de seguridad
- C) Microsoft Entra Connect
- D) Autenticación por SMS

**Respuesta: A.** Windows Hello for Business ofrece biometría ligada al dispositivo Windows sin contraseña. B activa MFA pero mantiene la contraseña. C sincroniza identidades. D es un segundo factor, no passwordless.

**Pregunta 3.** ¿Cuál de las siguientes afirmaciones sobre la autenticación sin contraseña es verdadera?
- A) Usa un único factor de autenticación
- B) Combina algo que tienes con algo que eres o sabes
- C) Requiere que la contraseña se almacene en el dispositivo
- D) Solo funciona con dispositivos Windows

**Respuesta: B.** Passwordless combina dispositivo/llave con biometría o PIN. A es falsa (es multifactor). C es falsa (no hay contraseña). D es falsa (FIDO2 y Authenticator funcionan en cualquier plataforma).

## 🧠 Resumen para el examen

1. Passwordless = sin contraseña, pero con dos factores.
2. Tres métodos: Windows Hello for Business, Microsoft Authenticator, FIDO2.
3. Windows Hello = biometría o PIN ligado al PC Windows.
4. Authenticator = el teléfono es la credencial (coincidencia de número + biometría).
5. FIDO2 = llave física, estándar abierto, resistente a phishing.
6. SMS y llamada de voz NO son passwordless.
7. El PIN de Windows Hello no es una contraseña.
8. Más seguro y más cómodo que contraseña + MFA.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
