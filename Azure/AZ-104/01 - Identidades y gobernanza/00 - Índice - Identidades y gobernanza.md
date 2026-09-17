---
tags: [az-104, azure, identidad, gobernanza, MOC]
tipo: MOC
modulo: Identidades y gobernanza
peso_examen: 20-25 %
---

# AZ-104 · Dominio 1 · Administrar identidades y gobernanza de Azure

> [!important] Peso en el examen: **20-25 %** (junto con cómputo, el dominio de mayor peso)
> Aquí caen preguntas de detalle: qué rol es el mínimo, qué se hereda, qué efecto de Policy usar, qué bloqueo aplicar, cuántos niveles tiene un grupo de administración. Es el dominio más "memorístico" y el que más se puede asegurar con estudio.

## Objetivos oficiales → notas

### 1.1 Administrar usuarios y grupos de Microsoft Entra

| Objetivo oficial | Nota |
|---|---|
| Crear usuarios y grupos | [[02 - Usuarios de Microsoft Entra ID]] · [[03 - Grupos de Microsoft Entra ID]] |
| Administrar propiedades de usuarios y grupos | [[02 - Usuarios de Microsoft Entra ID]] · [[03 - Grupos de Microsoft Entra ID]] |
| Administrar licencias en Microsoft Entra ID | [[04 - Licencias en Microsoft Entra ID]] |
| Administrar usuarios externos | [[05 - Usuarios externos e invitados (B2B)]] |
| Configurar el restablecimiento de contraseña de autoservicio (SSPR) | [[06 - Self-Service Password Reset (SSPR)]] |
| Contexto necesario | [[01 - Microsoft Entra ID para administradores]] · [[07 - Unidades administrativas y dispositivos]] ➕ |

### 1.2 Administrar el acceso a los recursos de Azure

| Objetivo oficial | Nota |
|---|---|
| Administrar roles integrados de Azure | [[08 - Azure RBAC - roles integrados y ámbitos]] |
| Asignar roles en distintos ámbitos | [[08 - Azure RBAC - roles integrados y ámbitos]] |
| Interpretar asignaciones de acceso | [[10 - Interpretar asignaciones de acceso]] |
| Contexto necesario | [[09 - Roles personalizados de Azure RBAC]] · [[18 - Identidades administradas y entidades de servicio]] ➕ |

### 1.3 Administrar suscripciones y gobernanza de Azure

| Objetivo oficial | Nota |
|---|---|
| Implementar y administrar Azure Policy | [[11 - Azure Policy]] |
| Configurar bloqueos de recursos | [[12 - Bloqueos de recursos (Locks)]] |
| Aplicar y administrar etiquetas | [[13 - Etiquetas (Tags)]] |
| Administrar grupos de recursos | [[14 - Grupos de recursos y movimiento de recursos]] |
| Administrar suscripciones | [[15 - Suscripciones y grupos de administración]] |
| Administrar costes con alertas, presupuestos y recomendaciones de Advisor | [[16 - Administración de costes (presupuestos, alertas y Advisor)]] |
| Configurar grupos de administración | [[15 - Suscripciones y grupos de administración]] |

### Extra (no está literal en el temario, pero aparece en escenarios)
- [[17 - Identidad híbrida - Microsoft Entra Connect]] ➕ (usuarios sincronizados, writeback de SSPR)
- [[18 - Identidades administradas y entidades de servicio]] ➕ (necesarias para Policy con remediación, CMK, App Service)

### Repaso
- [[99 - Repaso final - Identidades y gobernanza]]

## Orden de estudio sugerido

1. Entra ID (01 → 07): quién existe en el directorio.
2. RBAC (08 → 10): qué puede hacer cada uno sobre los recursos.
3. Gobernanza (11 → 16): qué reglas aplican a los recursos, cómo se organizan y cuánto cuestan.
4. Extras (17, 18) y repaso (99).

> [!tip] La idea central del dominio
> Tres preguntas distintas, tres herramientas distintas:
> - **¿Quién puede hacer qué?** → RBAC.
> - **¿Qué se puede crear y cómo?** → Azure Policy.
> - **¿Se puede borrar o modificar por error?** → Locks.
> Y todo se organiza en la jerarquía **Grupo de administración → Suscripción → Grupo de recursos → Recurso**, con herencia hacia abajo.

Volver: [[00 - AZ-104 Índice general (MOC)]]
