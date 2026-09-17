---
tags: [az-900, azure, conceptos-nube, modelos-nube]
modulo: Conceptos de la nube
peso_examen: Alto
---

# Modelos de nube: pública, privada e híbrida

## Concepto

El **modelo de nube** describe **dónde vive la infraestructura y quién la posee**. Hay tres modelos principales (público, privado, híbrido) y un cuarto que el examen menciona cada vez más: **multinube**.

**Problema que resuelve:** no todas las organizaciones pueden o quieren mover todo a un proveedor externo. Regulaciones, inversiones ya hechas o requisitos de latencia obligan a combinar entornos. Los modelos de nube dan nombre a esas combinaciones.

**Para qué se utiliza:** para decidir la estrategia de adopción: qué cargas van a la nube pública, cuáles se quedan en casa y cómo se conectan entre sí.

## Características principales

| Modelo | Quién posee la infraestructura | Dónde vive | Ejemplo |
|---|---|---|---|
| **Nube pública** | Un proveedor externo (Microsoft) | Centros de datos del proveedor, compartidos entre clientes | Azure, Microsoft 365 |
| **Nube privada** | La propia organización (o un tercero en exclusiva) | Centro de datos propio o dedicado | Azure Stack en tu CPD, VMware local |
| **Nube híbrida** | Ambos | Parte local, parte en nube pública, **conectadas** | Base de datos local + web en Azure |
| **Multinube** | Varios proveedores públicos | Azure + AWS + Google Cloud | Gobernar todo con [[Azure Arc]] |

### Nube pública
- **Sin inversión inicial** en hardware; pago por uso.
- Escalado prácticamente ilimitado y rápido.
- Recursos compartidos con otros clientes (aislados lógicamente).
- Menor control sobre el hardware y la ubicación exacta.

### Nube privada
- **Control total** sobre hardware, seguridad y configuración.
- Cumple regulaciones que exigen que los datos no salgan de la organización.
- **Mayor coste**: compras y mantienes el hardware; el escalado es lento.
- Sigue siendo "nube" porque ofrece autoservicio y elasticidad interna.

### Nube híbrida
- Combina lo mejor de ambas: mantienes lo sensible en local y usas la pública para el resto o para picos.
- Requiere **conectividad** entre entornos: [[05 - Azure VPN Gateway]] o [[08 - Azure ExpressRoute]].
- Requiere **identidad unificada**: [[Microsoft Entra Connect]].
- Requiere **gestión unificada**: [[Azure Arc]].
- **Azure VMware Solution** permite mover cargas VMware a Azure sin reescribirlas; se usa como paso intermedio en migraciones híbridas.

### Multinube
- Usar dos o más proveedores públicos a la vez, por resiliencia, por evitar dependencia de un único proveedor o porque cada uno destaca en algo.
- El reto es la gestión: [[Azure Arc]] permite ver y gobernar recursos de AWS y GCP desde Azure.

## Casos de uso

- **Pública**: startup que quiere lanzar rápido sin capital; empresa que quiere dejar de administrar hardware.
- **Privada**: banco o administración pública con regulación que prohíbe datos en infraestructura compartida; organización con hardware ya amortizado.
- **Híbrida**: empresa que mantiene su ERP en local pero pone la web y el análisis en Azure; extensión de capacidad a la nube solo en picos (*cloud bursting*).
- **Multinube**: empresa que usa Azure para Microsoft 365 y otro proveedor para una carga heredada, y quiere gobernar ambos desde un solo panel.

## Comparaciones

| Necesidad | Modelo | No confundir con |
|---|---|---|
| Lanzar sin inversión inicial | Pública | Privada (requiere comprar hardware) |
| Control físico total y datos que no salen del edificio | Privada | Pública con Private Endpoint (sigue siendo infraestructura compartida) |
| Mantener parte local y parte en Azure, conectadas | Híbrida | Multinube (son varios proveedores públicos, no local + público) |
| Usar Azure y AWS a la vez | Multinube | Híbrida |
| Gestionar servidores locales desde el portal de Azure | Híbrida con [[Azure Arc]] | Azure Migrate (eso los mueve) |

## Conceptos que debo memorizar

> [!important]
> - **Pública** = proveedor externo, sin CapEx, escalado rápido, menos control.
> - **Privada** = infraestructura propia, control total, más coste, escalado lento.
> - **Híbrida** = pública + privada **conectadas**. Es el modelo más flexible y el más preguntado.
> - **Multinube** = varios proveedores públicos.
> - Herramientas híbridas de Azure: **Arc** (gestión), **VPN/ExpressRoute** (red), **Entra Connect** (identidad), **VMware Solution** (migración).

## Tips para AZ-900

> [!tip]
> - Palabras clave para híbrida: *"parte de la carga en local"*, *"mantener el datacenter existente"*, *"extender a la nube"*, *"cumplir regulación con algunos datos"*.
> - Palabras clave para privada: *"control total"*, *"hardware propio"*, *"nadie más comparte"*, *"requisito legal de ubicación"*.
> - Palabras clave para pública: *"sin inversión inicial"*, *"pagar solo por lo que uso"*, *"escalar sin límite"*.
> - Trampa: una nube privada puede estar **hospedada por un tercero** siempre que sea de uso exclusivo. Lo que la define es la exclusividad, no el edificio.
> - Trampa: "Nube híbrida = usar Azure y AWS". **No**, eso es multinube.

## Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa quiere mantener su base de datos de clientes en su propio centro de datos por motivos legales, pero alojar su aplicación web en Azure para escalar en campañas. ¿Qué modelo de nube describe este escenario?

- A) Nube pública
- B) Nube privada
- C) Nube híbrida
- D) Multinube

**Respuesta: C.** Combina infraestructura local (base de datos) con nube pública (web), conectadas entre sí.
- A) Dejaría la base de datos en Azure, incumpliendo el requisito legal.
- B) No usaría Azure para la web.
- D) Implica varios proveedores públicos, no local + Azure.

**Pregunta 2.** ¿Cuál de las siguientes es una característica de la nube privada frente a la pública?

- A) Menor inversión inicial
- B) Mayor control sobre el hardware y la seguridad
- C) Escalado más rápido
- D) Los recursos se comparten con otros clientes

**Respuesta: B.** El control total es la ventaja principal de la privada.
- A y C) Son ventajas de la pública.
- D) Es una característica de la pública; la privada es de uso exclusivo.

## 🧠 Resumen para el examen

1. Tres modelos: pública (proveedor), privada (propia), híbrida (ambas conectadas). Cuarto: multinube (varios proveedores).
2. Pública = barata de empezar y rápida de escalar; privada = control total pero cara y lenta.
3. Híbrida = flexibilidad; exige conectar red, identidad y gestión.
4. Arc gestiona lo híbrido y multinube; VPN/ExpressRoute lo conectan; Entra Connect unifica identidades.
5. Azure VMware Solution = mover VMware a Azure sin reescribir.

---
