---
tags: [az-104, azure, identidad, rbac, acceso-efectivo]
modulo: Identidades y gobernanza
peso_examen: Muy alto
---

# Interpretar asignaciones de acceso

## ¿Qué es?

Es la habilidad de **predecir qué puede hacer un usuario** a partir de sus asignaciones RBAC (directas y por grupo) en la jerarquía de ámbitos, más los bloqueos y las directivas que se aplican. Es uno de los objetivos más preguntados del examen, normalmente en forma de "Sí/No" o de tablas con varios usuarios.

## ¿Para qué sirve?

- Responder "¿puede Ana borrar la VM?" sin probarlo.
- Auditar accesos excesivos.
- Resolver incidencias de "no tengo permiso".

## Conceptos clave (las reglas)

1. **Herencia**: una asignación en un ámbito aplica a todos los ámbitos hijos. MG → Sub → RG → Recurso.
2. **Acumulación**: los permisos de todas las asignaciones aplicables **se suman**.
3. **Grupos**: la pertenencia a grupos (incluidos anidados) aporta sus asignaciones.
4. **NotActions no deniega** (ver [[09 - Roles personalizados de Azure RBAC]]).
5. **Deny assignments** ganan sobre todo (raras).
6. **Locks** bloquean acciones **independientemente del rol** (incluso Owner).
7. **Azure Policy** puede rechazar la operación aunque el rol la permita.
8. **Roles de Entra ≠ roles RBAC**: Global Administrator no ve suscripciones salvo elevación.
9. **Plano de control ≠ plano de datos**.
10. Al **mover** recursos a otro RG/suscripción, las asignaciones hechas **en el recurso** se mantienen? ⚠️ No: las asignaciones de rol con ámbito de recurso **no se transfieren** con el recurso movido (hay que recrearlas). Las heredadas del nuevo RG sí aplican.
11. Al **transferir una suscripción a otro tenant**, todas las asignaciones RBAC se eliminan.
12. Una asignación en un ámbito hijo **no puede** quitar permisos del padre (por ejemplo Reader en RG no "reduce" a un Contributor de la suscripción).

## Cómo funciona: método para resolver preguntas

```
Paso 1. Lista al usuario y a todos sus grupos.
Paso 2. Para el recurso objetivo, sube por la jerarquía: recurso → RG → sub → MG.
Paso 3. Anota cada rol encontrado en cada nivel.
Paso 4. Une los permisos (suma). Si alguno permite la acción → permitido…
Paso 5. …salvo que exista un lock (ReadOnly/CanNotDelete), una deny assignment o una Policy que lo impida.
Paso 6. Si la acción es de datos (leer blob), comprueba que haya un rol de DATOS o clave/SAS.
```

Herramientas del portal:
- **IAM → Comprobar acceso** (Check access): roles efectivos de un usuario/grupo/SP en ese ámbito.
- **IAM → Asignaciones de roles**: filtro por ámbito "este recurso" o "heredado".
- **Usuario → Asignaciones de roles de Azure**: todas las asignaciones de un usuario.
- **Registro de actividad**: ver "AuthorizationFailed" para diagnosticar.

```bash
az role assignment list --assignee ana@contoso.com --all --include-groups --include-inherited -o table
```

```powershell
Get-AzRoleAssignment -SignInName ana@contoso.com -ExpandPrincipalGroups
```

## Ejemplo resuelto

Estructura:
- MG "Corp" → Sub1 → RG-Web → VM1
- Grupo *Ops* tiene **Reader** en MG "Corp".
- Ana (miembro de *Ops*) tiene **Contributor** en RG-Web.
- Luis tiene **Owner** en Sub1.
- VM1 tiene un lock **CanNotDelete**.

| Pregunta | Respuesta | Por qué |
|---|---|---|
| ¿Ana puede iniciar VM1? | **Sí** | Contributor en RG-Web se hereda a VM1 |
| ¿Ana puede crear un RG en Sub1? | **No** | Solo tiene Reader en la suscripción (heredado del MG) |
| ¿Ana puede asignar Reader a un compañero en RG-Web? | **No** | Contributor no asigna roles |
| ¿Luis puede borrar VM1? | **No** hasta quitar el lock | Owner permite borrar, pero el lock lo impide; Luis sí puede quitar el lock primero |
| ¿Luis puede crear usuarios en Entra? | **No** | Owner es RBAC, no rol de Entra |
| ¿Ana puede leer los blobs de una cuenta en RG-Web usando su cuenta de Entra? | **No** (con solo Contributor) | Falta rol de datos; sí podría listar las claves y usarlas |

## Comparaciones

| Herramienta de diagnóstico | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Check access (IAM)** | Roles efectivos en un ámbito | Rápido, incluye grupos | Primera comprobación |
| **Asignaciones de roles del usuario** | Todas las asignaciones | Vista global | Auditoría |
| **Registro de actividad** | Errores AuthorizationFailed | Muestra la operación exacta denegada | Incidencias |
| **`az role assignment list --include-groups --include-inherited`** | Script | Exportable | Auditorías masivas |

## AZ-104 Exam Tips

- ⭐ **Herencia hacia abajo + suma de permisos.** Nunca se "resta" con otra asignación.
- 🔥 Un **lock** vence a un Owner; el Owner debe **quitar el lock** y luego borrar.
- 🔥 📌 Global Administrator ≠ acceso a suscripciones.
- 🧠 Mover un recurso: las asignaciones hechas en el propio recurso **no viajan**.
- 🧠 Transferir suscripción a otro tenant: se **eliminan** todas las asignaciones RBAC.
- 💻 Usar **Check access** y `az role assignment list --include-inherited`.
- ⚠️ "Reader en la suscripción + Contributor en el RG" = Contributor en ese RG, Reader en el resto.
- ⚠️ Plano de datos: Contributor **no** lee blobs con Entra ID; sí puede **listar claves** de la cuenta.

## Errores comunes

- Creer que asignar Reader en un RG "baja" el nivel de alguien que tiene Contributor en la suscripción.
- Atribuir a RBAC un fallo que en realidad es un lock o una Policy.
- Olvidar las asignaciones por grupo al calcular permisos.

## Preguntas que podrían aparecer

**1.** El grupo G1 tiene Contributor en la suscripción. El usuario U1 pertenece a G1 y tiene Reader en RG1. ¿Puede U1 crear una VM en RG1?
- A) No, Reader en RG1 tiene prioridad · B) Sí, Contributor heredado de la suscripción se suma · C) Solo si se le asigna Owner · D) No, los grupos no heredan

<details><summary>Respuesta</summary>

**B.** Los permisos se acumulan; la asignación de Reader no reduce el Contributor heredado por el grupo.
</details>

**2.** Un usuario con el rol Owner en la suscripción recibe un error al intentar eliminar un grupo de recursos. ¿Cuál es la causa más probable?
- A) Le falta el rol User Access Administrator · B) Existe un bloqueo de recursos en el RG o en un recurso hijo · C) Azure Policy le deniega la lectura · D) Su cuenta es de tipo Guest

<details><summary>Respuesta</summary>

**B.** Los bloqueos impiden la eliminación aunque el usuario sea Owner. Debe eliminar el lock primero.
</details>

**3.** Mueves la VM1 de RG1 a RG2. Antes del movimiento, Pedro tenía Contributor asignado directamente en VM1 y Ana tenía Contributor en RG2. Tras el movimiento, ¿quién puede administrar VM1?
- A) Pedro y Ana · B) Solo Pedro · C) Solo Ana · D) Nadie hasta reasignar

<details><summary>Respuesta</summary>

**C.** Las asignaciones con ámbito en el recurso no se conservan al moverlo; Ana hereda Contributor desde RG2.
</details>

## Relacionado

- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[09 - Roles personalizados de Azure RBAC]]
- [[12 - Bloqueos de recursos (Locks)]]
- [[14 - Grupos de recursos y movimiento de recursos]]
- [[00 - Índice - Identidades y gobernanza]]
