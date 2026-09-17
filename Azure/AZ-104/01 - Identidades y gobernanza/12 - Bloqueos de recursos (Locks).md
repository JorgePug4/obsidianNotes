---
tags: [az-104, azure, gobernanza, locks]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Bloqueos de recursos (Resource Locks)

## ¿Qué es?

Un **bloqueo** protege una suscripción, un grupo de recursos o un recurso frente a **eliminación o modificación accidental**, sin importar los permisos RBAC del usuario. Es una medida de **seguridad frente a errores**, no de control de acceso.

## ¿Para qué sirve?

- Evitar que alguien borre por error la VNet de producción o la cuenta de almacenamiento con backups.
- Congelar recursos que no deben cambiar (ReadOnly).

## Conceptos clave

- Dos niveles 🧠:
  - **CanNotDelete** (Delete): se puede leer y modificar, **no eliminar**.
  - **ReadOnly**: solo lectura; no se puede modificar ni eliminar. Equivale a dar a todos el rol Reader sobre el recurso.
- **Herencia**: un lock en el RG aplica a todos sus recursos (y a los que se creen después); en la suscripción, a todo.
- **Prevalece el más restrictivo**: si el RG tiene ReadOnly y el recurso CanNotDelete, el recurso es ReadOnly.
- Se aplica **a todos**, incluidos Owner y Global Administrator elevado. Para borrar hay que **quitar el lock primero**.
- Permiso para gestionar locks: `Microsoft.Authorization/locks/*` → solo **Owner** y **User Access Administrator** (Contributor **no**).
- Afecta al **plano de control**, no al de datos: con un ReadOnly en una cuenta de almacenamiento se pueden seguir escribiendo blobs (con clave/SAS), pero no cambiar la configuración de la cuenta.
- Se pueden aplicar con ARM/Bicep (`Microsoft.Authorization/locks`), CLI, PowerShell y portal.

## Cómo funciona / efectos secundarios de ReadOnly que pregunta el examen

| Recurso con ReadOnly | Efecto inesperado |
|---|---|
| Cuenta de almacenamiento | No se pueden **listar las claves** (es una operación POST) → el portal no muestra blobs/archivos; las apps con clave siguen funcionando |
| Máquina virtual | No se puede **iniciar ni detener** (son acciones POST) |
| App Service | Impide ver algunas configuraciones en el portal |
| Grupo de recursos | No se pueden crear recursos nuevos ni mover recursos dentro |
| Suscripción | Bloquea despliegues en toda la suscripción |
| SQL Database | Impide operaciones de plano de control como escalado |

Un **CanNotDelete** en un RG impide también **mover** recursos fuera del RG? No: mover es permitido, pero borrar el RG no. Sin embargo, si un recurso con CanNotDelete se mueve, el lock viaja con él.

## Componentes / configuración relevante para el examen

- Portal: recurso/RG/suscripción → **Bloqueos** → Agregar → nombre, tipo (Read-only / Delete), notas.
- CLI:

```bash
az lock create --name lock-vnet --lock-type CanNotDelete --resource-group rg-prod --resource-name vnet-prod --resource-type Microsoft.Network/virtualNetworks
az lock create --name lock-rg --lock-type ReadOnly --resource-group rg-prod
az lock list --resource-group rg-prod -o table
az lock delete --name lock-rg --resource-group rg-prod
```

- PowerShell:

```powershell
New-AzResourceLock -LockName lock-vnet -LockLevel CanNotDelete -ResourceGroupName rg-prod -ResourceName vnet-prod -ResourceType Microsoft.Network/virtualNetworks
Get-AzResourceLock -ResourceGroupName rg-prod
Remove-AzResourceLock -LockName lock-rg -ResourceGroupName rg-prod
```

- Bicep:

```bicep
resource lock 'Microsoft.Authorization/locks@2020-05-01' = {
  name: 'lock-vnet'
  scope: vnet
  properties: { level: 'CanNotDelete', notes: 'Producción' }
}
```

## Ejemplo

Un Owner intenta borrar el RG `rg-prod` y recibe "The scope is locked". El RG no tiene lock, pero una de sus cuentas de almacenamiento tiene **CanNotDelete**. Como borrar el RG implica borrar la cuenta, la operación falla. Solución: quitar el lock del recurso hijo (y decidir si realmente debía borrarse).

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **CanNotDelete** | Proteger frente a borrado | Permite operar normalmente | Casi todos los recursos de producción |
| **ReadOnly** | Congelar configuración | Máxima protección | Recursos que no deben cambiar; cuidado con efectos secundarios |
| **RBAC Reader** | Limitar a personas concretas | Granular por identidad | Cuando solo algunos deben estar limitados |
| **Azure Policy DenyAction (delete)** ➕ | Impedir borrados por directiva | Escala, condiciones | Gobernanza a nivel de MG |
| **Soft delete / versioning** | Recuperar datos borrados | Recuperación | Datos dentro del recurso, no el recurso |

## AZ-104 Exam Tips

- ⭐ **CanNotDelete** = se puede modificar, no borrar. **ReadOnly** = ni modificar ni borrar.
- 🔥 Los locks **vencen a cualquier rol**, incluido Owner. Hay que **quitarlos** antes de borrar.
- 🔥 🧠 Crear/quitar locks: **Owner o User Access Administrator**. Contributor no puede.
- 🔥 ⚠️ ReadOnly en una **cuenta de almacenamiento** impide listar claves (el portal deja de mostrar datos); ReadOnly en una **VM** impide iniciarla/detenerla.
- 🧠 Se heredan hacia abajo y **gana el más restrictivo**.
- 💻 `az lock create`, `New-AzResourceLock`, y la hoja Bloqueos del portal.
- 📌 Lock (todos, accidental) vs RBAC (identidades, intencional) vs Policy (reglas de creación).

## Errores comunes

- Intentar resolver un fallo de eliminación con más permisos RBAC.
- Poner ReadOnly en una cuenta de almacenamiento y pensar que "se rompió" el acceso a los blobs desde el portal.
- Olvidar que un lock en un recurso hijo bloquea el borrado del RG completo.

## Preguntas que podrían aparecer

**1.** Aplicas un bloqueo ReadOnly a un grupo de recursos que contiene una máquina virtual. ¿Qué ocurre al intentar reiniciar la VM?
- A) Funciona; ReadOnly solo impide borrar · B) Falla; reiniciar es una operación que modifica el estado · C) Funciona solo para Owner · D) Falla solo si la VM tiene disco Premium

<details><summary>Respuesta</summary>

**B.** Iniciar, detener y reiniciar son operaciones POST bloqueadas por ReadOnly. Solo CanNotDelete las permitiría.
</details>

**2.** Un usuario con Contributor en RG1 debe evitar que se borren accidentalmente los recursos de RG1. Al intentar crear un bloqueo recibe un error. ¿Qué rol adicional mínimo necesita?
- A) Owner · B) User Access Administrator · C) Reader · D) Resource Policy Contributor

<details><summary>Respuesta</summary>

**B.** User Access Administrator incluye `Microsoft.Authorization/*`, que cubre los locks, y es menos amplio que Owner.
</details>

## Relacionado

- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[10 - Interpretar asignaciones de acceso]]
- [[11 - Azure Policy]]
- [[14 - Grupos de recursos y movimiento de recursos]]
- [[Bloqueos de recursos (Locks)]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
