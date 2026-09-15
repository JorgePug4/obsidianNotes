---
tags: [AZ-900, Azure, Seguridad, Redes]
up: "[[00 - Índice - Seguridad]]"
---

# Service Endpoints y Private Endpoints

> [!info] Relevancia: **baja** para AZ-900. No es un objetivo explícito, pero ayuda a responder preguntas sobre "acceder a un servicio PaaS sin salir a Internet".

## 1. Concepto

Los servicios PaaS de Azure (Storage, SQL Database, Key Vault...) tienen por defecto una **IP pública**. Estas dos opciones permiten que tus recursos en una **VNet** lleguen a ellos de forma más segura:

- **Service Endpoint** (punto de conexión de servicio): extiende la identidad de tu VNet al servicio. El tráfico viaja por la **red troncal de Microsoft** (no por Internet), pero el servicio **sigue teniendo IP pública**.
- **Private Endpoint** (punto de conexión privado, parte de **Azure Private Link**): crea una **tarjeta de red con IP privada de tu VNet** que apunta al servicio. El servicio pasa a ser accesible como si estuviera dentro de tu red.

- **Problema que resuelve**: exponer servicios PaaS a Internet.
- **Para qué se usa**: reducir la superficie de ataque y cumplir requisitos de red privada.

## 2. Características principales

- Service Endpoint se activa **a nivel de subred** y luego se restringe el firewall del servicio a esa subred.
- Private Endpoint asigna una **IP privada** al servicio; sirve incluso desde **on-premises** vía VPN/ExpressRoute.
- Private Endpoint es la opción **más segura** y la que Microsoft recomienda hoy.

## 3. Casos de uso

- Una VM necesita leer de una Storage Account sin que esa cuenta acepte tráfico de Internet → Private Endpoint.
- Quieres que solo tu subred de aplicaciones pueda hablar con Azure SQL, con configuración rápida → Service Endpoint.

## 4. Comparaciones

| | Service Endpoint | Private Endpoint (Private Link) |
|---|---|---|
| IP del servicio | Sigue siendo **pública** | Recibe una **IP privada** de tu VNet |
| Ruta del tráfico | Red troncal de Microsoft | Red troncal de Microsoft |
| Acceso desde on-premises | No | Sí (por VPN/ExpressRoute) |
| Granularidad | Toda la subred | Un recurso concreto |
| Coste | Gratis | De pago |
| Cuándo usar | Sencillez, solo desde Azure | Máxima seguridad, acceso híbrido |

## 5. Conceptos que debo memorizar

> [!important]
> - **Private Endpoint = IP privada** en tu VNet. **Service Endpoint = el servicio mantiene IP pública**.
> - Private Endpoint forma parte de **Azure Private Link**.
> - Ambos evitan que el tráfico salga a Internet público.

## 6. Tips para AZ-900

> [!tip]
> - Palabra clave **"IP privada"** o **"desde la red local (on-premises)"** → Private Endpoint.
> - Palabra clave **"sin salir de la red troncal de Microsoft, configuración a nivel de subred"** → Service Endpoint.
> - No confundir con **NSG** (filtra tráfico por reglas) ni con **Azure Firewall** (firewall gestionado). Los endpoints cambian *cómo se llega* al servicio, no *qué se permite*.

## 7. Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas que una Storage Account solo sea accesible mediante una dirección IP privada de tu red virtual. ¿Qué debes configurar?

- A) Service Endpoint
- B) Network Security Group
- C) Private Endpoint ✅
- D) Azure Bastion

*Correcta: C.* Solo el Private Endpoint da una IP privada al servicio. A mantiene la IP pública; B filtra tráfico pero no cambia la dirección; D sirve para conectarse por RDP/SSH a VMs, no a Storage.

**Pregunta 2.** ¿Qué afirmación sobre los Service Endpoints es correcta?

- A) Asignan una IP privada al servicio PaaS
- B) Permiten acceso desde redes on-premises
- C) Enrutan el tráfico por la red troncal de Microsoft ✅
- D) Sustituyen a los NSG

*Correcta: C.* A y B describen al Private Endpoint; D es falsa, son controles distintos.

## 🧠 Resumen para el examen

1. Ambos evitan que el tráfico hacia PaaS salga a Internet.
2. Private Endpoint = IP privada, parte de Private Link, funciona desde on-premises.
3. Service Endpoint = subred completa, el servicio conserva IP pública, gratis.
4. Private Endpoint es la opción recomendada por seguridad.
5. Tema de relevancia baja: reconócelo como opción de respuesta, no profundices.

---
⬅️ [[00 - Índice - Seguridad|Volver al índice de Seguridad]]
