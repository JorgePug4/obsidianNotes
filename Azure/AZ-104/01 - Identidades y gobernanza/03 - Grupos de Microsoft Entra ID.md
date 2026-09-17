---
tags: [az-104, azure, identidad, entra-id, grupos]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Grupos de Microsoft Entra ID

## ¿Qué es?

Un **grupo** es una colección de usuarios, dispositivos u otros grupos a la que se asignan permisos, licencias y directivas de una sola vez. Es la unidad recomendada de administración: **se asignan roles y licencias a grupos, no a personas**.

## ¿Para qué sirve?

- Asignar roles RBAC ("el grupo Dev-Team es Contributor en RG-Dev").
- Asignar licencias por grupo.
- Definir el ámbito de acceso condicional, SSPR o aplicaciones.
- Automatizar la pertenencia con reglas dinámicas.

## Conceptos clave

- **Tipo de grupo**:
  - **Security**: para permisos, licencias y directivas. Puede contener usuarios, dispositivos, entidades de servicio y otros grupos.
  - **Microsoft 365**: además crea buzón compartido, SharePoint, Teams. Solo contiene usuarios. Puede tener correo.
- **Tipo de pertenencia (membership type)**:
  - **Assigned** (asignada): un administrador añade y quita miembros.
  - **Dynamic User**: la pertenencia se calcula con una **regla** sobre atributos de usuario (`user.department -eq "Ventas"`). Requiere **Entra ID P1**.
  - **Dynamic Device**: regla sobre atributos de dispositivo. Requiere P1. Un grupo no puede ser dinámico de usuarios y dispositivos a la vez.
- **Propietarios (owners)**: pueden administrar miembros sin ser administradores del directorio.
- **Anidamiento**: un grupo de seguridad puede contener otro grupo. Ojo: para **licencias** y algunas funciones el anidamiento no se resuelve (solo miembros directos).
- **Grupos asignables a roles** (*isAssignableToRole*): grupos de seguridad a los que se pueden asignar **roles de Entra**; se decide al crearlos y no se puede cambiar después. Máximo 500 por tenant.
- **Grupos de seguridad con correo** y listas de distribución: se crean en Exchange, no en Entra.
- **Autoservicio**: los usuarios pueden crear grupos M365 (configurable) y solicitar unirse a grupos con aprobación del propietario.

## Cómo funciona

Reglas dinámicas (sintaxis que debes reconocer):

```
(user.department -eq "Ventas") and (user.country -eq "ES")
(user.jobTitle -contains "Ingeniero") or (user.userType -eq "Guest")
(device.deviceOSType -eq "Windows") and (device.deviceOSVersion -startsWith "10")
user.assignedPlans -any (assignedPlan.servicePlanId -eq "..." -and assignedPlan.capabilityStatus -eq "Enabled")
```

- Operadores: `-eq`, `-ne`, `-startsWith`, `-notStartsWith`, `-contains`, `-notContains`, `-match`, `-notMatch`, `-in`, `-notIn`, `-any`, `-all`.
- La evaluación es **asíncrona**; tras crear la regla puede tardar minutos (o más en tenants grandes). El estado de procesamiento se ve en la vista general del grupo.
- Cuando un grupo pasa de asignado a dinámico, **se eliminan los miembros asignados** y se recalculan.

## Componentes

| Elemento | Detalle |
|---|---|
| Nombre, descripción | Obligatorio el nombre |
| Tipo de grupo | Security / Microsoft 365 |
| Tipo de pertenencia | Assigned / Dynamic User / Dynamic Device |
| Propietarios | Recomendado al menos uno |
| Miembros | Usuarios, grupos, dispositivos, SP (según tipo) |
| Roles de Entra asignables | Solo si se marcó al crear |
| Licencias | [[04 - Licencias en Microsoft Entra ID]] |
| Azure role assignments | Roles RBAC del grupo |

## Configuración relevante para el examen

- **Rol mínimo** para crear y administrar grupos: **Groups Administrator** (User Administrator también puede).
- **Grupos dinámicos** requieren P1 para cada usuario miembro.
- **Cambiar M365 ↔ Security**: no se puede convertir el tipo de grupo una vez creado.
- **Eliminar un grupo M365**: recuperable 30 días. Un grupo de **seguridad** eliminado **no** se puede restaurar (a fecha de estas notas; verifica en Learn si cambia).
- Comandos:

```bash
az ad group create --display-name "Dev-Team" --mail-nickname devteam
az ad group member add --group "Dev-Team" --member-id <objectId>
az ad group member list --group "Dev-Team" -o table
```

## Ejemplo

El departamento de Ventas cambia constantemente de personal. En vez de mantener a mano el grupo "Ventas-App-Access", se crea un grupo dinámico con la regla `user.department -eq "Ventas"`. Al actualizar el atributo departamento en el usuario (o en AD local con sincronización), la pertenencia y por tanto la licencia y los permisos se ajustan solos.

## Comparaciones

| Tipo de grupo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Security, asignado** | Permisos y licencias controlados manualmente | Simple, sin licencia extra | Equipos estables, grupos pequeños |
| **Security, dinámico** | Pertenencia automática por atributos | Cero mantenimiento | Departamentos, ubicaciones, tipos de dispositivo (necesita P1) |
| **Microsoft 365** | Colaboración (Teams, SharePoint, buzón) | Incluye recursos de colaboración | Equipos de trabajo, proyectos |
| **Asignable a roles** | Delegar roles de Entra a un grupo | Gestión de administradores por grupo | Equipos de administradores (máx. 500 grupos) |

## 💻 Laboratorio: grupos dinámicos

1. Crear tres usuarios con `department = IT` y uno con `department = HR`.
2. Crear un grupo de seguridad **Dynamic User** con la regla `(user.department -eq "IT")`.
3. Esperar a que el estado de pertenencia dinámica sea "Actualizado" y verificar los 3 miembros.
4. Cambiar el departamento del usuario de HR a IT y confirmar que entra en el grupo.
5. Asignar al grupo el rol **Reader** en un grupo de recursos y comprobar el acceso con uno de los usuarios.

## AZ-104 Exam Tips

- ⭐ Asignar roles y licencias **a grupos**, nunca a usuarios sueltos.
- 🔥 🧠 **Grupos dinámicos = Entra ID P1.**
- 🧠 Regla dinámica: sintaxis `user.atributo -operador "valor"`; se combinan con `and` / `or`.
- 🧠 Un grupo no puede ser dinámico de usuarios y de dispositivos a la vez.
- 🧠 Grupos asignables a roles: se marca **al crearlos**; límite **500**.
- 📌 Security vs Microsoft 365: si la pregunta menciona Teams, SharePoint o buzón compartido → M365. Si menciona permisos, RBAC o licencias → Security.
- ⚠️ Cambiar de asignado a dinámico **borra** los miembros actuales.
- ⚠️ Licencias por grupo **no** respetan grupos anidados.

## Errores comunes

- Intentar usar un grupo M365 para asignar un rol de Entra (solo grupos de seguridad asignables a roles).
- Esperar que la pertenencia dinámica sea instantánea.
- Crear el grupo sin marcar "asignable a roles" y descubrir después que no se puede cambiar.

## Preguntas que podrían aparecer

**1.** Necesitas que todos los usuarios cuyo atributo *Country* sea "MX" reciban automáticamente una licencia de Microsoft 365 E3, con el mínimo esfuerzo administrativo. ¿Qué haces?
- A) Grupo de seguridad asignado + licencia por grupo · B) Grupo dinámico de usuarios con regla sobre country + licencia por grupo · C) Script semanal de PowerShell · D) Grupo Microsoft 365 dinámico

<details><summary>Respuesta</summary>

**B.** El grupo dinámico mantiene la pertenencia solo y la licencia por grupo se aplica automáticamente. D también podría funcionar, pero un grupo M365 crea recursos de colaboración innecesarios; la opción esperada es el grupo de seguridad dinámico.
</details>

**2.** Creas un grupo dinámico con la regla `user.department -eq "Finance"`, pero tras 2 minutos no aparece ningún miembro. ¿Qué es lo más probable?
- A) La regla es incorrecta · B) La evaluación dinámica aún está en proceso · C) Los usuarios no tienen licencia P2 · D) Hay que añadir los miembros manualmente

<details><summary>Respuesta</summary>

**B.** El procesamiento dinámico es asíncrono y puede tardar. La regla es válida; P2 no es necesario (P1 sí); en grupos dinámicos no se añaden miembros a mano.
</details>

## Relacionado

- [[02 - Usuarios de Microsoft Entra ID]]
- [[04 - Licencias en Microsoft Entra ID]]
- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[07 - Unidades administrativas y dispositivos]]
- [[00 - Índice - Identidades y gobernanza]]
