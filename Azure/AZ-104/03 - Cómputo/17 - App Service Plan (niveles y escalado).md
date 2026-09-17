---
tags: [az-104, azure, computo, app-service, plan, escalado]
modulo: Cómputo
peso_examen: Muy alto
---

# App Service Plan (niveles y escalado)

## ¿Qué es?

Un **plan de App Service** define el **conjunto de recursos de cómputo** (máquinas virtuales gestionadas) donde se ejecutan una o varias aplicaciones de App Service. Determina la **región**, el **sistema operativo** (Windows o Linux), el **nivel de precio (SKU)** y el **número de instancias**. **El plan es lo que se factura**, no la aplicación.

## ¿Para qué sirve?

- Elegir la capacidad y las características disponibles para las apps.
- Escalar vertical (cambiar de nivel) y horizontalmente (más instancias).
- Compartir recursos entre varias aplicaciones para ahorrar.

## Conceptos clave

### Niveles 🧠

| Nivel | Tipo de cómputo | Escalado horizontal | Slots | Dominio propio | TLS propio | Backup | VNet integration | Cuándo |
|---|---|---|---|---|---|---|---|---|
| **Free (F1)** | Compartido | No (1 instancia) | No | No | No | No | No | Pruebas |
| **Shared (D1)** ➕ | Compartido | No | No | **Sí** | No | No | No | Pruebas con dominio (solo Windows) |
| **Basic (B1-B3)** | Dedicado | Hasta 3 instancias, **manual** | No | Sí | Sí | Sí (solo producción) | Sí | Dev/test, cargas pequeñas |
| **Standard (S1-S3)** | Dedicado | Hasta 10, **autoescalado** | **5** | Sí | Sí | Sí | Sí | Producción |
| **Premium (P1v3-P3v3, Pv4)** | Dedicado (más CPU/RAM) | Hasta 30 (más con soporte), autoescalado | **20** | Sí | Sí | Sí | Sí | Producción exigente |
| **Isolated (I1v2-I3v2)** | **App Service Environment** (red aislada) | Hasta 100 | **20** | Sí | Sí | Sí | Sí (dedicada) | Aislamiento y cumplimiento |

- **Free y Shared** usan **cuotas de CPU por día** y no permiten escalar ni "Always On".
- **Always On** (Basic y superiores): evita que la app se descargue por inactividad; necesario para WebJobs continuos y Functions en plan de App Service.
- Las apps del **mismo plan comparten** CPU, memoria e instancias: una app puede afectar a las demás.
- **Un plan por región y SO**: no se pueden mezclar Windows y Linux en el mismo plan.
- **Cambiar de nivel (scale up)** es inmediato y sin recrear la app; se puede subir y bajar (con pérdida de características si se baja).
- **Mover una app a otro plan**: posible si ambos están en el **mismo grupo de recursos, región y SO** (misma webspace) 🧠.

### Escalado
- **Scale up (vertical)**: cambiar el nivel/tamaño del plan.
- **Scale out (horizontal)**: número de instancias.
  - **Manual**: fijar N instancias.
  - **Automático (autoscale)**: reglas por **métrica** (CPU, memoria, cola HTTP, tráfico) o **programación**, igual que en VMSS (métrica, agregación, operador, umbral, duración, acción, cool-down). Requiere **Standard o superior**.
  - **Escalado automático elástico / Automatic scaling** ➕ (Premium v2/v3): Azure gestiona las instancias con un máximo por app.
- **Flex Consumption / Consumption** son planes de **Azure Functions**, no de Web Apps.

## Cómo funciona

```bash
az appservice plan create --name plan-web --resource-group rg-web --sku S1 --number-of-workers 2
az appservice plan create --name plan-linux --resource-group rg-web --sku B1 --is-linux
az appservice plan update --name plan-web --resource-group rg-web --sku P1V3        # scale up
az appservice plan update --name plan-web --resource-group rg-web --number-of-workers 5  # scale out manual
# Autoescalado
az monitor autoscale create --resource-group rg-web --resource plan-web --resource-type Microsoft.Web/serverfarms \
  --name autoscale-web --min-count 2 --max-count 10 --count 2
az monitor autoscale rule create --resource-group rg-web --autoscale-name autoscale-web \
  --condition "CpuPercentage > 70 avg 10m" --scale out 2
az monitor autoscale rule create --resource-group rg-web --autoscale-name autoscale-web \
  --condition "CpuPercentage < 30 avg 10m" --scale in 1
az appservice plan list -o table
```

```powershell
New-AzAppServicePlan -ResourceGroupName rg-web -Name plan-web -Location westeurope -Tier Standard -NumberofWorkers 2
Set-AzAppServicePlan -ResourceGroupName rg-web -Name plan-web -Tier PremiumV3
Set-AzAppServicePlan -ResourceGroupName rg-web -Name plan-web -NumberofWorkers 5
```

Portal: **Planes de App Service** → Crear (o desde el asistente de la Web App) → SO, región, SKU. En la app: **Escalar verticalmente (plan de App Service)** y **Escalar horizontalmente (plan de App Service)**.

## Configuración relevante para el examen

| Requisito | Nivel mínimo |
|---|---|
| Dominio personalizado | **Shared** (Windows) / **Basic** |
| Certificado TLS/SSL propio | **Basic** |
| Ranuras de implementación | **Standard** (5 slots) |
| 20 ranuras | **Premium** |
| Autoescalado | **Standard** |
| Copia de seguridad | **Basic** (solo slot de producción) / **Standard+** (completo) |
| Aislamiento de red total (ASE) | **Isolated** |
| Always On | **Basic** |
| Escalar a 30 instancias | **Premium** |
| Coste mínimo para una demo sin dominio | **Free** |

## Ejemplo

Una web de producción necesita slots de staging, autoescalado y certificado propio. El plan **Standard S1** con 2 instancias cumple: 5 slots, autoescalado hasta 10 instancias y certificados. Cuando el equipo necesita 12 slots y VNet dedicada, se hace **scale up** a **P1v3** sin recrear la app.

## Comparaciones

| Plan | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Free/Shared** | Pruebas | Gratis / muy barato | Demos, aprendizaje |
| **Basic** | Dev/test, apps pequeñas | Dedicado, dominio y TLS | Entornos no críticos |
| **Standard** | Producción | Slots y autoescalado | La mayoría de webs |
| **Premium v3** | Producción exigente | Más CPU/RAM, 20 slots, más instancias | Alto tráfico, muchos entornos |
| **Isolated (ASE)** | Cumplimiento | Red dedicada, 100 instancias | Regulación, aislamiento |

## 💻 Laboratorio: plan y escalado

1. Crear un plan `plan-lab` (Linux, B1) y una Web App.
2. Intentar crear un slot: comprobar que Basic no lo permite.
3. Hacer **scale up** a S1 y crear el slot `staging`.
4. Configurar autoescalado: min 1, max 4, out si CPU > 70 % 10 min, in si CPU < 25 % 10 min.
5. Ver en **Escalar horizontalmente** el historial de ejecución de las reglas.

## AZ-104 Exam Tips

- ⭐ **Se factura el plan**, no la app; varias apps comparten un plan.
- 🔥 🧠 **Slots: Standard 5, Premium 20, Isolated 20**; Free/Shared/Basic **ninguno**.
- 🔥 🧠 **Autoescalado desde Standard**; Basic solo escalado manual (máx. 3 instancias).
- 🧠 **Free/Shared no admiten Always On, TLS propio ni escalado**.
- 🧠 Un plan es de **un solo SO** (Windows o Linux) y una región.
- 🧠 Mover una app entre planes: mismo **RG, región y SO**.
- 💻 `az appservice plan create/update`, reglas de autoescalado, scale up/out en el portal.
- 📌 Scale up (nivel) vs scale out (instancias).

## Errores comunes

- Intentar crear slots en un plan Basic.
- Mezclar apps Windows y Linux en el mismo plan.
- Creer que el escalado horizontal cambia el rendimiento por instancia (eso es scale up).

## Preguntas que podrían aparecer

**1.** Una aplicación en un plan Basic B1 necesita ranuras de implementación y autoescalado. ¿Cuál es el cambio mínimo?
- A) Cambiar a Shared · B) Escalar verticalmente a Standard · C) Crear otro plan Basic · D) Migrar a Isolated

<details><summary>Respuesta</summary>

**B.** Standard es el primer nivel con slots (5) y autoescalado.
</details>

**2.** ¿Cuántas ranuras de implementación admite el nivel Premium?
- A) 5 · B) 10 · C) 20 · D) 100

<details><summary>Respuesta</summary>

**C.** Premium admite 20 slots; Standard 5.
</details>

**3.** Tienes dos aplicaciones en el mismo plan S1 y una consume el 90 % de la CPU. ¿Qué ocurre con la otra?
- A) Nada, están aisladas · B) Se ve afectada porque comparten las instancias del plan · C) Se mueve automáticamente · D) Escala sola

<details><summary>Respuesta</summary>

**B.** Las apps de un mismo plan comparten los recursos de las instancias.
</details>

## Relacionado

- [[18 - Azure App Service - creación y configuración]]
- [[22 - App Service - ranuras de implementación (deployment slots)]]
- [[20 - App Service - copias de seguridad]]
- [[10 - Virtual Machine Scale Sets]]
- [[00 - Índice - Cómputo]]
