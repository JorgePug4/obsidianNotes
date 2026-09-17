---
tags: [az-104, azure, computo, app-service, slots, despliegue]
modulo: Cómputo
peso_examen: Alto
---

# App Service · ranuras de implementación (deployment slots)

## ¿Qué es?

Una **ranura (slot)** es una instancia adicional de la aplicación con su **propio nombre de host** (`<app>-<slot>.azurewebsites.net`) que se ejecuta **en el mismo plan de App Service**. Permite desplegar y validar una versión nueva y luego **intercambiarla (swap)** con producción **sin tiempo de inactividad**.

## ¿Para qué sirve?

- Probar en un entorno idéntico a producción antes de publicar.
- Despliegues sin caídas y **rollback inmediato** (swap inverso).
- Calentar la aplicación antes de recibir tráfico real.

## Conceptos clave

- **Disponibilidad** 🧠: **Standard (5 slots)**, **Premium (20)**, **Isolated (20)**. Free, Shared y **Basic no tienen slots**.
- Los slots **comparten el plan** (y por tanto CPU/memoria con producción).
- **Swap** 🧠: intercambia el **contenido y la configuración** entre dos slots. Durante el swap:
  1. Se aplica al slot de origen la configuración del destino (settings que **no** son de slot).
  2. Se espera a que la app **reinicie y se caliente** (warm-up) en el slot de origen.
  3. Se intercambian las rutas de tráfico (sin downtime).
- **Swap con vista previa (swap with preview)** ➕: hace la fase 1 y pausa para que valides con la configuración de producción antes de completar.
- **Configuración que NO se intercambia (pegada al slot)** 🧠:
  - App settings y cadenas de conexión marcadas como **"slot setting"**.
  - **Nombres de host**, certificados y enlaces TLS.
  - **Restricciones de acceso** (no viajan con el swap).
  - **Escalado** (es del plan) y **Always On** (configuración general **sí** se intercambia salvo indicación).
  - **Identidad administrada** (system-assigned es propia de cada slot), **private endpoints**, **diagnósticos**, **CORS** (depende de la configuración).
- **Configuración que SÍ se intercambia**: app settings y cadenas **no** marcadas como slot setting, configuración general (versión de runtime, plataforma), rutas de la aplicación, ajustes de WebJobs, handler mappings.
- **División de tráfico (traffic routing)**: enviar un porcentaje del tráfico a un slot (pruebas A/B) con la cookie `x-ms-routing-name`.
- **Auto swap** ➕: al desplegar en un slot, se intercambia automáticamente con producción (no disponible en Linux en algunos casos).
- **Warm-up**: `applicationInitialization` en `web.config` o `WEBSITE_SWAP_WARMUP_PING_PATH`/`WEBSITE_SWAP_WARMUP_PING_STATUSES`.
- **Rollback**: volver a hacer swap (intercambio inverso) restaura la versión anterior.

## Cómo funciona

```bash
# Crear slot clonando la configuración de producción
az webapp deployment slot create -g rg-web -n app-contoso --slot staging --configuration-source app-contoso
# Desplegar en el slot
az webapp deploy -g rg-web -n app-contoso --slot staging --src-path ./app.zip --type zip
# Marcar una app setting como slot setting
az webapp config appsettings set -g rg-web -n app-contoso --slot staging --slot-settings ENV=staging
# Swap
az webapp deployment slot swap -g rg-web -n app-contoso --slot staging --target-slot production
# Swap con vista previa
az webapp deployment slot swap -g rg-web -n app-contoso --slot staging --action preview
az webapp deployment slot swap -g rg-web -n app-contoso --slot staging --action swap      # completar
az webapp deployment slot swap -g rg-web -n app-contoso --slot staging --action reset     # cancelar
# División de tráfico 20 % a staging
az webapp traffic-routing set -g rg-web -n app-contoso --distribution staging=20
az webapp deployment slot list -g rg-web -n app-contoso -o table
az webapp deployment slot delete -g rg-web -n app-contoso --slot staging
```

```powershell
New-AzWebAppSlot -ResourceGroupName rg-web -Name app-contoso -Slot staging -AppServicePlan plan-web
Switch-AzWebAppSlot -ResourceGroupName rg-web -Name app-contoso -SourceSlotName staging -DestinationSlotName production
```

Portal: Web App → **Ranuras de implementación** → Agregar ranura (clonar configuración) → desplegar → **Intercambiar** (muestra un resumen de los cambios de configuración antes de confirmar).

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Desplegar sin downtime con posibilidad de rollback | Slot `staging` + **swap** (y swap inverso para revertir) |
| La cadena de conexión de producción no debe ir a staging | Marcarla como **slot setting** en ambos slots |
| Validar con la configuración de producción antes de publicar | **Swap with preview** |
| Enviar el 10 % de usuarios a la nueva versión | **Traffic routing** al slot |
| El plan es Basic y piden slots | **Escalar a Standard** |
| Evitar peticiones lentas tras el swap | **Warm-up** (`applicationInitialization` o ping path) |
| Cada slot con su propio dominio/certificado | Los hostnames **no se intercambian**; se configuran por slot |
| Despliegue automático con swap al finalizar | **Auto swap** |

## Ejemplo

El equipo publica una nueva versión: despliega el paquete en `staging`, comprueba `app-contoso-staging.azurewebsites.net`, hace **swap with preview** para validar con la configuración de producción (cadenas de conexión reales marcadas como slot settings) y completa el swap. Un error detectado 10 minutos después se revierte con un swap inverso en segundos.

## Comparaciones

| Técnica | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Slots + swap** | Blue/green en App Service | Sin downtime, rollback instantáneo, warm-up | Estándar en Standard+ |
| **Traffic routing** | Canary | Porcentaje de usuarios | Validar con tráfico real |
| **Revisiones de Container Apps** | Blue/green en contenedores | División de tráfico por revisión | Container Apps |
| **Despliegue directo a producción** | Simple | Rápido | Solo en dev/test |
| **Azure Front Door / Traffic Manager** | Multi-región | Conmutación entre regiones | DR y despliegues globales |

## 💻 Laboratorio: slots

1. En un plan S1, crear la app y el slot `staging` clonando la configuración.
2. Cambiar el contenido del slot (por ejemplo, un `index.html` distinto) y comprobar ambas URLs.
3. Añadir la app setting `ENV` con valor distinto en cada slot y marcarla como **slot setting**.
4. Ejecutar el **swap** y comprobar que el contenido cambia pero `ENV` permanece en cada slot.
5. Hacer un swap inverso para revertir y probar `traffic-routing` al 20 %.

## AZ-104 Exam Tips

- ⭐ Los slots comparten el **plan**; cada uno tiene su **hostname**.
- 🔥 🧠 **Standard 5 slots, Premium 20, Isolated 20; Basic y Free, ninguno.**
- 🔥 🧠 Las **slot settings NO se intercambian**; tampoco hostnames, certificados ni restricciones de acceso.
- 🧠 El swap aplica la configuración del destino y **calienta** la app antes de cambiar el tráfico → sin downtime.
- 🧠 **Rollback = swap inverso**.
- 💻 `az webapp deployment slot create/swap`, `--slot-settings`, `az webapp traffic-routing set`.
- ⚠️ `--action preview` / `swap` / `reset` para el swap con vista previa.

## Errores comunes

- No marcar como slot setting la cadena de conexión y apuntar staging a la base de datos de producción tras el swap.
- Esperar slots en un plan Basic.
- Olvidar que los slots consumen recursos del mismo plan.

## Preguntas que podrían aparecer

**1.** Tras intercambiar el slot `staging` con producción, la aplicación de producción apunta a la base de datos de pruebas. ¿Qué faltó?
- A) Habilitar Always On · B) Marcar la cadena de conexión como configuración de ranura (slot setting) · C) Reiniciar la app · D) Usar swap with preview

<details><summary>Respuesta</summary>

**B.** Sin marcarla como slot setting, la cadena de conexión viaja con el swap.
</details>

**2.** ¿Cuál es la forma más rápida de revertir un despliegue defectuoso publicado mediante swap?
- A) Restaurar una copia de seguridad · B) Volver a hacer swap (intercambio inverso) · C) Redesplegar desde Git · D) Escalar el plan

<details><summary>Respuesta</summary>

**B.** El swap inverso devuelve la versión anterior de inmediato, sin downtime.
</details>

**3.** ¿Qué nivel de plan es el mínimo para usar ranuras de implementación?
- A) Free · B) Basic · C) Standard · D) Premium

<details><summary>Respuesta</summary>

**C.** Standard incluye 5 ranuras; Basic no admite ninguna.
</details>

## Relacionado

- [[17 - App Service Plan (niveles y escalado)]]
- [[18 - Azure App Service - creación y configuración]]
- [[15 - Azure Container Apps]]
- [[20 - App Service - copias de seguridad]]
- [[00 - Índice - Cómputo]]
