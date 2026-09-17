---
tags: [az-900, azure, costes, herramientas, repaso]
modulo: Costes y herramientas
---

# 🎯 Repaso final de Costes y herramientas

Nota de consolidación de la administración de costes y las herramientas de administración (dominio 3). Úsala el día antes del examen.

## Los conceptos más importantes

1. Factores de coste: **tipo de recurso, consumo, mantenimiento, geografía, tráfico de red, tipo de suscripción**.
2. **Ingress gratis, egress se paga.** El precio varía por región.
3. Ahorro: **reservas** (1/3 años, estable), **Spot** (interrumpible), **Hybrid Benefit** (licencias propias), apagar/desasignar, redimensionar, niveles de acceso.
4. **Calculadora de precios** = estimar coste de una solución en Azure. **Calculadora de TCO** = comparar on-premises vs Azure a varios años.
5. **Cost Management** = gasto real, previsión, **presupuestos** y **alertas**. El presupuesto avisa, no bloquea.
6. **Etiquetas** = nombre:valor para organizar y atribuir coste; no se heredan; se fuerzan con Policy.
7. **Portal** = gráfico. **Cloud Shell** = terminal en navegador con CLI y PowerShell listos.
8. **CLI** = `az …`. **PowerShell** = `Verbo-Az…`. Ambos multiplataforma.
9. **IaC**: plantillas **ARM** (JSON, declarativas, idempotentes) y **Bicep** (sintaxis simple que compila a ARM).
10. **Azure Arc** extiende la gestión a recursos fuera de Azure (ver [[Azure Arc]]).

## Tabla de herramientas y su propósito

| Herramienta | Pregunta que responde | Palabra clave de examen |
|---|---|---|
| Calculadora de precios | ¿Cuánto costará esta solución al mes? | *estimate, before deploying* |
| Calculadora de TCO | ¿Cuánto ahorro migrando desde mi CPD? | *on-premises vs Azure, electricity, IT labor, years* |
| Cost Management | ¿Cuánto he gastado y cuánto gastaré? | *actual cost, budget, alert, forecast* |
| Etiquetas | ¿A quién pertenece este coste? | *name-value, categorize, cost center* |
| Azure Advisor | ¿Cómo gasto menos? | *recommendation* |
| Instancias reservadas | ¿Cómo pago menos por lo estable? | *1 or 3 years, up to 72 %* |
| VMs Spot | ¿Cómo aprovecho capacidad sobrante? | *interruptible, evicted* |
| Azure Hybrid Benefit | ¿Cómo reutilizo mis licencias? | *existing Windows/SQL licenses* |
| Portal de Azure | ¿Dónde hago clic? | *graphical, web* |
| App móvil | ¿Cómo superviso desde el teléfono? | *iOS, Android* |
| Cloud Shell | ¿Cómo ejecuto comandos sin instalar nada? | *browser-based shell, Bash and PowerShell* |
| CLI de Azure | ¿Cómo automatizo con `az`? | *az command, cross-platform* |
| Azure PowerShell | ¿Cómo automatizo con cmdlets? | *New-Az, Get-Az* |
| Plantillas ARM | ¿Cómo despliego lo mismo muchas veces? | *JSON, declarative, idempotent* |
| Bicep | ¿Cómo escribo plantillas más legibles? | *simpler syntax, compiles to ARM* |
| Azure Arc | ¿Cómo gestiono servidores fuera de Azure? | *on-premises, multicloud* |

## Las diferencias que más fácilmente puedo confundir

> [!warning] Pares de confusión clásicos
> | Confusión | Cómo distinguirlos |
> |---|---|
> | **Calculadora de precios vs TCO** | Precios = coste de servicios de Azure. TCO = comparar tu CPD con Azure a años vista. |
> | **Calculadoras vs Cost Management** | Calculadoras = estiman antes. Cost Management = mide el gasto real después. |
> | **Cost Management vs Advisor** | Cost Management muestra y presupuesta. Advisor recomienda. |
> | **Presupuesto vs Cuota** | Presupuesto = límite de gasto que avisa. Cuota = límite técnico de la suscripción. |
> | **Etiquetas vs Suscripciones** | Etiquetas desglosan una factura. Suscripciones generan facturas separadas. |
> | **Etiquetas vs Policy** | Etiquetas organizan. Policy obliga a etiquetar. |
> | **Reserva vs Spot** | Reserva = descuento por compromiso con capacidad garantizada. Spot = descuento por capacidad sobrante sin garantía. |
> | **Reserva vs Hybrid Benefit** | Reserva = descuento por tiempo. Hybrid Benefit = descuento por licencia propia. Se combinan. |
> | **CLI vs PowerShell** | `az vm create` vs `New-AzVM`. |
> | **Cloud Shell vs CLI local** | Cloud Shell = en el navegador, sin instalar. CLI local = instalada en tu equipo. |
> | **ARM vs Bicep** | Mismo motor. ARM = JSON. Bicep = sintaxis simplificada que compila a ARM. |
> | **Declarativo vs Imperativo** | Plantillas (qué) vs scripts (cómo). |
> | **Egress vs Ingress** | Salida se paga. Entrada gratis. |

## 10 tips de examen

> [!tip]
> 1. "Antes de desplegar, ¿cuánto costará?" → **calculadora de precios**. "¿Cuánto ahorro frente a mi CPD?" → **TCO**.
> 2. "Cuánto llevo gastado", "previsión", "alerta al 80 % del presupuesto" → **Cost Management**.
> 3. "Reducir coste" como **recomendación** → **Advisor**. Como **medición** → Cost Management.
> 4. "Atribuir coste a departamento/proyecto" → **etiquetas**. "Obligar a etiquetar" → **Policy**.
> 5. "Carga estable durante años" → **reserva**. "Puede interrumpirse" → **Spot**. "Ya tengo licencias" → **Hybrid Benefit**.
> 6. "Datos que salen de Azure" → se pagan (**egress**). "Datos que entran" → gratis.
> 7. "Sin instalar nada, desde el navegador" → **Cloud Shell**.
> 8. `az` → **CLI**. `Verbo-Az` → **PowerShell**.
> 9. "Desplegar de forma **repetible/consistente/idéntica**" → **plantillas ARM o Bicep**. "JSON" → ARM. "Más legible" → Bicep.
> 10. Un presupuesto **nunca detiene** recursos; solo **avisa**.

## 10 preguntas de repaso tipo AZ-900

**1.** ¿Qué herramienta compara el coste de ejecutar cargas de trabajo en el centro de datos propio frente a Azure durante varios años?
- A) Calculadora de precios · B) Calculadora de TCO ✅ · C) Cost Management · D) Azure Advisor

**2.** Necesitas recibir una notificación cuando el gasto de una suscripción supere los 1.000 € en el mes. ¿Qué configuras?
- A) Una alerta de métricas · B) Un presupuesto en Cost Management ✅ · C) Una directiva de Azure · D) Una recomendación de Advisor

**3.** ¿Cuál de los siguientes factores NO afecta al coste de Azure?
- A) La región del recurso · B) El tráfico de salida a Internet · C) El nombre del recurso ✅ · D) El tipo de recurso

**4.** Una empresa quiere que el coste de cada recurso se atribuya a su centro de coste correspondiente al analizar la factura. ¿Qué debe usar?
- A) Grupos de administración · B) Etiquetas ✅ · C) Bloqueos · D) Zonas de disponibilidad

**5.** ¿Qué opción de compra ofrece el mayor descuento para cargas de trabajo que pueden interrumpirse?
- A) Instancias reservadas · B) Pago por uso · C) Máquinas virtuales Spot ✅ · D) Azure Hybrid Benefit

**6.** Un administrador quiere ejecutar comandos de PowerShell contra Azure desde un navegador sin instalar módulos. ¿Qué usa?
- A) El portal de Azure · B) Azure Cloud Shell ✅ · C) La app móvil · D) Azure Arc

**7.** ¿Cuál de estos comandos pertenece a la CLI de Azure?
- A) `New-AzResourceGroup` · B) `az group create` ✅ · C) `Get-AzVM` · D) `Set-AzContext`

**8.** Tu equipo quiere definir la infraestructura en archivos versionados y desplegarla de forma idéntica en varios entornos. ¿Qué enfoque describe esta necesidad?
- A) Administración desde el portal · B) Infraestructura como código ✅ · C) Azure Advisor · D) Cost Management

**9.** ¿Qué lenguaje ofrece Microsoft para escribir plantillas de Azure con una sintaxis más sencilla que el JSON de ARM?
- A) YAML · B) Terraform · C) Bicep ✅ · D) PowerShell

**10.** ¿Cuál de las siguientes afirmaciones sobre el tráfico de red en Azure es correcta?
- A) El tráfico de entrada se cobra y el de salida es gratuito
- B) El tráfico de entrada es gratuito y el de salida a Internet se cobra ✅
- C) Todo el tráfico es gratuito
- D) Todo el tráfico se cobra por igual

## 🧠 Última pasada antes del examen

> [!important] Lo que no puede fallarte
> - Precios = estimar · TCO = comparar con CPD · Cost Management = medir y presupuestar · Advisor = recomendar.
> - Etiquetas atribuyen coste; Policy las obliga; suscripciones separan facturas.
> - Reserva = estable · Spot = interrumpible · Hybrid Benefit = licencias.
> - Egress se paga; ingress no; precio distinto por región.
> - Portal · app móvil · Cloud Shell (navegador) · CLI `az` · PowerShell `Verbo-Az`.
> - ARM = JSON declarativo idempotente · Bicep = sintaxis simple sobre ARM.

Volver al índice: [[00 - Índice - Costes y herramientas]]
