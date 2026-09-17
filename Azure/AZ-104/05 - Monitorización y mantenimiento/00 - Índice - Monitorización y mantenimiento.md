---
tags: [az-104, azure, monitorizacion, backup, MOC]
tipo: MOC
modulo: Monitorización y mantenimiento
peso_examen: 10-15 %
---

# AZ-104 · Dominio 5 · Supervisar y mantener recursos de Azure

> [!important] Peso en el examen: **10-15 %** (el dominio más pequeño, pero el más fácil de asegurar)
> Dos bloques: **Azure Monitor** (métricas, logs, KQL, alertas, Insights, Network Watcher) y **protección de datos** (Azure Backup y Azure Site Recovery). Las preguntas suelen ser "qué elemento configuras" y "qué vault/política usas".

## Objetivos oficiales → notas

### 5.1 Supervisar recursos en Azure

| Objetivo oficial | Nota |
|---|---|
| Interpretar métricas en Azure Monitor | [[02 - Métricas en Azure Monitor]] |
| Configurar la configuración de registros en Azure Monitor | [[03 - Logs - Log Analytics y configuración de diagnóstico]] |
| Consultar y analizar registros en Azure Monitor | [[04 - Consultas KQL]] |
| Configurar reglas de alerta, grupos de acciones y reglas de procesamiento de alertas | [[05 - Alertas, grupos de acciones y reglas de procesamiento]] |
| Configurar e interpretar la supervisión de VMs, cuentas de almacenamiento y redes con Azure Monitor Insights | [[06 - Azure Monitor Insights (VM, Storage, Network)]] |
| Usar Azure Network Watcher y Connection Monitor | [[07 - Network Watcher y Connection Monitor]] |
| Contexto | [[01 - Azure Monitor - visión general]] |

### 5.2 Implementar copias de seguridad y recuperación

| Objetivo oficial | Nota |
|---|---|
| Crear un almacén de Recovery Services | [[08 - Azure Backup - Recovery Services vault y Backup vault]] |
| Crear un almacén de Azure Backup | [[08 - Azure Backup - Recovery Services vault y Backup vault]] |
| Crear y configurar directivas de copia de seguridad | [[09 - Directivas de copia de seguridad]] |
| Realizar copia de seguridad y restauración con Azure Backup | [[10 - Operaciones de copia de seguridad y restauración]] |
| Configurar Azure Site Recovery para recursos de Azure | [[11 - Azure Site Recovery]] |
| Realizar una conmutación por error a una región secundaria con Site Recovery | [[11 - Azure Site Recovery]] |
| Configurar e interpretar informes y alertas de copias de seguridad | [[12 - Informes y alertas de copias de seguridad]] |

### Repaso
- [[99 - Repaso final - Monitorización y mantenimiento]]

## Orden de estudio sugerido

1. Azure Monitor de arriba abajo (01 → 07).
2. Backup y DR (08 → 12).
3. Repaso (99).

> [!tip] La idea central del dominio
> **Métricas** = números casi en tiempo real, baratos, retención corta. **Logs** = eventos ricos, se consultan con **KQL**, necesitan **configuración de diagnóstico** y un **área de trabajo de Log Analytics**. **Alertas** = condición + **grupo de acciones**. Para datos: **Recovery Services vault** (VMs, Files, SQL/SAP en VM, on-premises, **ASR**) vs **Backup vault** (discos, blobs, PostgreSQL, AKS).

Volver: [[00 - AZ-104 Índice general (MOC)]] · Repaso de fundamentos: [[Azure Monitor]] (AZ-900)
