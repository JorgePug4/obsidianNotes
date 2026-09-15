# Azure RBAC

## Concepto

**RBAC** (Role-Based Access Control, control de acceso basado en roles) es el sistema con el que decides **quién** puede hacer **qué** sobre **qué recursos** de Azure. En lugar de dar permisos uno por uno, asignas un **rol** (paquete de permisos) a una identidad sobre un **ámbito**.

Problema que resuelve: dar a todos el rol de Propietario "para que no falle nada" es un riesgo enorme. RBAC permite aplicar el **principio de privilegio mínimo**: cada persona tiene solo lo que necesita.

Para qué se usa: separar responsabilidades (quien administra red no toca bases de datos), dar acceso de solo lectura a auditores, delegar administración de un grupo de recursos a un equipo.

## Características principales

Una **asignación de rol** tiene tres partes. Memorízalas:

```
Entidad de seguridad  +  Rol  +  Ámbito  =  Asignación de rol
(quién)                  (qué)   (dónde)
```

- **Entidad de seguridad** (security principal): usuario, grupo, entidad de servicio o identidad administrada.
- **Definición de rol**: lista de acciones permitidas. Azure trae roles integrados y permite roles personalizados.
- **Ámbito**: grupo de administración, suscripción, grupo de recursos o recurso individual. Se **hereda hacia abajo**.

Roles integrados que debes conocer para AZ-900:

| Rol | Qué permite | Qué NO permite |
|---|---|---|
| **Propietario** (Owner) | Todo, incluida la gestión de acceso de otros | Nada, es el máximo |
| **Colaborador** (Contributor) | Crear y gestionar todos los recursos | **Asignar roles a otros** |
| **Lector** (Reader) | Ver todo | Cambiar nada |
| **Administrador de acceso de usuario** (User Access Administrator) | Gestionar el acceso de usuarios | Gestionar recursos |

- Los permisos son **acumulativos**: si tienes Lector en la suscripción y Colaborador en un RG, en ese RG eres Colaborador.
- RBAC se basa en **permitir**. Un rol no "quita" permisos; con **excepciones de denegación** (deny assignments) se pueden bloquear acciones, pero eso queda fuera de AZ-900.
- RBAC funciona sobre el **plano de control** (gestionar recursos), no sobre el contenido dentro del recurso (por ejemplo, no gestiona las filas de una base de datos).

> [!warning] Propietario vs Colaborador
> La única diferencia relevante para el examen: el **Colaborador no puede dar permisos a otros**. Si la pregunta dice "puede gestionar recursos pero no debe poder asignar roles", la respuesta es Colaborador.

> [!warning] RBAC de Azure vs roles de Microsoft Entra ID
> Los roles de Entra ID (Administrador global, Administrador de usuarios) gestionan el **directorio** (usuarios, grupos, licencias). Los roles de **Azure RBAC** gestionan **recursos de Azure** (VMs, redes, almacenamiento). Un Administrador global de Entra ID no tiene acceso automático a las suscripciones.

## Casos de uso

- Un auditor externo necesita **ver** toda la suscripción sin modificar nada: **Lector** a nivel de suscripción.
- El equipo de desarrollo gestiona su propio RG: **Colaborador** sobre ese RG.
- El responsable de seguridad decide quién accede pero no despliega nada: **Administrador de acceso de usuario**.
- Quieres dar permisos a 50 personas del mismo departamento: asigna el rol a un **grupo**, no a cada usuario.

## Comparaciones

| | RBAC | Azure Policy | Locks |
|---|---|---|---|
| Pregunta | ¿Quién puede? | ¿Qué está permitido crear/configurar? | ¿Se puede borrar por error? |
| Objeto principal | Identidades | Recursos | Recursos |
| Ejemplo | "Ana puede reiniciar VMs del RG X" | "Solo VMs tamaño B" | "No borrar esta VM" |

## Conceptos que debo memorizar

> [!important]
> - Asignación = **Entidad de seguridad + Rol + Ámbito**.
> - Cuatro roles básicos: **Propietario, Colaborador, Lector, Administrador de acceso de usuario**.
> - **Colaborador no asigna roles**.
> - Ámbitos: MG, suscripción, RG, recurso. **Herencia hacia abajo**.
> - Permisos **acumulativos**.
> - **Privilegio mínimo**: dar el rol más restrictivo que permita hacer el trabajo.
> - Mejor práctica: asignar roles a **grupos**, no a usuarios individuales.

## Tips para AZ-900

> [!tip]
> - "Puede hacer todo excepto gestionar el acceso" → **Colaborador**.
> - "Solo necesita consultar" → **Lector**.
> - "Debe poder dar permisos pero no crear recursos" → **Administrador de acceso de usuario**.
> - "El menor privilegio posible" aparece en muchas preguntas: elige siempre el rol **más restrictivo** que cumpla el escenario.
> - Trampa: "un usuario tiene Lector en la suscripción y Colaborador en un RG; ¿puede crear una VM en ese RG?" **Sí**, los permisos se suman.
> - Trampa: "un Propietario de la suscripción no puede borrar un recurso". Causa probable: hay un **lock**, no un problema de RBAC.

## Ejemplo de pregunta de examen

**Pregunta 1.** Un desarrollador debe poder crear y eliminar cualquier recurso de un grupo de recursos, pero no debe poder dar acceso a otras personas. Siguiendo el principio de privilegio mínimo, ¿qué rol le asignas?

- A) Propietario
- B) Colaborador
- C) Lector
- D) Administrador de acceso de usuario

**Respuesta: B.** Colaborador gestiona recursos sin poder asignar roles.
- A) Propietario también podría dar acceso, viola el privilegio mínimo.
- C) Lector no puede crear nada.
- D) Ese rol gestiona acceso, no recursos.

**Pregunta 2.** Asignas el rol Lector a un usuario en la suscripción y el rol Colaborador en el grupo de recursos `RG-Web`. ¿Qué puede hacer el usuario en `RG-Web`?

- A) Solo leer.
- B) Crear y gestionar recursos.
- C) Nada, hay conflicto de roles.
- D) Solo asignar roles.

**Respuesta: B.** Los permisos se acumulan; en `RG-Web` tiene Lector (heredado) más Colaborador.
- A) Ignora la asignación de Colaborador.
- C) RBAC no genera conflictos, suma permisos.
- D) Colaborador no asigna roles.

**Pregunta 3.** ¿Cuál de los siguientes NO es un componente de una asignación de rol en Azure RBAC?

- A) Entidad de seguridad
- B) Definición de rol
- C) Ámbito
- D) Etiqueta

**Respuesta: D.** Las etiquetas organizan recursos; no forman parte de RBAC.
- A, B y C) Son exactamente los tres componentes.

## 🧠 Resumen para el examen

1. RBAC responde "¿quién puede hacer qué y dónde?".
2. Asignación de rol = entidad de seguridad + rol + ámbito.
3. Propietario > Colaborador (sin gestión de acceso) > Lector.
4. Administrador de acceso de usuario solo gestiona permisos.
5. Se hereda hacia abajo y los permisos se acumulan.
6. Privilegio mínimo: el rol más restrictivo que sirva.
7. Asigna a grupos, no a personas.
8. Roles de Entra ID ≠ roles de Azure RBAC.

---
