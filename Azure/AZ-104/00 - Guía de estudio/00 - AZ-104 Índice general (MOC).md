---
tags: [az-104, azure, certificacion, MOC]
tipo: MOC
certificación: Microsoft Certified Azure Administrator Associate (AZ-104)
---

# 🗺️ AZ-104 · Índice general (mapa de conocimiento)

Punto de entrada de toda la carpeta AZ-104. Sigue la misma filosofía que las notas de AZ-900: cada tema es una nota independiente con concepto, funcionamiento, configuración, comparaciones, tips de examen, errores comunes y preguntas tipo. La diferencia es el **nivel**: AZ-900 pregunta *qué servicio elegir*; AZ-104 pregunta *cómo configurarlo, qué límite aplica y qué pasa si lo haces mal*.

> [!important] Temario oficial vigente (Microsoft Learn, versión del 17 de abril de 2026)
> | # | Dominio | Peso |
> |---|---|---|
> | 1 | Administrar identidades y gobernanza de Azure | **20-25 %** |
> | 2 | Implementar y administrar almacenamiento | **15-20 %** |
> | 3 | Implementar y administrar recursos de cómputo de Azure | **20-25 %** |
> | 4 | Implementar y administrar redes virtuales | **15-20 %** |
> | 5 | Supervisar y mantener recursos de Azure | **10-15 %** |
>
> Los pesos de identidad (subió) y redes (bajó) cambiaron en 2026 respecto a versiones anteriores. Ver [[01 - Guía del examen AZ-104]] para el detalle.

## Cómo usar esta carpeta

1. Lee primero [[01 - Guía del examen AZ-104]] (formato, estrategia, cambios recientes).
2. Estudia cada dominio en orden. Empieza siempre por su nota `00 - Índice`, que mapea los objetivos oficiales a notas.
3. Al terminar un dominio, haz su `99 - Repaso final` (checklist + preguntas).
4. Cuando termines los cinco dominios, ve a `06 - Repaso global`: resumen, memorización, errores frecuentes, banco de preguntas y simulacro.
5. Los apartados **💻 Laboratorio** de cada nota se hacen en una suscripción de prueba (Azure free account o Microsoft Learn sandbox).

## Mapa de dominios

### 00 · Guía de estudio
- [[01 - Guía del examen AZ-104]]
- [[02 - Herramientas del administrador (Portal, CLI, PowerShell, Cloud Shell, ARM)]]

### 01 · Identidades y gobernanza (20-25 %)
Índice: [[00 - Índice - Identidades y gobernanza]]
- Entra ID: [[01 - Microsoft Entra ID para administradores]] · [[02 - Usuarios de Microsoft Entra ID]] · [[03 - Grupos de Microsoft Entra ID]] · [[04 - Licencias en Microsoft Entra ID]] · [[05 - Usuarios externos e invitados (B2B)]] · [[06 - Self-Service Password Reset (SSPR)]] · [[07 - Unidades administrativas y dispositivos]]
- Acceso: [[08 - Azure RBAC - roles integrados y ámbitos]] · [[09 - Roles personalizados de Azure RBAC]] · [[10 - Interpretar asignaciones de acceso]]
- Gobernanza: [[11 - Azure Policy]] · [[12 - Bloqueos de recursos (Locks)]] · [[13 - Etiquetas (Tags)]] · [[14 - Grupos de recursos y movimiento de recursos]] · [[15 - Suscripciones y grupos de administración]] · [[16 - Administración de costes (presupuestos, alertas y Advisor)]]
- Extra: [[17 - Identidad híbrida - Microsoft Entra Connect]] · [[18 - Identidades administradas y entidades de servicio]]
- [[99 - Repaso final - Identidades y gobernanza]]

### 02 · Almacenamiento (15-20 %)
Índice: [[00 - Índice - Almacenamiento]]
- Cuentas: [[01 - Cuentas de almacenamiento]] · [[02 - Redundancia de almacenamiento]] · [[06 - Cifrado de cuentas de almacenamiento]] · [[07 - Replicación de objetos (Object Replication)]] · [[08 - Azure Storage Explorer y AzCopy]]
- Acceso: [[03 - Firewalls y redes virtuales de Azure Storage]] · [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]] · [[05 - Claves de acceso y autorización con Microsoft Entra ID]]
- Blob: [[09 - Azure Blob Storage]] · [[10 - Niveles de acceso de Blob (Hot, Cool, Cold, Archive)]] · [[11 - Administración del ciclo de vida de Blob]] · [[12 - Versionado, instantáneas y eliminación temporal de Blob]]
- Files: [[13 - Azure Files]] · [[14 - Acceso basado en identidad para Azure Files]] · [[15 - Instantáneas y eliminación temporal en Azure Files]] · [[16 - Azure File Sync]]
- [[99 - Repaso final - Almacenamiento]]

### 03 · Cómputo (20-25 %)
Índice: [[00 - Índice - Cómputo]]
- IaC: [[01 - Azure Resource Manager y plantillas ARM]] · [[02 - Bicep]] · [[03 - Implementar, exportar y convertir plantillas]]
- VMs: [[04 - Máquinas virtuales - creación y configuración]] · [[05 - Tamaños de VM y redimensionamiento]] · [[06 - Discos administrados]] · [[07 - Azure Disk Encryption y cifrado de discos]] · [[08 - Mover una VM (grupo de recursos, suscripción o región)]] · [[09 - Alta disponibilidad - Availability Sets y Availability Zones]] · [[10 - Virtual Machine Scale Sets]] · [[11 - Extensiones de VM y automatización]] · [[12 - Imágenes y Azure Compute Gallery]]
- Contenedores: [[13 - Azure Container Registry]] · [[14 - Azure Container Instances]] · [[15 - Azure Container Apps]] · [[16 - Comparación de servicios de contenedores]]
- App Service: [[17 - App Service Plan (niveles y escalado)]] · [[18 - Azure App Service - creación y configuración]] · [[19 - App Service - certificados, TLS y dominios personalizados]] · [[20 - App Service - copias de seguridad]] · [[21 - App Service - redes]] · [[22 - App Service - ranuras de implementación (deployment slots)]]
- [[99 - Repaso final - Cómputo]]

### 04 · Redes virtuales (15-20 %)
Índice: [[00 - Índice - Redes virtuales]]
- VNet: [[01 - Azure Virtual Network y subredes]] · [[02 - Direcciones IP públicas y privadas]] · [[03 - Peering de redes virtuales]] · [[04 - Rutas definidas por el usuario (UDR) y NVA]]
- Seguridad: [[05 - Network Security Group (NSG)]] · [[06 - Application Security Group (ASG)]] · [[07 - Reglas de seguridad efectivas]] · [[08 - Azure Bastion]] · [[09 - Service Endpoints]] · [[10 - Private Endpoint y Private Link]]
- DNS y balanceo: [[11 - Azure DNS (zonas públicas)]] · [[12 - Azure Private DNS y resolución de nombres]] · [[13 - Azure Load Balancer]] · [[14 - Solución de problemas de balanceo de carga]]
- Diagnóstico y contexto: [[15 - Solución de problemas de conectividad de red]] · [[16 - Conectividad híbrida - VPN Gateway y ExpressRoute]] · [[17 - Comparación de balanceadores (Load Balancer, Application Gateway, Front Door, Traffic Manager)]]
- [[99 - Repaso final - Redes virtuales]]

### 05 · Monitorización y mantenimiento (10-15 %)
Índice: [[00 - Índice - Monitorización y mantenimiento]]
- Monitor: [[01 - Azure Monitor - visión general]] · [[02 - Métricas en Azure Monitor]] · [[03 - Logs - Log Analytics y configuración de diagnóstico]] · [[04 - Consultas KQL]] · [[05 - Alertas, grupos de acciones y reglas de procesamiento]] · [[06 - Azure Monitor Insights (VM, Storage, Network)]] · [[07 - Network Watcher y Connection Monitor]]
- Backup y DR: [[08 - Azure Backup - Recovery Services vault y Backup vault]] · [[09 - Directivas de copia de seguridad]] · [[10 - Operaciones de copia de seguridad y restauración]] · [[11 - Azure Site Recovery]] · [[12 - Informes y alertas de copias de seguridad]]
- [[99 - Repaso final - Monitorización y mantenimiento]]

### 06 · Repaso global
- [[01 - Resumen general de AZ-104]]
- [[02 - Guía de memorización (números, límites y nombres)]]
- [[03 - Tabla comparativa de servicios]]
- [[04 - Errores y confusiones frecuentes]]
- [[05 - Checklist final antes del examen]]
- [[06 - Banco de preguntas de práctica]]
- [[07 - Simulacro final AZ-104]]
- [[08 - Índice de laboratorios]]

## Leyenda de símbolos usada en todas las notas

| Símbolo | Significado |
|---|---|
| ⭐ | Concepto imprescindible |
| ⚠️ | Concepto que suele confundirse |
| 🔥 | Alta importancia para el examen |
| 🧠 | Debo memorizarlo (número, límite, nombre exacto) |
| 💻 | Debo saber hacerlo en Azure (portal, CLI o PowerShell) |
| 📌 | Diferencia importante entre servicios |
| ➕ | Conocimiento adicional, no exigido explícitamente por el temario |

## Relación con AZ-900

Las notas de la carpeta `Azure/AZ-900` cubren los mismos servicios a nivel de concepto. Sus puntos de entrada son [[AZ-900 - Autenticación y Autorización (índice)]], [[Almacenamiento en Azure (Índice)]], [[00 - Introduccion a Azure Networking]], [[00 - Índice - Seguridad]] y [[00 Indice - Monitorizacion y Gestion]]. Cuando una nota de AZ-104 profundiza un tema que ya tienes en AZ-900, se enlaza (por ejemplo [[03 - Grupo de seguridad de red (NSG)]] → [[05 - Network Security Group (NSG)]]). Si un concepto de AZ-900 te falla, vuelve a esa nota antes de seguir.
