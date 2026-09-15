# Azure Policy

## Concepto

**Azure Policy** es el servicio que te permite **definir reglas** sobre cómo deben ser los recursos y **evaluar** si los recursos existentes y nuevos las cumplen. Es el "reglamento interno" de tu nube.

Problema que resuelve: sin Policy, un equipo puede crear una VM en Brasil cuando la ley te obliga a mantener datos en la UE, o desplegar una VM de 64 núcleos cuando el presupuesto solo permite tamaños pequeños.

Para qué se usa: garantizar **cumplimiento** (compliance), estandarizar configuraciones y detectar recursos **no conformes**.

## Características principales

- Una **definición de política** describe la regla y el **efecto** (qué pasa si se incumple).
- Una **iniciativa** (initiative) es un **conjunto de políticas** que se asignan juntas, por ejemplo "Cumplimiento ISO 27001".
- Se **asigna** a un ámbito: grupo de administración, suscripción o grupo de recursos. Se hereda hacia abajo.
- Evalúa **recursos nuevos** (puede bloquear su creación) y **recursos existentes** (los marca como no conformes).
- Efectos que debes reconocer para AZ-900:

| Efecto | Qué hace |
|---|---|
| **Deny** | Impide crear o modificar el recurso si incumple la regla |
| **Audit** | Permite el recurso pero lo marca como **no conforme** en el informe |
| **Append / Modify** | Añade o cambia propiedades (por ejemplo, agrega una etiqueta) |
| **DeployIfNotExists** | Despliega algo si falta (por ejemplo, un agente de monitorización) |

- Azure incluye cientos de **políticas integradas** (built-in) listas para usar.
- Puedes ver un **panel de cumplimiento** con el porcentaje de recursos conformes.

> [!warning] Policy no gestiona permisos de usuarios
> Si la pregunta menciona "un usuario intenta crear una VM y no tiene permiso", eso es **RBAC**. Si dice "un usuario con permisos crea una VM pero Azure la rechaza porque está en una región no permitida", eso es **Policy**.

> [!warning] Policy evalúa lo existente, pero no lo borra
> Al asignar una política, los recursos que ya incumplen quedan marcados como **no conformes**. Azure no los elimina ni los modifica por sí solo (salvo que uses tareas de remediación, tema de AZ-104).

## Casos de uso

- **Restringir regiones**: solo permitir `westeurope` y `northeurope`.
- **Restringir tamaños de VM**: solo la serie B y D.
- **Forzar etiquetas**: todo recurso debe llevar `Environment` y `Owner`.
- **Auditar cifrado**: marcar como no conforme cualquier cuenta de almacenamiento sin HTTPS obligatorio.
- **Iniciativa de cumplimiento**: asignar de una vez todas las políticas necesarias para PCI-DSS.

## Comparaciones

| | Azure Policy | Azure RBAC | Locks |
|---|---|---|---|
| Controla | Propiedades y configuración de **recursos** | **Quién** accede y qué acciones puede ejecutar | Borrado/modificación **accidental** |
| Responde a | "¿Está permitido este recurso así?" | "¿Puede esta persona hacer esto?" | "¿Puede alguien borrar esto por error?" |
| Ejemplo | "Solo VMs en Europa" | "Ana es Colaboradora de este RG" | "Esta VNet no se puede borrar" |
| Ámbito | MG, suscripción, RG | MG, suscripción, RG, recurso | Suscripción, RG, recurso |

## Conceptos que debo memorizar

> [!important]
> - **Definición** = la regla. **Iniciativa** = grupo de definiciones. **Asignación** = aplicarla a un ámbito.
> - Efectos clave: **Deny** (bloquea), **Audit** (registra sin bloquear).
> - Se hereda de grupo de administración hacia abajo.
> - Evalúa recursos **nuevos y existentes**.
> - Tiene **políticas integradas** y permite **políticas personalizadas**.
> - Se usa para **cumplimiento**, no para permisos.

## Tips para AZ-900

> [!tip]
> - Si ves "**iniciativa**", piensa "muchas políticas empaquetadas para un objetivo de cumplimiento".
> - "**No conforme**" (non-compliant) siempre apunta a Azure Policy.
> - "**Solo se pueden crear recursos en la región X**" → Policy, nunca RBAC.
> - Pregunta trampa: "¿Azure Policy puede impedir que un usuario inicie sesión?" **No**. Eso es Microsoft Entra ID / Acceso condicional.
> - Pregunta trampa: "¿Una política asignada a una suscripción afecta a un grupo de recursos que se cree la semana que viene?" **Sí**, por herencia.

## Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas asegurarte de que todos los recursos nuevos de una suscripción incluyan la etiqueta `CostCenter`. Los recursos sin la etiqueta no deben poder crearse. ¿Qué debes usar?

- A) Azure RBAC
- B) Un bloqueo de recursos
- C) Azure Policy con efecto Deny
- D) Azure Advisor

**Respuesta: C.** Policy con Deny rechaza la creación de recursos que no cumplan la regla.
- A) RBAC controla quién puede crear, no cómo debe estar configurado lo creado.
- B) Un lock protege contra borrado o modificación, no valida etiquetas.
- D) Advisor solo recomienda, no impone.

**Pregunta 2.** Asignas una política que exige que todas las cuentas de almacenamiento usen HTTPS. Ya existen 10 cuentas que no lo usan. ¿Qué ocurre con esas 10 cuentas?

- A) Azure las elimina.
- B) Azure las modifica para usar HTTPS.
- C) Se marcan como no conformes.
- D) La política no se puede asignar mientras existan.

**Respuesta: C.** Policy evalúa los recursos existentes y los reporta como no conformes.
- A y B) Policy no borra ni cambia recursos existentes por sí sola.
- D) La asignación es posible aunque existan recursos que incumplan.

**Pregunta 3.** ¿Qué es una iniciativa de Azure Policy?

- A) Una política que solo audita.
- B) Un conjunto de definiciones de políticas agrupadas para un objetivo común.
- C) Un rol personalizado de RBAC.
- D) Una plantilla ARM.

**Respuesta: B.**
- A) Eso es una definición con efecto Audit.
- C) Los roles pertenecen a RBAC.
- D) Las plantillas ARM despliegan recursos, no evalúan cumplimiento.

## 🧠 Resumen para el examen

1. Policy = reglas de configuración y cumplimiento sobre recursos.
2. Definición → Iniciativa (agrupa) → Asignación (a un ámbito).
3. Deny bloquea, Audit solo registra.
4. Evalúa recursos nuevos y existentes; los existentes quedan "no conformes".
5. Se hereda desde grupo de administración.
6. Casos típicos: regiones permitidas, tamaños de VM, etiquetas obligatorias.
7. No gestiona quién accede: eso es RBAC.

---
