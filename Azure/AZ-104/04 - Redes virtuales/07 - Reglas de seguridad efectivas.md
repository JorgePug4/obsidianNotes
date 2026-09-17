---
tags: [az-104, azure, redes, nsg, diagnostico, reglas-efectivas]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Reglas de seguridad efectivas

## ¿Qué es?

Las **reglas de seguridad efectivas** son la vista combinada de **todas** las reglas que se aplican a una interfaz de red: las del **NSG de la subred**, las del **NSG de la NIC**, las **predeterminadas** y las expandidas de **etiquetas de servicio y ASGs**. Es la herramienta principal para responder "¿por qué no conecta?".

## ¿Para qué sirve?

- Ver qué regla concreta permite o bloquea un flujo.
- Depurar solapamientos entre NSG de subred y de NIC.
- Responder preguntas de examen del tipo "¿se permite este tráfico?".

## Conceptos clave

### Cómo se evalúa un flujo 🧠

```
Tráfico ENTRANTE a una VM:
   1. NSG de la SUBRED  →  si deniega, fin
   2. NSG de la NIC     →  si deniega, fin
   3. Se entrega a la VM

Tráfico SALIENTE de una VM:
   1. NSG de la NIC     →  si deniega, fin
   2. NSG de la SUBRED  →  si deniega, fin
   3. Sale de la VNet
```

- Dentro de cada NSG: orden por **prioridad ascendente**, **primera coincidencia gana**.
- **Ambos** NSGs deben permitir → el resultado es el **más restrictivo**.
- Las reglas son **stateful**: no hace falta regla de vuelta.
- Si no hay NSG en la subred, solo aplica el de la NIC (y viceversa). Si no hay ninguno, aplican solo las reglas predeterminadas de la plataforma (que permiten todo dentro de la VNet y bloquean entrada desde Internet).

### Herramientas 🧠

| Herramienta | Qué responde |
|---|---|
| **Reglas de seguridad efectivas** (NIC o VM → Redes) | Lista completa y ordenada de reglas aplicadas |
| **Verificación de flujo de IP** (IP flow verify, Network Watcher) | ¿Se permite este flujo concreto? Devuelve **Allow/Deny y el nombre de la regla** |
| **Next hop** (Network Watcher) | ¿Por dónde sale el tráfico? (enrutamiento, no filtrado) |
| **Connection troubleshoot** | Prueba de conectividad extremo a extremo |
| **Rutas efectivas** | Rutas aplicadas a la NIC |
| **VNet flow logs** | Registro de flujos permitidos/denegados |

## Cómo funciona

```bash
# Reglas efectivas de una NIC
az network nic list-effective-nsg -g rg-net -n nic-web01 -o json
# ¿Se permite este flujo?
az network watcher test-ip-flow -g rg-net --vm vm-web01 --direction Inbound --protocol TCP \
  --local 10.0.1.4:443 --remote 203.0.113.5:60000
# Conectividad extremo a extremo
az network watcher test-connectivity -g rg-net --source-resource vm-web01 --dest-address 10.0.2.4 --dest-port 1433
# Rutas efectivas
az network nic show-effective-route-table -g rg-net -n nic-web01 -o table
```

```powershell
Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName nic-web01 -ResourceGroupName rg-net
Test-AzNetworkWatcherIPFlow -NetworkWatcher $nw -TargetVirtualMachineId $vm.Id -Direction Inbound -Protocol TCP `
  -LocalIPAddress 10.0.1.4 -LocalPort 443 -RemoteIPAddress 203.0.113.5 -RemotePort 60000
```

Portal: VM o NIC → **Redes** → **Reglas de seguridad efectivas**; Network Watcher → **Verificación de flujo de IP**.

## Ejercicio resuelto (tipo examen)

**NSG de subred `nsg-sub`:**
| Prio | Dirección | Origen | Puerto | Acción |
|---|---|---|---|---|
| 100 | Inbound | Internet | 443 | Allow |
| 200 | Inbound | Internet | 80 | Allow |
| 300 | Inbound | 10.0.2.0/24 | 1433 | Deny |

**NSG de NIC `nsg-nic`:**
| Prio | Dirección | Origen | Puerto | Acción |
|---|---|---|---|---|
| 150 | Inbound | Internet | 443 | Deny |
| 160 | Inbound | 10.0.2.0/24 | 1433 | Allow |

| Flujo | Resultado | Motivo |
|---|---|---|
| Internet → 443 | **Denegado** | La subred lo permite (100) pero la NIC lo deniega (150) |
| Internet → 80 | **Permitido** | Subred 200 Allow; la NIC no tiene regla → predeterminada `DenyAllInBound`… ⚠️ **Denegado**: la NIC deniega por la regla predeterminada 65500 |
| 10.0.2.0/24 → 1433 | **Denegado** | La subred lo deniega en 300 antes de llegar a la NIC |
| 10.0.1.5 (misma subred) → 3389 | **Permitido** | Regla predeterminada `AllowVNetInBound` en ambos NSGs |

> [!warning] La trampa más frecuente
> Cuando el NSG de la NIC existe pero **no tiene una regla Allow** para ese tráfico, la regla predeterminada **DenyAllInBound (65500)** lo bloquea. Añadir la regla solo en la subred no basta.

## Configuración relevante para el examen

| Síntoma | Diagnóstico |
|---|---|
| No conecto por RDP desde Internet | Reglas efectivas: buscar Allow 3389; comprobar ambos NSGs |
| Funciona desde la VNet pero no desde fuera | La regla Allow solo cubre `VirtualNetwork` |
| La sonda del balanceador marca la VM no saludable | Falta Allow desde `AzureLoadBalancer` |
| La VM no sale a Internet | Regla Deny outbound a `Internet` o UDR a NVA |
| Quiero saber qué regla bloquea | **IP flow verify** devuelve el nombre de la regla |
| El tráfico sale por donde no debe | **Next hop** / rutas efectivas (es enrutamiento, no NSG) |

## Comparaciones

| Herramienta | Filtrado | Enrutamiento | Extremo a extremo |
|---|---|---|---|
| **Reglas efectivas** | ✔ | ✘ | ✘ |
| **IP flow verify** | ✔ (dice la regla) | ✘ | ✘ |
| **Rutas efectivas** | ✘ | ✔ | ✘ |
| **Next hop** | ✘ | ✔ | ✘ |
| **Connection troubleshoot** | ✔ | ✔ | ✔ |
| **VNet flow logs** | ✔ (histórico) | ✘ | ✘ |

## 💻 Laboratorio: reglas efectivas

1. Crear un NSG en la subred con Allow 3389 desde Internet y comprobar el acceso RDP.
2. Crear un segundo NSG en la NIC **sin** reglas de entrada y comprobar que RDP deja de funcionar.
3. Abrir **Reglas de seguridad efectivas** y localizar `DenyAllInBound` en el NSG de la NIC.
4. Ejecutar **IP flow verify** para el puerto 3389 y anotar el nombre de la regla que bloquea.
5. Añadir la regla Allow en el NSG de la NIC y repetir la comprobación.

## AZ-104 Exam Tips

- ⭐ **Ambos NSGs deben permitir**: entrada subred→NIC, salida NIC→subred.
- 🔥 🧠 Un NSG sin Allow para ese flujo **deniega** por la regla predeterminada 65500.
- 🔥 🧠 **IP flow verify** devuelve **Allow/Deny y la regla responsable**.
- 🧠 Las reglas son **stateful**: no necesitas la regla de vuelta.
- 💻 `az network nic list-effective-nsg`, `az network watcher test-ip-flow`, portal → Reglas de seguridad efectivas.
- 📌 NSG = filtrado; rutas efectivas/Next hop = enrutamiento.

## Errores comunes

- Añadir la regla solo en la subred cuando la NIC tiene su propio NSG.
- Crear la regla de vuelta "por si acaso" (innecesaria, es stateful).
- Usar Next hop para diagnosticar un bloqueo de puerto.

## Preguntas que podrían aparecer

**1.** El NSG de la subred permite el puerto 443 desde Internet. La NIC tiene un NSG sin ninguna regla de entrada personalizada. ¿Llega el tráfico HTTPS a la VM?
- A) Sí · B) No, la regla predeterminada DenyAllInBound del NSG de la NIC lo bloquea · C) Solo desde la VNet · D) Depende de la prioridad

<details><summary>Respuesta</summary>

**B.** Ambos NSG deben permitirlo; el de la NIC deniega por defecto.
</details>

**2.** ¿Qué herramienta te dice exactamente qué regla de NSG está bloqueando una conexión concreta?
- A) Next hop · B) Rutas efectivas · C) Verificación de flujo de IP (IP flow verify) · D) Connection Monitor

<details><summary>Respuesta</summary>

**C.** IP flow verify evalúa el flujo y devuelve la regla que lo permite o lo deniega.
</details>

## Relacionado

- [[05 - Network Security Group (NSG)]]
- [[06 - Application Security Group (ASG)]]
- [[15 - Solución de problemas de conectividad de red]]
- [[07 - Network Watcher y Connection Monitor]]
- [[00 - Índice - Redes virtuales]]
