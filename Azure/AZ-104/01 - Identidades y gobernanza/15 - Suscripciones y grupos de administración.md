---
tags: [az-104, azure, gobernanza, suscripciones, management-groups]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Suscripciones y grupos de administración

## ¿Qué es?

- Una **suscripción** es el contenedor de **facturación y límites** de Azure: agrupa recursos, define cuotas y se asocia a un tenant de Entra ID y a una cuenta de facturación.
- Un **grupo de administración (management group, MG)** es un contenedor **por encima de las suscripciones** para aplicar gobernanza (Policy, RBAC, presupuestos) a varias suscripciones de una vez.

## ¿Para qué sirve?

- Separar entornos (Dev/Test/Prod), departamentos o clientes por suscripción: facturación y cuotas independientes.
- Aplicar una directiva ("solo regiones europeas") o un rol ("auditores = Reader") a **todas** las suscripciones desde un MG.

## Conceptos clave

### Suscripciones
- Tipos/ofertas: **Free** (200 USD 30 días + 12 meses de servicios gratis), **Pay-As-You-Go**, **Enterprise Agreement (EA)**, **Microsoft Customer Agreement (MCA)**, **CSP** (partner), **Visual Studio / Dev/Test**, **Student**.
- Una suscripción → un tenant; un tenant → muchas suscripciones.
- **Límites (cuotas)** por suscripción: por ejemplo vCPUs por región y familia (se amplían con solicitud de soporte), 980 RGs, 4000 asignaciones RBAC, etc.
- **Estados**: Active, Disabled (impago, límite de gasto alcanzado, cancelada), Deleted, Past due, Warned. Desactivada = recursos parados, se conservan 90 días.
- **Límite de gasto (spending limit)**: solo en suscripciones con crédito (Free, Visual Studio); al alcanzarlo se deshabilita hasta el siguiente período o se quita el límite.
- Acciones administrativas: **cambiar de directorio** (transferir a otro tenant; se pierden RBAC, Policy, identidades administradas), **transferir la propiedad de facturación**, **cancelar**, **renombrar**, **mover entre grupos de administración**.
- Roles: el **Owner** de la suscripción administra recursos; el **Account Administrator / Billing account owner** administra facturación. Se pueden crear presupuestos y alertas ([[16 - Administración de costes (presupuestos, alertas y Advisor)]]).

### Grupos de administración
- **Root management group (Tenant Root Group)**: creado automáticamente; contiene todo. Por defecto nadie tiene acceso; un Global Administrator puede **elevar acceso**.
- Jerarquía: hasta **6 niveles de profundidad** (sin contar el raíz ni las suscripciones) 🧠.
- **10 000 MGs** por tenant 🧠.
- Cada MG y cada suscripción tiene **un solo padre**; un MG puede tener muchos hijos (MGs o suscripciones).
- **Herencia**: Policy y RBAC asignados en un MG aplican a todo lo de debajo.
- Nombre de MG (ID) inmutable; el nombre para mostrar sí se puede cambiar.
- Permiso para crear MGs: por defecto cualquier usuario del tenant puede crear MGs (configurable en Configuración → **Autorización requerida**); mover una suscripción a un MG requiere `Microsoft.Management/managementGroups/write` en el destino y Owner (o `moveResources`) en la suscripción.
- **Configuración del MG raíz**: ámbito por defecto para nuevas suscripciones (Default management group).

## Cómo funciona

```
Tenant Root Group
 ├── MG "Corp"
 │    ├── MG "Prod"  ──► Policy: Allowed locations = EU; RBAC: SecOps = Reader
 │    │    ├── Sub "Prod-Web"
 │    │    └── Sub "Prod-Data"
 │    └── MG "NonProd"
 │         ├── Sub "Dev"
 │         └── Sub "Test"
 └── MG "Sandbox"
      └── Sub "Sandbox-01"
```

```bash
az account management-group create --name mg-corp --display-name "Corp"
az account management-group create --name mg-prod --display-name "Prod" --parent mg-corp
az account management-group subscription add --name mg-prod --subscription <subId>
az account management-group list
az account list --all -o table
az account subscription rename --id <subId> --name "Prod-Web"   # (extensión)
```

```powershell
New-AzManagementGroup -GroupName mg-corp -DisplayName "Corp"
New-AzManagementGroup -GroupName mg-prod -DisplayName "Prod" -ParentId "/providers/Microsoft.Management/managementGroups/mg-corp"
New-AzManagementGroupSubscription -GroupName mg-prod -SubscriptionId <subId>
Get-AzManagementGroup -Recurse -Expand
```

## Configuración relevante para el examen

| Necesidad | Solución |
|---|---|
| Aplicar una Policy a 20 suscripciones | Asignarla en un MG padre |
| Dar Reader a auditores en todas las suscripciones | RBAC en el Tenant Root Group (tras elevar acceso) |
| Crear suscripciones nuevas de forma programática | Cuenta de facturación EA/MCA + `az account alias create` ➕ |
| Aumentar cuota de vCPU | Suscripción → Uso + cuotas → solicitar aumento |
| Separar facturación Dev y Prod | Suscripciones distintas |
| Suscripción de un tenant a otro | Cambiar directorio (perderás RBAC, Policy, identidades administradas, Key Vault access policies) |
| Ver todas las suscripciones en el portal | Filtro global de suscripciones/directorios |

## Ejemplo

Fabrikam tiene 12 suscripciones. Quiere que ninguna, salvo la de Sandbox, pueda crear recursos fuera de Europa, y que el equipo de seguridad tenga Reader en todas. Crea MG "Corp" (11 subs) y MG "Sandbox" (1 sub). Asigna *Allowed locations* en "Corp" y el rol Reader al grupo SecOps en el Tenant Root Group. Las suscripciones nuevas se crean bajo "Corp" configurándolo como MG por defecto.

## Comparaciones

| Ámbito | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Grupo de administración** | Gobernanza sobre varias suscripciones | Herencia masiva | Organizaciones con varias suscripciones |
| **Suscripción** | Unidad de facturación y cuotas | Aislamiento de coste y límites | Separar entornos, departamentos, clientes |
| **Grupo de recursos** | Ciclo de vida de recursos | Borrado y RBAC por aplicación | Cada aplicación/entorno |
| **Recurso** | Ámbito mínimo | Máxima granularidad | Excepciones puntuales |

## AZ-104 Exam Tips

- ⭐ Jerarquía: **Root MG → MG (hasta 6 niveles) → Suscripción → RG → Recurso**; herencia hacia abajo.
- 🔥 🧠 **6 niveles** de profundidad; **10 000** MGs por tenant; **un solo padre**.
- 🔥 📌 Suscripción = **facturación + cuotas**; MG = **gobernanza** (no factura).
- 🧠 Cambiar una suscripción de tenant **borra RBAC y Policy**.
- 🧠 Root MG: nadie tiene acceso por defecto; Global Administrator debe **elevar**.
- 💻 Crear MGs, mover suscripciones, asignar Policy/RBAC en un MG, solicitar aumento de cuota.
- ⚠️ Una suscripción deshabilitada mantiene los recursos **90 días**.
- ⚠️ Los recursos no se pueden crear "en" un MG; solo en RGs.

## Errores comunes

- Pensar que un MG agrupa recursos (agrupa suscripciones).
- Asignar la misma Policy en cada suscripción cuando un MG lo resuelve.
- Olvidar que al mover una suscripción a otro MG hereda las directivas del nuevo padre (y puede volverse no conforme).

## Preguntas que podrían aparecer

**1.** Tienes 15 suscripciones y necesitas que el grupo "Auditores" tenga acceso de lectura a todas ellas y a las que se creen en el futuro, con el mínimo esfuerzo. ¿Qué haces?
- A) Asignar Reader en cada suscripción · B) Asignar Reader en el grupo de administración raíz · C) Crear un rol personalizado · D) Usar Azure Policy

<details><summary>Respuesta</summary>

**B.** Una asignación en el Tenant Root Group se hereda a todas las suscripciones actuales y futuras.
</details>

**2.** ¿Cuántos niveles de grupos de administración se pueden anidar por debajo del grupo raíz?
- A) 3 · B) 6 · C) 10 · D) Ilimitados

<details><summary>Respuesta</summary>

**B.** Seis niveles, sin contar el raíz ni el nivel de suscripción.
</details>

**3.** Cambias la suscripción Sub1 al tenant Fabrikam. ¿Qué ocurre con las asignaciones de roles RBAC existentes?
- A) Se conservan · B) Se eliminan y deben recrearse · C) Se convierten en roles de Entra · D) Solo se conservan las de Owner

<details><summary>Respuesta</summary>

**B.** Al transferir una suscripción a otro directorio, todas las asignaciones RBAC (y otras configuraciones ligadas a identidades) se pierden.
</details>

## Relacionado

- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[11 - Azure Policy]]
- [[14 - Grupos de recursos y movimiento de recursos]]
- [[16 - Administración de costes (presupuestos, alertas y Advisor)]]
- [[01 - Microsoft Entra ID para administradores]]
- [[00 - Índice - Identidades y gobernanza]]
