---
tags: [az-104, azure, identidad, rbac, roles-personalizados]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Roles personalizados de Azure RBAC

## ¿Qué es?

Un **rol personalizado (custom role)** es una definición de rol creada por ti cuando ningún rol integrado encaja con el principio de privilegio mínimo. Se define en **JSON** (o PowerShell/CLI/portal) con listas de acciones permitidas y excluidas.

## ¿Para qué sirve?

- "Puede reiniciar VMs pero no crearlas ni borrarlas".
- "Puede leer todo y además gestionar soporte".
- Recortar un rol integrado quitando acciones concretas.

## Conceptos clave

- **Actions**: operaciones de plano de control permitidas (`Microsoft.Compute/virtualMachines/start/action`). Admite comodines `*`.
- **NotActions**: operaciones **excluidas del rol** (se restan de Actions). ⚠️ **No es una denegación**: si otro rol las concede, el usuario las tiene.
- **DataActions / NotDataActions**: lo mismo para el plano de datos (`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`).
- **AssignableScopes**: dónde puede asignarse el rol (MG, suscripciones, RGs). No se pueden usar comodines para suscripciones.
- **Permisos efectivos de un rol = Actions − NotActions** (y DataActions − NotDataActions).
- Se crea a partir de una copia de un rol integrado, desde cero o desde JSON.
- Límite: **5000 roles personalizados por tenant** (2000 en Azure China).
- Rol necesario para crear roles personalizados: **Owner** o **User Access Administrator** en el ámbito de AssignableScopes (necesita `Microsoft.Authorization/roleDefinitions/write`).

## Cómo funciona

Ejemplo de definición:

```json
{
  "Name": "VM Operator",
  "IsCustom": true,
  "Description": "Puede iniciar, reiniciar y detener VMs, y leer recursos.",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Network/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

```bash
az role definition create --role-definition vm-operator.json
az role definition update --role-definition vm-operator.json
az role definition delete --name "VM Operator"
az provider operation show --namespace Microsoft.Compute   # listar operaciones disponibles
```

```powershell
New-AzRoleDefinition -InputFile vm-operator.json
$role = Get-AzRoleDefinition "Virtual Machine Contributor"
$role.Id = $null; $role.Name = "VM Operator"; $role.IsCustom = $true
$role.Actions.Clear(); $role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
$role.AssignableScopes.Clear(); $role.AssignableScopes.Add("/subscriptions/<id>")
New-AzRoleDefinition -Role $role
Get-AzProviderOperation "Microsoft.Compute/virtualMachines/*"
```

Formato de las operaciones: `{Proveedor}/{tipoRecurso}/{operación}`; las que terminan en `/action` son operaciones especiales (start, restart, listKeys, regenerateKey…).

## Componentes

| Propiedad | Obligatoria | Nota |
|---|---|---|
| Name | Sí | Único en el tenant |
| Id | Auto | GUID |
| IsCustom | Sí (`true`) | |
| Description | Recomendada | |
| Actions | Sí (puede ser `[]`) | Comodines permitidos |
| NotActions | No | Solo resta del propio rol |
| DataActions / NotDataActions | No | Plano de datos |
| AssignableScopes | Sí, al menos uno | Ámbitos donde se puede asignar |

## Configuración relevante para el examen

- **Portal**: Suscripción → IAM → Agregar → **Agregar rol personalizado** → Clonar un rol / Empezar desde cero / Desde JSON → permisos → ámbitos asignables → revisar.
- Para ver qué operaciones existen: `Get-AzProviderOperation` / `az provider operation show`.
- Si un rol personalizado necesita asignarse en varias suscripciones, deben estar todas en **AssignableScopes** o usar un **grupo de administración** como ámbito asignable.
- **Modificar** un rol en uso: los cambios aplican de inmediato a todas las asignaciones.
- No se puede **borrar** un rol personalizado con asignaciones activas.

## Ejemplo

El equipo de operaciones nocturnas debe **reiniciar** VMs de producción y abrir tickets de soporte. Ningún rol integrado da exactamente eso (Virtual Machine Contributor permite borrar). Se crea "VM Restart Operator" con `Microsoft.Compute/virtualMachines/restart/action`, `*/read` y `Microsoft.Support/*`, con AssignableScopes en la suscripción de producción, y se asigna al grupo "NOC" en el RG de producción.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Rol integrado** | Necesidad estándar | Sin mantenimiento, Microsoft lo actualiza | Siempre que uno encaje |
| **Rol personalizado** | Necesidad no cubierta | Privilegio mínimo exacto | Cuando el integrado da de más o de menos |
| **Rol integrado + condición (ABAC)** ➕ | Restringir roles de datos por atributos | Sin crear roles nuevos | Storage: acceso por tag/contenedor |
| **Rol de Entra personalizado** | Permisos del directorio | Granular en Entra | Delegación de administración de apps/usuarios (P1) |

## AZ-104 Exam Tips

- ⭐ Permisos de un rol = **Actions − NotActions**.
- 🔥 ⚠️ **NotActions no deniega**: solo excluye del rol. Si el usuario tiene otro rol que sí incluye la acción, la puede ejecutar.
- 🧠 **AssignableScopes** obligatorio; para usarlo en varias suscripciones, incluirlas todas o usar un MG.
- 🧠 Límite **5000** roles personalizados por tenant.
- 💻 Reconocer el JSON y los comandos `New-AzRoleDefinition` / `az role definition create`.
- 📌 Acciones `/action` = operaciones especiales (start, restart, listKeys).
- ⚠️ Los roles de Storage Blob Data usan **DataActions**, no Actions.

## Errores comunes

- Intentar asignar el rol en una suscripción que no está en AssignableScopes.
- Usar NotActions creyendo que bloquea a un usuario que tiene Contributor por otra vía.
- Escribir `Microsoft.Compute/virtualMachines/restart` sin `/action`.

## Preguntas que podrían aparecer

**1.** Creas un rol personalizado con `Actions: ["*"]` y `NotActions: ["Microsoft.Compute/virtualMachines/delete"]` y lo asignas a Juan en RG1. Juan también tiene Contributor en RG1. ¿Puede Juan borrar VMs en RG1?
- A) No, NotActions lo impide · B) Sí, porque Contributor lo permite · C) Solo si es Owner · D) Depende del orden de asignación

<details><summary>Respuesta</summary>

**B.** NotActions solo recorta el rol personalizado. Contributor incluye el borrado de VMs, y los permisos se suman.
</details>

**2.** Quieres asignar un rol personalizado en tres suscripciones distintas del mismo grupo de administración. ¿Cuál es la forma más sencilla?
- A) Crear tres roles · B) Establecer el grupo de administración como AssignableScope · C) Usar comodín `/subscriptions/*` · D) Asignarlo en el tenant

<details><summary>Respuesta</summary>

**B.** Un MG como ámbito asignable cubre todas sus suscripciones. Los comodines no se admiten en AssignableScopes.
</details>

## Relacionado

- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[10 - Interpretar asignaciones de acceso]]
- [[15 - Suscripciones y grupos de administración]]
- [[00 - Índice - Identidades y gobernanza]]
