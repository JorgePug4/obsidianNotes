---
tags: [az-900, azure, costes, calculadora-precios, tco]
modulo: Costes y herramientas
peso_examen: Alto
---

# Calculadora de precios y calculadora de TCO

## Concepto

Azure ofrece **dos calculadoras gratuitas y públicas** (no necesitan cuenta) que responden a preguntas distintas:

- **Calculadora de precios** (Pricing Calculator): *¿cuánto me costará al mes esta solución en Azure?*
- **Calculadora de TCO** (Total Cost of Ownership): *¿cuánto ahorraré a lo largo de varios años si migro mi infraestructura local a Azure?*

**Problema que resuelve:** antes de desplegar nada, necesitas un presupuesto; y antes de decidir migrar, necesitas comparar el coste real de tu centro de datos (hardware, electricidad, personal, licencias) con lo que costaría en la nube.

**Para qué se utiliza:** planificación de costes, presupuestos de proyecto y justificación de la migración ante dirección.

## Características principales

### Calculadora de precios

- Eliges **servicios** (VM, almacenamiento, SQL, App Service…), su **región**, **tamaño/nivel**, **cantidad**, **horas de uso** y **opciones de compra** (pago por uso, reserva, Hybrid Benefit).
- Devuelve una **estimación mensual** (y anual) del coste.
- Permite **guardar y compartir** estimaciones y **exportar** a Excel.
- Muestra precios por región, así que sirve para comparar dónde es más barato.
- Es una **estimación**: el coste real depende del consumo.

### Calculadora de TCO

- Introduces tu **infraestructura actual**: servidores, bases de datos, almacenamiento, red, y sus costes asociados: **electricidad, licencias, personal de TI, mantenimiento, espacio**.
- Puedes ajustar **suposiciones** (precio del kWh, salario, coste del hardware).
- Compara el coste de mantener todo **on-premises** frente a ejecutarlo en **Azure** durante **1 a 5 años**.
- Genera un **informe** con el ahorro estimado, desglosado por categoría (cómputo, datacenter, red, almacenamiento, personal).
- Se usa para tomar la decisión de migrar y justificarla.

### Comparativa

| | Calculadora de precios | Calculadora de TCO |
|---|---|---|
| Pregunta que responde | ¿Cuánto costará esta solución en Azure? | ¿Cuánto ahorro migrando a Azure? |
| Entrada | Servicios de Azure que planeas usar | Tu infraestructura y costes **actuales** |
| Salida | Coste mensual/anual estimado | Comparación on-premises vs Azure a varios años |
| Horizonte | Mes / año | 1 a 5 años |
| Momento | Antes de desplegar o al diseñar | Antes de decidir migrar |
| Necesita cuenta | No | No |

## Casos de uso

- Un arquitecto diseña una solución con 4 VMs, una base de datos y 2 TB de blobs y necesita el presupuesto mensual: **calculadora de precios**.
- El CIO pregunta si compensa cerrar el CPD en tres años: **calculadora de TCO**.
- Comparar si la misma VM es más barata en West Europe o en North Europe: **calculadora de precios**.

## Comparaciones

| Necesidad | Herramienta | No confundir con |
|---|---|---|
| Estimar coste de una solución nueva | Calculadora de precios | Cost Management (muestra gasto real, no estimaciones) |
| Comparar on-premises vs Azure | Calculadora de TCO | Calculadora de precios (no conoce tus costes locales) |
| Ver cuánto he gastado este mes | [[03 - Microsoft Cost Management y etiquetas|Cost Management]] | Calculadoras (solo estiman) |
| Recomendaciones para ahorrar | [[Azure Advisor]] | Calculadoras |

## Conceptos que debo memorizar

> [!important]
> - **Calculadora de precios** = estimar el coste **futuro** de servicios de Azure.
> - **Calculadora de TCO** = comparar el coste de **on-premises vs Azure** a varios años, incluyendo **electricidad, personal y licencias**.
> - Ambas son **gratuitas**, **públicas** y dan **estimaciones**, no facturas.
> - Para el gasto **real** se usa **Cost Management**.

## Tips para AZ-900

> [!tip]
> - "Estimar el coste mensual de una solución" → **calculadora de precios**.
> - "Comparar con el centro de datos actual", "ahorro a tres años", "costes de electricidad y personal" → **TCO**.
> - "Cuánto llevo gastado" → **Cost Management**, no una calculadora.
> - Trampa: "La calculadora de TCO estima el coste de nuevos servicios de Azure". **Falso**; compara infraestructuras.
> - Trampa: "La calculadora de precios requiere una suscripción". **Falso**; es pública.

## Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa quiere saber cuánto dinero ahorraría durante los próximos cinco años si trasladara sus 50 servidores físicos a Azure, teniendo en cuenta electricidad y personal. ¿Qué herramienta debe usar?

- A) Calculadora de precios
- B) Calculadora de TCO
- C) Microsoft Cost Management
- D) Azure Advisor

**Respuesta: B.** La calculadora de TCO compara el coste total local con el de Azure a varios años.
- A) Solo estima el coste de servicios de Azure, sin conocer los costes locales.
- C) Muestra gasto real de recursos ya desplegados.
- D) Recomienda optimizaciones sobre recursos existentes.

**Pregunta 2.** Necesitas presentar un presupuesto mensual aproximado para una solución que aún no has desplegado, compuesta por tres máquinas virtuales y una base de datos SQL. ¿Qué herramienta utilizas?

- A) Calculadora de TCO
- B) Microsoft Cost Management
- C) Calculadora de precios
- D) Azure Monitor

**Respuesta: C.** La calculadora de precios estima el coste de servicios que planeas usar.
- A) Compara on-premises con Azure, no presupuesta soluciones nuevas.
- B) Requiere recursos desplegados para mostrar gasto.
- D) Monitoriza rendimiento, no costes.

## 🧠 Resumen para el examen

1. Calculadora de precios = estimar coste mensual de servicios de Azure antes de desplegar.
2. Calculadora de TCO = comparar coste local vs Azure a 1–5 años, incluyendo electricidad, licencias y personal.
3. Ambas gratuitas y públicas; solo estiman.
4. Gasto real → Cost Management. Recomendaciones → Advisor.

---
