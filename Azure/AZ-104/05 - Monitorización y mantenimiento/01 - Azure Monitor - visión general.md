---
tags: [az-104, azure, monitorizacion, azure-monitor]
modulo: Monitorización y mantenimiento
peso_examen: Alto
---

# Azure Monitor · visión general

## ¿Qué es?

**Azure Monitor** es la plataforma de observabilidad de Azure: recopila, analiza y actúa sobre la telemetría de los recursos de Azure, de otras nubes y de entornos locales. Fundamentos en [[Azure Monitor]] (AZ-900).

## ¿Para qué sirve?

- Saber si un recurso está sano, cuánto consume y por qué falla.
- Disparar alertas y automatizaciones.
- Analizar tendencias y capacidad.

## Conceptos clave

### Dos tipos de datos 🧠

| | **Métricas** | **Logs** |
|---|---|---|
| Naturaleza | Valores numéricos en series temporales | Registros con estructura (tablas) |
| Almacenamiento | Base de datos de métricas (automática) | **Área de trabajo de Log Analytics** |
| Recopilación | **Automática** para recursos de Azure (platform metrics) | Requiere **configuración de diagnóstico** o agente |
| Retención | **93 días** (platform metrics) | Configurable (30 días a 12 años según plan) |
| Consulta | Explorador de métricas, gráficos | **KQL** (Kusto Query Language) |
| Latencia | Casi en tiempo real (~1 min) | Minutos |
| Coste | Gratis (métricas de plataforma) | Por ingesta y retención |

### Fuentes de datos 🧠
- **Registro de actividad (Activity log)**: operaciones del **plano de control** (quién creó/borró/modificó). Retención **90 días** gratis; para más, exportar con configuración de diagnóstico.
- **Métricas de plataforma**: automáticas por recurso.
- **Registros de recurso (resource logs)**: antes "logs de diagnóstico"; **no se recopilan por defecto**, hay que crear una **configuración de diagnóstico**.
- **Datos del sistema operativo invitado**: requieren **Azure Monitor Agent (AMA)** + **Data Collection Rules (DCR)**. El **Log Analytics Agent (MMA) está retirado**.
- **Application Insights**: telemetría de aplicaciones (APM): peticiones, dependencias, excepciones, disponibilidad, mapa de aplicación, Live Metrics.
- **Registros de Entra ID** (inicios de sesión, auditoría): se exportan al área de trabajo.

### Destinos de los datos 🧠
Una configuración de diagnóstico puede enviar a:
1. **Log Analytics workspace** (consultar con KQL)
2. **Cuenta de almacenamiento** (archivado barato a largo plazo)
3. **Event Hub** (a herramientas externas: SIEM, Splunk)
4. **Partner solutions** ➕

### Capacidades
- **Explorador de métricas**, **Logs (KQL)**, **Libros (Workbooks)**, **Insights** (VM, Storage, Network, Container, App), **Alertas**, **Autoescalado**, **Change Analysis**, **Service Health** y **Resource Health**.

## Cómo funciona

```
Recursos de Azure ──métricas (automático)──────────► Base de datos de métricas ──► Explorador, alertas, autoescalado
                 ──logs de recurso (diagnóstico)──► Log Analytics / Storage / Event Hub
VMs (AMA + DCR)  ──logs y perf del invitado───────► Log Analytics
Aplicaciones     ──Application Insights───────────► Log Analytics
Activity log     ──plano de control───────────────► 90 días + exportación
                                                     │
                                            KQL, Workbooks, Alertas → Grupos de acciones
```

## Componentes y dónde se configuran

| Componente | Dónde |
|---|---|
| Métricas | Recurso → Métricas / Monitor → Métricas |
| Configuración de diagnóstico | Recurso → **Configuración de diagnóstico** |
| Área de trabajo | Log Analytics workspaces |
| Consultas | Monitor → Logs |
| Alertas | Monitor → Alertas → Reglas de alerta |
| Grupos de acciones | Monitor → Grupos de acciones |
| Insights | Monitor → Insights (VM, Storage, Network…) |
| Agente | VM → Extensiones (AzureMonitorWindowsAgent/LinuxAgent) + DCR |

## Configuración relevante para el examen

| Necesidad | Solución |
|---|---|
| Saber quién eliminó una VM | **Registro de actividad** (90 días) |
| Conservar logs de auditoría 2 años | Configuración de diagnóstico → **cuenta de almacenamiento** o workspace con retención larga |
| Enviar logs a un SIEM externo | Configuración de diagnóstico → **Event Hub** |
| Consultar con KQL | Destino **Log Analytics** |
| Recopilar el registro de eventos de Windows | **AMA + DCR** |
| Métricas de CPU de una VM sin agente | Métricas de plataforma (automáticas) |
| Memoria libre de una VM | **Requiere agente** (no es métrica de plataforma) |
| Telemetría de una aplicación web | **Application Insights** |

## Comparaciones

| Servicio | Responde a | Cuándo utilizarlo |
|---|---|---|
| **Azure Monitor** | ¿Cómo está funcionando mi recurso? | Telemetría, alertas, diagnóstico |
| **Service Health** | ¿Hay una incidencia de Azure que me afecta? | Caídas del proveedor, mantenimientos |
| **Resource Health** | ¿Está sano *este* recurso? | Diagnóstico puntual |
| **Azure Advisor** | ¿Qué debería mejorar? | Recomendaciones de coste, seguridad, fiabilidad |
| **Defender for Cloud** | ¿Estoy seguro? | Postura y amenazas |
| **Microsoft Sentinel** | SIEM/SOAR | Correlación de seguridad a escala |

## AZ-104 Exam Tips

- ⭐ **Métricas = números automáticos (93 días)**; **logs = eventos que requieren configuración**.
- 🔥 🧠 Los **registros de recurso NO se recopilan por defecto**: hay que crear una **configuración de diagnóstico**.
- 🔥 🧠 Destinos: **Log Analytics, Storage, Event Hub**.
- 🧠 **Activity log: 90 días** gratis.
- 🧠 Métricas del **invitado** (memoria, disco lógico) requieren **AMA + DCR**.
- 💻 Crear configuración de diagnóstico, consultar métricas, abrir Logs.
- 📌 Monitor (telemetría) vs Service Health (incidencias de Azure) vs Advisor (recomendaciones).

## Preguntas que podrían aparecer

**1.** Necesitas consultar los registros de un Key Vault con KQL, pero la tabla aparece vacía. ¿Qué falta?
- A) Un grupo de acciones · B) Una configuración de diagnóstico que envíe los registros al área de trabajo · C) Application Insights · D) Un agente en el Key Vault

<details><summary>Respuesta</summary>

**B.** Los registros de recurso no se recopilan hasta que se crea la configuración de diagnóstico.
</details>

**2.** ¿Durante cuánto tiempo se conserva el registro de actividad sin configuración adicional?
- A) 30 días · B) 90 días · C) 1 año · D) Indefinidamente

<details><summary>Respuesta</summary>

**B.** 90 días; para más tiempo hay que exportarlo.
</details>

## Relacionado

- [[02 - Métricas en Azure Monitor]]
- [[03 - Logs - Log Analytics y configuración de diagnóstico]]
- [[05 - Alertas, grupos de acciones y reglas de procesamiento]]
- [[Azure Monitor]] (AZ-900)
- [[00 - Índice - Monitorización y mantenimiento]]
