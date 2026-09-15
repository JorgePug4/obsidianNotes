---
tags: [az-900, azure, redes, networking]
modulo: Redes
---

# Introducción a Azure Networking (AZ-900)

## Concepto

Azure Networking es el conjunto de servicios que conectan tus recursos en la nube entre sí, con Internet y con tu red local (on-premises). Sin red no hay comunicación: una máquina virtual sin VNet no puede hablar con nadie.

**Problema que resuelve:** en un centro de datos físico necesitas switches, cables, firewalls y routers. En Azure todo eso es **software definido**: lo creas en minutos desde el portal, sin comprar hardware.

**Para qué se utiliza:**
- Aislar recursos en redes privadas.
- Filtrar tráfico (seguridad).
- Repartir carga entre servidores.
- Conectar la oficina con Azure.
- Entregar contenido rápido a usuarios de todo el mundo.

## Mapa del módulo

| Nota | Pregunta que responde | Peso en AZ-900 |
|---|---|---|
| [[01 - Azure Virtual Network]] | ¿Dónde viven mis recursos? | Alto |
| [[02 - Espacio de direcciones y subredes]] | ¿Cómo organizo las IPs? | Medio |
| [[03 - Grupo de seguridad de red (NSG)]] | ¿Qué tráfico dejo pasar? | Medio |
| [[04 - Azure Load Balancer]] | ¿Cómo reparto tráfico entre VMs? | Bajo-Medio |
| [[05 - Azure VPN Gateway]] | ¿Cómo conecto mi oficina por Internet cifrado? | Alto |
| [[06 - Azure Application Gateway]] | ¿Cómo reparto tráfico web con inteligencia? | Bajo |
| [[07 - Azure Content Delivery Network]] | ¿Cómo entrego contenido rápido globalmente? | Bajo |
| [[08 - Azure ExpressRoute]] | ¿Cómo conecto mi oficina SIN Internet? | Alto |
| [[09 - Repaso final de Redes]] | Todo junto | - |

> [!important] Qué evalúa el AZ-900 en redes (temario oficial 2026)
> El objetivo oficial dice: "Describir redes virtuales, incluyendo el propósito de las **Azure Virtual Networks**, **subredes**, **peering**, **Azure DNS**, **VPN Gateway** y **ExpressRoute**".
> Load Balancer, Application Gateway, NSG y CDN ya no son objetivos explícitos, pero siguen apareciendo como distractores o en preguntas de "elige el servicio adecuado". Conócelos a nivel de concepto y caso de uso, sin profundizar.

## La idea central para el examen

Casi todas las preguntas de redes en AZ-900 se reducen a **elegir el servicio correcto según el escenario**. Aprende a asociar palabras clave con servicios:

| Si la pregunta dice... | La respuesta suele ser... |
|---|---|
| "red privada", "aislar recursos", "subred" | Virtual Network |
| "conectar dos VNets" | Peering |
| "conexión cifrada por Internet", "sitio a sitio" | VPN Gateway |
| "conexión privada dedicada", "no pasa por Internet" | ExpressRoute |
| "filtrar tráfico por puerto/IP", "permitir/denegar" | NSG |
| "distribuir tráfico capa 4 / TCP-UDP" | Load Balancer |
| "distribuir tráfico web capa 7 / HTTP, por URL" | Application Gateway |
| "cachear contenido estático cerca del usuario" | CDN |
| "resolver nombres de dominio" | Azure DNS |

> [!tip] Capas OSI que sí necesitas
> Solo dos: **capa 4** (transporte: TCP/UDP, puertos) y **capa 7** (aplicación: HTTP/HTTPS, URLs). Load Balancer trabaja en capa 4; Application Gateway en capa 7. Es la distinción de redes más preguntada.

## 🧠 Resumen para el examen

1. La red en Azure es **software definido**: sin hardware, se crea desde el portal.
2. La **VNet** es la base de todo; el resto de servicios se conecta a ella.
3. **VPN Gateway** = Internet cifrado. **ExpressRoute** = línea privada sin Internet.
4. **NSG** filtra tráfico con reglas permitir/denegar por IP y puerto.
5. **Load Balancer** = capa 4. **Application Gateway** = capa 7 (HTTP).
6. **CDN** acerca el contenido al usuario mediante caché en puntos de presencia.
7. El examen pregunta **cuál servicio usar**, no cómo configurarlo.
