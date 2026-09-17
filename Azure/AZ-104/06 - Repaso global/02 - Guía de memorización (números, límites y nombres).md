---
tags: [az-104, azure, repaso, memorizacion, limites]
modulo: Repaso global
---

# 🧠 Guía de memorización: números, límites y nombres

Todo lo que conviene saber de memoria, agrupado para repasar en 20 minutos. Los valores son los vigentes en el temario actual; Azure puede ampliarlos (muchos son "por defecto, ampliable con soporte").

## 1. Identidad y gobernanza

| Dato | Valor |
|---|---|
| Usuario eliminado recuperable | **30 días** |
| Invitación B2B válida | **90 días** |
| Reconfirmación de registro SSPR | **180 días** (por defecto) |
| Métodos requeridos para SSPR | **1 o 2** |
| Ámbito SSPR "Selected" | **1 grupo** |
| Dispositivos por usuario | **50** |
| Grupos asignables a roles | **500** |
| Asignaciones RBAC por suscripción | **4000** |
| Asignaciones RBAC por grupo de administración | **500** |
| Roles personalizados por tenant | **5000** |
| Prioridad de roles | No existe: los permisos **se suman** |
| Tags por recurso | **50** (nombre 512 / **128 en Storage**; valor 256) |
| Grupos de recursos por suscripción | **980** |
| Niveles de grupos de administración | **6** (bajo el raíz) |
| Grupos de administración por tenant | **10 000** |
| Evaluación de Azure Policy | **cada 24 h** (~30 min tras asignar) |
| Suscripción deshabilitada conserva recursos | **90 días** |
| Sincronización de Entra Connect | **30 min** |

**Nombres exactos**: Owner, Contributor, Reader, User Access Administrator, Virtual Machine Contributor, Storage Blob Data Contributor, Resource Policy Contributor, Cost Management Contributor, Tag Contributor, Backup Operator, Guest Inviter, User Administrator, Password Administrator, License Administrator, Groups Administrator.

## 2. Almacenamiento

| Dato | Valor |
|---|---|
| Nombre de cuenta | **3-24**, minúsculas y números, único global |
| Durabilidad LRS / ZRS / GRS-GZRS | **11 / 12 / 16 nueves** |
| Copias LRS / ZRS / GRS | 3 / 3 (zonas) / **6** |
| Reglas de firewall | 200 IP + 200 VNet |
| Directivas de acceso almacenadas | **5** por contenedor/recurso compartido |
| User delegation SAS máxima | **7 días** |
| Retención mínima Cool / Cold / Archive | **30 / 90 / 180 días** |
| Rehidratación Standard / High | **≤ 15 h / < 1 h** (< 10 GB) |
| Reglas de ciclo de vida | **100**; ejecución **diaria** (hasta 24 h) |
| Soft delete (blobs, contenedores, Files) | **1-365 días** |
| Object replication | **10 reglas** por directiva, **2 destinos** por origen |
| Tamaño máximo de recurso compartido | **100 TiB** |
| Snapshots por recurso compartido | **200** |
| Puerto SMB | **445** |
| Cloud endpoints por sync group | **1** |
| Agente de File Sync | **Windows Server 2016+** |

## 3. Cómputo

| Dato | Valor |
|---|---|
| Fault domains / update domains | **3 / 20** (por defecto 5 UD) |
| SLA VM única Premium / set / zonas | **99,9 % / 99,95 % / 99,99 %** |
| Instancias máximas de VMSS | **1000** |
| Nombre de VM Windows / Linux | **15 / 64** caracteres |
| Contraseña de administrador | 12-123 caracteres |
| Disco administrado máximo | **32 TiB** (Ultra 64 TiB) |
| Despliegues guardados por RG | **800** |
| Slots Standard / Premium / Isolated | **5 / 20 / 20** |
| Instancias Basic / Standard / Premium | **3 / 10 / 30** |
| Backup de App Service: tamaño | **10 GB** (4 GB por base de datos) |
| Backup automático de App Service | **cada hora, 30 días** |
| Réplicas por defecto de Container Apps | **0-10** (máx. 1000) |
| Regla HTTP de Container Apps | ~**10** peticiones concurrentes por réplica |
| ACR Basic / Standard / Premium | **10 / 100 / 500 GiB** incluidos |

**Nombres exactos**: `sysprep /generalize /oobe /shutdown`, `waagent -deprovision+user`, AzureBastionSubnet, GatewaySubnet, AzureFirewallSubnet, `Microsoft.Web/serverFarms` (delegación de App Service), `Microsoft.ContainerInstance/containerGroups` (delegación de ACI).

## 4. Redes

| Dato | Valor |
|---|---|
| IPs reservadas por subred | **5** |
| Subred mínima / máxima | **/29 … /2** |
| AzureBastionSubnet | **/26** mínimo |
| GatewaySubnet | **/27** recomendado |
| AzureFirewallSubnet | **/26** |
| Prioridad de reglas NSG | **100-4096** |
| Reglas predeterminadas NSG | **65000, 65001, 65500** |
| IP de plataforma (sondas, DNS, DHCP) | **168.63.129.16** |
| Backend pool de Standard LB | **1000** |
| Idle timeout del LB | **4 min** (4-30) |
| Intervalo de sonda por defecto | **5 s** |
| SLA Standard LB / VPN Gateway | **99,99 % / 99,95 %** |
| Vínculos con autorregistro por VNet | **1** |
| VNets por suscripción y región | 1000 |
| Subredes por VNet | 3000 |

**Fórmula**: IPs utilizables = **2^(32 − prefijo) − 5**.

| Prefijo | /29 | /28 | /27 | /26 | /25 | /24 | /23 | /22 | /16 |
|---|---|---|---|---|---|---|---|---|---|
| Utilizables | 3 | 11 | 27 | 59 | 123 | **251** | 507 | 1019 | 65 531 |

## 5. Monitorización y backup

| Dato | Valor |
|---|---|
| Retención de métricas de plataforma | **93 días** |
| Retención del registro de actividad | **90 días** |
| Retención de Log Analytics | 30 incluidos, hasta **730** (archivo 12 años) |
| Configuraciones de diagnóstico por recurso | **5** |
| Severidades de alerta | **Sev 0 a Sev 4** |
| Soft delete de Backup | **14 días** (hasta 180 en el modelo actual) |
| Instant restore Standard / Enhanced | **1-5 (2) / 1-30 (7) días** |
| Frecuencia Enhanced | cada **4, 6, 8, 12 h** o diaria |
| Retención máxima de copias de VM | **9999 días** |
| Copias diarias del agente MARS | **3** |
| Puntos crash-consistent de ASR | **cada 5 minutos** |
| Retención de puntos de ASR | **24 h** por defecto (hasta 15 días) |

## 6. Fechas de retirada que pueden aparecer

| Servicio | Fecha |
|---|---|
| Log Analytics Agent (MMA) | **retirado en agosto de 2024** |
| Basic Load Balancer y Basic Public IP | **30/09/2025** |
| Azure Blueprints | en retirada (fin previsto **31/01/2027**) |
| NSG flow logs | **30/09/2027** (usar VNet flow logs) |
| Azure Disk Encryption | anunciada para **15/09/2028** |

## 7. Reglas de oro que no son números

- Una suscripción → **un** tenant; un tenant → muchas suscripciones.
- El **lock vence a cualquier rol**.
- **Policy no borra** recursos existentes.
- Los permisos RBAC **se suman**; nunca se restan.
- **Ambos NSGs** (subred y NIC) deben permitir.
- El **peering no es transitivo**.
- **UDR > BGP > sistema**; a igualdad, prefijo más específico.
- **Service endpoint no llega desde on-premises**; el private endpoint sí.
- El **DNS de Azure no cruza VNets**.
- **Sin configuración de diagnóstico no hay logs** de recurso.
- Backup = **recuperar datos**; Site Recovery = **mantener el servicio**.

## Relacionado

- [[01 - Resumen general de AZ-104]]
- [[03 - Tabla comparativa de servicios]]
- [[05 - Checklist final antes del examen]]
- [[00 - AZ-104 Índice general (MOC)]]
