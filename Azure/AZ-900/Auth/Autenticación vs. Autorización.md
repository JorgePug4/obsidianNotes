---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Autenticación vs. Autorización

## Concepto

- **Autenticación (AuthN)**: demostrar **quién eres**. Es el proceso de verificar una identidad (usuario, contraseña, MFA, huella, certificado).
- **Autorización (AuthZ)**: decidir **qué puedes hacer** una vez identificado. Define permisos sobre recursos (leer, escribir, eliminar, administrar).

Problema que resuelve: separar "confirmar identidad" de "conceder acceso". Puedes autenticarte correctamente y aun así no tener permiso para tocar un recurso.

Orden: **primero autenticación, después autorización**. Sin identidad verificada no hay nada que autorizar.

## Características principales

- La autenticación establece la identidad; la autorización establece el **nivel de acceso**.
- En Azure, la autenticación la gestiona **[[Microsoft Entra ID]]**; la autorización sobre recursos de Azure la gestiona **Azure RBAC** (Role-Based Access Control).
- Los métodos de autenticación en Azure son: contraseña, **[[Autenticación multifactor (MFA)|MFA]]**, **[[Autenticación sin contraseña (Passwordless)|passwordless]]** y **[[Inicio de sesión único (SSO)|SSO]]**.
- La autorización se expresa con roles (Lector, Colaborador, Propietario) asignados a un ámbito (suscripción, grupo de recursos, recurso).

> [!warning] Confusión frecuente
> Los estudiantes mezclan ambos términos porque en la práctica ocurren en segundos. Truco: AuthN responde *"¿eres tú?"* y AuthZ responde *"¿tienes permiso?"*. MFA pertenece a AuthN. RBAC pertenece a AuthZ.

## Casos de uso

- Entras al portal de Azure con usuario, contraseña y código del Authenticator → **autenticación**.
- Intentas eliminar una VM y recibes "no tienes permisos" → falló la **autorización** (la autenticación fue correcta).
- Tarjeta de embarque: el pasaporte te identifica (AuthN); el ticket indica tu asiento y si puedes entrar a la sala VIP (AuthZ).

## Comparaciones

| | Autenticación | Autorización |
|---|---|---|
| Pregunta | ¿Quién eres? | ¿Qué puedes hacer? |
| Momento | Primero | Después |
| Ejemplos | Contraseña, MFA, huella, tarjeta inteligente | Roles RBAC, permisos, políticas |
| Servicio en Azure | Microsoft Entra ID | Azure RBAC (y Entra ID para apps) |
| Resultado | Identidad verificada (token) | Acceso permitido o denegado |

## Conceptos que debo memorizar

> [!important]
> - **Autenticación = identidad**. **Autorización = permisos**.
> - MFA, passwordless y SSO son **métodos de autenticación**.
> - RBAC es **autorización**.
> - Siempre ocurre AuthN antes que AuthZ.

## Tips para AZ-900

> [!tip]
> - Si la opción menciona **contraseña, código, biometría, dispositivo**, es autenticación.
> - Si menciona **rol, permiso, leer/escribir, ámbito**, es autorización.
> - Pregunta trampa: "Un usuario inicia sesión correctamente pero no puede crear recursos". La causa es de **autorización**, no de autenticación.
> - Palabras clave: *verify identity* → AuthN; *grant access / level of access* → AuthZ.

## Ejemplo de pregunta de examen

**Pregunta 1.** ¿Cuál de los siguientes es un ejemplo de autorización?
- A) Escribir la contraseña para entrar al portal de Azure
- B) Aprobar la notificación de Microsoft Authenticator
- C) Asignar el rol Lector a un usuario en un grupo de recursos
- D) Usar la huella dactilar con Windows Hello

**Respuesta: C.** Asignar un rol define qué puede hacer el usuario (autorización). A, B y D son métodos para demostrar identidad (autenticación).

**Pregunta 2.** Un usuario inicia sesión con éxito en Azure pero recibe un error al intentar eliminar una máquina virtual. ¿Qué proceso falló?
- A) Autenticación
- B) Autorización
- C) MFA
- D) SSO

**Respuesta: B.** El inicio de sesión fue exitoso, así que la identidad está verificada. La falta de permiso para eliminar es un fallo de autorización. A, C y D pertenecen al proceso de identificarse, que ya se completó.

## 🧠 Resumen para el examen

1. Autenticación = ¿quién eres? Autorización = ¿qué puedes hacer?
2. Primero autenticación, luego autorización.
3. Contraseña, MFA, biometría, SSO → autenticación.
4. Roles, permisos, RBAC → autorización.
5. "Inicia sesión pero no puede hacer X" → problema de autorización.
6. Microsoft Entra ID autentica; Azure RBAC autoriza.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
