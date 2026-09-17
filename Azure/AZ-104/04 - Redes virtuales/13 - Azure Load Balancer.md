---
tags: [az-104, azure, redes, load-balancer, balanceo]
modulo: Redes virtuales
peso_examen: Muy alto
---

# Azure Load Balancer

## ¿Qué es?

**Azure Load Balancer** distribuye tráfico **TCP/UDP (capa 4)** entre un conjunto de recursos backend (VMs, VMSS) dentro de una región. Puede ser **público** (frontend con IP pública) o **interno/privado** (frontend con IP privada de la VNet). Fundamentos en [[04 - Azure Load Balancer]] (AZ-900).

## ¿Para qué sirve?

- Repartir carga entre varias VMs y sacar de rotación las que fallan.
- Publicar una aplicación con alta disponibilidad (zonas).
- Balancear tráfico interno entre capas (web → app → base de datos).
- Dar salida a Internet controlada (reglas de salida).

## Conceptos clave

### SKUs 🧠

| | **Basic (retirado el 30/09/2025)** | **Standard** |
|---|---|---|
| Backend | Hasta 300 instancias | **1000** |
| Zonas | No | **Zona-redundante o zonal** |
| Seguridad | Abierto por defecto | **Cerrado por defecto: requiere NSG** |
| SLA | No | **99,99 %** |
| Sondas | TCP, HTTP | TCP, HTTP, **HTTPS** |
| Reglas de salida | No | **Sí** |
| Backend pool | Solo VMs de un availability set/VMSS | VMs, VMSS, **por IP** |

- Para despliegues nuevos: **Standard**. La IP pública debe ser también **Standard**.

### Componentes 🧠

| Componente | Qué es |
|---|---|
| **Frontend IP configuration** | IP pública (LB público) o privada (LB interno). Puede haber varias |
| **Backend pool** | VMs/NICs/VMSS/IPs que reciben el tráfico |
| **Health probe** | Comprobación periódica: protocolo (TCP/HTTP/HTTPS), puerto, ruta, **intervalo** (5 s por defecto) y umbral de fallos |
| **Load balancing rule** | Frontend:puerto → backend:puerto, protocolo, **distribución de sesión**, **Floating IP**, **tiempo de espera de inactividad** (4-30 min, por defecto 4) |
| **Inbound NAT rule** | Reenvío de puerto a **una** VM concreta (por ejemplo 50001 → 3389 de vm1) |
| **Outbound rule** (Standard) | SNAT explícito con puertos asignados |

### Distribución 🧠
- **Hash de 5 tuplas** (por defecto): IP origen, puerto origen, IP destino, puerto destino, protocolo → cada conexión puede ir a una VM distinta.
- **Client IP (2 tuplas)**: misma IP de cliente → misma VM (afinidad de sesión).
- **Client IP and protocol (3 tuplas)**: IP + protocolo → misma VM.

### Sondas de estado 🧠
- Origen: **168.63.129.16** (permitir con la etiqueta `AzureLoadBalancer` en el NSG).
- Si la sonda falla, la instancia se saca de rotación (las conexiones existentes pueden mantenerse según configuración).
- Si **todas** las instancias están no saludables, el comportamiento depende (Standard corta el tráfico).

### Otros
- **Zonas**: frontend **zona-redundante** (recomendado) o **zonal**.
- **HA Ports** (LB interno Standard): balancea **todos los puertos** a la vez, para NVAs.
- **Cross-region Load Balancer** ➕: frontend global sobre balanceadores regionales.
- **Gateway Load Balancer** ➕: insertar NVAs de terceros de forma transparente.
- Límite: una VM solo puede estar en backend pools de balanceadores del **mismo SKU**.

## Cómo funciona

```bash
# LB público estándar zona-redundante
az network lb create -g rg-net -n lb-web --sku Standard --public-ip-address pip-lb --frontend-ip-name fe-web --backend-pool-name bp-web
# Sonda HTTP
az network lb probe create -g rg-net --lb-name lb-web -n probe-http --protocol Http --port 80 --path /health --interval 5 --threshold 2
# Regla de balanceo
az network lb rule create -g rg-net --lb-name lb-web -n rule-http --protocol Tcp \
  --frontend-port 80 --backend-port 80 --frontend-ip-name fe-web --backend-pool-name bp-web \
  --probe-name probe-http --idle-timeout 4 --load-distribution Default --disable-outbound-snat true
# Añadir VMs al pool
az network nic ip-config address-pool add -g rg-net --nic-name nic-web01 --ip-config-name ipconfig1 --lb-name lb-web --address-pool bp-web
# Regla NAT de entrada (RDP a una VM concreta)
az network lb inbound-nat-rule create -g rg-net --lb-name lb-web -n rdp-vm1 --protocol Tcp --frontend-port 50001 --backend-port 3389 --frontend-ip-name fe-web
# LB interno
az network lb create -g rg-net -n lb-int --sku Standard --vnet-name vnet-hub --subnet snet-app --private-ip-address 10.0.2.100 --frontend-ip-name fe-int --backend-pool-name bp-app
# Regla de salida
az network lb outbound-rule create -g rg-net --lb-name lb-web -n out-rule --frontend-ip-configs fe-web --protocol All --address-pool bp-web --idle-timeout 15
```

```powershell
$probe = New-AzLoadBalancerProbeConfig -Name probe-http -Protocol Http -Port 80 -RequestPath /health -IntervalInSeconds 5 -ProbeCount 2
$rule = New-AzLoadBalancerRuleConfig -Name rule-http -FrontendIpConfiguration $fe -BackendAddressPool $bp -Probe $probe -Protocol Tcp -FrontendPort 80 -BackendPort 80
New-AzLoadBalancer -ResourceGroupName rg-net -Name lb-web -Location westeurope -Sku Standard -FrontendIpConfiguration $fe -BackendAddressPool $bp -Probe $probe -LoadBalancingRule $rule
```

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| Balancear tráfico HTTP entre 3 VMs web | LB **público** Standard + sonda HTTP + regla 80→80 |
| Balancear entre capas internas | LB **interno** con IP privada |
| Cada usuario debe ir siempre a la misma VM | Distribución **Client IP** (2 tuplas) |
| RDP a cada VM por un puerto distinto | **Reglas NAT de entrada** (50001→3389, 50002→3389…) |
| Balancear todos los puertos hacia una NVA | **HA Ports** (LB interno Standard) |
| SLA 99,99 % | LB **Standard** con frontend **zona-redundante** y VMs en varias zonas |
| Enrutar por URL o terminar TLS | **No es Load Balancer** → Application Gateway |
| Balanceo entre regiones | **Front Door / Traffic Manager / Cross-region LB** |
| Salida a Internet con puertos SNAT controlados | **Reglas de salida** |

## Ejemplo

Tres VMs web en las zonas 1, 2 y 3 sirven una aplicación. Se crea un **Standard Load Balancer público** con IP zona-redundante, backend pool con las tres NICs, sonda HTTP a `/health` cada 5 s y regla 80→80 con distribución por defecto. El NSG de la subred permite 80 desde `Internet` y desde `AzureLoadBalancer`. El SLA resultante es del 99,99 %.

## Comparaciones

| Servicio | Capa | Ámbito | Cuándo utilizarlo |
|---|---|---|---|
| **Load Balancer** | 4 (TCP/UDP) | Regional | Cualquier protocolo, alto rendimiento, bajo coste |
| **Application Gateway** | 7 (HTTP/S) | Regional | Enrutamiento por URL/host, TLS, WAF, cookies |
| **Front Door** | 7 | **Global** | Web global, caché, WAF, aceleración |
| **Traffic Manager** | DNS | **Global** | Enrutar por DNS a regiones/endpoints |
| **NAT Gateway** | — | Subred | Solo salida a Internet |

Detalle completo en [[17 - Comparación de balanceadores (Load Balancer, Application Gateway, Front Door, Traffic Manager)]].

## 💻 Laboratorio: balanceador público

1. Crear dos VMs Linux en zonas distintas con nginx (páginas distinguibles) y sin IP pública.
2. Crear un **Standard LB** público con sonda HTTP y regla 80→80; añadir ambas NICs al backend pool.
3. Permitir 80 desde `Internet` y `AzureLoadBalancer` en el NSG.
4. Acceder varias veces a la IP del LB y ver cómo alterna entre las VMs.
5. Detener nginx en una VM y comprobar que la sonda la saca de rotación.

## AZ-104 Exam Tips

- ⭐ **Capa 4**: no entiende URLs ni TLS. Para eso, Application Gateway.
- 🔥 🧠 **Standard: cerrado por defecto (necesita NSG), zona-redundante, SLA 99,99 %**; Basic está retirado.
- 🔥 🧠 Componentes: **frontend, backend pool, health probe, regla de balanceo** (+ NAT de entrada y reglas de salida).
- 🧠 Distribución por defecto: **hash de 5 tuplas**; afinidad con **Client IP**.
- 🧠 Sondas desde **168.63.129.16** → permitir `AzureLoadBalancer`.
- 🧠 La IP pública y el LB deben tener el **mismo SKU**.
- 💻 `az network lb create/probe create/rule create`, añadir NICs al pool, NAT rules.
- 📌 LB público (Internet) vs interno (IP privada).

## Errores comunes

- Bloquear las sondas en el NSG y ver todo el backend no saludable.
- Mezclar IP Basic con LB Standard.
- Usar Load Balancer cuando el enunciado pide enrutamiento por ruta o WAF.

## Preguntas que podrían aparecer

**1.** Todas las instancias del backend aparecen como no saludables aunque la aplicación funciona. ¿Cuál es la causa más probable?
- A) La sonda usa HTTPS · B) El NSG no permite el tráfico desde la etiqueta AzureLoadBalancer (168.63.129.16) · C) El SKU es Standard · D) Falta una regla NAT

<details><summary>Respuesta</summary>

**B.** Las sondas se originan en 168.63.129.16 y deben permitirse explícitamente.
</details>

**2.** Necesitas que todas las peticiones de un mismo cliente vayan siempre a la misma máquina virtual. ¿Qué configuras?
- A) Floating IP · B) Distribución de sesión Client IP · C) HA Ports · D) Regla NAT de entrada

<details><summary>Respuesta</summary>

**B.** La distribución por IP de cliente (2 tuplas) mantiene la afinidad de sesión.
</details>

**3.** Debes permitir RDP a tres VMs concretas detrás de un balanceador usando puertos 50001, 50002 y 50003. ¿Qué configuras?
- A) Tres reglas de balanceo · B) Tres reglas NAT de entrada · C) HA Ports · D) Reglas de salida

<details><summary>Respuesta</summary>

**B.** Las reglas NAT de entrada mapean un puerto del frontend a una instancia concreta.
</details>

## Relacionado

- [[14 - Solución de problemas de balanceo de carga]]
- [[17 - Comparación de balanceadores (Load Balancer, Application Gateway, Front Door, Traffic Manager)]]
- [[09 - Alta disponibilidad - Availability Sets y Availability Zones]]
- [[05 - Network Security Group (NSG)]]
- [[04 - Azure Load Balancer]] (AZ-900)
- [[00 - Índice - Redes virtuales]]
