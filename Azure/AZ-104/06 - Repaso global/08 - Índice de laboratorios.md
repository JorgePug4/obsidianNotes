---
tags: [az-104, azure, laboratorios, practica]
modulo: Repaso global
---

# 💻 Índice de laboratorios AZ-104

Todos los laboratorios de la carpeta, en orden recomendado. Hazlos en una **suscripción de prueba** (cuenta gratuita de Azure, crédito de Visual Studio o entornos de Microsoft Learn) y **elimina los recursos al terminar** para no gastar crédito.

> [!warning] Antes de empezar
> - Crea un grupo de recursos por laboratorio (`rg-lab-XX`) para poder borrarlo entero con `az group delete --name rg-lab-XX --yes --no-wait`.
> - Usa tamaños pequeños (`Standard_B2s`) y apaga o desasigna las VMs al terminar.
> - Vigila el **límite de gasto** y crea un presupuesto con alerta al 50 % ([[16 - Administración de costes (presupuestos, alertas y Advisor)]]).

## Dominio 1 · Identidades y gobernanza

| # | Laboratorio | Nota |
|---|---|---|
| 1.1 | Crear usuarios individualmente, en bloque con CSV y restaurar uno eliminado | [[02 - Usuarios de Microsoft Entra ID]] |
| 1.2 | Crear un grupo dinámico con regla por departamento y asignarle un rol | [[03 - Grupos de Microsoft Entra ID]] |
| 1.3 | Configurar SSPR para un grupo piloto con dos métodos | [[06 - Self-Service Password Reset (SSPR)]] |
| 1.4 | Asignar RBAC en varios ámbitos y comprobar el acceso efectivo | [[08 - Azure RBAC - roles integrados y ámbitos]] |
| 1.5 | Asignar la directiva "Allowed locations" y una de herencia de etiquetas con remediación | [[11 - Azure Policy]] |

## Dominio 2 · Almacenamiento

| # | Laboratorio | Nota |
|---|---|---|
| 2.1 | Crear una cuenta GPv2, cambiar redundancia y activar protección de datos | [[01 - Cuentas de almacenamiento]] |
| 2.2 | Restringir la cuenta a una subred con service endpoint | [[03 - Firewalls y redes virtuales de Azure Storage]] |
| 2.3 | Generar SAS ad hoc y con directiva almacenada, y revocarla | [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]] |
| 2.4 | Configurar una regla de ciclo de vida y observar el cambio de nivel | [[11 - Administración del ciclo de vida de Blob]] |
| 2.5 | Crear y montar un recurso compartido de Azure Files, con instantánea y restauración | [[13 - Azure Files]] · [[15 - Instantáneas y eliminación temporal en Azure Files]] |
| 2.6 | Copiar datos con AzCopy (copy, sync y server-to-server) | [[08 - Azure Storage Explorer y AzCopy]] |

## Dominio 3 · Cómputo

| # | Laboratorio | Nota |
|---|---|---|
| 3.1 | Exportar una plantilla, convertirla a Bicep y redesplegarla | [[03 - Implementar, exportar y convertir plantillas]] |
| 3.2 | Crear una VM con disco de datos, desasignarla y ver la facturación | [[04 - Máquinas virtuales - creación y configuración]] |
| 3.3 | Añadir, ampliar y cambiar el tipo de un disco administrado | [[06 - Discos administrados]] |
| 3.4 | Cifrar discos con ADE y con encryption at host | [[07 - Azure Disk Encryption y cifrado de discos]] |
| 3.5 | Desplegar dos VMs en zonas distintas tras un Load Balancer | [[09 - Alta disponibilidad - Availability Sets y Availability Zones]] |
| 3.6 | Crear un VMSS con reglas de autoescalado y generar carga | [[10 - Virtual Machine Scale Sets]] |
| 3.7 | Publicar una imagen en Azure Compute Gallery | [[12 - Imágenes y Azure Compute Gallery]] |
| 3.8 | Construir una imagen con `az acr build` y desplegarla en ACI con identidad administrada | [[13 - Azure Container Registry]] · [[14 - Azure Container Instances]] |
| 3.9 | Desplegar una Container App con escalado a cero y división de tráfico | [[15 - Azure Container Apps]] |
| 3.10 | Crear una Web App con slot, swap y configuración de ranura | [[22 - App Service - ranuras de implementación (deployment slots)]] |

## Dominio 4 · Redes

| # | Laboratorio | Nota |
|---|---|---|
| 4.1 | Diseñar una VNet con subredes reservadas y calcular direcciones | [[01 - Azure Virtual Network y subredes]] |
| 4.2 | Montar hub-and-spoke con peering y comprobar la no transitividad | [[03 - Peering de redes virtuales]] |
| 4.3 | Forzar el tráfico por una NVA con UDR e IP forwarding | [[04 - Rutas definidas por el usuario (UDR) y NVA]] |
| 4.4 | Configurar NSG en subred y NIC y ver las reglas efectivas | [[05 - Network Security Group (NSG)]] · [[07 - Reglas de seguridad efectivas]] |
| 4.5 | Desplegar Azure Bastion y conectarse sin IP pública | [[08 - Azure Bastion]] |
| 4.6 | Crear un private endpoint con su zona DNS privada | [[10 - Private Endpoint y Private Link]] |
| 4.7 | Crear zonas DNS privadas con autorregistro y resolución entre VNets | [[12 - Azure Private DNS y resolución de nombres]] |
| 4.8 | Montar un Load Balancer público con sonda y provocar un fallo | [[13 - Azure Load Balancer]] |
| 4.9 | Diagnosticar con IP flow verify, Next hop y flow logs | [[15 - Solución de problemas de conectividad de red]] |

## Dominio 5 · Monitorización y mantenimiento

| # | Laboratorio | Nota |
|---|---|---|
| 5.1 | Crear un área de trabajo, configurar diagnóstico e instalar AMA con DCR | [[03 - Logs - Log Analytics y configuración de diagnóstico]] |
| 5.2 | Ejecutar consultas KQL sobre Heartbeat, Perf y AzureActivity | [[04 - Consultas KQL]] |
| 5.3 | Crear un grupo de acciones, una alerta de eliminación de VM y una regla de supresión | [[05 - Alertas, grupos de acciones y reglas de procesamiento]] |
| 5.4 | Habilitar VM Insights y revisar el mapa de dependencias | [[06 - Azure Monitor Insights (VM, Storage, Network)]] |
| 5.5 | Crear un Connection Monitor entre dos VMs | [[07 - Network Watcher y Connection Monitor]] |
| 5.6 | Proteger una VM con Azure Backup y restaurar un archivo | [[10 - Operaciones de copia de seguridad y restauración]] |
| 5.7 | Habilitar Site Recovery, ejecutar test failover y limpiarlo | [[11 - Azure Site Recovery]] |

## Ruta rápida (si tienes poco tiempo)

Si solo puedes hacer **ocho** laboratorios, haz estos: **1.4** (RBAC), **1.5** (Policy), **2.3** (SAS), **3.2** (VM), **3.10** (slots), **4.4** (NSG), **4.6** (private endpoint) y **5.6** (Backup). Cubren los patrones más preguntados.

## Limpieza

```bash
# Ver qué grupos de laboratorio existen
az group list --query "[?starts_with(name,'rg-lab')].name" -o tsv
# Borrarlos todos
for rg in $(az group list --query "[?starts_with(name,'rg-lab')].name" -o tsv); do az group delete --name $rg --yes --no-wait; done
```

> [!warning] Recursos que siguen costando si los olvidas
> Azure Bastion, VPN Gateway, Application Gateway, Azure Firewall, IPs públicas Standard, discos de VMs eliminadas, vaults con elementos protegidos, replicación de Site Recovery y planes de App Service dedicados.

## Relacionado

- [[05 - Checklist final antes del examen]]
- [[02 - Herramientas del administrador (Portal, CLI, PowerShell, Cloud Shell, ARM)]]
- [[00 - AZ-104 Índice general (MOC)]]
