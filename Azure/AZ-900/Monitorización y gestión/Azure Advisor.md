# Azure Advisor

> [!important] Nota
> No está en el curso, pero **sí en el outline oficial del AZ-900** ("Describir herramientas de monitorización: Azure Advisor, Azure Monitor, Azure Service Health"). Lo incluyo breve porque las preguntas lo enfrentan a Monitor y Service Health.

## Concepto

**Azure Advisor** es un **consultor automático y gratuito** que analiza tus recursos y te da **recomendaciones personalizadas** para mejorarlos.

## Características principales

Las recomendaciones se agrupan en **cinco categorías**:

| Categoría | Ejemplo de recomendación |
|---|---|
| **Confiabilidad** (Reliability) | "Activa copias de seguridad en esta VM" |
| **Seguridad** (Security) | "Habilita MFA", "cierra este puerto" |
| **Rendimiento** (Performance) | "Cambia a discos Premium" |
| **Coste** (Cost) | "Apaga o reduce esta VM infrautilizada", "compra instancias reservadas" |
| **Excelencia operativa** (Operational Excellence) | "Añade etiquetas", "crea alertas de Service Health" |

- Es **gratuito**.
- Puedes **posponer o descartar** recomendaciones y configurar **alertas** cuando aparezcan nuevas.
- Se accede desde el portal y ofrece un **Advisor Score**.

## Conceptos que debo memorizar

> [!important]
> - Advisor = **recomendaciones** en **5 categorías**: Confiabilidad, Seguridad, Rendimiento, Coste, Excelencia operativa.
> - Gratuito y personalizado a tus recursos.
> - No aplica nada por sí solo; tú decides.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *recomendación, mejores prácticas, optimizar coste, reducir gasto, mejorar seguridad, VM infrautilizada*.
> - "¿Cómo reduzco costes?" → Advisor (categoría Coste). No confundir con Cost Management (que muestra y analiza el gasto).
> - Trampa: "Advisor te avisa de incidentes de Azure". **No**. Eso es Service Health.
> - Trampa: "Advisor muestra la CPU de mi VM". **No**. Eso es Monitor.

## Ejemplo de pregunta de examen

**Pregunta 1.** Quieres identificar máquinas virtuales infrautilizadas para reducir la factura mensual. ¿Qué herramienta te ofrece esa recomendación?

- A) Azure Monitor
- B) Azure Service Health
- C) Azure Advisor
- D) Azure Policy

**Respuesta: C.** Advisor, en su categoría Coste, detecta recursos infrautilizados.
- A) Monitor muestra las métricas, pero no la recomendación de apagar.
- B) Service Health no analiza coste.
- D) Policy evalúa cumplimiento.

## 🧠 Resumen para el examen

1. Advisor = recomendaciones gratuitas y personalizadas.
2. Cinco categorías: Confiabilidad, Seguridad, Rendimiento, Coste, Excelencia operativa.
3. Recomienda, no ejecuta.
4. Advisor recomienda; Monitor mide; Service Health informa de Azure.

---
