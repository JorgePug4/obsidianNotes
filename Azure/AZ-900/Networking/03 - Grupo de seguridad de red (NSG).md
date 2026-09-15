---
tags: [az-900, azure, redes, seguridad, nsg]
modulo: Redes
peso_examen: Medio
---

# Grupo de seguridad de red (NSG)

## Concepto

Un **Network Security Group (NSG)** es un firewall básico que filtra el tráfico de red hacia y desde los recursos de una [[01 - Azure Virtual Network]]. Funciona como un portero con una lista: cada regla dice "permitir" o "denegar" según origen, destino, puerto y protocolo.

**Problema que resuelve:** sin filtrado, cualquier recurso de la VNet podría recibir tráfico en cualquier puerto. El NSG restringe qué conexiones son válidas.

**Para qué se utiliza:**
- Permitir solo el puerto 443 a los servidores web.
- Bloquear el acceso RDP (3389) desde Internet.
- Impedir que la subred web hable con la subred de base de datos salvo por el puerto necesario.

## Características principales

- **Reglas de entrada (inbound) y de salida (outbound)** separadas.
- Cada regla define: **nombre, prioridad, origen, destino, puerto, protocolo (TCP/UDP/Any), acción (Allow/Deny)**.
- **Prioridad:** número de 100 a 4096. **Menor número = mayor prioridad**. Azure evalúa las reglas en orden y se detiene en la primera que coincide.
- **Reglas predeterminadas** que no se pueden borrar (pero sí sobrescribir con reglas de mayor prioridad):
  - Entrada: permite tráfico dentro de la VNet, permite tráfico del balanceador de Azure, **deniega todo lo demás**.
  - Salida: permite tráfico dentro de la VNet, permite salida a Internet, deniega todo lo demás.
- **Se asocia a una subred o a una interfaz de red (NIC)**. Nunca a la VNet completa.
- Un NSG se puede reutilizar en varias subredes o NICs.
- Trabaja en **capa 3-4** (IPs y puertos), no entiende HTTP ni URLs.
- Es **stateful**: si permites una conexión entrante, la respuesta sale sin necesitar una regla de salida.

> [!warning] Confusiones frecuentes
> - **NSG vs. Azure Firewall:** NSG es filtrado básico por IP/puerto, gratuito y por subred/NIC. Azure Firewall es un servicio administrado, con inspección de amenazas, filtrado por nombre de dominio (FQDN) y reglas centralizadas para toda la VNet. En el examen: "filtrado básico" → NSG; "inteligencia de amenazas", "FQDN", "centralizado" → Azure Firewall.
> - **NSG vs. Application Security Group (ASG):** el ASG agrupa VMs por función (por ejemplo "servidores web") para referenciarlas en las reglas del NSG en vez de usar IPs. No sustituye al NSG.
> - **Prioridad:** un error clásico es pensar que 4000 tiene más prioridad que 100. Es al revés.

## Casos de uso

- Servidor web: regla que permite entrada TCP 80 y 443 desde cualquier origen; todo lo demás queda denegado por la regla predeterminada.
- Servidor de base de datos: regla que solo permite el puerto 1433 desde la subred de aplicación.
- Administración: regla que permite RDP 3389 solo desde la IP pública de la oficina.

## Comparaciones

| | **NSG** | **Azure Firewall** |
|---|---|---|
| Nivel | Capa 3-4 (IP, puerto) | Capa 3-7 (incluye FQDN, aplicaciones) |
| Alcance | Subred o NIC | Toda la VNet, hub central |
| Coste | Gratuito | De pago (servicio administrado) |
| Inteligencia de amenazas | No | Sí |
| Cuándo usar | Filtrado básico entre subredes | Perímetro de seguridad centralizado |

## Conceptos que debo memorizar

> [!important]
> - NSG = reglas **Allow/Deny** por **origen, destino, puerto, protocolo**.
> - Se asocia a **subred** o **NIC**.
> - Prioridad **100-4096**, **menor número gana**.
> - Reglas por defecto: **permite dentro de la VNet, deniega todo lo demás desde fuera**.
> - Es **stateful**.
> - No inspecciona contenido HTTP; para eso está Application Gateway con WAF o Azure Firewall.

## Tips para AZ-900

> [!tip] Palabras clave
> "filtrar tráfico", "permitir o denegar", "puerto", "reglas de entrada y salida", "subred o interfaz de red".

> [!tip] Preguntas trampa
> - "¿Puede asociarse un NSG a una VNet?" → **No**. Solo a subred o NIC.
> - "¿Cuál regla se aplica si dos coinciden?" → la de **menor número de prioridad**.
> - "¿Bloquea el NSG el tráfico entre VMs de la misma VNet por defecto?" → **No**, la regla predeterminada lo permite.
> - Si la pregunta menciona **"filtrar por URL"** o **"proteger contra inyección SQL"** la respuesta no es NSG, es **Web Application Firewall (WAF)** en Application Gateway.

## Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas permitir el tráfico HTTPS entrante a un grupo de máquinas virtuales y bloquear el resto de puertos. ¿Qué servicio de Azure utilizas?

- A) Azure Load Balancer
- B) Network Security Group
- C) Azure Content Delivery Network
- D) Azure VPN Gateway

**Respuesta correcta: B.** El NSG filtra tráfico por puerto y protocolo con reglas Allow/Deny.
- A es incorrecta: el balanceador reparte tráfico, no filtra puertos.
- C es incorrecta: CDN cachea contenido.
- D es incorrecta: VPN Gateway crea túneles cifrados con on-premises.

**Pregunta 2.** Un NSG tiene dos reglas de entrada: la regla A (prioridad 200) deniega el puerto 80 y la regla B (prioridad 300) permite el puerto 80. ¿Qué ocurre con el tráfico al puerto 80?

- A) Se permite, porque la regla B se evalúa primero.
- B) Se deniega, porque la regla A tiene mayor prioridad.
- C) Se permite, porque las reglas Allow siempre ganan.
- D) Se produce un error de configuración.

**Respuesta correcta: B.** Menor número = mayor prioridad. La regla 200 se evalúa antes y deniega.
- A es incorrecta: 300 se evalúa después de 200.
- C es incorrecta: no existe preferencia por Allow; manda la prioridad.
- D es incorrecta: tener reglas contradictorias es válido.

**Pregunta 3.** ¿A cuáles de estos elementos puede asociarse un Network Security Group? (elige la correcta)

- A) Solo a una máquina virtual
- B) A una subred o a una interfaz de red
- C) A una Virtual Network completa
- D) A una suscripción

**Respuesta correcta: B.**
- A es incorrecta: se asocia a la NIC de la VM, y también a subredes.
- C y D son incorrectas: el NSG no opera a esos niveles.

## 🧠 Resumen para el examen

1. NSG = **firewall básico** con reglas Allow/Deny.
2. Filtra por **origen, destino, puerto y protocolo** (capa 3-4).
3. Se asocia a **subred** o **NIC**, nunca a la VNet.
4. Prioridad **100-4096**; **menor número gana**.
5. Reglas predeterminadas: permiten tráfico interno de la VNet y deniegan lo demás desde fuera.
6. **Stateful**: la respuesta a una conexión permitida sale sola.
7. Para inspección avanzada (FQDN, amenazas) → **Azure Firewall**; para HTTP/URL → **WAF**.
