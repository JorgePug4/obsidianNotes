---
tags: [az-900, azure, redes, load-balancer]
modulo: Redes
peso_examen: Bajo-Medio
---

# Azure Load Balancer

## Concepto

**Azure Load Balancer** distribuye el tráfico de red entrante entre varias máquinas virtuales para que ninguna se sature y para que, si una falla, las demás sigan atendiendo. Trabaja en **capa 4** (TCP/UDP): mira IPs y puertos, no el contenido de la petición.

**Problema que resuelve:** una sola VM es un punto único de fallo y tiene capacidad limitada. Con varias VMs detrás de un balanceador consigues **alta disponibilidad** y **escalabilidad**.

**Para qué se utiliza:**
- Repartir conexiones entre un grupo de VMs (backend pool).
- Detectar VMs caídas mediante **sondas de estado (health probes)** y dejar de enviarles tráfico.
- Exponer un conjunto de VMs con una única IP.

## Características principales

- **Capa 4 (transporte):** reparte según IP y puerto, protocolo TCP o UDP. No entiende HTTP.
- **Tipos según alcance:**
  - **Público:** tiene IP pública; recibe tráfico de Internet y lo reparte a VMs con IP privada.
  - **Interno (privado):** solo tiene IP privada; reparte tráfico dentro de la VNet (por ejemplo, de la capa web a la capa de aplicación).
- **Componentes:** IP frontend, **backend pool** (las VMs), **health probe** (comprueba que cada VM responde), reglas de balanceo.
- **SKU:** Basic (en retirada) y **Standard** (recomendado, con soporte de zonas de disponibilidad). En AZ-900 basta con saber que Standard es el actual.
- **Regional:** opera dentro de una única región. Para balancear entre regiones existen otros servicios (ver comparación).
- Alta disponibilidad: si una VM no responde al health probe, deja de recibir tráfico de inmediato.

> [!warning] Confusiones frecuentes
> - **Load Balancer vs. Application Gateway:** los dos "balancean", pero Load Balancer es **capa 4** (cualquier protocolo TCP/UDP) y Application Gateway es **capa 7** (solo HTTP/HTTPS, enruta por URL, termina SSL, incluye WAF). Ver [[06 - Azure Application Gateway]].
> - **Load Balancer vs. Traffic Manager / Front Door:** Load Balancer es regional y trabaja a nivel de red. Traffic Manager es un balanceador **basado en DNS** y **global**. Front Door es capa 7 y **global**.
> - **Load Balancer no es Virtual Machine Scale Sets:** el Scale Set crea y elimina VMs según demanda; el balanceador reparte tráfico entre las que existen. Suelen usarse juntos.

## Casos de uso

- Tres VMs con un servidor web idéntico detrás de un Load Balancer público: los usuarios entran por una única IP y el tráfico se reparte entre las tres.
- Un Load Balancer interno entre los servidores web y los servidores de aplicación, de forma que la capa de aplicación no tenga IP pública.
- Un servicio de juegos por UDP que necesita repartir conexiones: Application Gateway no sirve (solo HTTP), Load Balancer sí.

## Comparaciones

| Servicio | Capa OSI | Alcance | Protocolos | Cuándo usarlo |
|---|---|---|---|---|
| **Load Balancer** | 4 | Regional | TCP, UDP | Repartir tráfico genérico entre VMs de una región |
| **Application Gateway** | 7 | Regional | HTTP, HTTPS | Tráfico web con enrutamiento por URL, SSL, WAF |
| **Traffic Manager** | DNS | Global | Cualquiera (responde con DNS) | Dirigir usuarios a la región más cercana o sana |
| **Front Door** | 7 | Global | HTTP, HTTPS | Aplicaciones web globales con aceleración y WAF |

## Conceptos que debo memorizar

> [!important]
> - Load Balancer = **capa 4**, **TCP/UDP**, **regional**.
> - **Público** (IP pública, tráfico desde Internet) o **interno** (IP privada, tráfico dentro de la VNet).
> - **Backend pool** = las VMs que reciben tráfico.
> - **Health probe** = comprueba que las VMs estén sanas.
> - Proporciona **alta disponibilidad** y **escalabilidad**, no seguridad ni caché.

## Tips para AZ-900

> [!tip] Palabras clave
> "distribuir tráfico entre máquinas virtuales", "capa 4", "TCP o UDP", "alta disponibilidad", "sondas de estado".

> [!tip] Preguntas trampa
> - Si la pregunta menciona **HTTP, URL, cookies, certificados SSL o WAF** → **no** es Load Balancer, es **Application Gateway**.
> - Si la pregunta menciona **varias regiones** o **DNS** → **Traffic Manager** o **Front Door**.
> - Si la pregunta menciona **crear VMs automáticamente según la carga** → **Virtual Machine Scale Sets**, no Load Balancer.
> - "¿El Load Balancer cifra el tráfico?" → **No**. Solo lo reparte.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tienes cuatro máquinas virtuales que ejecutan el mismo servicio sobre TCP en el puerto 5000. Necesitas repartir las conexiones entre ellas y dejar de enviar tráfico a las que fallen. ¿Qué servicio utilizas?

- A) Azure Application Gateway
- B) Azure Load Balancer
- C) Azure ExpressRoute
- D) Network Security Group

**Respuesta correcta: B.** Tráfico TCP genérico entre VMs de una región con detección de fallos = Load Balancer.
- A es incorrecta: Application Gateway solo balancea HTTP/HTTPS.
- C es incorrecta: ExpressRoute conecta on-premises con Azure.
- D es incorrecta: el NSG filtra, no reparte.

**Pregunta 2.** ¿En qué capa del modelo OSI opera Azure Load Balancer?

- A) Capa 2
- B) Capa 4
- C) Capa 7
- D) Capa 3

**Respuesta correcta: B.** Load Balancer trabaja en la capa de transporte (TCP/UDP).
- C corresponde a Application Gateway y Front Door.
- A y D no son niveles en los que opere este servicio como balanceador.

**Pregunta 3.** Quieres que los servidores de aplicación de tu VNet reciban tráfico repartido desde los servidores web, sin que los de aplicación tengan una IP pública. ¿Qué tipo de Load Balancer necesitas?

- A) Load Balancer público
- B) Load Balancer interno
- C) Application Gateway con IP pública
- D) Azure Traffic Manager

**Respuesta correcta: B.** El Load Balancer interno usa solo IP privada y reparte tráfico dentro de la VNet.
- A y C exponen una IP pública, lo contrario de lo pedido.
- D es un servicio global basado en DNS, no un balanceador interno.

## 🧠 Resumen para el examen

1. Load Balancer reparte tráfico **capa 4 (TCP/UDP)** entre VMs.
2. Es **regional**.
3. **Público** = IP pública, tráfico desde Internet. **Interno** = IP privada, tráfico dentro de la VNet.
4. **Backend pool** + **health probe** = alta disponibilidad.
5. No entiende HTTP: para URL, SSL o WAF usa **Application Gateway**.
6. Para varias regiones usa **Traffic Manager** (DNS) o **Front Door** (capa 7).
7. Se combina con **Virtual Machine Scale Sets** para escalar.
