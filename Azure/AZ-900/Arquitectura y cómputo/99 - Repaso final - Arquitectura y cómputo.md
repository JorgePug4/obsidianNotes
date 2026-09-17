---
tags: [az-900, azure, arquitectura, computo, repaso]
modulo: Arquitectura y cómputo
---

# 🎯 Repaso final de Arquitectura y cómputo

Nota de consolidación de los componentes arquitectónicos y los servicios de cómputo (dominio 2). Úsala el día antes del examen.

## Los conceptos más importantes

1. **Jerarquía física**: datacenter → zona de disponibilidad → región → geografía.
2. **Zonas**: mínimo 3 por región, independientes en energía, refrigeración y red; protegen contra fallo de **datacenter**.
3. **Pares de regiones**: misma geografía, ≥300 millas, actualizaciones secuenciales; protegen contra fallo de **región**.
4. **Regiones soberanas**: Azure Government y Azure China, aisladas.
5. **Jerarquía lógica**: grupo de administración → suscripción → grupo de recursos → recurso; herencia hacia abajo.
6. **Grupo de recursos**: un recurso vive en uno solo; no se anidan; regiones mixtas; borrarlo borra todo.
7. **Suscripción** = facturación y límites; muchas por tenant.
8. **VM** = IaaS; **VMSS** = autoescalado horizontal; **conjunto de disponibilidad** = dominios de error y actualización; **AVD** = escritorios multisesión.
9. **Contenedores**: ACI (simple) < Container Apps (serverless con escala a cero) < AKS (Kubernetes administrado).
10. **App Service** = PaaS web/API; **Functions** = serverless por eventos, pago por ejecución.

## Tabla de servicios y su propósito

| Servicio / concepto | Propósito en una frase | Palabra clave de examen |
|---|---|---|
| Región | Unidad de despliegue con varios datacenters | *deploy to, latency, services vary* |
| Zona de disponibilidad | Datacenters independientes dentro de la región | *datacenter failure, minimum three* |
| Par de regiones | Región hermana a ≥300 millas para desastres | *regional disaster, 300 miles, sequential updates* |
| Región soberana | Nube aislada para gobiernos | *Azure Government, compliance* |
| Grupo de recursos | Contenedor lógico por ciclo de vida | *delete all at once, one group per resource* |
| Suscripción | Unidad de facturación y límites | *separate billing, quota* |
| Azure Resource Manager | Capa de gestión que procesa toda petición | *deployment and management service* |
| Máquina virtual | Servidor IaaS con control total | *operating system, lift-and-shift* |
| VM Scale Sets | VMs idénticas con autoescalado | *automatically add instances* |
| Conjunto de disponibilidad | Repartir VMs en dominios de error y actualización | *hardware failure, planned maintenance* |
| Azure Virtual Desktop | Escritorios Windows en la nube | *desktop from any device, multi-session* |
| Container Instances | Contenedor sin infraestructura | *simplest, quickest container* |
| Container Apps | Microservicios serverless en contenedores | *scale to zero, no Kubernetes management* |
| Kubernetes Service | Orquestación de contenedores administrada | *orchestrate, cluster* |
| Container Registry | Registro privado de imágenes | *store images* |
| App Service | Hospedaje PaaS de web y API | *web app, API, multiple languages* |
| Functions | Código serverless por eventos | *event-driven, pay per execution* |
| Logic Apps | Flujos serverless sin código | *workflow, no-code, connectors* |

## Las diferencias que más fácilmente puedo confundir

> [!warning] Pares de confusión clásicos
> | Confusión | Cómo distinguirlos |
> |---|---|
> | **Zona vs Par de regiones** | Zona = fallo de datacenter, misma región, síncrono. Par = fallo de región, ≥300 millas, asíncrono. |
> | **Zona vs Conjunto de disponibilidad** | Zona = datacenters distintos. Conjunto = racks distintos en el mismo datacenter. |
> | **Región vs Geografía** | Región = donde despliegas. Geografía = frontera legal de residencia de datos. |
> | **Grupo de recursos vs Suscripción** | Grupo = ciclo de vida y permisos. Suscripción = facturación y límites. |
> | **Suscripción vs Tenant** | Tenant = identidades. Suscripción = facturación; confía en un tenant. |
> | **VMSS vs Load Balancer** | VMSS crea/elimina VMs. Load Balancer reparte tráfico entre las existentes. |
> | **VM vs Contenedor** | VM incluye SO completo. Contenedor comparte kernel y es ligero. |
> | **ACI vs AKS** | ACI = un contenedor rápido y simple. AKS = orquestación de muchos. |
> | **Container Apps vs AKS** | Container Apps oculta Kubernetes y escala a cero. AKS te da el clúster. |
> | **App Service vs Functions** | App Service = app completa, plan siempre pagado. Functions = código por eventos, pago por ejecución. |
> | **Functions vs Logic Apps** | Functions = código. Logic Apps = diseñador visual sin código. |
> | **Detener vs Desasignar VM** | Solo desasignar deja de cobrar cómputo. |

## 10 tips de examen

> [!tip]
> 1. "Fallo de un **centro de datos**" → zonas. "Fallo de una **región**" → par de regiones / GRS. "Fallo de un **rack** o mantenimiento" → conjunto de disponibilidad.
> 2. "¿Cuántas zonas como mínimo?" → **3**. "¿Distancia entre regiones emparejadas?" → **≥300 millas**.
> 3. "Borrar todo de golpe" → eliminar el **grupo de recursos**.
> 4. "Facturas separadas" → **suscripciones**. "Aplicar a varias suscripciones" → **grupo de administración**.
> 5. Los recursos de un grupo **pueden estar en regiones distintas**; un recurso está en **un solo** grupo; los grupos **no se anidan**.
> 6. "Añadir VMs automáticamente" → **VMSS**. "Escritorio Windows desde cualquier dispositivo" → **AVD**.
> 7. "Contenedor de la forma **más simple**" → **ACI**. "**Orquestar**" → **AKS**. "Escala a cero sin Kubernetes" → **Container Apps**.
> 8. "Web/API sin gestionar SO" → **App Service**. "Código por **evento**, pago por **ejecución**" → **Functions**. "Sin código" → **Logic Apps**.
> 9. Control de mayor a menor: **VM > contenedor > App Service > Functions**. La pregunta "necesita instalar software en el SO" siempre es VM.
> 10. Servicios **globales** que no piden región: Entra ID, DNS, Traffic Manager, Front Door.

## 10 preguntas de repaso tipo AZ-900

**1.** ¿Cuál es el número mínimo de zonas de disponibilidad en una región de Azure que las admite?
- A) 1 · B) 2 · C) 3 ✅ · D) 5

**2.** Una empresa necesita que sus datos sigan disponibles aunque un huracán deje fuera de servicio toda la región donde están desplegados. ¿Qué característica de Azure lo permite?
- A) Zonas de disponibilidad · B) Pares de regiones ✅ · C) Conjuntos de disponibilidad · D) Grupos de recursos

**3.** ¿Cuál de las siguientes afirmaciones sobre los grupos de recursos es verdadera?
- A) Un recurso puede pertenecer a varios grupos de recursos
- B) Un grupo de recursos puede contener recursos de varias regiones ✅
- C) Los grupos de recursos pueden anidarse
- D) Eliminar un grupo de recursos conserva sus recursos

**4.** ¿Qué elemento de la jerarquía de Azure representa la unidad de facturación?
- A) Grupo de recursos · B) Grupo de administración · C) Suscripción ✅ · D) Tenant

**5.** Necesitas que una aplicación en máquinas virtuales aumente y reduzca el número de instancias según la carga. ¿Qué servicio usas?
- A) Conjunto de disponibilidad · B) Virtual Machine Scale Sets ✅ · C) Azure Load Balancer · D) Azure Virtual Desktop

**6.** ¿Qué protege un conjunto de disponibilidad?
- A) Contra la caída de una región
- B) Contra la caída de un centro de datos completo
- C) Contra fallos de hardware y mantenimientos planificados dentro de un centro de datos ✅
- D) Contra la pérdida de datos en discos

**7.** Un equipo quiere ejecutar un contenedor Docker de forma puntual sin crear ni administrar un clúster. ¿Qué servicio es el más adecuado?
- A) Azure Kubernetes Service · B) Azure Container Instances ✅ · C) Azure Virtual Machines · D) Azure App Service

**8.** ¿Qué servicio permite ejecutar código en respuesta a eventos, escalando a cero y pagando solo por ejecución?
- A) Azure App Service · B) Azure Functions ✅ · C) Azure Virtual Machines · D) Azure Kubernetes Service

**9.** ¿Qué servicio proporciona escritorios Windows virtualizados accesibles desde cualquier dispositivo, con soporte multisesión?
- A) Azure Virtual Machines · B) Azure App Service · C) Azure Virtual Desktop ✅ · D) Azure Container Apps

**10.** Un desarrollador quiere hospedar una aplicación web en Node.js con despliegues sin tiempo de inactividad y sin administrar el sistema operativo. ¿Qué servicio elige?
- A) Azure Virtual Machines · B) Azure App Service ✅ · C) Azure Container Instances · D) Azure Functions

## 🧠 Última pasada antes del examen

> [!important] Lo que no puede fallarte
> - Datacenter → zona (mín. 3) → región → geografía. Par de regiones ≥300 millas.
> - Zona = fallo de datacenter · Par = fallo de región · Conjunto de disponibilidad = fallo de rack.
> - Grupo de administración → suscripción (facturación) → grupo de recursos (ciclo de vida) → recurso.
> - VM = IaaS · VMSS = autoescalado · AVD = escritorios.
> - ACI simple · Container Apps escala a cero · AKS orquesta.
> - App Service = web/API · Functions = eventos y pago por ejecución · Logic Apps = sin código.

Volver al índice: [[00 - Índice - Arquitectura y cómputo]]
