---
tags: [az-900, azure, redes, application-gateway, waf]
modulo: Redes
peso_examen: Bajo
---

# Azure Application Gateway

## Concepto

**Azure Application Gateway** es un balanceador de tráfico **web** que trabaja en **capa 7** (HTTP/HTTPS). A diferencia del [[04 - Azure Load Balancer]], entiende el contenido de la petición: puede leer la URL, el nombre de host o las cookies y decidir a qué servidor enviarla.

**Problema que resuelve:** una aplicación web moderna tiene varias partes (imágenes, API, vídeos) que conviene atender con servidores distintos. Además necesita terminar HTTPS y protegerse de ataques web. Un balanceador de capa 4 no puede hacer nada de eso.

**Para qué se utiliza:**
- Enrutar `/imagenes/*` a un grupo de servidores y `/api/*` a otro.
- Descargar el trabajo de cifrado SSL de los servidores web (**terminación SSL**).
- Proteger la aplicación con **Web Application Firewall (WAF)**.

## Características principales

- **Capa 7 (aplicación):** solo HTTP, HTTPS, HTTP/2 y WebSocket.
- **Enrutamiento basado en URL (path-based):** `/video` → pool A, `/images` → pool B.
- **Enrutamiento multi-sitio:** varios dominios (`contoso.com`, `fabrikam.com`) en un solo gateway.
- **Terminación SSL/TLS:** el gateway descifra el tráfico y lo envía en claro (o recifrado) a los servidores, que así trabajan menos.
- **Afinidad de sesión basada en cookies:** un usuario siempre va al mismo servidor.
- **Web Application Firewall (WAF):** protección contra ataques comunes (inyección SQL, cross-site scripting) según las reglas **OWASP**.
- **Redirección:** por ejemplo, de HTTP a HTTPS.
- **Regional:** opera dentro de una región. Para lo mismo pero global, existe **Azure Front Door**.
- **Escalado automático** (SKU v2) y soporte de zonas de disponibilidad.

> [!warning] Confusiones frecuentes
> - **Application Gateway vs. Load Balancer:** ambos balancean, pero App Gateway es **capa 7 / solo web**, Load Balancer es **capa 4 / cualquier TCP-UDP**. Es la comparación más preguntada.
> - **Application Gateway vs. Azure Front Door:** funcionalidad similar (capa 7, WAF), pero Front Door es **global** y Application Gateway es **regional**.
> - **WAF vs. Azure Firewall vs. NSG:** WAF protege **aplicaciones web** (capa 7, OWASP). Azure Firewall protege la **red** (capa 3-7, FQDN, amenazas). NSG filtra por **IP y puerto** (capa 3-4).

## Casos de uso

- Una tienda online sirve el catálogo desde unos servidores y el checkout desde otros más potentes: Application Gateway enruta por ruta URL.
- Una empresa hospeda tres sitios web en el mismo grupo de VMs y necesita una única IP pública con un certificado por dominio.
- Una aplicación pública recibe intentos de inyección SQL: se activa el **WAF** en el Application Gateway.

## Comparaciones

| | **Load Balancer** | **Application Gateway** | **Front Door** |
|---|---|---|---|
| Capa | 4 | 7 | 7 |
| Protocolos | TCP, UDP | HTTP, HTTPS | HTTP, HTTPS |
| Alcance | Regional | Regional | Global |
| Enrutamiento por URL | No | Sí | Sí |
| Terminación SSL | No | Sí | Sí |
| WAF | No | Sí | Sí |
| Afinidad por cookie | No (solo por IP) | Sí | Sí |
| Cuándo usarlo | Tráfico genérico entre VMs | App web en una región | App web multirregión con aceleración |

## Conceptos que debo memorizar

> [!important]
> - Application Gateway = balanceador **capa 7**, **solo HTTP/HTTPS**.
> - **Enrutamiento por URL** y **multi-sitio**.
> - **Terminación SSL**.
> - **WAF** incluido (OWASP: inyección SQL, XSS).
> - **Regional**. Su equivalente global es **Front Door**.

## Tips para AZ-900

> [!tip] Palabras clave
> "HTTP", "HTTPS", "URL", "ruta", "capa 7", "terminación SSL", "WAF", "aplicación web", "cookies".

> [!tip] Preguntas trampa
> - "Balancear tráfico UDP" → Application Gateway **no** puede. Load Balancer sí.
> - "Proteger una aplicación web contra inyección SQL" → **WAF** (en Application Gateway o Front Door), no NSG ni Azure Firewall.
> - "Usuarios en todo el mundo con la menor latencia" → **Front Door**, no Application Gateway.
> - "Enviar `/api` a un servidor y `/web` a otro" → **Application Gateway** (enrutamiento por ruta).

> [!tip] Peso en el examen
> Application Gateway ya no figura como objetivo explícito en el temario AZ-900 vigente. Aparece sobre todo como distractor frente a Load Balancer o como respuesta cuando se menciona WAF. No dediques tiempo a los SKUs ni a la configuración.

## Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas que las peticiones a `https://contoso.com/videos` vayan a un grupo de servidores y las peticiones a `https://contoso.com/images` vayan a otro. ¿Qué servicio de Azure lo permite?

- A) Azure Load Balancer
- B) Azure Application Gateway
- C) Network Security Group
- D) Azure VPN Gateway

**Respuesta correcta: B.** El enrutamiento basado en URL es una función de capa 7 exclusiva de Application Gateway (y Front Door).
- A es incorrecta: Load Balancer no lee la URL.
- C es incorrecta: NSG filtra por IP y puerto.
- D es incorrecta: VPN Gateway conecta redes.

**Pregunta 2.** ¿Qué característica de Azure Application Gateway protege una aplicación web frente a ataques como la inyección SQL y el cross-site scripting?

- A) Health probe
- B) Terminación SSL
- C) Web Application Firewall (WAF)
- D) Afinidad de sesión

**Respuesta correcta: C.** El WAF aplica reglas OWASP contra estos ataques.
- A comprueba si los servidores responden.
- B descifra el tráfico HTTPS.
- D mantiene al usuario en el mismo servidor.

**Pregunta 3.** ¿Cuál es la diferencia principal entre Azure Load Balancer y Azure Application Gateway?

- A) Load Balancer es global y Application Gateway es regional.
- B) Load Balancer opera en capa 4 y Application Gateway en capa 7.
- C) Load Balancer solo funciona con HTTP y Application Gateway con cualquier protocolo.
- D) No hay diferencia; son el mismo servicio con distinto nombre.

**Respuesta correcta: B.**
- A es incorrecta: ambos son regionales.
- C es incorrecta: es al revés.
- D es incorrecta: son servicios distintos.

## 🧠 Resumen para el examen

1. Application Gateway = balanceador **capa 7** para tráfico **HTTP/HTTPS**.
2. **Enruta por URL** y por **nombre de host** (multi-sitio).
3. Hace **terminación SSL**.
4. Incluye **WAF** (OWASP: inyección SQL, XSS).
5. Es **regional**; **Front Door** es su versión global.
6. Load Balancer = capa 4 y TCP/UDP; Application Gateway = capa 7 y solo web.
7. Peso bajo en AZ-900: reconócelo, no lo configures.
