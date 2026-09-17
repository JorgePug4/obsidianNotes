---
tags: [az-900, azure, computo, serverless, functions, app-service, paas]
modulo: Arquitectura y cómputo
peso_examen: Medio
---

# Azure Functions y App Service

## Concepto

Son las dos opciones **PaaS** para hospedar código sin administrar servidores:

- **Azure App Service**: plataforma para **aplicaciones web, APIs REST y back-ends móviles**. Subes tu código (o contenedor) y Azure lo ejecuta, lo escala y lo parchea.
- **Azure Functions**: plataforma **serverless** para ejecutar **pequeños fragmentos de código en respuesta a eventos**, pagando solo por la ejecución.

**Problema que resuelve:** un desarrollador quiere publicar una web o una API sin instalar IIS, configurar Linux ni aplicar parches. O quiere que un trozo de código se ejecute cuando pasa algo (llega un archivo, un mensaje, una hora) sin mantener un servidor encendido para eso.

**Para qué se utiliza:**
- App Service: sitios web, APIs, aplicaciones de negocio, back-ends de apps móviles.
- Functions: procesamiento por eventos, integraciones, tareas programadas, microservicios pequeños.

## Características principales

### Azure App Service
- Soporta **.NET, Java, Node.js, Python, PHP, Ruby** y contenedores, en Windows o Linux.
- Tipos de aplicación: **Web Apps**, **API Apps**, **WebJobs** (tareas en segundo plano) y **Mobile Apps**.
- Se ejecuta en un **plan de App Service** que define **tamaño, región y nivel** (Free, Basic, Standard, Premium, Isolated). El plan se paga **aunque la app no reciba tráfico**.
- Incluye **escalado automático**, **balanceo de carga**, **ranuras de implementación** (deployment slots) para desplegar sin caída, integración continua con GitHub/Azure DevOps, certificados TLS y dominios personalizados.
- Alta disponibilidad y parches del SO gestionados por Azure.
- Sin acceso al SO subyacente: si necesitas instalar algo a nivel de sistema, es [[03 - Máquinas virtuales, Scale Sets y conjuntos de disponibilidad|VM]].

### Azure Functions
- **Serverless y basado en eventos**: la función se ejecuta cuando la dispara un **trigger**: petición HTTP, temporizador, nuevo blob, mensaje en cola, evento de Event Grid o Service Bus, cambio en Cosmos DB.
- **Escala automáticamente** de cero a miles de instancias y **vuelve a cero** cuando no hay eventos.
- Plan de **consumo**: pagas por **ejecuciones y tiempo de ejecución** (con un millón de ejecuciones gratuitas al mes). También hay planes Premium y dedicado (dentro de un plan de App Service) para casos con arranque en frío inaceptable.
- Funciones **sin estado** por defecto (stateless); **Durable Functions** permite flujos con estado.
- Ideal para tareas **cortas** (el plan de consumo limita la duración).

### Otras opciones serverless relacionadas
- **Azure Logic Apps**: flujos de trabajo **sin código** (diseñador visual) que conectan cientos de servicios (Office 365, SQL, Salesforce…). Functions es *código*; Logic Apps es *diseñador*.
- **Azure Container Apps**: contenedores serverless (ver [[04 - Contenedores - ACI, AKS y Container Apps]]).

### Opciones de hospedaje de aplicaciones (comparativa del temario)

| Opción | Tipo | Control | Gestión del SO | Escalado | Cuándo |
|---|---|---|---|---|---|
| **Máquina virtual** | IaaS | Máximo | Tú | Manual o VMSS | Software con requisitos de SO |
| **Contenedores** (ACI/AKS/Container Apps) | PaaS | Alto | No | Automático (según servicio) | Microservicios, portabilidad |
| **App Service** | PaaS | Medio | No | Automático | Web y APIs estándar |
| **Functions** | Serverless | Bajo | No | Automático a cero | Código por eventos |

## Casos de uso

- Portal corporativo en .NET con despliegue sin caída: **App Service** con ranuras de implementación.
- Redimensionar cada imagen que un usuario sube a Blob Storage: **Function** con trigger de blob.
- Enviar un informe cada lunes a las 08:00: **Function** con trigger de temporizador.
- Cuando llega un correo con adjunto, guardarlo en SharePoint y avisar en Teams, sin programar: **Logic Apps**.

## Comparaciones

| Necesidad | Servicio | No confundir con |
|---|---|---|
| Hospedar una web o API sin gestionar SO | App Service | VM (tendrías que gestionar el SO) |
| Ejecutar código solo cuando ocurre un evento y pagar por ejecución | Functions | App Service (cobra el plan siempre) |
| Flujo de integración sin escribir código | Logic Apps | Functions (requiere código) |
| Desplegar nueva versión sin caída | Ranuras de implementación de App Service | Crear otra VM |
| Tarea en segundo plano ligada a una web | WebJobs (App Service) | Functions (independiente) |

## Conceptos que debo memorizar

> [!important]
> - **App Service** = PaaS para **web, API, móvil**; multi-lenguaje; **plan** de App Service que se paga siempre; escalado automático; ranuras de implementación; **sin acceso al SO**.
> - **Functions** = **serverless**, por **eventos (triggers)**, **escala a cero**, pago por **ejecución**; sin estado por defecto.
> - **Logic Apps** = flujos serverless **sin código**.
> - Escala de control: VM > contenedores > App Service > Functions.

## Tips para AZ-900

> [!tip]
> - "Aplicación web", "API REST", "varios lenguajes", "sin gestionar el servidor" → **App Service**.
> - "Cuando ocurra X ejecuta código", "pagar solo por ejecución", "escala a cero" → **Functions**.
> - "Flujo de trabajo", "sin código", "conectar servicios" → **Logic Apps**.
> - "Desplegar sin tiempo de inactividad" → **ranuras de implementación** de App Service.
> - Trampa: "App Service permite instalar software en el sistema operativo". **Falso**.
> - Trampa: "Functions es ideal para procesos de varias horas". **Falso**; son tareas cortas.
> - Trampa: "App Service no cobra si no hay tráfico". **Falso** (salvo el nivel Free); el plan se paga siempre.

## Ejemplo de pregunta de examen

**Pregunta 1.** Un desarrollador necesita hospedar una API REST escrita en Python y no quiere administrar ningún servidor ni aplicar parches del sistema operativo. ¿Qué servicio es el más adecuado?

- A) Azure Virtual Machines
- B) Azure App Service
- C) Azure Virtual Desktop
- D) Azure Container Registry

**Respuesta: B.** App Service hospeda APIs en Python y gestiona el SO por ti.
- A) Exigiría administrar el SO y los parches.
- C) Es un servicio de escritorios.
- D) Almacena imágenes de contenedor, no ejecuta código.

**Pregunta 2.** Quieres que un fragmento de código se ejecute automáticamente cada vez que se cargue un archivo en una cuenta de almacenamiento, y solo pagar por las ejecuciones. ¿Qué servicio debes usar?

- A) Azure App Service
- B) Azure Functions
- C) Azure Kubernetes Service
- D) Azure Virtual Machine Scale Sets

**Respuesta: B.** Functions con un trigger de blob es exactamente ese escenario: por eventos y pago por ejecución.
- A) Cobra el plan aunque no se ejecute nada.
- C y D) Requieren infraestructura encendida y no se facturan por ejecución.

## 🧠 Resumen para el examen

1. App Service = PaaS para web, API y móvil; multi-lenguaje; escalado automático; ranuras de implementación; sin acceso al SO.
2. Functions = serverless por eventos; escala a cero; pago por ejecución; tareas cortas.
3. Logic Apps = flujos de trabajo serverless sin código.
4. Hospedaje de apps: VM (máximo control) > contenedores > App Service > Functions (mínima gestión).
5. El plan de App Service se paga siempre; Functions en plan de consumo solo cuando se ejecuta.

---
