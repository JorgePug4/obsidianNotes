---
tags: [az-104, azure, monitorizacion, metricas]
modulo: Monitorización y mantenimiento
peso_examen: Alto
---

# Métricas en Azure Monitor

## ¿Qué es?

Las **métricas** son valores **numéricos** recopilados a intervalos regulares que describen el comportamiento de un recurso (CPU, IOPS, peticiones, latencia). Se guardan en una base de datos de series temporales optimizada para consultas rápidas y alertas casi en tiempo real.

## ¿Para qué sirve?

- Ver el rendimiento actual y su tendencia.
- Disparar **alertas de métrica** y **autoescalado**.
- Comparar recursos y dimensiones.

## Conceptos clave 🧠

- **Métricas de plataforma**: automáticas para casi todos los recursos, **sin configuración ni coste**, retención **93 días**.
- **Granularidad**: normalmente **1 minuto** (algunas 5 min); se agregan al visualizar.
- **Agregación**: **Average, Minimum, Maximum, Sum, Count**. Cada métrica tiene una agregación por defecto.
- **Dimensiones**: atributos que permiten dividir la métrica (por ejemplo, *Percentage CPU* por instancia; *Transactions* por `ApiName`, `ResponseType`, `GeoType`). Se filtran y se aplica *splitting*.
- **Métricas personalizadas**: enviadas por el agente, Application Insights o la API.
- **Métricas del invitado** ⚠️: memoria disponible, espacio en disco lógico… **no** son métricas de plataforma; requieren **AMA + DCR** (se pueden enviar a métricas o a logs).
- **Explorador de métricas**: elegir ámbito, espacio de nombres, métrica, agregación; añadir filtros y división; fijar en un panel o en un **workbook**.
- **Exportación**: con **configuración de diagnóstico** se pueden enviar métricas a Log Analytics (tabla `AzureMetrics`), Storage o Event Hub.
- **Métricas multi-recurso** y **a nivel de suscripción** para alertas.

## Métricas típicas por servicio (reconocerlas)

| Recurso | Métricas clave |
|---|---|
| **Máquina virtual** | Percentage CPU, Network In/Out Total, Disk Read/Write Bytes, Disk Read/Write Operations/Sec, **Available Memory Bytes** (requiere agente en versiones antiguas; hoy disponible como métrica de plataforma en muchas series) |
| **Cuenta de almacenamiento** | Used capacity, Transactions, Ingress/Egress, SuccessE2ELatency, Availability |
| **Load Balancer** | Data path availability, **Health probe status (DipAvailability)**, SNAT connection count, Byte count |
| **App Service** | CPU Time, Requests, Http 5xx, Response Time, Memory working set, Data In/Out |
| **SQL Database** | DTU/CPU percentage, Deadlocks, Storage percentage |
| **VMSS** | Percentage CPU (para autoescalado) |
| **Recovery Services vault** | Backup Health Events, Jobs |

## Cómo funciona

```bash
# Listar definiciones de métricas de un recurso
az monitor metrics list-definitions --resource <resourceId> -o table
# Consultar una métrica
az monitor metrics list --resource <resourceId> --metric "Percentage CPU" --interval PT1M --aggregation Average --start-time 2026-09-17T00:00:00Z
# Con dimensión
az monitor metrics list --resource <storageId> --metric Transactions --filter "ApiName eq '*'" --interval PT1H
```

```powershell
Get-AzMetricDefinition -ResourceId $vm.Id
Get-AzMetric -ResourceId $vm.Id -MetricName "Percentage CPU" -TimeGrain 00:01:00 -AggregationType Average
```

Portal: recurso → **Métricas** → ámbito, espacio de nombres, métrica, agregación → *Agregar filtro* / *Aplicar división* → Anclar al panel o Nueva regla de alerta.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Ver la CPU media de una VM en las últimas 4 horas | Explorador de métricas, agregación **Average** |
| Detectar picos puntuales | Agregación **Maximum** |
| Contar transacciones de Storage por tipo de API | Métrica *Transactions* con **división por ApiName** |
| Alertar si la CPU supera el 80 % durante 5 min | Alerta de métrica ([[05 - Alertas, grupos de acciones y reglas de procesamiento]]) |
| Escalar un VMSS por CPU | Autoescalado basado en métrica |
| Conservar métricas más de 93 días | Exportar a **Log Analytics** o Storage con configuración de diagnóstico |
| Métrica de memoria libre del SO | **AMA + DCR** (métrica del invitado) |
| Comparar dos VMs en el mismo gráfico | Métricas **multi-recurso** |

## Ejemplo

Un equipo investiga lentitud de una web. En Métricas de la App Service seleccionan *Response Time* (Average) y *Http 5xx* (Sum) en el mismo gráfico durante 24 horas; observan que los 5xx coinciden con picos de *CPU Time*. Crean una alerta de métrica para *Http 5xx > 10 en 5 minutos* y activan el autoescalado del plan.

## Comparaciones

| | **Métricas** | **Logs** |
|---|---|---|
| Para | Rendimiento y tendencias numéricas | Diagnóstico detallado y auditoría |
| Consulta | Explorador de métricas | **KQL** |
| Alertas | Rápidas y baratas | Más flexibles, algo más lentas |
| Retención | 93 días | Configurable |
| Coste | Gratis (plataforma) | Por ingesta/retención |

## AZ-104 Exam Tips

- ⭐ Las métricas de plataforma son **automáticas, gratuitas y con 93 días** de retención.
- 🔥 🧠 Agregaciones: **Average, Min, Max, Sum, Count**.
- 🔥 🧠 Las métricas del **invitado** (memoria, disco lógico) suelen requerir **agente**.
- 🧠 **Dimensiones** = filtrar y dividir (por instancia, API, código de respuesta).
- 🧠 Para retención larga → exportar con configuración de diagnóstico.
- 💻 Explorador de métricas, crear alerta desde la métrica, anclar a panel.
- 📌 Métrica (número, rápido) vs log (evento, KQL).

## Errores comunes

- Buscar "memoria" en las métricas de plataforma de una VM antigua sin agente.
- Usar Average cuando el enunciado pide detectar picos (Maximum).
- Esperar métricas de más de 93 días sin exportación.

## Preguntas que podrían aparecer

**1.** Necesitas conservar las métricas de rendimiento de tus VMs durante un año. ¿Qué haces?
- A) Nada, se guardan 1 año · B) Crear una configuración de diagnóstico que envíe las métricas a Log Analytics o a una cuenta de almacenamiento · C) Usar Application Insights · D) Crear una alerta

<details><summary>Respuesta</summary>

**B.** Las métricas de plataforma solo se conservan 93 días; para más hay que exportarlas.
</details>

**2.** Quieres ver las transacciones de una cuenta de almacenamiento separadas por tipo de operación. ¿Qué utilizas?
- A) Un filtro de tiempo · B) La división (splitting) por la dimensión ApiName · C) Un log de actividad · D) Una alerta

<details><summary>Respuesta</summary>

**B.** Las dimensiones permiten dividir la métrica por atributos como ApiName.
</details>

## Relacionado

- [[01 - Azure Monitor - visión general]]
- [[05 - Alertas, grupos de acciones y reglas de procesamiento]]
- [[06 - Azure Monitor Insights (VM, Storage, Network)]]
- [[10 - Virtual Machine Scale Sets]]
- [[00 - Índice - Monitorización y mantenimiento]]
