---
tags: [az-104, azure, redes, balanceo, comparacion, waf]
modulo: Redes virtuales
peso_examen: Alto
---

# Comparación de balanceadores

## ¿Qué es?

Azure ofrece cuatro servicios de distribución de tráfico. El examen pregunta **cuál elegir** según la capa (4 o 7), el ámbito (regional o global) y las funciones necesarias (TLS, WAF, caché, enrutamiento por URL).

## Tabla principal 🧠

| Servicio | Capa | Ámbito | Protocolos | Funciones clave | Cuándo utilizarlo |
|---|---|---|---|---|---|
| **Azure Load Balancer** | **4** (TCP/UDP) | **Regional** | Cualquier TCP/UDP | Sondas, NAT, reglas de salida, HA Ports, zonas | Balanceo interno o público de VMs, cualquier protocolo |
| **Application Gateway** | **7** (HTTP/S) | **Regional** | HTTP, HTTPS, HTTP/2, WebSocket | **Enrutamiento por URL y host**, terminación TLS, **WAF**, afinidad por cookies, reescritura de encabezados, redirección | Aplicaciones web dentro de una región |
| **Azure Front Door** | **7** | **Global** | HTTP/S | Anycast global, **WAF**, caché/CDN, aceleración, SSL offload, failover entre regiones, reglas | Aplicaciones web globales |
| **Traffic Manager** | **DNS** (no es proxy) | **Global** | Cualquiera (resuelve nombres) | Métodos de enrutamiento (prioridad, ponderado, rendimiento, geográfico, subred, multivalor), sondas de endpoint | Dirigir usuarios a la región/endpoint adecuado, failover entre regiones |

> [!important] Regla mental
> - ¿**TCP/UDP** o protocolo no HTTP? → **Load Balancer**
> - ¿**HTTP** con URL/host, TLS o **WAF**, en **una** región? → **Application Gateway**
> - ¿**HTTP** **global**, con caché y WAF? → **Front Door**
> - ¿Decidir **por DNS** a qué región va el usuario? → **Traffic Manager**

## Detalles que caen en el examen

### Application Gateway
- **SKU v2** (Standard_v2 y WAF_v2) con **autoescalado** y zona-redundancia; v1 en retirada.
- Componentes: **frontend IP**, **listeners** (básico o multi-sitio), **reglas de enrutamiento** (por ruta o por host), **backend pools**, **HTTP settings** (puerto, protocolo, afinidad, sondas), **health probes**, **WAF policy**.
- Requiere su **propia subred** (solo para gateways, /24 recomendado).
- **WAF**: reglas gestionadas (OWASP CRS) en modo **Detección** o **Prevención**.

### Front Door
- **Standard/Premium**: Premium añade **Private Link al origen**, WAF con reglas de bots gestionadas y protección mejorada.
- Caché de contenido, compresión, reglas de enrutamiento, dominios y certificados gestionados.
- Sustituye a **Azure CDN** en despliegues nuevos (Azure CDN clásico está en retirada).

### Traffic Manager
- **No transporta datos**: solo responde consultas DNS con el endpoint elegido; el cliente conecta directamente.
- **Métodos de enrutamiento** 🧠: *Priority* (failover), *Weighted* (reparto), *Performance* (menor latencia), *Geographic* (por origen), *MultiValue*, *Subnet*.
- Sondas HTTP/HTTPS/TCP sobre los endpoints; TTL configurable (afecta a la velocidad del failover).
- Endpoints: Azure, externos o anidados.

### Load Balancer
- Detalle completo en [[13 - Azure Load Balancer]].

## Combinaciones habituales

```
Usuarios globales
   └─► Front Door (WAF, caché, global)
          └─► Application Gateway regional (WAF, enrutamiento por URL)
                 └─► Load Balancer interno / VMSS / App Service
```

o bien

```
Traffic Manager (DNS, elige región)
   ├─► Región 1: Application Gateway → VMs
   └─► Región 2: Application Gateway → VMs
```

## Comparaciones adicionales

| Pareja | Diferencia en una línea |
|---|---|
| **Load Balancer vs Application Gateway** | Capa 4 (puertos) vs capa 7 (URL, host, TLS, WAF) |
| **Application Gateway vs Front Door** | Regional vs **global** (Front Door añade caché y anycast) |
| **Front Door vs Traffic Manager** | Proxy global que transporta tráfico vs **solo DNS** |
| **Traffic Manager vs Load Balancer** | Global por DNS vs regional por conexión |
| **Front Door vs CDN** | Front Door incluye la funcionalidad de CDN (el CDN clásico está en retirada) |
| **WAF en App Gateway vs en Front Door** | Regional vs global; ambos con OWASP |

## AZ-104 Exam Tips

- 🔥 📌 **Capa 4 = Load Balancer**; **capa 7 regional = Application Gateway**; **capa 7 global = Front Door**; **DNS = Traffic Manager**.
- 🧠 Traffic Manager **no ve el tráfico**, solo resuelve nombres; el failover depende del **TTL**.
- 🧠 **WAF** solo existe en Application Gateway y Front Door.
- 🧠 Application Gateway necesita una **subred dedicada**.
- 🧠 Métodos de Traffic Manager: prioridad, ponderado, rendimiento, geográfico, multivalor, subred.
- 💻 En AZ-104 se pide **configurar Load Balancer**; los demás hay que **reconocerlos** y saber cuándo elegirlos.

## Preguntas que podrían aparecer

**1.** Una aplicación web debe enrutar `/api` a un grupo de servidores y `/web` a otro, con terminación TLS y protección frente a inyección SQL, dentro de una sola región. ¿Qué servicio usas?
- A) Load Balancer · B) Application Gateway con WAF · C) Traffic Manager · D) NAT Gateway

<details><summary>Respuesta</summary>

**B.** El enrutamiento por ruta, la terminación TLS y el WAF son propios de Application Gateway.
</details>

**2.** Quieres dirigir a cada usuario a la región de Azure con menor latencia y hacer failover si una región cae, sin que el tráfico pase por un proxy. ¿Qué servicio eliges?
- A) Front Door · B) Traffic Manager con método de rendimiento · C) Load Balancer entre regiones · D) Application Gateway

<details><summary>Respuesta</summary>

**B.** Traffic Manager decide por DNS y no transporta el tráfico; el método Performance elige la región de menor latencia.
</details>

**3.** Necesitas balancear tráfico UDP entre varias máquinas virtuales. ¿Qué servicio lo permite?
- A) Application Gateway · B) Front Door · C) Azure Load Balancer · D) Traffic Manager

<details><summary>Respuesta</summary>

**C.** Solo el Load Balancer opera en capa 4 con TCP y UDP.
</details>

## Relacionado

- [[13 - Azure Load Balancer]]
- [[14 - Solución de problemas de balanceo de carga]]
- [[11 - Azure DNS (zonas públicas)]]
- [[06 - Azure Application Gateway]] (AZ-900)
- [[00 - Índice - Redes virtuales]]
