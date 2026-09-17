---
tags: [az-104, azure, gobernanza, costes, cost-management, advisor]
modulo: Identidades y gobernanza
peso_examen: Medio-Alto
---

# Administración de costes: presupuestos, alertas y Azure Advisor

## ¿Qué es?

**Microsoft Cost Management** es el conjunto de herramientas para **analizar, presupuestar, alertar y optimizar** el gasto en Azure. **Azure Advisor** es el servicio de recomendaciones que, entre otras cosas, señala oportunidades de **ahorro**.

## ¿Para qué sirve?

- Saber cuánto se gasta, en qué y por quién (análisis de costes, etiquetas).
- Avisar antes de pasarse del presupuesto (alertas).
- Reducir coste: apagar VMs infrautilizadas, reservas, tamaños adecuados.

## Conceptos clave

- **Análisis de costes (Cost analysis)**: vistas por servicio, ubicación, RG, etiqueta, con **previsión (forecast)**. Ámbitos: cuenta de facturación, MG, suscripción, RG.
- **Presupuesto (Budget)**: importe y periodo (mensual, trimestral, anual) con fecha de caducidad, ámbito (suscripción, RG, MG) y **filtros** (por etiqueta, RG, servicio). Define **umbrales de alerta** (% del importe, sobre coste **real** o **previsto**) y destinatarios de correo o **grupo de acciones**.
- ⚠️ **Un presupuesto no detiene el gasto por sí solo**. Para actuar (apagar VMs), se enlaza un **grupo de acciones** con un runbook de Automation / Logic App / Function.
- **Alertas de coste**: tres tipos 🧠 → **alertas de presupuesto**, **alertas de crédito** (EA con crédito prepago, al 90 % y 100 %) y **alertas de cuota de gasto de departamento** (EA).
- **Exportaciones**: programar la exportación diaria/semanal/mensual de datos de coste a una cuenta de almacenamiento.
- **Cost allocation** y **facturas**: en el ámbito de cuenta de facturación.
- **Calculadora de precios / TCO**: estimaciones previas (contenido de AZ-900).
- **Azure Advisor**: recomendaciones en cinco categorías 🧠 → **Coste**, **Seguridad**, **Confiabilidad**, **Excelencia operativa**, **Rendimiento**. Ejemplos de coste: VMs infrautilizadas (apagar o redimensionar), discos sin adjuntar, IPs públicas sin uso, comprar **reservas** o **planes de ahorro**, circuitos ExpressRoute sin usar, gateways inactivos.
- **Opciones de ahorro**: **Reservas** (1 o 3 años, VM/SQL/Storage… hasta ~72 %), **Azure Savings Plan for compute** (compromiso por hora), **Azure Hybrid Benefit** (licencias Windows/SQL propias), **Spot VMs** (capacidad sobrante, interrumpibles), **Dev/Test pricing**, apagado automático de VMs, **autoscale**, niveles de acceso de Storage.

## Cómo funciona un presupuesto

```
Cost Management → Presupuestos → Agregar
  Ámbito: suscripción / RG / MG (+filtros por tag)
  Nombre, período de restablecimiento (mensual), fecha inicio/fin
  Importe: 5000 €
  Alertas:  80 % del coste real   → correo a finops@contoso.com
            100 % del coste previsto → grupo de acciones "Apagar-Dev" (runbook)
```

- Las alertas se evalúan aproximadamente cada 4-8 horas con los datos de coste disponibles (que tienen retraso de hasta ~24 h).
- Roles: crear presupuestos requiere **Cost Management Contributor** (o Owner/Contributor de la suscripción); ver costes: **Cost Management Reader** o **Billing Reader**.

```bash
az consumption budget create --budget-name dev-monthly --amount 500 --category cost --time-grain monthly --start-date 2026-10-01 --end-date 2027-09-30 --resource-group rg-dev
az advisor recommendation list --category Cost -o table
```

```powershell
New-AzConsumptionBudget -Name dev-monthly -Amount 500 -Category Cost -TimeGrain Monthly -StartDate 2026-10-01 -EndDate 2027-09-30 -ContactEmail finops@contoso.com -NotificationKey n1 -NotificationThreshold 80
Get-AzAdvisorRecommendation -Category Cost
```

## Componentes

| Componente | Dónde | Qué aporta |
|---|---|---|
| Análisis de costes | Cost Management → Análisis de costes | Desglose y previsión |
| Presupuestos | Cost Management → Presupuestos | Umbrales y alertas |
| Alertas de coste | Cost Management → Alertas de costes | Presupuesto, crédito, cuota de departamento |
| Exportaciones | Cost Management → Exportaciones | CSV a Storage |
| Azure Advisor | Advisor → Coste | Recomendaciones con ahorro estimado; se pueden posponer/descartar; **alertas de Advisor** cuando aparecen nuevas |
| Reservas / Savings plans | Reservations | Compromisos con descuento |

## Configuración relevante para el examen

- Presupuesto **por RG** o **por etiqueta**: usar filtros al crear el presupuesto.
- Alerta sobre **coste previsto** (forecast) para avisar antes de superar el límite.
- Enlazar **grupo de acciones** para automatizar respuestas.
- **Advisor** permite configurar el umbral de CPU para considerar una VM "infrautilizada" y excluir suscripciones/RGs.
- Ver costes de un tenant/varias suscripciones: ámbito de **cuenta de facturación** o **MG**.
- Los **invitados** y usuarios sin rol de facturación no ven costes.

## Ejemplo

El equipo de Dev tiene un límite de 500 €/mes. Se crea un presupuesto mensual en `rg-dev` con alerta al 80 % real (correo) y al 100 % previsto (grupo de acciones que ejecuta un runbook para desasignar todas las VMs del RG). Además, Advisor recomienda redimensionar dos VMs D8 a D4 por baja utilización, con un ahorro estimado del 45 %.

## Comparaciones

| Herramienta | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Cost analysis** | Ver y prever gasto | Filtros, agrupación, forecast | Análisis periódico |
| **Presupuesto + alerta** | Avisar de desviaciones | Umbrales real/previsto, action groups | Control de gasto por equipo/proyecto |
| **Azure Advisor (Coste)** | Encontrar ahorros | Recomendaciones concretas | Optimización continua |
| **Reservas** | Descuento por compromiso 1/3 años | Hasta ~72 % | Cargas estables 24×7 |
| **Savings Plan** | Descuento por compromiso de gasto/hora | Flexible entre familias/regiones | Cómputo variable pero constante en gasto |
| **Spot VMs** | Capacidad sobrante | Muy barato | Cargas interrumpibles (batch, test) |
| **Azure Hybrid Benefit** | Reusar licencias | Reduce coste de Windows/SQL | Empresas con Software Assurance |

## AZ-104 Exam Tips

- 🔥 ⚠️ **Un presupuesto no bloquea el gasto**; solo alerta. Para actuar → **grupo de acciones** + automatización.
- 🧠 Tipos de alertas de coste: **presupuesto, crédito, cuota de departamento**.
- 🧠 Categorías de Advisor: **Coste, Seguridad, Confiabilidad, Excelencia operativa, Rendimiento**.
- 🧠 Rol para crear presupuestos: **Cost Management Contributor**; para leer: **Cost Management Reader / Billing Reader**.
- 💻 Crear un presupuesto con filtro por etiqueta y alerta de coste previsto; leer recomendaciones de Advisor.
- 📌 Reservas (recurso concreto) vs Savings plan (gasto en cómputo) vs Spot (interrumpible) vs Hybrid Benefit (licencias).
- ⚠️ Los datos de coste tienen retraso; las alertas no son en tiempo real.

## Errores comunes

- Esperar que el presupuesto apague recursos al 100 %.
- Crear el presupuesto en la suscripción cuando se pedía por grupo de recursos o por etiqueta.
- Confundir Advisor (recomendaciones) con Azure Monitor (telemetría) o Service Health (incidencias).

## Preguntas que podrían aparecer

**1.** Necesitas recibir un correo cuando el gasto previsto de la suscripción para el mes supere los 10 000 €, y además que las VMs del RG "Dev" se apaguen automáticamente al llegar al 100 % del gasto real. ¿Qué configuras?
- A) Dos alertas de métrica en Azure Monitor · B) Un presupuesto con una alerta sobre coste previsto (correo) y otra sobre coste real ligada a un grupo de acciones con un runbook · C) Una directiva de Azure Policy · D) Solo Azure Advisor

<details><summary>Respuesta</summary>

**B.** Los presupuestos permiten alertas sobre coste real y previsto, y pueden invocar grupos de acciones para automatizar respuestas. El presupuesto por sí solo no apaga nada.
</details>

**2.** ¿Qué servicio identifica máquinas virtuales infrautilizadas y sugiere reducir su tamaño con el ahorro estimado?
- A) Azure Monitor · B) Azure Service Health · C) Azure Advisor · D) Microsoft Defender for Cloud

<details><summary>Respuesta</summary>

**C.** Advisor, en su categoría Coste, detecta VMs con baja utilización y recomienda redimensionarlas o apagarlas.
</details>

## Relacionado

- [[13 - Etiquetas (Tags)]]
- [[15 - Suscripciones y grupos de administración]]
- [[05 - Alertas, grupos de acciones y reglas de procesamiento]]
- [[Azure Advisor]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
