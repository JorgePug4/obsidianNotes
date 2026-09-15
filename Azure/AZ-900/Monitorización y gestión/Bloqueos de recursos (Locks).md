# Bloqueos de recursos (Locks)

## Concepto

Un **bloqueo de recursos** (resource lock) es una protección que impide **borrar** o **modificar** un recurso, un grupo de recursos o una suscripción **por accidente**, sin importar los permisos RBAC de quien lo intente.

Problema que resuelve: un administrador con rol Propietario borra por error el grupo de recursos de producción un viernes por la tarde. RBAC no lo evitaría porque el Propietario tiene permiso. El lock sí.

Para qué se usa: proteger recursos críticos (bases de datos, redes virtuales, cuentas de almacenamiento con datos de negocio).

## Características principales

Solo existen **dos tipos**:

| Tipo | Nombre en el portal | Qué impide | Qué permite |
|---|---|---|---|
| **CanNotDelete** | Eliminar | Borrar el recurso | Leer y **modificar** |
| **ReadOnly** | Solo lectura | Borrar **y modificar** | Solo leer |

- Se aplican a **suscripción, grupo de recursos o recurso**.
- Se **heredan**: un lock en el grupo de recursos protege todos sus recursos.
- Aplican a **todos los usuarios y roles**, incluidos Propietarios.
- Para borrar o cambiar el recurso hay que **quitar el lock primero** (necesitas permisos de Propietario o Administrador de acceso de usuario para gestionar locks).
- El lock más restrictivo gana si hay varios.

> [!warning] ReadOnly tiene efectos secundarios inesperados
> Un lock ReadOnly puede bloquear operaciones que parecen "de lectura" pero que internamente escriben. Ejemplo clásico: listar claves de una cuenta de almacenamiento o iniciar/detener una VM. Para AZ-900 basta con saber que **ReadOnly congela el recurso**.

> [!warning] Locks vs RBAC
> RBAC decide **si tienes permiso**. El lock decide **si la operación está permitida sobre el recurso**, sin importar tu permiso. Ambos deben "dejar pasar" para que la acción ocurra.

## Casos de uso

- Red virtual principal de producción: **CanNotDelete**, porque hay que poder añadir subredes pero nunca borrarla.
- Cuenta de almacenamiento con archivos legales: **ReadOnly**, nadie debe cambiarla ni borrarla.
- Grupo de recursos completo de producción: **CanNotDelete** heredado a todos sus recursos.

## Comparaciones

| Escenario | CanNotDelete | ReadOnly |
|---|---|---|
| Puedo cambiar el tamaño de la VM | Sí | No |
| Puedo borrar la VM | No | No |
| Puedo ver la VM | Sí | Sí |
| Puedo añadir una etiqueta | Sí | No |

## Conceptos que debo memorizar

> [!important]
> - Dos tipos: **CanNotDelete** (Eliminar) y **ReadOnly** (Solo lectura).
> - Aplican a **todos**, incluidos Propietarios.
> - Ámbitos: suscripción, grupo de recursos, recurso. Se heredan.
> - Para borrar hay que **quitar el lock primero**.
> - Protegen contra **accidentes**, no contra atacantes (un atacante con permisos de Propietario podría quitar el lock).

## Tips para AZ-900

> [!tip]
> - Palabras clave: *accidental, por error, evitar eliminación, proteger, incluso los administradores*.
> - "Debe poder modificarse pero no borrarse" → **CanNotDelete**.
> - "No debe cambiarse de ninguna forma" → **ReadOnly**.
> - Trampa: "Un usuario con rol Propietario no puede borrar un recurso. ¿Por qué?" → Hay un lock. No es RBAC.
> - Trampa: "¿Cuántos tipos de lock existen?" → **Dos**. Si una opción menciona un tercero (por ejemplo "CanNotModify"), es falsa.
> - Trampa: "¿El lock impide que un Propietario lo elimine?" → El Propietario **sí puede quitar el lock** y luego borrar. El lock frena el accidente, no la intención.

## Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas evitar que se elimine accidentalmente una red virtual, pero los administradores deben poder seguir añadiendo subredes. ¿Qué debes configurar?

- A) Un bloqueo ReadOnly
- B) Un bloqueo CanNotDelete
- C) Una política de Azure con efecto Audit
- D) Quitar el rol Propietario a todos

**Respuesta: B.** CanNotDelete impide borrar pero permite modificar.
- A) ReadOnly impediría añadir subredes.
- C) Audit solo registra, no impide.
- D) Impracticable y no protege contra accidentes de quien conserve permisos.

**Pregunta 2.** Un usuario tiene el rol Propietario de una suscripción. Intenta borrar una cuenta de almacenamiento y recibe un error. ¿Cuál es la causa más probable?

- A) No tiene permisos suficientes.
- B) Existe un bloqueo de recursos en la cuenta o en su grupo de recursos.
- C) La cuenta está en otra región.
- D) Azure Policy no permite borrar recursos.

**Respuesta: B.** Los locks bloquean incluso a Propietarios.
- A) Propietario tiene todos los permisos.
- C) La región no afecta al borrado.
- D) Policy no se usa para impedir borrados.

## 🧠 Resumen para el examen

1. Lock = protección contra borrado o modificación accidental.
2. Solo dos tipos: CanNotDelete y ReadOnly.
3. ReadOnly también impide modificar.
4. Aplican a todos los roles, Propietario incluido.
5. Se heredan de grupo de recursos y suscripción.
6. Para borrar: quitar el lock y luego borrar.
7. "Propietario no puede borrar" = hay un lock.

---
