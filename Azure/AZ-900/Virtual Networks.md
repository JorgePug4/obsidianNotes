## 1. ¿Qué es?

Una **Virtual Network (VNet)** es la representación en la nube de una red privada tradicional. Es el **contenedor de red fundamental** de Azure: dentro de ella viven las VMs, y a través de ella se conectan entre sí, con internet, y con redes on-premises.

Piensa en la VNet como el **datacenter privado que ya no tienes que cablear tú mismo**.

---

## 2. Conceptos fundamentales

- **VNet**: red aislada lógicamente, vive en **una sola región** y **una sola suscripción**.
- **Subred (subnet)**: subdivisión del espacio de direcciones de la VNet. Los recursos se despliegan dentro de subredes, nunca directo en la VNet.
- **NIC (Network Interface Card)**: la interfaz de red que conecta una VM a una subred. Una NIC = una subred (no puede estar en dos a la vez).
- **NSG (Network Security Group)**: firewall de capa 4 (stateful), filtra tráfico entrante/saliente.
- **UDR (User Defined Route) / Route Table**: tabla de rutas personalizada que sobrescribe el ruteo automático de Azure.
- **Peering**: conexión directa entre dos VNets (no pasa por internet).
- **Service Endpoint**: extiende la identidad de la VNet a un servicio PaaS (por IP pública, optimizado).
- **Private Endpoint**: le da una **IP privada** dentro de tu VNet a un servicio PaaS.
- **ASG (Application Security Group)**: agrupa NICs por rol lógico (web, db) para simplificar reglas de NSG.

---

## 3. ¿Cómo funciona?

**Flujo típico al crear una VNet:**

1. Defines un **espacio de direcciones** (ej. `10.0.0.0/16`) usando rangos privados RFC 1918.
2. Divides ese espacio en **subredes** (ej. `10.0.1.0/24` para web, `10.0.2.0/24` para db).
3. Despliegas recursos (VMs, App Service con integración VNet, etc.) dentro de esas subredes.
4. Aplicas **NSGs** a nivel subred y/o NIC para controlar el tráfico.
5. Si necesitas conectar con otra VNet → **Peering**.
6. Si necesitas conectar con on-premises → **VPN Gateway** o **ExpressRoute**.
7. Si necesitas salida controlada a internet → **NAT Gateway**.

**Evaluación de tráfico en NSG:**

- **Entrante**: NSG de la **subred** → luego NSG de la **NIC**.
- **Saliente**: NSG de la **NIC** → luego NSG de la **subred**.
- Ambos deben permitir el tráfico; si uno deniega, se bloquea.
- Se evalúa por **prioridad numérica** (menor número = mayor prioridad), de 100 a 4096.

---

## 4. Puntos importantes (memorizar)

- **5 IPs reservadas por subred**: red, gateway, 2 para DNS de Azure, broadcast. → Un `/24` da **251 IPs usables**, no 254.
- **Subred mínima**: `/29`. Recomendado `/27`+ en producción.
- **VNet Peering NO es transitivo**: A↔B y B↔C no implica A↔C.
- **Precedencia de rutas**: UDR > BGP > rutas del sistema. Gana el **prefijo más largo** (longest prefix match).
- **IP DNS interna de Azure**: `168.63.129.16` — nunca bloquearla en NSG.
- **Acceso saliente a internet ya NO es automático** en VNets nuevas → se necesita NAT Gateway, IP pública en la NIC, o Load Balancer con reglas outbound.
- **Subredes con nombre reservado**:
    - `GatewaySubnet` → mínimo `/29`, recomendado `/27`
    - `AzureBastionSubnet` → mínimo `/26`
    - `AzureFirewallSubnet` → `/26`
- **Standard Public IP** = _deny by default_ (sin NSG que permita, no entra nada), y solo puede ser **estática**.

---

## 5. Ejemplos prácticos

**Escenario 1 — Segmentación por capas:**

```
VNet: 10.0.0.0/16
 ├── Subred-Web:   10.0.1.0/24  (NSG: permite 80/443 desde Internet)
 ├── Subred-App:   10.0.2.0/24  (NSG: permite tráfico solo desde Subred-Web)
 └── Subred-DB:    10.0.3.0/24  (NSG: permite 1433 solo desde Subred-App)
```

Aquí cada capa solo habla con la que le corresponde, aislamiento por diseño.

**Escenario 2 — Hub and Spoke (arquitectura común en empresas):**

```
VNet-Hub (Firewall, VPN Gateway)
   ├── Peering → VNet-Spoke-Prod
   └── Peering → VNet-Spoke-Dev
```

Como el peering no es transitivo, Spoke-Prod y Spoke-Dev **no se ven entre sí** a menos que el tráfico pase forzado por el Firewall del Hub (usando UDR + `Allow forwarded traffic`).

**Escenario 3 — Conectar una base de datos PaaS de forma privada:** Una Azure SQL Database normalmente tiene IP pública. Con un **Private Endpoint**, la base de datos obtiene una IP privada (ej. `10.0.3.5`) dentro de tu VNet, y el tráfico nunca sale a internet.

---

## 6. Ventajas y desventajas

||Ventajas|Desventajas / limitaciones|
|---|---|---|
|**VNet en general**|Aislamiento lógico, control total de IP/ruteo, integración con on-premises|Vive en una sola región (no cruza regiones sin peering)|
|**NSG**|Gratis, simple, granular (subred/NIC)|Solo capa 4, sin inspección de contenido|
|**Peering**|Sin salto por internet, baja latencia, entre regiones/tenants|No transitivo, requiere config en ambos lados|
|**Service Endpoint**|Gratis, fácil de habilitar|Sigue usando IP pública del servicio, no accesible on-prem|
|**Private Endpoint**|IP privada real, accesible on-premises, tráfico nunca sale a internet|Tiene costo, requiere configurar Private DNS Zone|

---

## 7. Errores comunes

- **Confundir Service Endpoint con Private Endpoint**: Service Endpoint sigue usando la IP pública del servicio (solo "asegura" el camino); Private Endpoint le da una IP privada real. Si la pregunta/escenario menciona _on-premises_, la respuesta casi siempre es Private Endpoint.
- **Configurar peering solo de un lado**: queda en estado `Initiated`, no `Connected`. Hay que configurarlo en ambas VNets.
- **Olvidar el "Allow forwarded traffic"** en peering cuando el tráfico pasa por una NVA/Firewall del hub → el tráfico se descarta.
- **No habilitar IP Forwarding** en la NIC de una NVA que recibe tráfico vía UDR → el reenvío falla silenciosamente.
- **Calcular mal el tamaño de subred** olvidando las 5 IPs reservadas.
- **Asumir que un NSG basta como firewall completo**: NSG no hace inspección de capa 7, para eso se necesita Azure Firewall o un NVA.
- **Bloquear sin querer `168.63.129.16`** en un NSG, rompiendo health probes y comunicación del agente de VM.

---

## 8. Buenas prácticas

- Usa **NSG a nivel subred** como regla general; usa NIC solo para excepciones puntuales.
- Diseña con **Hub and Spoke** cuando tengas múltiples entornos/equipos, centralizando seguridad y conectividad en el hub.
- Usa **ASG** en vez de listas de IPs cuando agrupes recursos por rol (web, app, db).
- Prefiere **Private Endpoint** sobre Service Endpoint cuando necesites cumplimiento estricto o acceso on-premises.
- Reserva rangos de IP pensando en crecimiento futuro (no uses `/28` si vas a escalar).
- Usa **NAT Gateway** en vez de IP pública directa en VMs para salida a internet: evita agotamiento de puertos SNAT.
- Documenta el espacio de direcciones de todas tus VNets **antes** de crear peering, para evitar solapamientos.
- Usa **Network Watcher** (`IP Flow Verify`, `Next Hop`, `Effective Security Rules`) para depurar, no adivines.

---

## 9. Resumen para mi libreta

- VNet = red privada aislada, 1 región, 1 suscripción.
- Subred = división de la VNet; 5 IPs siempre reservadas.
- NSG = firewall L4 stateful, evalúa por prioridad (menor gana).
- Entrante: NSG subred → NIC. Saliente: NIC → subred.
- Peering = conexión directa entre VNets, **no transitiva**.
- UDR > BGP > rutas del sistema; gana prefijo más largo.
- Service Endpoint = IP pública optimizada. Private Endpoint = IP privada real (sirve on-prem).
- Salida a internet ya no es automática → usar NAT Gateway.
- IP especial de Azure: `168.63.129.16` (no bloquear).

---

## 10. Preguntas de entrevista

**1. ¿Qué diferencia hay entre un Service Endpoint y un Private Endpoint?**

> Un Service Endpoint extiende la identidad de la VNet hacia un servicio PaaS, pero el tráfico sigue usando la IP pública del servicio (aunque protegida). Un Private Endpoint, en cambio, asigna una IP privada dentro de la propia VNet al servicio PaaS, permitiendo que el tráfico nunca salga a internet y que sea accesible incluso desde on-premises vía VPN/ExpressRoute.

**2. ¿Por qué el VNet Peering no es transitivo y cómo se resuelve en una arquitectura real?**

> Porque cada peering es una relación punto a punto; Azure no propaga automáticamente la conectividad a una tercera VNet. En arquitecturas reales se resuelve con un diseño **Hub and Spoke**, donde todo el tráfico entre spokes pasa por una NVA o Azure Firewall en el hub, usando rutas definidas por el usuario (UDR) y habilitando "Allow forwarded traffic".

**3. Tienes dos VMs en la misma subred que no pueden comunicarse. ¿Qué revisarías primero?**

> Primero el NSG asociado a la subred y a las NICs (con Effective Security Rules en Network Watcher), luego confirmaría que la regla `AllowVnetInBound` no fue sobrescrita por una regla de mayor prioridad (número menor) que la deniegue, y finalmente revisaría si hay algún firewall a nivel de sistema operativo bloqueando el puerto.

**4. ¿Cómo asegurarías que una base de datos PaaS solo sea accesible desde tu red privada?**

> Configurando un Private Endpoint para el recurso, deshabilitando el acceso público, y creando una Private DNS Zone vinculada a la VNet para que el nombre del recurso resuelva a la IP privada en lugar de la pública.

**5. ¿Qué pasa si mando tráfico hacia una NVA (Network Virtual Appliance) usando una Route Table y no funciona?**

> Lo más probable es que falte habilitar **IP Forwarding** en la NIC de la NVA. Sin eso, la VM descarta el tráfico que no va dirigido a su propia IP, aunque la ruta esté bien definida.

**6. ¿Cuál es la diferencia entre NSG y Azure Firewall, y cuándo usarías cada uno?**

> NSG es un firewall de capa 4, stateful, gratuito, que filtra por IP/puerto/protocolo a nivel de subred o NIC — es la primera línea de defensa. Azure Firewall es un servicio PaaS completamente administrado con capacidades de capa 7 (filtrado por FQDN, inteligencia de amenazas, logging centralizado), ideal para inspeccionar y controlar tráfico saliente de toda una arquitectura Hub and Spoke.

---

## 🧠 Lo que debo recordar

1. Una VNet es regional y de una sola suscripción; las subredes viven dentro de ella.
2. Cada subred pierde 5 IPs por reservas de Azure (calcula siempre con eso en mente).
3. NSG es stateful y se evalúa por prioridad; el orden de evaluación cambia según sea entrante o saliente.
4. El Peering conecta VNets directamente pero **no es transitivo** — para eso se usa Hub and Spoke.
5. Precedencia de ruteo: UDR > BGP > rutas del sistema, y siempre gana el prefijo más específico.
6. Private Endpoint = IP privada real y accesible on-premises; Service Endpoint = optimización sobre IP pública.
7. La salida a internet ya no es automática por defecto: hay que configurarla explícitamente (idealmente con NAT Gateway).
8. `168.63.129.16` es la IP de infraestructura de Azure — jamás bloquearla.
9. IP Forwarding en la NIC es obligatorio para que una NVA pueda reenviar tráfico.
10. Network Watcher es la herramienta de diagnóstico oficial: úsala antes de adivinar.