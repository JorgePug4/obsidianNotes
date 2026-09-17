---
tags: [az-900, azure, costes, cost-management, etiquetas, tags]
modulo: Costes y herramientas
peso_examen: Alto
---

# Microsoft Cost Management y etiquetas

## Concepto

**Microsoft Cost Management** (antes *Cost Management + Billing*) es la herramienta del portal para **ver, analizar, presupuestar y alertar** sobre el gasto **real** de Azure. Las **etiquetas** (tags) son pares **nombre:valor** que añades a los recursos para **organizarlos y atribuir** el coste a un equipo, proyecto o entorno.

**Problema que resuelve:** con el modelo de consumo, la factura cambia cada mes. Sin visibilidad y sin límites, el gasto se descontrola; sin etiquetas, nadie sabe qué parte de la factura corresponde a cada departamento.

**Para qué se utiliza:**
- Saber cuánto se ha gastado y en qué.
- Pronosticar el gasto del mes.
- Fijar **presupuestos** y recibir **alertas** al acercarse al límite.
- Repartir el coste entre equipos (chargeback / showback).

## Características principales

### Microsoft Cost Management

| Función | Qué hace |
|---|---|
| **Análisis de costes** (Cost analysis) | Gráficos y tablas del gasto acumulado y diario, filtrables por suscripción, grupo de recursos, servicio, región o **etiqueta** |
| **Presupuestos** (Budgets) | Defines un límite de gasto por periodo y ámbito |
| **Alertas** | Aviso por correo cuando el gasto real o previsto alcanza un porcentaje del presupuesto; también alertas de crédito y de límite de gasto |
| **Previsión** (Forecast) | Proyección del gasto hasta fin de periodo según la tendencia |
| **Exportaciones** | Envío programado de datos de coste a una cuenta de almacenamiento |
| **Recomendaciones** | Enlaza con las recomendaciones de coste de [[Azure Advisor]] |

- Se accede desde el portal y funciona en cualquier **ámbito** de la jerarquía: grupo de administración, suscripción o grupo de recursos.
- Un **presupuesto no detiene el gasto**; solo avisa. (Puede combinarse con automatizaciones, pero eso no entra en el examen.)
- Es **gratuito** para Azure.

### Etiquetas (tags)

- Par **nombre : valor**, por ejemplo `Entorno: Producción`, `CentroDeCoste: 1234`, `Propietario: ana@empresa.com`.
- Se aplican a **recursos, grupos de recursos y suscripciones**. **No se heredan** automáticamente (un recurso no hereda las etiquetas de su grupo), salvo que lo fuerces con [[Azure Policy]].
- Límite de **50 etiquetas** por recurso.
- Usos:
  - **Gestión de costes**: filtrar la factura por etiqueta y repartirla entre departamentos.
  - **Operaciones**: identificar criticidad, ventanas de mantenimiento, propietario.
  - **Seguridad y cumplimiento**: marcar nivel de confidencialidad.
  - **Automatización**: scripts que actúan según etiqueta (apagar todo lo etiquetado `Entorno: Dev` por la noche).
- Para **obligar** a que todo recurso tenga una etiqueta se usa **Azure Policy** (efecto Deny o Modify/Append); las etiquetas por sí solas no imponen nada.

## Casos de uso

- Finanzas quiere saber cuánto gasta cada departamento: etiqueta `Departamento` en todos los recursos y análisis de costes agrupado por esa etiqueta.
- El equipo de desarrollo tiene 500 € al mes: presupuesto de 500 € en su grupo de recursos con alertas al 80 % y al 100 %.
- Prever si se superará el gasto del trimestre: vista de previsión en análisis de costes.
- Evitar recursos sin dueño: política que deniega la creación sin la etiqueta `Propietario`.

## Comparaciones

| Necesidad | Herramienta | No confundir con |
|---|---|---|
| Ver el gasto real y la previsión | Cost Management | [[02 - Calculadora de precios y calculadora de TCO|Calculadoras]] (estiman antes de desplegar) |
| Recibir aviso al llegar al 80 % del presupuesto | Presupuesto + alerta de Cost Management | Alerta de métricas de [[Azure Monitor]] (rendimiento, no coste) |
| Recomendaciones para gastar menos | [[Azure Advisor]] | Cost Management (muestra, no recomienda; aunque enlaza a Advisor) |
| Atribuir coste a un proyecto | Etiquetas | Grupos de recursos (organizan, pero un proyecto puede abarcar varios) |
| Obligar a etiquetar | Azure Policy | Etiquetas (no se imponen solas) |
| Facturas separadas | Suscripciones | Etiquetas (desglosan una factura, no la dividen) |

## Conceptos que debo memorizar

> [!important]
> - **Cost Management** = análisis de gasto real, **presupuestos**, **alertas**, previsión y exportación. Gratuito.
> - Un **presupuesto avisa, no bloquea**.
> - **Etiqueta** = par nombre:valor para organizar y **atribuir coste**. Hasta 50 por recurso. **No se heredan**.
> - Para forzar etiquetas: **Azure Policy**.
> - Advisor **recomienda** ahorrar; Cost Management **mide** el gasto.

## Tips para AZ-900

> [!tip]
> - "Cuánto he gastado", "previsión de gasto", "presupuesto", "alerta cuando llegue a X €" → **Cost Management**.
> - "Identificar a qué departamento pertenece el coste" → **etiquetas**.
> - "Asegurar que todos los recursos tienen etiqueta" → **Azure Policy**.
> - Trampa: "Un presupuesto de Cost Management apaga los recursos al superarse". **Falso**; solo alerta.
> - Trampa: "Los recursos heredan las etiquetas del grupo de recursos". **Falso**.
> - Trampa: "Cost Management recomienda cambiar el tamaño de las VMs". Eso es **Advisor**.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu empresa quiere ver el coste mensual desglosado por departamento. Cada departamento usa recursos en varios grupos de recursos de la misma suscripción. ¿Qué debes hacer?

- A) Crear una suscripción por departamento
- B) Aplicar etiquetas a los recursos con el nombre del departamento y filtrar en Cost Management
- C) Crear un grupo de administración por departamento
- D) Usar la calculadora de TCO

**Respuesta: B.** Las etiquetas permiten agrupar el coste en análisis de costes sin reorganizar recursos.
- A) Funciona, pero reorganiza toda la infraestructura; la pregunta no pide facturas separadas.
- C) Los grupos de administración agrupan suscripciones, no recursos dentro de una.
- D) La calculadora de TCO compara on-premises con Azure, no analiza gasto real.

**Pregunta 2.** Quieres recibir un correo electrónico cuando el gasto de una suscripción alcance el 90 % de los 2.000 € mensuales asignados. ¿Qué debes configurar?

- A) Una alerta de Azure Service Health
- B) Una recomendación de Azure Advisor
- C) Un presupuesto con alerta en Microsoft Cost Management
- D) Una regla de Azure Policy

**Respuesta: C.** Los presupuestos de Cost Management envían alertas al alcanzar umbrales del gasto.
- A) Service Health avisa de incidentes de la plataforma.
- B) Advisor recomienda, no vigila presupuestos.
- D) Policy controla la configuración de recursos, no el gasto acumulado.

## 🧠 Resumen para el examen

1. Cost Management = ver, analizar, prever y presupuestar el gasto real, con alertas. Gratuito.
2. El presupuesto avisa; no detiene el gasto.
3. Etiquetas = nombre:valor para organizar y atribuir coste; hasta 50; no se heredan.
4. Forzar etiquetas = Azure Policy.
5. Cost Management mide; Advisor recomienda; las calculadoras estiman.

---
