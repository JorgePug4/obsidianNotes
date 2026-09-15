# Azure Arc

## Concepto

**Azure Arc** extiende la **gestión de Azure** a recursos que **no están en Azure**: servidores en tu datacenter, clústeres de Kubernetes en otra nube, bases de datos en AWS o Google Cloud. Los proyectas en Azure como si fueran recursos nativos y los gestionas con las mismas herramientas.

Problema que resuelve: una empresa con servidores on-premises, VMs en AWS y recursos en Azure tiene tres formas distintas de gestionar, aplicar políticas y monitorizar. Arc unifica todo en un solo plano de control: el portal de Azure.

Para qué se usa: escenarios **híbridos y multinube**. Aplicar Azure Policy, RBAC, etiquetas, Azure Monitor y Defender for Cloud a recursos que viven fuera de Azure.

## Características principales

- Los recursos externos aparecen en el portal como **recursos de Azure Arc** con un ID de recurso, dentro de un grupo de recursos.
- Se instala un **agente** en el servidor o clúster para conectarlo.
- Recursos que puede gestionar:
  - **Servidores** (Windows y Linux, físicos o virtuales, en cualquier sitio).
  - **Clústeres de Kubernetes**.
  - **Servicios de datos** (SQL Managed Instance, PostgreSQL) desplegados fuera de Azure.
  - Hosts de virtualización (VMware, SCVMM).
- Una vez conectado, puedes aplicar:
  - **Azure Policy** y **RBAC**.
  - **Etiquetas**.
  - **Azure Monitor** y **Log Analytics**.
  - **Microsoft Defender for Cloud**.
  - **Update Manager**.
- Arc **no mueve** los recursos a Azure. Siguen ejecutándose donde están; solo se gestionan desde Azure.

> [!warning] Arc no es migración
> Azure Arc **gestiona** recursos fuera de Azure. **Azure Migrate** los **traslada** a Azure. Si la pregunta dice "mover a Azure", es Migrate. Si dice "gestionar desde Azure sin mover", es Arc.

> [!warning] Arc vs VPN / ExpressRoute
> Arc no conecta redes. Conectar tu datacenter con Azure a nivel de red es tarea de **VPN Gateway** o **ExpressRoute**. Arc conecta la **gestión**, no el tráfico.

## Casos de uso

- 300 servidores Windows on-premises: conectarlos con Arc para aplicar la misma Azure Policy que a las VMs de Azure.
- Clúster de Kubernetes en AWS: verlo y gobernarlo desde el portal de Azure.
- Auditoría de seguridad unificada: Defender for Cloud evaluando servidores de Azure, on-premises y Google Cloud a la vez.
- Inventario único de todos los servidores de la empresa, estén donde estén.

## Comparaciones

| Necesidad | Servicio |
|---|---|
| Gestionar servidores on-premises desde Azure sin moverlos | **Azure Arc** |
| Migrar servidores on-premises a Azure | Azure Migrate |
| Conectar la red on-premises con Azure | VPN Gateway / ExpressRoute |
| Ejecutar servicios de Azure en tu propio hardware | Azure Stack (HCI / Hub) |
| Aplicar Azure Policy a una VM de AWS | **Azure Arc** |

## Conceptos que debo memorizar

> [!important]
> - Arc = **gestión híbrida y multinube** desde Azure.
> - Gestiona **servidores, Kubernetes y servicios de datos** fuera de Azure.
> - Los recursos **no se mueven**; se proyectan en Azure como recursos Arc.
> - Permite aplicar **Policy, RBAC, etiquetas, Monitor y Defender** a recursos externos.
> - Requiere un **agente**.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *híbrido, multinube, on-premises, AWS, Google Cloud, gestionar desde Azure, plano de control único, sin migrar*.
> - "Aplicar Azure Policy a servidores fuera de Azure" → **Arc**.
> - "Ver servidores de AWS en el portal de Azure" → **Arc**.
> - Trampa: "Azure Arc migra servidores a Azure". **Falso**. Eso es Azure Migrate.
> - Trampa: "Azure Arc solo funciona con Windows". **Falso**. Windows y Linux.
> - Trampa: "Azure Arc conecta redes on-premises con Azure". **Falso**. Eso es VPN / ExpressRoute.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu empresa tiene servidores Linux en su datacenter y máquinas virtuales en AWS. Quieres aplicar las mismas políticas de Azure Policy y monitorizarlos con Azure Monitor sin trasladarlos a Azure. ¿Qué servicio debes usar?

- A) Azure Migrate
- B) Azure Arc
- C) Azure ExpressRoute
- D) Azure Blueprints

**Respuesta: B.** Arc conecta recursos externos al plano de gestión de Azure.
- A) Migrate los movería a Azure, y no quieres moverlos.
- C) ExpressRoute conecta redes, no aplica políticas.
- D) Blueprints desplegaba entornos dentro de Azure y está en retirada.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre Azure Arc es correcta?

- A) Traslada las máquinas virtuales on-premises a Azure.
- B) Permite gestionar clústeres de Kubernetes que se ejecutan fuera de Azure desde el portal de Azure.
- C) Solo admite recursos ubicados en regiones de Azure.
- D) Sustituye a Azure Monitor.

**Respuesta: B.**
- A) Arc no migra.
- C) Su propósito es precisamente gestionar recursos **fuera** de Azure.
- D) Arc se integra con Monitor, no lo sustituye.

**Pregunta 3.** Un administrador quiere ver en un único inventario todos los servidores de la empresa: los de Azure, los del datacenter y los de Google Cloud. ¿Qué servicio lo permite?

- A) Azure Service Health
- B) Azure Arc
- C) Azure Advisor
- D) Log Analytics

**Respuesta: B.** Arc registra servidores externos como recursos de Azure, de modo que aparecen en el inventario junto a los nativos.
- A) Service Health informa del estado de Azure.
- C) Advisor recomienda mejoras.
- D) Log Analytics consulta registros; podría almacenar datos de esos servidores, pero no es la herramienta de inventario y gestión.

## 🧠 Resumen para el examen

1. Arc = extender la gestión de Azure a recursos on-premises y multinube.
2. Gestiona servidores (Windows/Linux), Kubernetes y servicios de datos.
3. No migra ni conecta redes; solo gestiona.
4. Los recursos aparecen en el portal como recursos Arc.
5. Permite Policy, RBAC, etiquetas, Monitor y Defender sobre recursos externos.
6. Necesita un agente instalado en el recurso.
7. Arc (gestionar) ≠ Migrate (mover) ≠ ExpressRoute (conectar red).

---
