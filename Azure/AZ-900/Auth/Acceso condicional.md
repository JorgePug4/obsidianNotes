---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Acceso condicional

## Concepto

**Acceso condicional** es la herramienta de [[Microsoft Entra ID]] que aplica reglas del tipo **"SI se cumple X, ENTONCES exige Y"** al momento del inicio de sesión. Analiza **señales** (quién, desde dónde, con qué dispositivo, a qué app, con qué riesgo) y decide si **permite, bloquea o exige un control adicional** como [[Autenticación multifactor (MFA)|MFA]].

Problema que resuelve: exigir MFA a todo el mundo todo el tiempo es molesto; no exigirlo nunca es peligroso. Acceso condicional adapta la exigencia al contexto.

Para qué se usa: ser el **motor de directivas de Confianza cero** en Microsoft.

## Características principales

Estructura de una directiva:

1. **Señales / condiciones** (SI):
   - Usuario o grupo
   - Ubicación (IP, país)
   - Dispositivo (conforme, unido a Entra, plataforma)
   - Aplicación a la que se accede
   - Riesgo del inicio de sesión (detección en tiempo real; requiere P2)
2. **Decisión** (ENTONCES):
   - **Conceder** acceso
   - **Conceder con condición**: exigir MFA, dispositivo conforme, cambio de contraseña
   - **Bloquear** acceso

Puntos clave:
- Requiere licencia **Microsoft Entra ID P1** (o P2 para directivas basadas en riesgo).
- Se evalúa **en cada inicio de sesión**, no solo una vez.
- Puede exigir MFA solo cuando el usuario está **fuera de la red corporativa**, por ejemplo.
- Implementa los principios de "verificar explícitamente" y "privilegio mínimo" de [[Confianza cero (Zero Trust)]].

> [!warning] Confusión frecuente
> - **Acceso condicional ≠ MFA**. MFA es un método de autenticación; Acceso condicional es el motor que decide **cuándo** exigirlo (entre otras acciones).
> - **Acceso condicional ≠ RBAC**. Acceso condicional controla si **puedes iniciar sesión** y bajo qué condiciones (autenticación/acceso). RBAC controla **qué puedes hacer** una vez dentro (autorización).
> - **Acceso condicional ≠ valores predeterminados de seguridad**. Los valores predeterminados son un conjunto fijo y gratuito; Acceso condicional es personalizable y de pago.

## Casos de uso

- Exigir MFA solo cuando el usuario inicia sesión desde fuera de la oficina.
- Bloquear el acceso desde países donde la empresa no opera.
- Permitir Outlook solo desde dispositivos administrados por la empresa.
- Exigir MFA para acceder al portal de Azure, pero no para Teams.
- Bloquear protocolos de autenticación heredados (legacy authentication).

## Comparaciones

| | Valores predeterminados de seguridad | Acceso condicional |
|---|---|---|
| Costo | Gratis (Free) | Entra ID P1 (P2 para riesgo) |
| Personalización | Ninguna: reglas fijas | Total: usuarios, apps, ubicaciones, dispositivos |
| MFA | Para todos, siempre que se solicita | Solo cuando se cumplen las condiciones |
| Bloquear por ubicación | No | Sí |
| Exigir dispositivo conforme | No | Sí |
| Ideal para | Organizaciones pequeñas, punto de partida | Organizaciones que necesitan control granular |

## Conceptos que debo memorizar

> [!important]
> - Acceso condicional = **directivas "si… entonces…"** en el inicio de sesión.
> - **Señales**: usuario, ubicación, dispositivo, aplicación, riesgo.
> - **Decisiones**: conceder, conceder con MFA/condición, bloquear.
> - Requiere **Entra ID P1** (riesgo → P2).
> - Es el **motor de Confianza cero** en Entra ID.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *based on location*, *from an untrusted device*, *only when*, *require MFA when*, *signals*, *if-then policy*.
> - Cuando la pregunta describe una **condición** (ubicación, dispositivo, app) seguida de una **acción** (exigir MFA, bloquear), la respuesta es Acceso condicional.
> - Si la pregunta pide "MFA para todos sin licencias adicionales" → valores predeterminados de seguridad, NO Acceso condicional.
> - Trampa: Acceso condicional no asigna permisos sobre recursos. Eso es RBAC.
> - Trampa: "Acceso condicional está incluido en la edición Free" es falsa.

## Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa quiere exigir MFA únicamente cuando los usuarios inicien sesión desde fuera de la red corporativa. ¿Qué característica de Microsoft Entra ID debe usar?
- A) Valores predeterminados de seguridad
- B) Acceso condicional
- C) Azure RBAC
- D) Microsoft Entra Connect

**Respuesta: B.** Aplicar MFA según la ubicación es una directiva condicional. A exige MFA sin distinguir ubicación. C gestiona permisos sobre recursos. D sincroniza identidades locales.

**Pregunta 2.** ¿Cuál de las siguientes es una señal que Acceso condicional puede evaluar?
- A) El rol RBAC asignado al usuario en la suscripción
- B) La ubicación desde la que se inicia sesión
- C) El tamaño del grupo de recursos
- D) El costo mensual de la suscripción

**Respuesta: B.** La ubicación (IP o país) es una señal estándar. A es autorización posterior, no una señal de inicio de sesión. C y D no tienen relación con la identidad.

**Pregunta 3.** ¿Qué licencia se requiere como mínimo para usar directivas de Acceso condicional?
- A) Microsoft Entra ID Free
- B) Microsoft Entra ID P1
- C) Microsoft 365 Business Basic
- D) Azure Pay-As-You-Go

**Respuesta: B.** Acceso condicional es una característica de P1 (P2 añade directivas basadas en riesgo). A no lo incluye. C y D no son ediciones de Entra ID.

## 🧠 Resumen para el examen

1. Acceso condicional = reglas "si… entonces…" al iniciar sesión.
2. Señales: usuario, ubicación, dispositivo, app, riesgo.
3. Decisiones: conceder, exigir MFA, bloquear.
4. Requiere Entra ID P1; riesgo en tiempo real requiere P2.
5. Es distinto de MFA (método) y de RBAC (permisos).
6. Valores predeterminados de seguridad = versión gratuita y fija.
7. Pone en práctica Confianza cero: verificar explícitamente.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
