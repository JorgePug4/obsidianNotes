---
tags: [az-104, azure, monitorizacion, insights, vm-insights]
modulo: Monitorización y mantenimiento
peso_examen: Alto
---

# Azure Monitor Insights (VM, Storage, Network)

## ¿Qué es?

Los **Insights** son experiencias de supervisión **preconfiguradas** (libros interactivos sobre métricas y logs) para tipos de recurso concretos: **VM Insights**, **Storage Insights**, **Network Insights**, **Container Insights**, **Application Insights**. Evitan construir consultas y paneles desde cero.

El temario los cita explícitamente: "configurar e interpretar la supervisión de máquinas virtuales, cuentas de almacenamiento y redes con Azure Monitor Insights".

## VM Insights 🧠

- **Qué aporta**: rendimiento (CPU, memoria, disco, red) de VMs, VMSS y servidores con **Azure Arc**, y **mapa de dependencias** (procesos y conexiones entre servidores).
- **Requisitos**: **Azure Monitor Agent (AMA)** + **Data Collection Rule** de VM Insights; el **mapa** requiere además el **Dependency Agent**.
- Tablas: `InsightsMetrics` (rendimiento) y `VMConnection`/`VMBoundPort` (mapa).
- Vistas: **Get Started** (habilitar), **Performance** (gráficos y top N), **Map** (dependencias).
- Se habilita por VM, por grupo de recursos o a escala con **Azure Policy**.
- Útil para: detectar cuellos de botella, ver qué servidor habla con cuál antes de una migración, y responder "¿qué VM tiene poca memoria?" (métrica que **no** está en las métricas de plataforma clásicas).

## Storage Insights 🧠

- **Qué aporta**: vista de todas las cuentas de almacenamiento: **disponibilidad**, **latencia E2E y del servidor**, **transacciones**, **capacidad usada**, errores por tipo de API.
- **Requisitos**: métricas de plataforma (automáticas); para el detalle de logs por servicio (blob, file, queue, table) hace falta **configuración de diagnóstico** hacia Log Analytics.
- Útil para: diagnosticar throttling (`ServerBusyError`), latencias altas o picos de transacciones.

## Network Insights 🧠

- **Qué aporta**: vista topológica y de salud de los recursos de red (Load Balancer, Application Gateway, Public IP, NSG, VPN/ExpressRoute, Front Door, Firewall), con métricas y dependencias.
- Incluye accesos a **Network Watcher**, **Connection Monitor**, **flow logs** y **Traffic Analytics**.
- Útil para: ver de un vistazo el estado de sondas de un balanceador, el rendimiento de un gateway o los NSG con más denegaciones.

## Otros Insights

| Insight | Para |
|---|---|
| **Container Insights** | AKS y Container Apps: nodos, pods, logs |
| **Application Insights** | Aplicaciones: peticiones, dependencias, excepciones, disponibilidad, Live Metrics |
| **Key Vault, Cosmos DB, SQL, Backup Insights** | Vistas específicas por servicio |

## Cómo funciona

```bash
# Habilitar VM Insights (AMA + DCR de VM Insights)
az vm extension set -g rg-web --vm-name vm-web01 -n AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor
az monitor data-collection rule association create -g rg-mon -n assoc-vmi --rule-id <dcrVMInsightsId> --resource <vmId>
# Consultar datos de VM Insights
az monitor log-analytics query -w <workspaceId> --analytics-query "InsightsMetrics | where Name == 'AvailableMB' | summarize avg(Val) by Computer"
```

Portal: **Monitor → Insights → Máquinas virtuales / Cuentas de almacenamiento / Redes**, o desde el propio recurso → **Insights**.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Ver memoria disponible y espacio en disco de todas las VMs | **VM Insights** (AMA + DCR) |
| Saber qué servidores dependen entre sí antes de migrar | **VM Insights → Map** (Dependency Agent) |
| Diagnosticar latencia y throttling de una cuenta de almacenamiento | **Storage Insights** |
| Ver el estado de las sondas de varios balanceadores a la vez | **Network Insights** |
| Supervisar contenedores de AKS | **Container Insights** |
| Telemetría de una web (excepciones, dependencias) | **Application Insights** |
| Habilitar VM Insights en todas las VMs nuevas | **Azure Policy DeployIfNotExists** |

## Ejemplo

Antes de migrar una aplicación, el equipo habilita **VM Insights** con Dependency Agent en los 12 servidores implicados. El mapa revela una dependencia olvidada con un servidor de licencias. Además, **Storage Insights** muestra latencias E2E altas en la cuenta de almacenamiento a las 9:00, lo que lleva a cambiar el nivel de rendimiento a Premium.

## Comparaciones

| Herramienta | Nivel | Qué muestra | Requisitos |
|---|---|---|---|
| **Métricas** | Recurso | Números en bruto | Ninguno |
| **Logs (KQL)** | Todo | Eventos y consultas libres | Diagnóstico/agente |
| **Insights** | Tipo de recurso | Vistas preconstruidas | Según insight (agente/diagnóstico) |
| **Workbooks** | Personalizado | Informes a medida | Datos disponibles |
| **Service Health / Resource Health** | Plataforma | Incidencias y estado | Ninguno |

## AZ-104 Exam Tips

- 🔥 🧠 **VM Insights requiere AMA + DCR**; el **mapa de dependencias** requiere además el **Dependency Agent**.
- 🧠 Memoria y disco lógico del invitado se ven en **VM Insights** (`InsightsMetrics`), no en las métricas de plataforma clásicas.
- 🧠 **Storage Insights**: disponibilidad, latencia, transacciones, capacidad.
- 🧠 **Network Insights**: topología y salud de recursos de red.
- 💻 Habilitar Insights desde el recurso o desde Monitor; a escala con Azure Policy.
- 📌 Insight (vista preconfigurada) vs workbook (personalizado) vs métrica/log (datos crudos).

## Preguntas que podrían aparecer

**1.** Necesitas ver el consumo de memoria y las dependencias entre procesos de 20 máquinas virtuales. ¿Qué habilitas?
- A) Métricas de plataforma · B) VM Insights con Azure Monitor Agent y Dependency Agent · C) Network Insights · D) Application Insights

<details><summary>Respuesta</summary>

**B.** VM Insights aporta las métricas del invitado y, con el Dependency Agent, el mapa de dependencias.
</details>

**2.** ¿Qué Insight utilizarías para investigar latencias y errores de limitación en varias cuentas de almacenamiento?
- A) VM Insights · B) Storage Insights · C) Container Insights · D) Network Insights

<details><summary>Respuesta</summary>

**B.** Storage Insights muestra disponibilidad, latencia E2E y del servidor, y errores por tipo.
</details>

## Relacionado

- [[03 - Logs - Log Analytics y configuración de diagnóstico]]
- [[02 - Métricas en Azure Monitor]]
- [[07 - Network Watcher y Connection Monitor]]
- [[11 - Extensiones de VM y automatización]]
- [[00 - Índice - Monitorización y mantenimiento]]
