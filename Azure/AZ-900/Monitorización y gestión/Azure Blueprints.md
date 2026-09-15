# Azure Blueprints

> [!warning] Relevancia para AZ-900: BAJA
> Microsoft está retirando Azure Blueprints (Preview). Fases de restricción desde el 31 de julio de 2026 y **retiro completo el 31 de enero de 2027**. El servicio ya **no aparece en el outline oficial** del AZ-900 de 2026. Su sustituto es **Deployment Stacks** con **Template Specs**, que tampoco entra en el AZ-900. Lee esta nota para entender el concepto por si aparece en material antiguo, pero no inviertas tiempo en memorizar detalles.

## Concepto

**Azure Blueprints** permitía definir un **paquete repetible de entorno**: grupos de recursos, asignaciones de roles, asignaciones de políticas y plantillas ARM, todo junto, para desplegar suscripciones nuevas que cumplieran los estándares de la organización desde el minuto uno.

Problema que resolvía: cada nueva suscripción necesitaba la misma configuración base (redes, políticas, roles). Blueprints lo empaquetaba y lo desplegaba de una vez.

## Características principales

- **Artefactos** que podía incluir: grupos de recursos, asignaciones de rol, asignaciones de política, plantillas ARM.
- Mantenía una **relación viva** entre la definición y lo desplegado (a diferencia de una plantilla ARM, que se olvida del recurso tras desplegarlo).
- Tenía **versionado**.

## Comparaciones

| | Blueprints | Plantilla ARM / Bicep | Azure Policy |
|---|---|---|---|
| Propósito | Desplegar un **entorno completo** con políticas y roles | Desplegar **recursos** | Evaluar **cumplimiento** |
| Incluye RBAC y Policy | Sí | No | Solo Policy |
| Relación con lo desplegado | La conserva | No la conserva | No aplica |
| Estado 2026 | **En retirada** | Vigente | Vigente |

## Conceptos que debo memorizar

> [!important]
> - Blueprints = plantilla de **entorno** (recursos + roles + políticas), no solo de recursos.
> - Está **en retirada**; sustituto: Deployment Stacks + Template Specs.
> - Si una pregunta antigua ofrece Blueprints como respuesta para "desplegar un entorno estándar repetible con políticas y RBAC incluidos", esa era la respuesta esperada.

## Tips para AZ-900

> [!tip]
> - En exámenes actuales lo más probable es que **no aparezca**. Si aparece como distractor, descártalo para preguntas sobre cumplimiento (Policy), permisos (RBAC) o monitorización.
> - Diferencia con ARM: ARM despliega recursos; Blueprints desplegaba recursos **más** gobernanza.

## Ejemplo de pregunta de examen

**Pregunta 1.** (Estilo antiguo) ¿Qué servicio permitía definir un conjunto repetible de grupos de recursos, asignaciones de rol y políticas para desplegar en nuevas suscripciones?

- A) Azure Policy
- B) Azure Blueprints
- C) Azure Monitor
- D) Azure Arc

**Respuesta: B.**
- A) Policy solo cubre políticas, no roles ni grupos de recursos.
- C) Monitor observa, no despliega.
- D) Arc gestiona recursos fuera de Azure.

## 🧠 Resumen para el examen

1. Blueprints empaquetaba recursos + RBAC + Policy en un despliegue repetible.
2. Está en retirada (retiro final: enero 2027).
3. No aparece en el outline actual del AZ-900.
4. Si aparece en material viejo, distínguelo de ARM (solo recursos) y de Policy (solo cumplimiento).

---
