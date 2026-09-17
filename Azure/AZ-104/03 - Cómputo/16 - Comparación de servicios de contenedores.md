---
tags: [az-104, azure, computo, contenedores, comparacion]
modulo: Cómputo
peso_examen: Alto
---

# Comparación de servicios de contenedores

## ¿Qué es?

Resumen decisivo de los servicios de contenedores que evalúa AZ-104 (**ACR**, **ACI**, **Container Apps**) más los que aparecen como distractores (**AKS**, **App Service**, **Functions**). El examen casi siempre pregunta **cuál elegir** ante un escenario y **cómo se escala** cada uno.

## Tabla principal

| Servicio | Qué es | Escalado | Escala a cero | Red | Cuándo utilizarlo |
|---|---|---|---|---|---|
| **Azure Container Registry (ACR)** | Registro privado de imágenes | Almacenamiento por SKU | — | Private Link (Premium) | Guardar y distribuir imágenes |
| **Azure Container Instances (ACI)** | Contenedor suelto sin orquestador | **Manual** (crear más grupos) | No (pagas por segundo mientras exista) | IP pública con DNS **o** subred de VNet | Jobs, tareas, contenedores efímeros |
| **Azure Container Apps** | Contenedores serverless con KEDA/Dapr | **Automático** (HTTP, eventos, CPU/memoria) | **Sí** (min=0 con reglas de evento) | Ingress externo/interno, VNet | Microservicios y APIs con tráfico variable |
| **Azure Kubernetes Service (AKS)** ➕ | Kubernetes gestionado | HPA, cluster autoscaler, KEDA | Con KEDA | CNI/kubenet, private cluster | Plataformas complejas, control total |
| **App Service (contenedor)** | Web app que ejecuta un contenedor | Escalado del plan (manual/auto) | No | VNet integration, private endpoint | Webs y APIs con slots, dominios, auth |
| **Azure Functions (contenedor)** ➕ | Funciones serverless | Por eventos | Sí (plan de consumo) | VNet en Premium | Código orientado a eventos |

## Escalado: cómo se ajusta "el tamaño" en cada uno

| Servicio | "Tamaño" | "Escalado" |
|---|---|---|
| **ACI** | vCPU y memoria por contenedor (recrear el grupo para cambiarlo) | Crear/eliminar container groups manualmente |
| **Container Apps** | vCPU y memoria por réplica (combinaciones válidas) | min/max réplicas + reglas (HTTP, TCP, KEDA, CPU, memoria) |
| **AKS** | Tamaño de VM de los nodos | Node pools, cluster autoscaler, HPA |
| **App Service** | Nivel del plan (B1, S1, P1v3…) | Escalar vertical (nivel) y horizontal (instancias, autoscale) |

## Reglas mentales para el examen 🧠

> [!important] Palabras clave → servicio
> - "sin orquestador", "tarea que se ejecuta y termina", "arranque rápido", "pago por segundo" → **ACI**
> - "escala a cero", "microservicios", "revisiones", "división de tráfico", "KEDA", "Dapr", "ingress" → **Container Apps**
> - "Kubernetes", "kubectl", "Helm", "control total del clúster" → **AKS**
> - "registro privado de imágenes", "geo-replicación de imágenes", "AcrPull" → **ACR**
> - "aplicación web", "ranuras de implementación", "dominio y certificado", "autenticación integrada" → **App Service**
> - "función", "desencadenador", "pago por ejecución" → **Functions**

## Comparaciones adicionales

| Pareja | Diferencia en una línea |
|---|---|
| **ACI vs Container Apps** | ACI no escala automáticamente ni a cero; Container Apps sí, y añade ingress, revisiones y KEDA |
| **Container Apps vs AKS** | Container Apps oculta Kubernetes (sin nodos ni kubectl); AKS da control total y toda la API de K8s |
| **Container Apps vs App Service** | Container Apps escala a cero y está pensado para microservicios; App Service ofrece slots, auth integrada y planes fijos |
| **ACI vs Container Apps Jobs** | Ambos ejecutan tareas; Jobs añade cron, reintentos y paralelismo dentro del entorno |
| **ACR Basic/Standard vs Premium** | Premium añade geo-replicación, private link, content trust y CMK |
| **Identidad administrada vs usuario admin de ACR** | Sin secretos y con RBAC vs cuenta compartida (desaconsejada) |

## AZ-104 Exam Tips

- 🔥 📌 **ACI = sin autoescalado**; **Container Apps = escala a cero**; **AKS = Kubernetes completo**.
- 🔥 🧠 Para descargar imágenes privadas desde cualquiera de ellos: **identidad administrada con AcrPull**.
- 🧠 Cambiar vCPU/memoria en ACI implica **recrear** el container group; en Container Apps se actualiza la app (nueva revisión).
- 🧠 ACI en VNet: subred **delegada**; Container Apps: entorno con VNet e ingress interno.
- 💻 Saber crear los tres desde el portal y con CLI.

## Preguntas que podrían aparecer

**1.** Una API con tráfico impredecible debe no generar coste durante la noche y escalar automáticamente durante el día. ¿Qué servicio eliges?
- A) Azure Container Instances · B) Azure Container Apps · C) Virtual Machine Scale Set · D) Azure Container Registry

<details><summary>Respuesta</summary>

**B.** Container Apps escala a cero con reglas HTTP y crece automáticamente con la demanda.
</details>

**2.** Necesitas ejecutar un contenedor de migración de datos una sola vez, con la menor complejidad posible. ¿Qué usas?
- A) AKS · B) ACI con restart policy Never · C) Container Apps con min replicas 1 · D) App Service

<details><summary>Respuesta</summary>

**B.** ACI ejecuta el contenedor sin orquestador y con la directiva Never finaliza tras completarse.
</details>

**3.** ¿Qué servicio almacena las imágenes que consumen ACI, Container Apps y AKS?
- A) Azure Storage · B) Azure Container Registry · C) Azure Compute Gallery · D) Azure Artifacts

<details><summary>Respuesta</summary>

**B.** ACR es el registro privado de imágenes de contenedor. Compute Gallery almacena imágenes de VM.
</details>

## Relacionado

- [[13 - Azure Container Registry]]
- [[14 - Azure Container Instances]]
- [[15 - Azure Container Apps]]
- [[17 - App Service Plan (niveles y escalado)]]
- [[00 - Índice - Cómputo]]
