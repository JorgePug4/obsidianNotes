---
tags: [az-900, azure, redes, cdn]
modulo: Redes
peso_examen: Bajo
---

# Azure Content Delivery Network (CDN)

## Concepto

**Azure CDN** es una red distribuida de servidores (**puntos de presencia** o **POP**, también llamados **edge servers**) repartidos por todo el mundo que guardan una **copia en caché** de tu contenido estático y lo sirven desde el punto más cercano al usuario.

**Problema que resuelve:** si tu web está en West Europe y un usuario en Sídney pide una imagen, la petición cruza medio planeta. Con CDN, la imagen ya está cacheada en un servidor de Sídney y llega en milisegundos.

**Para qué se utiliza:**
- Acelerar la entrega de contenido estático: imágenes, vídeos, CSS, JavaScript, descargas de software.
- Reducir la carga en el servidor de origen (Storage, App Service, VM).
- Absorber picos de tráfico (lanzamientos, eventos).

## Características principales

- **Caché en el edge:** el primer usuario de una zona provoca que el POP pida el contenido al origen y lo guarde. Los siguientes usuarios lo reciben desde el POP.
- **Origen:** puede ser Azure Storage (Blob), App Service, Cloud Services, una VM o cualquier servidor web público.
- **TTL (tiempo de vida):** cuánto tiempo mantiene el POP la copia antes de volver a pedirla al origen.
- **Perfil de CDN:** el recurso contenedor; dentro creas **endpoints** (cada uno apunta a un origen).
- **Contenido estático** es el objetivo natural. Para contenido dinámico existe la aceleración de sitio dinámico, fuera del alcance de AZ-900.
- **Compresión, HTTPS con dominio personalizado, reglas de caché** configurables.
- **Global por diseño:** al contrario que la mayoría de servicios de red, no está atado a una región.

> [!warning] Confusiones frecuentes
> - **CDN vs. Front Door:** los dos son globales y usan la red edge de Microsoft. CDN se centra en **cachear contenido estático**. Front Door es un **balanceador global de capa 7** con WAF y enrutamiento de aplicaciones dinámicas. Microsoft ha ido integrando la CDN clásica dentro de Front Door; en el examen, "caché de contenido estático global" sigue apuntando a CDN.
> - **CDN vs. Load Balancer:** CDN no reparte tráfico entre tus servidores; sirve copias desde sus propios servidores edge.
> - **CDN vs. replicación de Storage:** la replicación (GRS, etc.) protege los datos ante desastres; la CDN acelera la entrega a usuarios. Son cosas distintas.

## Casos de uso

- Una empresa de medios publica vídeos que ven usuarios en todos los continentes; la CDN los cachea cerca de cada audiencia.
- Un fabricante de software aloja instaladores en Blob Storage y usa CDN para que las descargas sean rápidas en cualquier país.
- Una web de venta de entradas espera un pico de tráfico masivo el día del lanzamiento; la CDN sirve las páginas estáticas y protege el origen.

## Comparaciones

| | **Azure CDN** | **Azure Front Door** | **Load Balancer** |
|---|---|---|---|
| Función principal | Caché de contenido estático | Balanceo global capa 7 + aceleración | Balanceo regional capa 4 |
| Alcance | Global (POPs) | Global (POPs) | Regional |
| Contenido dinámico | Limitado | Sí | N/A |
| WAF | No (en la CDN clásica) | Sí | No |
| Palabra clave | "caché", "estático", "cerca del usuario" | "aplicación global", "WAF global" | "TCP/UDP entre VMs" |

## Conceptos que debo memorizar

> [!important]
> - CDN = **caché** de contenido en **puntos de presencia (POP)** cercanos al usuario.
> - Reduce **latencia** y **carga en el origen**.
> - Ideal para **contenido estático**: imágenes, vídeo, CSS, JS, descargas.
> - Es un servicio **global**.
> - El **TTL** define cuánto dura la copia en caché.

## Tips para AZ-900

> [!tip] Palabras clave
> "usuarios en todo el mundo", "reducir latencia", "contenido estático", "caché", "puntos de presencia", "imágenes y vídeos".

> [!tip] Preguntas trampa
> - "Distribuir tráfico entre VMs" → **no** es CDN, es Load Balancer.
> - "Mantener una copia de los datos en otra región por si hay desastre" → **no** es CDN, es replicación de Storage.
> - "Acelerar la entrega de imágenes a usuarios de Asia desde un origen en Europa" → **CDN**.
> - La CDN **no** aloja tu aplicación; siempre necesita un **origen**.

> [!tip] Peso en el examen
> CDN no figura en el temario AZ-900 vigente como objetivo propio. Puede aparecer como distractor o en una pregunta de "elige el servicio para reducir la latencia global". Con el concepto basta.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu sitio web se aloja en West Europe y los usuarios de Australia se quejan de que las imágenes cargan despacio. ¿Qué servicio de Azure reduce la latencia con el menor cambio en la arquitectura?

- A) Azure Load Balancer
- B) Azure Content Delivery Network
- C) Azure ExpressRoute
- D) Network Security Group

**Respuesta correcta: B.** La CDN cachea las imágenes en POPs cercanos a Australia.
- A es incorrecta: Load Balancer reparte tráfico dentro de una región.
- C es incorrecta: ExpressRoute conecta redes corporativas con Azure.
- D es incorrecta: NSG filtra tráfico.

**Pregunta 2.** ¿Qué tipo de contenido se beneficia más de Azure CDN?

- A) Consultas a una base de datos transaccional
- B) Contenido estático como imágenes, vídeos y archivos CSS
- C) Tráfico de una VPN Site-to-Site
- D) Conexiones RDP a máquinas virtuales

**Respuesta correcta: B.** El contenido estático se cachea sin cambios entre usuarios.
- A cambia en cada consulta, no se cachea bien.
- C y D no son contenido web servible desde caché.

**Pregunta 3.** ¿Qué es un punto de presencia (POP) en el contexto de Azure CDN?

- A) Una región de Azure donde se despliega la VNet
- B) Un servidor perimetral que almacena en caché el contenido cerca de los usuarios
- C) Una subred especial para el gateway
- D) Una regla de enrutamiento del Application Gateway

**Respuesta correcta: B.**
- A confunde POP con región.
- C describe la GatewaySubnet.
- D describe una función de Application Gateway.

## 🧠 Resumen para el examen

1. CDN = **caché global** de contenido en **puntos de presencia**.
2. Objetivo: **menor latencia** para usuarios lejanos y **menos carga** en el origen.
3. Ideal para **contenido estático** (imágenes, vídeo, CSS, JS, descargas).
4. Necesita un **origen** (Blob Storage, App Service, VM, servidor externo).
5. **TTL** controla cuánto vive el contenido en caché.
6. No balancea entre tus VMs ni replica datos: solo **entrega más rápido**.
7. **Front Door** es la evolución global con balanceo y WAF; CDN se reconoce por "caché".
