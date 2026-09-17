---
tags: [az-104, azure, redes, troubleshooting, network-watcher]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Solución de problemas de conectividad de red

## ¿Qué es?

Metodología y herramientas para diagnosticar por qué dos recursos no se comunican en Azure. Es un objetivo explícito del temario y aparece en preguntas de escenario ("¿qué herramienta usas para…?").

## Método en 6 pasos 🧠

```
1. ¿DNS?        ¿El nombre resuelve a la IP correcta?      → nslookup / zonas privadas
2. ¿Rutas?      ¿Por dónde sale el tráfico?                → Rutas efectivas / Next hop
3. ¿Filtrado?   ¿Algún NSG lo bloquea?                     → Reglas efectivas / IP flow verify
4. ¿Servicio?   ¿La app escucha en ese puerto?             → curl/Test-NetConnection dentro de la VM
5. ¿SO?         ¿Firewall local, servicio parado?          → Windows Firewall / iptables
6. ¿Extremo?    Prueba completa origen→destino             → Connection troubleshoot / Connection Monitor
```

## Herramientas de Network Watcher 🧠

| Herramienta | Responde a | Nota |
|---|---|---|
| **IP flow verify** | ¿Se permite este flujo? | Devuelve Allow/Deny y **la regla NSG** responsable |
| **NSG diagnostics** ➕ | Evaluación completa de NSG para un flujo | Más detalle que IP flow verify |
| **Next hop** | ¿Cuál es el siguiente salto? | Detecta UDR, peering, gateway, None |
| **Effective security rules** | Reglas NSG combinadas | Subred + NIC |
| **Connection troubleshoot** | Prueba puntual origen→destino | Latencia, saltos, causa del fallo |
| **Connection Monitor** | Monitorización **continua** | Alertas, múltiples orígenes/destinos, híbrido |
| **Packet capture** | Captura de paquetes en la VM | Requiere la extensión Network Watcher Agent; guarda en Storage |
| **VPN troubleshoot** | Diagnóstico de gateway y conexiones | VPN/ExpressRoute |
| **VNet flow logs** (sustituyen a **NSG flow logs**, retirados el 30/09/2027) | Registro de flujos permitidos/denegados | Se envían a Storage; con **Traffic Analytics** en Log Analytics |
| **Topology** | Diagrama de la red | Vista rápida |
| **Network Watcher se habilita por región** automáticamente | | Recurso `NetworkWatcherRG` |

## Causas frecuentes por síntoma

| Síntoma | Causas probables |
|---|---|
| No conecto por RDP/SSH desde Internet | NSG sin Allow; IP Standard cerrada; VM sin IP pública (usar Bastion); servicio parado |
| VM a VM en la misma VNet falla | NSG de NIC/subred; firewall del SO; VM apagada |
| VM a VM en VNets emparejadas falla | Peering no conectado, rangos solapados, NSG, **no transitivo** |
| No llego a on-premises | UDR incorrecta, BGP, rangos solapados, reglas del dispositivo VPN |
| No resuelvo un nombre interno | Falta zona privada vinculada; DNS personalizado mal configurado; VM sin reiniciar tras cambiar DNS |
| El private endpoint resuelve a IP pública | Falta la zona `privatelink.*` o el vínculo |
| Sin salida a Internet | Regla Deny outbound; UDR `0.0.0.0/0` a NVA; falta NAT Gateway/IP pública |
| El tráfico no pasa por el firewall | UDR ausente o IP forwarding deshabilitado en la NVA |
| Sondas del balanceador fallan | Ver [[14 - Solución de problemas de balanceo de carga]] |

## Cómo funciona

```bash
# ¿Bloquea un NSG?
az network watcher test-ip-flow -g rg-net --vm vm-app01 --direction Outbound --protocol TCP --local 10.0.1.4:50000 --remote 10.0.2.4:1433
# ¿Por dónde sale?
az network watcher show-next-hop -g rg-net --vm vm-app01 --source-ip 10.0.1.4 --dest-ip 10.0.2.4
# Prueba extremo a extremo
az network watcher test-connectivity -g rg-net --source-resource vm-app01 --dest-address 10.0.2.4 --dest-port 1433
# Reglas y rutas efectivas
az network nic list-effective-nsg -g rg-net -n nic-app01
az network nic show-effective-route-table -g rg-net -n nic-app01 -o table
# Captura de paquetes
az network watcher packet-capture create -g rg-net --vm vm-app01 -n captura1 --storage-account st001 --time-limit 300
# Flow logs de VNet
az network watcher flow-log create -g rg-net -n flowlog-vnet --vnet vnet-hub --storage-account st001 --workspace <lawId> --traffic-analytics true
```

```powershell
Test-AzNetworkWatcherIPFlow -NetworkWatcher $nw -TargetVirtualMachineId $vm.Id -Direction Outbound -Protocol TCP -LocalIPAddress 10.0.1.4 -LocalPort 50000 -RemoteIPAddress 10.0.2.4 -RemotePort 1433
Get-AzNetworkWatcherNextHop -NetworkWatcher $nw -TargetVirtualMachineId $vm.Id -SourceIPAddress 10.0.1.4 -DestinationIPAddress 10.0.2.4
Test-AzNetworkWatcherConnectivity -NetworkWatcher $nw -SourceId $vm.Id -DestinationAddress 10.0.2.4 -DestinationPort 1433
```

## Ejemplo

Una VM de `snet-app` no conecta con SQL en `snet-data`. **IP flow verify** devuelve *Deny* por la regla `deny-all-from-app` del NSG de la subred de datos. Se añade una regla Allow TCP 1433 desde el ASG `asg-app` con prioridad 100. Tras el cambio, **Connection troubleshoot** confirma la conectividad y se activa **Connection Monitor** para vigilarla de forma continua.

## Comparaciones

| Herramienta | Momento | Ámbito | Cuándo utilizarla |
|---|---|---|---|
| **IP flow verify** | Puntual | NSG | Saber qué regla bloquea |
| **Next hop** | Puntual | Enrutamiento | Saber a dónde va el tráfico |
| **Connection troubleshoot** | Puntual | Extremo a extremo | Diagnóstico completo una vez |
| **Connection Monitor** | **Continuo** | Extremo a extremo, híbrido | Vigilancia y alertas |
| **Packet capture** | Puntual | Paquetes | Análisis profundo |
| **VNet flow logs + Traffic Analytics** | Histórico | Todos los flujos | Auditoría y patrones |
| **Topology** | Visual | Red | Entender el entorno |

## 💻 Laboratorio: diagnóstico

1. Crear dos VMs en subredes distintas con un NSG que bloquee el puerto 1433.
2. Ejecutar **IP flow verify** y anotar la regla que deniega.
3. Corregir la regla y repetir; luego usar **Connection troubleshoot** para validar.
4. Crear una UDR con next hop **None** hacia la subred destino y ejecutar **Next hop** para verla.
5. Habilitar **VNet flow logs** con Traffic Analytics y revisar los flujos denegados.

## AZ-104 Exam Tips

- ⭐ Orden mental: **DNS → rutas → NSG → aplicación → SO**.
- 🔥 🧠 **IP flow verify** = qué regla bloquea. **Next hop** = por dónde sale. **Connection troubleshoot** = prueba completa. **Connection Monitor** = continuo.
- 🧠 **NSG flow logs se retiran el 30/09/2027** → usar **VNet flow logs**.
- 🧠 La captura de paquetes requiere la **extensión Network Watcher Agent**.
- 🧠 Network Watcher se habilita **por región**.
- 💻 Todas las herramientas desde el portal (Network Watcher) y CLI.
- ⚠️ Si el problema es de resolución de nombres, ninguna herramienta de NSG lo mostrará: revisa DNS primero.

## Errores comunes

- Usar Next hop para diagnosticar un bloqueo de puerto.
- No comprobar el firewall del sistema operativo.
- Olvidar que el peering no es transitivo al diagnosticar VNets.

## Preguntas que podrían aparecer

**1.** Necesitas saber si una regla de NSG está bloqueando el tráfico entre dos máquinas virtuales. ¿Qué herramienta usas?
- A) Next hop · B) Verificación de flujo de IP · C) Topology · D) Packet capture

<details><summary>Respuesta</summary>

**B.** IP flow verify indica si el flujo se permite o deniega y qué regla lo decide.
</details>

**2.** Quieres monitorizar de forma continua la latencia y la disponibilidad entre una VM de Azure y un servidor on-premises, con alertas. ¿Qué usas?
- A) Connection troubleshoot · B) Connection Monitor · C) IP flow verify · D) Resource Health

<details><summary>Respuesta</summary>

**B.** Connection Monitor realiza pruebas continuas y permite alertas.
</details>

**3.** ¿Qué sustituye a los NSG flow logs, que se retiran?
- A) Diagnostic settings · B) VNet flow logs · C) Activity log · D) Packet capture

<details><summary>Respuesta</summary>

**B.** Los registros de flujo de red virtual son el reemplazo, con Traffic Analytics.
</details>

## Relacionado

- [[07 - Reglas de seguridad efectivas]]
- [[04 - Rutas definidas por el usuario (UDR) y NVA]]
- [[12 - Azure Private DNS y resolución de nombres]]
- [[07 - Network Watcher y Connection Monitor]]
- [[14 - Solución de problemas de balanceo de carga]]
- [[00 - Índice - Redes virtuales]]
