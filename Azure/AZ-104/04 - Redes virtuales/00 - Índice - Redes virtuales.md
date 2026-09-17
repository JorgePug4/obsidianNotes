---
tags: [az-104, azure, redes, networking, MOC]
tipo: MOC
modulo: Redes virtuales
peso_examen: 15-20 %
---

# AZ-104 · Dominio 4 · Implementar y administrar redes virtuales

> [!important] Peso en el examen: **15-20 %** (bajó desde el 20-25 % en la revisión de 2026)
> En AZ-900 bastaba con saber qué servicio elegir. Aquí hay que **calcular subredes**, **predecir el resultado de un NSG**, **decidir entre service endpoint y private endpoint**, **configurar DNS** y **diagnosticar** por qué algo no conecta.

## Objetivos oficiales → notas

### 4.1 Configurar y administrar redes virtuales en Azure

| Objetivo oficial | Nota |
|---|---|
| Crear y configurar redes virtuales y subredes | [[01 - Azure Virtual Network y subredes]] |
| Crear y configurar el emparejamiento (peering) de redes virtuales | [[03 - Peering de redes virtuales]] |
| Configurar direcciones IP públicas | [[02 - Direcciones IP públicas y privadas]] |
| Configurar rutas de red definidas por el usuario | [[04 - Rutas definidas por el usuario (UDR) y NVA]] |
| Solucionar problemas de conectividad de red | [[15 - Solución de problemas de conectividad de red]] |

### 4.2 Configurar el acceso seguro a redes virtuales

| Objetivo oficial | Nota |
|---|---|
| Crear y configurar grupos de seguridad de red y grupos de seguridad de aplicación | [[05 - Network Security Group (NSG)]] · [[06 - Application Security Group (ASG)]] |
| Evaluar las reglas de seguridad efectivas en los NSG | [[07 - Reglas de seguridad efectivas]] |
| Implementar Azure Bastion | [[08 - Azure Bastion]] |
| Configurar puntos de conexión de servicio en subredes | [[09 - Service Endpoints]] |
| Configurar puntos de conexión privados | [[10 - Private Endpoint y Private Link]] |

### 4.3 Configurar la resolución de nombres y el equilibrio de carga

| Objetivo oficial | Nota |
|---|---|
| Configurar Azure DNS | [[11 - Azure DNS (zonas públicas)]] · [[12 - Azure Private DNS y resolución de nombres]] |
| Configurar un balanceador de carga interno o público | [[13 - Azure Load Balancer]] |
| Solucionar problemas de equilibrio de carga | [[14 - Solución de problemas de balanceo de carga]] |
| Contexto necesario | [[16 - Conectividad híbrida - VPN Gateway y ExpressRoute]] ➕ · [[17 - Comparación de balanceadores (Load Balancer, Application Gateway, Front Door, Traffic Manager)]] |

### Repaso
- [[99 - Repaso final - Redes virtuales]]

## Orden de estudio sugerido

1. Fundamentos de VNet, IPs, peering y rutas (01-04).
2. Seguridad: NSG, ASG, reglas efectivas, Bastion, endpoints (05-10).
3. DNS y balanceo (11-14).
4. Contexto híbrido y comparativa de balanceadores (16-17).
5. Diagnóstico (15) y repaso (99).

> [!tip] La idea central del dominio
> Dibuja siempre el escenario. La mayoría de preguntas se resuelven sabiendo: **quién enruta** (rutas del sistema → UDR → BGP), **quién filtra** (NSG de subred + NSG de NIC, ambos deben permitir) y **cómo se resuelve el nombre** (DNS de Azure, DNS privado, DNS propio).

Volver: [[00 - AZ-104 Índice general (MOC)]] · Repaso de fundamentos: [[00 - Introduccion a Azure Networking]] (AZ-900)
