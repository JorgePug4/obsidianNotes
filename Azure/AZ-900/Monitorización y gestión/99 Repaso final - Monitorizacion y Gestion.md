# 🎯 Repaso final de Monitorización y Gestión

## Los conceptos más importantes

1. **Jerarquía de Azure**: Grupo de administración → Suscripción → Grupo de recursos → Recurso. Todo se hereda hacia abajo.
2. **Azure Policy** controla **qué** se puede crear y cómo. Efectos clave: Deny y Audit. Las iniciativas agrupan políticas.
3. **RBAC** controla **quién** puede hacer qué. Asignación = entidad de seguridad + rol + ámbito. Colaborador no asigna roles.
4. **Locks** protegen contra accidentes: CanNotDelete y ReadOnly. Aplican incluso a Propietarios.
5. **Azure Monitor** recopila métricas y registros. Incluye Log Analytics (KQL), Application Insights (apps) y Alertas (grupos de acciones).
6. **Service Health** informa de incidentes y mantenimientos de **Azure** que te afectan. Azure Status es global; Resource Health es por recurso.
7. **Advisor** da recomendaciones en 5 categorías: confiabilidad, seguridad, rendimiento, coste, excelencia operativa.
8. **Azure Arc** gestiona desde Azure recursos que están fuera de Azure, sin moverlos.
9. **CAF** es una guía de adopción, no un servicio. **Blueprints** está en retirada y ya no entra en el examen.
10. **Microsoft Purview** gobierna **datos** (catálogo, clasificación), no recursos.

## Tabla de servicios y su propósito

| Servicio | Propósito en una frase | Pregunta que responde |
|---|---|---|
| Grupos de administración | Gobernar varias suscripciones a la vez | ¿Cómo aplico algo a 20 suscripciones? |
| Azure Policy | Reglas de configuración y cumplimiento | ¿Está permitido crear esto así? |
| Azure RBAC | Permisos de identidades sobre recursos | ¿Quién puede hacer qué? |
| Bloqueos (Locks) | Evitar borrado o cambio accidental | ¿Cómo protejo esto de errores? |
| Etiquetas (Tags) | Organizar y atribuir coste | ¿De quién es y a qué proyecto pertenece? |
| Microsoft Purview | Gobernanza de datos | ¿Dónde están mis datos sensibles? |
| Azure Monitor | Métricas, logs y alertas de tus recursos | ¿Cómo está mi VM / mi app? |
| Log Analytics | Consultar registros con KQL | ¿Qué pasó y cuándo? |
| Application Insights | Monitorizar aplicaciones (APM) | ¿Por qué mi app va lenta o falla? |
| Alertas + Grupos de acciones | Notificar o actuar ante una condición | ¿Cómo me avisan si algo pasa? |
| Azure Service Health | Estado de Azure aplicado a tus suscripciones | ¿Azure tiene un problema que me afecte? |
| Azure Status | Estado global público de Azure | ¿Azure está caído en algún sitio? |
| Resource Health | Estado de un recurso concreto | ¿Esta VM está disponible? |
| Azure Advisor | Recomendaciones personalizadas | ¿Cómo mejoro coste, seguridad, rendimiento? |
| Azure Arc | Gestión híbrida y multinube desde Azure | ¿Cómo gobierno servidores fuera de Azure? |
| Cloud Adoption Framework | Guía para adoptar la nube | ¿Cómo planificamos la migración? |
| Azure Blueprints (retirada) | Desplegar entornos con RBAC y Policy | (Ya no entra en el examen) |

## Las diferencias que más fácilmente puedo confundir

| Confusión | Cómo distinguirlas |
|---|---|
| **Policy vs RBAC** | Policy = recursos y configuración ("solo en Europa"). RBAC = personas y permisos ("Ana es Lectora"). |
| **Lock vs RBAC** | Si un **Propietario** no puede borrar, es un **lock**. RBAC nunca bloquea a un Propietario. |
| **CanNotDelete vs ReadOnly** | CanNotDelete permite modificar. ReadOnly congela todo. |
| **Propietario vs Colaborador** | Colaborador hace todo **menos asignar roles**. |
| **Monitor vs Service Health** | Monitor = **tus** recursos (CPU alta). Service Health = **Azure** (incidente en la región). |
| **Monitor vs Advisor** | Monitor **mide**. Advisor **recomienda**. |
| **Service Health vs Azure Status** | Service Health = personalizado a tus suscripciones. Azure Status = global y público. |
| **Log Analytics vs Application Insights** | Log Analytics = consultar logs de cualquier origen con KQL. Application Insights = telemetría de aplicaciones. |
| **Métricas vs Registros** | Métricas = números casi en tiempo real. Registros = eventos detallados para investigar. |
| **Arc vs Migrate** | Arc **gestiona** sin mover. Migrate **traslada** a Azure. |
| **Arc vs ExpressRoute/VPN** | Arc conecta la **gestión**. VPN/ExpressRoute conectan la **red**. |
| **CAF vs un servicio** | CAF es **documentación y metodología**. Nunca es "el servicio que..." |
| **Purview vs Policy** | Purview = gobernanza de **datos**. Policy = gobernanza de **recursos**. |
| **Advisor vs Cost Management** | Advisor recomienda ahorrar. Cost Management muestra y analiza el gasto. |
| **Iniciativa vs Definición** | Definición = una regla. Iniciativa = varias reglas agrupadas. |

## 10 tips de examen

> [!tip]
> 1. Lee el escenario buscando el **sujeto**: si es una persona, piensa RBAC; si es un recurso, piensa Policy; si es un accidente, piensa Lock.
> 2. "**Varias suscripciones**" casi siempre apunta a **grupos de administración**.
> 3. "**No conforme**" (non-compliant) siempre es **Azure Policy**.
> 4. "**Privilegio mínimo**" te obliga a elegir el rol **más restrictivo** que cumpla la tarea.
> 5. Si un **Propietario no puede borrar**, la respuesta es **lock**, no falta de permisos.
> 6. "**Interrupción / mantenimiento planificado / incidente de Azure**" es **Service Health**.
> 7. "**CPU, memoria, umbral, alerta, SMS**" es **Azure Monitor**.
> 8. "**Recomendación / reducir coste / mejores prácticas**" es **Advisor**.
> 9. "**On-premises / AWS / multinube / sin mover**" es **Azure Arc**.
> 10. Descarta **Blueprints** como respuesta en exámenes actuales; ya no está en el outline y está en retirada.

## 10 preguntas de repaso tipo AZ-900

**1.** Necesitas asegurarte de que ningún usuario pueda crear máquinas virtuales fuera de la región `westeurope`, aunque tenga el rol Propietario. ¿Qué debes usar?

- A) Azure RBAC
- B) Azure Policy
- C) Un bloqueo ReadOnly
- D) Azure Advisor

**Respuesta: B.** Policy restringe regiones sin importar el rol del usuario. RBAC no controla propiedades del recurso; el lock protege recursos existentes; Advisor solo recomienda.

---

**2.** Un usuario debe poder gestionar todos los recursos de una suscripción, pero no debe poder conceder acceso a otros usuarios. ¿Qué rol le asignas?

- A) Propietario
- B) Colaborador
- C) Lector
- D) Administrador de acceso de usuario

**Respuesta: B.** Colaborador gestiona recursos sin gestionar acceso. Propietario también podría conceder acceso (viola privilegio mínimo); Lector no gestiona; Administrador de acceso de usuario solo gestiona permisos.

---

**3.** Aplicas un bloqueo ReadOnly a un grupo de recursos que contiene una máquina virtual. ¿Qué puede hacer un usuario con rol Propietario sobre esa VM?

- A) Borrarla
- B) Cambiar su tamaño
- C) Verla
- D) Añadirle una etiqueta

**Respuesta: C.** ReadOnly solo permite leer. Borrar, redimensionar y etiquetar son operaciones de escritura o eliminación bloqueadas.

---

**4.** Tu aplicación web devuelve errores y quieres ver las excepciones del código y los tiempos de respuesta de cada página. ¿Qué debes usar?

- A) Azure Service Health
- B) Application Insights
- C) Azure Advisor
- D) Resource Health

**Respuesta: B.** Application Insights es la herramienta de monitorización de aplicaciones. Service Health y Resource Health tratan del estado de Azure y de recursos, no del código; Advisor recomienda mejoras.

---

**5.** Quieres recibir un correo electrónico cuando Microsoft programe un mantenimiento que afecte a las máquinas virtuales de tu suscripción. ¿Qué debes configurar?

- A) Una alerta de métricas en Azure Monitor
- B) Una alerta de Azure Service Health
- C) Una recomendación de Azure Advisor
- D) Una política de Azure

**Respuesta: B.** Las alertas de Service Health notifican mantenimientos e incidentes de la plataforma. Las alertas de métricas vigilan tus recursos; Advisor y Policy no notifican mantenimientos.

---

**6.** ¿Qué herramienta consulta los registros almacenados en un área de trabajo mediante el lenguaje KQL?

- A) Azure Advisor
- B) Log Analytics
- C) Azure Policy
- D) Azure Status

**Respuesta: B.** Log Analytics usa KQL sobre el área de trabajo. Las otras opciones no consultan registros.

---

**7.** Tu empresa tiene 500 servidores en su datacenter. Quieres aplicarles Azure Policy y verlos en el portal de Azure sin migrarlos. ¿Qué servicio usas?

- A) Azure Migrate
- B) Azure Arc
- C) Azure ExpressRoute
- D) Azure Blueprints

**Respuesta: B.** Arc proyecta los servidores en Azure para gestionarlos. Migrate los movería; ExpressRoute conecta redes; Blueprints está en retirada y no gestiona recursos externos.

---

**8.** ¿Qué herramienta te muestra recomendaciones para reducir costes, mejorar la seguridad y aumentar la confiabilidad de tus recursos?

- A) Azure Monitor
- B) Azure Service Health
- C) Azure Advisor
- D) Azure Arc

**Respuesta: C.** Advisor ofrece recomendaciones en cinco categorías. Monitor mide, Service Health informa de Azure y Arc gestiona recursos externos.

---

**9.** ¿Cuál es el orden correcto de la jerarquía de ámbitos en Azure, de mayor a menor?

- A) Suscripción → Grupo de administración → Grupo de recursos → Recurso
- B) Grupo de administración → Suscripción → Grupo de recursos → Recurso
- C) Grupo de recursos → Suscripción → Grupo de administración → Recurso
- D) Grupo de administración → Grupo de recursos → Suscripción → Recurso

**Respuesta: B.** El grupo de administración contiene suscripciones, que contienen grupos de recursos, que contienen recursos.

---

**10.** Una organización quiere seguir una metodología de Microsoft con fases de estrategia, planificación, preparación, adopción, gobernanza y administración para su paso a la nube. ¿Qué debe consultar?

- A) Azure Policy
- B) Microsoft Cloud Adoption Framework
- C) Azure Blueprints
- D) Azure Well-Architected Framework

**Respuesta: B.** El CAF define esas fases. Policy es un servicio de cumplimiento; Blueprints está en retirada; el Well-Architected Framework trata del diseño de cargas concretas, no de la adopción organizativa.

---

> [!important] Última comprobación antes del examen
> Si puedes explicar en una frase la diferencia entre **Policy / RBAC / Lock** y entre **Monitor / Service Health / Advisor**, tienes cubierta la mayoría de las preguntas de este módulo.
