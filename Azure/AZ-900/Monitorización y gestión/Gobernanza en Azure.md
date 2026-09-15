# Gobernanza en Azure

## Concepto

La **gobernanza** es el conjunto de reglas, procesos y herramientas con las que una organización controla **qué se puede hacer en Azure, quién puede hacerlo y cómo**. Sin gobernanza, cada equipo crea recursos como quiere: regiones no permitidas, VMs demasiado caras, sin etiquetas, sin control de costes.

Problema que resuelve: la nube es fácil de usar y esa facilidad genera desorden. La gobernanza pone barandillas para mantener **cumplimiento, consistencia y control de costes** sin frenar a los equipos.

Para qué se usa: aplicar estándares corporativos a gran escala, cumplir regulaciones (GDPR, ISO, HIPAA) y evitar errores caros.

## Características principales

Azure organiza los recursos en una **jerarquía** y la gobernanza se aplica de arriba hacia abajo (**herencia**):

```
Grupos de administración (Management Groups)
   └── Suscripciones (Subscriptions)
         └── Grupos de recursos (Resource Groups)
               └── Recursos (Resources)
```

- Lo que aplicas en un nivel superior **se hereda** por todos los niveles inferiores.
- Un **grupo de administración** puede contener otros grupos de administración y suscripciones (hasta 6 niveles de profundidad, sin contar el raíz).
- Herramientas de gobernanza que evalúa el AZ-900:

| Herramienta | Pregunta que responde |
|---|---|
| [[Azure Policy]] | ¿**Qué** se puede crear y cómo debe estar configurado? |
| [[Azure RBAC]] | ¿**Quién** puede hacer **qué** sobre un recurso? |
| [[Bloqueos de recursos (Locks)]] | ¿Cómo evito **borrados o cambios accidentales**? |
| Etiquetas (Tags) | ¿Cómo **organizo y clasifico** recursos (coste, dueño, entorno)? |
| Microsoft Purview | ¿Cómo **gobierno los datos** (catálogo, clasificación, linaje) en Azure y fuera de Azure? |
| Grupos de administración | ¿Cómo aplico políticas y permisos a **muchas suscripciones a la vez**? |

> [!warning] Confusión típica: Policy vs RBAC
> **Azure Policy** controla propiedades de los **recursos** (región, SKU, etiquetas). **RBAC** controla permisos de las **personas e identidades**. Si la pregunta habla de "usuarios", "permisos" o "acceso", es RBAC. Si habla de "configuración", "estándar", "solo se permite crear en…", es Policy.

## Casos de uso

- Una empresa con 40 suscripciones quiere que **ninguna** cree recursos fuera de Europa: grupo de administración + Azure Policy.
- Quieres que solo el equipo de finanzas vea el coste, pero no toque nada: RBAC con rol Lector a nivel de suscripción.
- Un becario no puede borrar la base de datos de producción por error: bloqueo `CanNotDelete`.
- Auditoría quiere saber dónde vive la información personal de clientes: Microsoft Purview.

## Comparaciones

| Necesidad | Herramienta correcta | No confundir con |
|---|---|---|
| Forzar etiqueta "CostCenter" en todo recurso nuevo | Azure Policy | Tags solos (no se pueden forzar sin Policy) |
| Dar permiso de administrador de VMs a un grupo | RBAC | Policy |
| Impedir borrado de un recurso aunque seas Propietario | Lock | RBAC (el Propietario podría borrarlo sin lock) |
| Aplicar la misma regla a 15 suscripciones | Grupo de administración | Hacerlo suscripción por suscripción |
| Descubrir y clasificar datos sensibles | Microsoft Purview | Azure Policy |

## Conceptos que debo memorizar

> [!important]
> - Jerarquía: **Grupo de administración → Suscripción → Grupo de recursos → Recurso**.
> - La gobernanza se **hereda hacia abajo**.
> - **Policy = qué / cómo**. **RBAC = quién**. **Lock = proteger contra accidentes**.
> - **Purview = gobernanza de datos** (catálogo, clasificación, cumplimiento), no de recursos.
> - Los grupos de administración existen para gestionar **múltiples suscripciones**.

## Tips para AZ-900

> [!tip]
> - Cuando la pregunta diga "**varias suscripciones**" y "**aplicar a todas**", la respuesta suele ser **grupo de administración**.
> - Palabras clave para Policy: *cumplimiento, estándar, obligar, solo permitir, auditar configuración, no conforme*.
> - Palabras clave para RBAC: *usuario, grupo, permiso, acceso, rol, privilegio mínimo*.
> - Palabras clave para Locks: *accidental, borrado, modificación, proteger, incluso administradores*.
> - Palabras clave para Purview: *datos, catálogo, clasificar, dónde están, linaje, sensible*.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu empresa tiene 20 suscripciones de Azure. Necesitas aplicar la misma política de cumplimiento a todas ellas con el menor esfuerzo administrativo. ¿Qué debes usar?

- A) Grupos de recursos
- B) Grupos de administración
- C) Etiquetas
- D) Azure Blueprints

**Respuesta: B.** Los grupos de administración agrupan suscripciones y todo lo que asignas al grupo (políticas, RBAC) se hereda por todas ellas.
- A) Un grupo de recursos vive **dentro** de una suscripción, no puede abarcar varias.
- C) Las etiquetas clasifican, no aplican políticas.
- D) Blueprints está en retirada y, además, su función era desplegar entornos, no gestionar herencia entre suscripciones.

**Pregunta 2.** Un administrador asigna una política en un grupo de administración. ¿Qué ocurre con las suscripciones que están dentro de ese grupo?

- A) Nada, la política solo aplica al grupo de administración.
- B) Heredan la política automáticamente.
- C) Deben aceptar la política manualmente.
- D) Solo la heredan las suscripciones creadas después.

**Respuesta: B.** La herencia es automática y hacia abajo en toda la jerarquía.
- A y C) Contradicen el modelo de herencia.
- D) La herencia aplica a las existentes y a las nuevas.

## 🧠 Resumen para el examen

1. Gobernanza = control de qué, quién y cómo en Azure.
2. Jerarquía de 4 niveles: Management Group → Subscription → Resource Group → Resource.
3. Herencia siempre hacia abajo.
4. Policy controla recursos; RBAC controla personas; Locks protegen de accidentes.
5. Grupos de administración = gobernar muchas suscripciones a la vez.
6. Purview = gobernanza de datos, multi-nube y on-premises.
7. Las etiquetas organizan y ayudan al coste, pero no se fuerzan solas: necesitan Policy.

---
