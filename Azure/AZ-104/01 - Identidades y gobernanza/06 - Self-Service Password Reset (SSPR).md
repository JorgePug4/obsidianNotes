---
tags: [az-104, azure, identidad, entra-id, sspr]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Self-Service Password Reset (SSPR)

## ¿Qué es?

**SSPR** (restablecimiento de contraseña de autoservicio) permite que los usuarios **restablezcan o desbloqueen** su propia contraseña sin llamar al helpdesk, tras verificar su identidad con métodos de autenticación registrados.

## ¿Para qué sirve?

- Reducir tickets de soporte (la causa nº 1 de llamadas al helpdesk).
- Permitir que un usuario recupere el acceso a cualquier hora.
- Con **writeback**, que la nueva contraseña se escriba también en el **AD DS local** en entornos híbridos.

## Conceptos clave

- **Ámbito de habilitación**: **None / Selected (un solo grupo) / All**. Con *Selected* solo se puede elegir **un grupo** (que puede contener grupos anidados).
- **Métodos de autenticación** disponibles para SSPR: notificación de app móvil (Authenticator), código de app móvil, correo electrónico, teléfono móvil (SMS/llamada), teléfono de oficina, **preguntas de seguridad** (solo para SSPR, nunca para MFA).
- **Número de métodos requeridos**: **1 o 2**.
- **Registro**: se puede exigir el registro al iniciar sesión y volver a confirmar la información cada N días (por defecto 180, 0 = nunca).
- **Notificaciones**: avisar al usuario cuando restablece su contraseña; avisar a todos los administradores cuando otro administrador restablece la suya.
- **Bloqueo de cuenta**: SSPR también sirve para **desbloquear** cuentas sin cambiar la contraseña (en híbrido, con writeback).
- **Administradores**: siempre tienen SSPR habilitado con dos métodos y **no pueden usar preguntas de seguridad**.
- **Directiva de métodos de autenticación combinada**: hoy el registro de MFA y SSPR está unificado (combined security information registration).

## Cómo funciona

```
Usuario → "¿Olvidó su contraseña?" (https://aka.ms/sspr)
   → Entra comprueba que el usuario está en el ámbito habilitado
   → Verifica 1 o 2 métodos (según la directiva)
   → Aplica la directiva de contraseñas (longitud, contraseñas prohibidas)
   → Cambia la contraseña en la nube
   → Si hay writeback habilitado → la escribe en AD DS local (Entra Connect / Cloud Sync)
```

## Componentes

| Componente | Detalle |
|---|---|
| Propiedades | Ámbito None/Selected/All |
| Métodos de autenticación | Cuáles y cuántos (1 o 2) |
| Registro | Obligar al iniciar sesión; días para reconfirmar |
| Notificaciones | A usuarios / a administradores |
| Personalización | Enlace de ayuda propio |
| Integración local | **Password writeback** (requiere Entra Connect o Cloud Sync con writeback y P1) |
| Protección de contraseñas | Contraseñas prohibidas globales y personalizadas (P1 para lista personalizada) |

## Configuración relevante para el examen (licencias)

| Escenario | Licencia mínima |
|---|---|
| **Cambio** de contraseña de usuario cloud (conoce la actual) | Free |
| **Restablecimiento** de contraseña de usuario cloud-only | Microsoft 365 Business Standard o superior, o **Entra ID P1/P2** |
| Restablecimiento / desbloqueo con **writeback** a AD DS local | **Entra ID P1/P2** o M365 Business Premium |
| Lista personalizada de contraseñas prohibidas | P1/P2 |

- **Rol mínimo para configurar SSPR**: Global Administrator (la configuración de SSPR es de tenant); *Authentication Policy Administrator* gestiona métodos.
- Para que funcione el writeback: Entra Connect con la opción **Password writeback** habilitada, cuenta de servicio con permisos en AD, y **usuarios sincronizados**. Los usuarios sincronizados sin writeback **no pueden** usar SSPR (verían un error).

## Ejemplo

Contoso (híbrido, con Entra Connect) quiere que los empleados de la sede de Madrid puedan restablecer su contraseña de Windows desde casa. Pasos: comprar P1, habilitar writeback en Entra Connect, crear el grupo "SSPR-Piloto", configurar SSPR = Selected → ese grupo, exigir 2 métodos (app móvil + móvil) y registro obligatorio al inicio de sesión. Tras el piloto, cambiar a All.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **SSPR cloud** | Usuarios de nube | Sin infraestructura | Tenants cloud-only |
| **SSPR + writeback** | Usuarios sincronizados desde AD DS | La contraseña sirve en Windows local | Híbrido con Entra Connect / Cloud Sync |
| **Restablecimiento por helpdesk** | Rol Password Administrator | Control | Cuando SSPR no está permitido por política |
| **Passwordless** ([[Autenticación sin contraseña (Passwordless)]]) | Eliminar contraseñas | Más seguro | Estrategia a largo plazo |

## 💻 Laboratorio: habilitar SSPR

1. Crear grupo de seguridad "SSPR-Pilot" con dos usuarios de prueba.
2. Entra ID → Protección → Restablecimiento de contraseña → Propiedades → **Selected** → elegir el grupo.
3. Métodos de autenticación: exigir **2** métodos; habilitar app móvil, correo y teléfono móvil.
4. Registro: obligar a registrarse al iniciar sesión.
5. Iniciar sesión con un usuario de prueba, registrar métodos y probar en `https://aka.ms/sspr`.
6. Comprobar en los registros de auditoría el evento de restablecimiento.

## AZ-104 Exam Tips

- ⭐ Ámbito **Selected = un único grupo**.
- 🔥 🧠 **Writeback requiere P1 y Entra Connect (o Cloud Sync)**; sin writeback, los usuarios sincronizados no pueden usar SSPR.
- 🧠 Métodos requeridos: **1 o 2**. Preguntas de seguridad: **solo para SSPR** y **nunca para administradores**.
- 🧠 Reconfirmación de registro: **180 días** por defecto.
- 📌 Cambio de contraseña (Free) ≠ restablecimiento (requiere licencia).
- ⚠️ SSPR no es MFA. MFA protege el inicio de sesión; SSPR recupera la contraseña. Comparten el registro de métodos.

## Errores comunes

- Habilitar SSPR para "todos" y que los usuarios híbridos fallen por no haber activado writeback.
- Seleccionar preguntas de seguridad como método y esperar que funcionen para administradores.
- Elegir varios grupos en modo Selected: solo se puede uno.

## Preguntas que podrían aparecer

**1.** Tu tenant sincroniza usuarios desde AD DS con Microsoft Entra Connect. Habilitas SSPR para todos, pero los usuarios sincronizados reciben un error al intentar restablecer la contraseña. ¿Qué falta?
- A) Licencia P2 · B) Habilitar password writeback en Entra Connect · C) Cambiar el ámbito a Selected · D) Registrar más métodos

<details><summary>Respuesta</summary>

**B.** Sin writeback, la contraseña no puede escribirse en AD DS y SSPR bloquea a los usuarios sincronizados. Se necesita P1 (no P2) y la opción activada en Entra Connect.
</details>

**2.** Quieres habilitar SSPR solo para los grupos "Ventas" y "Marketing". ¿Cuál es la forma correcta?
- A) Seleccionar ambos grupos en Selected · B) Crear un grupo que contenga a ambos y seleccionarlo · C) Usar All y excluir el resto · D) No es posible

<details><summary>Respuesta</summary>

**B.** El modo Selected solo admite un grupo, pero ese grupo puede contener grupos anidados.
</details>

## Relacionado

- [[02 - Usuarios de Microsoft Entra ID]]
- [[17 - Identidad híbrida - Microsoft Entra Connect]]
- [[Autenticación multifactor (MFA)]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
