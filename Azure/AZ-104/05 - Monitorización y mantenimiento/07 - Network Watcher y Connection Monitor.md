---
tags: [az-104, azure, monitorizacion, network-watcher, connection-monitor]
modulo: Monitorización y mantenimiento
peso_examen: Alto
---

# Network Watcher y Connection Monitor

## ¿Qué es?

**Azure Network Watcher** es el conjunto de herramientas de **diagnóstico, supervisión y registro** de redes en Azure. **Connection Monitor** es su servicio de **supervisión continua de conectividad** entre orígenes y destinos (en Azure, on-premises o Internet).

El temario lo cita en el dominio de monitorización: "usar Azure Network Watcher y Connection Monitor". El detalle de diagnóstico puntual está en [[15 - Solución de problemas de conectividad de red]].

## Componentes 🧠

### Diagnóstico (bajo demanda)
| Herramienta | Responde |
|---|---|
| **IP flow verify** | ¿Un NSG permite o bloquea este flujo? (indica la regla) |
| **NSG diagnostics** | Evaluación completa de NSG para un flujo |
| **Next hop** | ¿Cuál es el siguiente salto del tráfico? |
| **Effective security rules** | Reglas NSG combinadas de una NIC |
| **Connection troubleshoot** | Prueba puntual origen→destino con latencia y saltos |
| **Packet capture** | Captura de paquetes (requiere extensión Network Watcher Agent) |
| **VPN troubleshoot** | Diagnóstico de gateways y conexiones |

### Supervisión (continua)
| Herramienta | Qué hace |
|---|---|
| **Connection Monitor** | Pruebas periódicas de conectividad (TCP, ICMP, HTTP) entre **orígenes** (VMs, VMSS, servidores Arc, agentes on-premises) y **destinos** (VM, IP, URL, servicio PaaS). Mide **porcentaje de comprobaciones fallidas** y **latencia (RTT)**, con umbrales y **alertas** |
| **Topology** | Diagrama de la red |

### Registro
| Herramienta | Qué hace |
|---|---|
| **VNet flow logs** | Registro de flujos IP de la red virtual hacia una cuenta de almacenamiento. **Sustituyen a los NSG flow logs**, que se retiran el **30/09/2027** 🧠 |
| **Traffic Analytics** | Procesa los flow logs en **Log Analytics** y ofrece mapas de tráfico, hosts más activos, puertos maliciosos, flujos denegados |

## Conceptos clave

- Network Watcher se habilita **por región** (recurso en `NetworkWatcherRG`); normalmente ya está activo.
- **Connection Monitor** 🧠 se compone de: **grupos de prueba** con **orígenes**, **destinos** y **configuración de prueba** (protocolo, puerto, frecuencia, umbrales de éxito).
  - Requiere el **Network Watcher Agent** (VMs de Azure) o el agente de monitorización para equipos on-premises y Arc.
  - Los resultados se guardan en **Log Analytics** (tablas `NWConnectionMonitor*`) y permiten **alertas**.
  - Frecuencia de prueba desde 30 segundos.
  - Sustituye a las funciones antiguas *Connection Monitor (clásico)* y *Network Performance Monitor*.
- **Flow logs**: se habilitan sobre la **VNet** (o NSG, en el modelo antiguo), requieren **cuenta de almacenamiento** y opcionalmente Traffic Analytics con workspace.
- **Métricas de red** disponibles en Azure Monitor para gateways, balanceadores y NICs.

## Cómo funciona

```bash
# Connection Monitor
az network watcher connection-monitor create -n cm-app -g rg-net --location westeurope \
  --endpoint-source-name vm-app01 --endpoint-source-resource-id <vmId> \
  --endpoint-dest-name sql --endpoint-dest-address 10.0.2.4 \
  --test-config-name tcp1433 --protocol Tcp --tcp-port 1433 --frequency 60
az network watcher connection-monitor list -l westeurope -o table
# VNet flow logs con Traffic Analytics
az network watcher flow-log create -g rg-net -n fl-vnet --vnet vnet-hub --storage-account <stId> \
  --workspace <lawId> --traffic-analytics true --interval 10 --retention 30
# Diagnóstico puntual
az network watcher test-connectivity -g rg-net --source-resource vm-app01 --dest-address 10.0.2.4 --dest-port 1433
az network watcher show-topology -g rg-net --location westeurope
```

```powershell
$nw = Get-AzNetworkWatcher -Location westeurope
Test-AzNetworkWatcherConnectivity -NetworkWatcher $nw -SourceId $vm.Id -DestinationAddress 10.0.2.4 -DestinationPort 1433
Set-AzNetworkWatcherFlowLog -NetworkWatcher $nw -Name fl-vnet -TargetResourceId $vnet.Id -StorageId $st.Id -Enabled $true
```

Portal: **Network Watcher** → Supervisión (Connection Monitor, Topology), Diagnóstico de red (IP flow verify, Next hop, Connection troubleshoot, Packet capture), Registros (Flow logs, Traffic Analytics).

## Configuración relevante para el examen

| Escenario | Herramienta |
|---|---|
| Vigilar de forma continua la conectividad VM ↔ base de datos con alertas | **Connection Monitor** |
| Medir latencia entre dos regiones o hacia on-premises | **Connection Monitor** |
| Saber qué regla NSG bloquea ahora mismo | **IP flow verify** |
| Ver a dónde envía el tráfico una UDR | **Next hop** |
| Analizar qué flujos se deniegan en toda la VNet | **VNet flow logs + Traffic Analytics** |
| Capturar paquetes para analizar con Wireshark | **Packet capture** (requiere agente) |
| Diagnosticar un túnel VPN caído | **VPN troubleshoot** |
| Ver el diagrama de la red | **Topology** |

## Ejemplo

Tras varias quejas de lentitud intermitente entre la capa de aplicación y SQL, se crea un **Connection Monitor** con origen las 4 VMs de aplicación y destino la IP privada de SQL por TCP 1433 cada 30 segundos, con umbral de fallo del 5 % y RTT de 10 ms. Los resultados van a Log Analytics y una alerta avisa cuando se superan los umbrales. En paralelo se habilitan **VNet flow logs con Traffic Analytics** para detectar flujos denegados.

## Comparaciones

| Herramienta | Momento | Qué mide | Cuándo utilizarla |
|---|---|---|---|
| **Connection troubleshoot** | Puntual | Conectividad, saltos, latencia | Diagnóstico único |
| **Connection Monitor** | **Continuo** | Disponibilidad y latencia con umbrales | Supervisión y alertas |
| **IP flow verify** | Puntual | Permiso NSG | Bloqueos |
| **Flow logs + Traffic Analytics** | Histórico | Todos los flujos | Auditoría y patrones |
| **Packet capture** | Puntual | Paquetes | Análisis profundo |
| **Network Insights** | Continuo | Salud de recursos de red | Vista general |

## AZ-104 Exam Tips

- 🔥 🧠 **Connection Monitor = supervisión continua con alertas**; *Connection troubleshoot* = prueba puntual.
- 🔥 🧠 **NSG flow logs se retiran (30/09/2027)** → **VNet flow logs**; con **Traffic Analytics** se analizan en Log Analytics.
- 🧠 Connection Monitor necesita el **Network Watcher Agent** en las VMs de origen.
- 🧠 Network Watcher se habilita **por región**.
- 💻 Crear Connection Monitor, habilitar flow logs, ejecutar diagnósticos.
- 📌 Diagnóstico (puntual) vs supervisión (continua) vs registro (histórico).

## Preguntas que podrían aparecer

**1.** Necesitas medir de forma continua la latencia y las pérdidas entre máquinas virtuales de dos regiones, con alertas cuando se superen los umbrales. ¿Qué usas?
- A) Connection troubleshoot · B) Connection Monitor · C) Packet capture · D) Topology

<details><summary>Respuesta</summary>

**B.** Connection Monitor realiza pruebas periódicas con umbrales y alertas.
</details>

**2.** ¿Qué combinación te permite analizar en Log Analytics todos los flujos permitidos y denegados de una red virtual?
- A) NSG flow logs clásicos · B) VNet flow logs con Traffic Analytics · C) Packet capture · D) Effective security rules

<details><summary>Respuesta</summary>

**B.** Es el reemplazo actual de los NSG flow logs y añade el análisis en Log Analytics.
</details>

## Relacionado

- [[15 - Solución de problemas de conectividad de red]]
- [[14 - Solución de problemas de balanceo de carga]]
- [[06 - Azure Monitor Insights (VM, Storage, Network)]]
- [[05 - Alertas, grupos de acciones y reglas de procesamiento]]
- [[00 - Índice - Monitorización y mantenimiento]]
