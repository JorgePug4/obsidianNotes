---
tags: [az-104, azure, computo, contenedores, container-apps, keda]
modulo: Cómputo
peso_examen: Alto
---

# Azure Container Apps

## ¿Qué es?

**Azure Container Apps** es una plataforma **serverless de contenedores** construida sobre Kubernetes (con **KEDA**, **Dapr** y **Envoy**) que ejecuta microservicios y aplicaciones con **escalado automático**, **escalado a cero**, ingress HTTPS, revisiones y despliegue progresivo, **sin gestionar nodos ni clústeres**.

Está explícitamente en el temario vigente de AZ-104 ("aprovisionar un contenedor mediante Azure Container Apps" y "administrar el tamaño y el escalado").

## ¿Para qué sirve?

- APIs y microservicios con tráfico variable (escalan a cero cuando no hay peticiones).
- Procesadores de eventos (colas, Event Hubs, Kafka).
- Trabajos programados o por eventos (**Container Apps Jobs**).
- Migrar contenedores sin adoptar Kubernetes completo.

## Conceptos clave

### Jerarquía 🧠

```
Entorno (Container Apps Environment)  ← límite de seguridad y red; comparte VNet y Log Analytics
 ├── Container App "api"
 │     ├── Revisión 1 (inmutable)   ── réplicas (contenedores)
 │     └── Revisión 2  ← división de tráfico 80/20
 └── Container App "worker"
```

- **Entorno**: una o varias apps que comparten **red virtual**, **espacio de trabajo de Log Analytics** y certificados. Las apps del mismo entorno se comunican por nombre interno.
- **Revisión**: versión inmutable de la app. Modo **Single** (una activa) o **Multiple** (varias con **división de tráfico** para blue/green y canary).
- **Réplica**: instancia en ejecución de una revisión.
- **Contenedor**: puede haber varios por app (sidecars), con **vCPU/memoria** por contenedor.

### Escalado 🧠
- **min replicas** y **max replicas** (por defecto **0 a 10**; máximo configurable hasta **1000**).
- **Escala a cero** cuando `min = 0` y solo con **reglas basadas en eventos** (HTTP, TCP, colas, personalizadas con KEDA).
- **Regla HTTP**: número de **peticiones concurrentes** por réplica (por defecto ~10).
- **Reglas KEDA**: Service Bus, Storage Queue, Event Hubs, Kafka, Redis, cron, personalizadas.
- ⚠️ Las reglas de **CPU y memoria no permiten escalar a cero** (necesitan una réplica en ejecución para medir).
- **Recursos por réplica**: combinaciones válidas de vCPU/memoria (por ejemplo, 0,25 vCPU/0,5 GiB … 2 vCPU/4 GiB en consumo; más en Dedicated); la memoria debe ser ~2x la vCPU en el plan de consumo.
- **Planes**: **Consumption** (serverless, pago por uso, escala a cero) y **Dedicated workload profiles** ➕ (instancias reservadas, más CPU/memoria, GPU).

### Otras características
- **Ingress**: externo (público, HTTPS con certificado gestionado) o interno (solo dentro del entorno/VNet); **puerto de destino**; soporte HTTP/2, gRPC, TCP.
- **Dominios personalizados y certificados** gestionados.
- **Secretos** de la app y referencias a **Key Vault**; **identidad administrada** para ACR y otros servicios.
- **Dapr** para comunicación entre servicios, estado y pub/sub.
- **Jobs**: ejecución puntual, programada (cron) o por eventos, con reintentos y paralelismo.
- **Observabilidad**: logs a Log Analytics, **Log stream**, **Console** (exec), métricas.
- **Redes**: entorno con VNet propia (subred dedicada con tamaño mínimo, p. ej. /23 para Consumption según perfil), UDR, NAT gateway.

## Cómo funciona

```bash
az extension add --name containerapp
az provider register --namespace Microsoft.App
# Entorno
az containerapp env create --name env-contoso --resource-group rg-apps --location westeurope
# App desde ACR con identidad administrada
az containerapp create --name api --resource-group rg-apps --environment env-contoso \
  --image acrcontoso.azurecr.io/miapp:1.0 --registry-server acrcontoso.azurecr.io --registry-identity system \
  --target-port 8080 --ingress external --cpu 0.5 --memory 1.0Gi \
  --min-replicas 0 --max-replicas 10
# Regla de escalado HTTP
az containerapp update --name api --resource-group rg-apps --scale-rule-name http-rule --scale-rule-type http \
  --scale-rule-http-concurrency 20 --min-replicas 1 --max-replicas 30
# Regla KEDA con cola de Service Bus
az containerapp update --name worker --resource-group rg-apps \
  --scale-rule-name queue-rule --scale-rule-type azure-servicebus \
  --scale-rule-metadata queueName=orders messageCount=5 --scale-rule-auth connection=sb-connection
# Revisiones y tráfico
az containerapp revision list --name api --resource-group rg-apps -o table
az containerapp ingress traffic set --name api --resource-group rg-apps --revision-weight <rev1>=80 <rev2>=20
# Logs
az containerapp logs show --name api --resource-group rg-apps --follow
```

Portal: **Aplicaciones de contenedor** → Crear → entorno (nuevo o existente), contenedor (imagen, ACR/Docker Hub, CPU/memoria), **Ingress** (habilitado, tráfico externo/limitado al entorno, puerto de destino), y tras crear: **Escalado** (réplicas mín./máx. y reglas), **Revisiones y réplicas**, **Secretos**, **Identidad**.

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| No pagar nada cuando no hay tráfico | **min replicas = 0** + regla HTTP/KEDA |
| Una réplica siempre caliente (evitar arranque en frío) | **min replicas = 1** |
| Escalar por longitud de cola | Regla **KEDA** (Service Bus / Storage Queue) |
| Escalar por CPU | Regla de CPU (⚠️ no escala a cero) |
| Despliegue canary 90/10 | Modo de revisión **Multiple** + división de tráfico |
| API interna solo para otras apps del entorno | **Ingress interno** |
| Dominio propio con HTTPS | Dominio personalizado + certificado administrado |
| Descargar imagen de ACR sin secretos | **Identidad administrada** + AcrPull |
| Trabajo por lotes con cron | **Container Apps Job** |
| Más CPU/memoria de la que permite el consumo | **Workload profile** dedicado |

## Ejemplo

Una API recibe tráfico solo en horario laboral. Se despliega en Container Apps con `min=0`, `max=20` y una regla HTTP de 20 peticiones concurrentes por réplica: fuera de horario escala a cero (coste ~0) y en picos crece automáticamente. Un `worker` en el mismo entorno tiene ingress interno y escala con una regla KEDA sobre una cola de Service Bus (1 réplica por cada 5 mensajes). Las nuevas versiones se publican como revisión con 10 % de tráfico antes de promoverlas al 100 %.

## Comparaciones

| Servicio | Escala a cero | Orquestación | Cuándo utilizarlo |
|---|---|---|---|
| **Container Apps** | **Sí** (con reglas de evento) | Gestionada (KEDA/Dapr/Envoy) | Microservicios, APIs, workers con tráfico variable |
| **ACI** | No (pagas mientras exista) | Ninguna | Contenedores sueltos, jobs simples |
| **AKS** | Con KEDA/manual | Kubernetes completo | Control total, ecosistema K8s |
| **App Service (contenedor)** | No (Always On) | PaaS web | Webs con slots, dominios, autenticación integrada |
| **Azure Functions** | Sí | Serverless funciones | Código orientado a eventos |

## 💻 Laboratorio: Container Apps

1. Crear el entorno `env-lab` y una app con la imagen `mcr.microsoft.com/k8se/quickstart:latest`, ingress externo y puerto 80.
2. Comprobar la URL pública y ver **Log stream**.
3. Configurar escalado: min 0, max 5, regla HTTP con concurrencia 10. Generar carga y observar las réplicas.
4. Actualizar la imagen para crear una **nueva revisión**, cambiar el modo a Multiple y repartir el tráfico 50/50.
5. Crear una app con ingress interno y llamarla desde la primera por su nombre interno.

## AZ-104 Exam Tips

- ⭐ Jerarquía **Entorno → App → Revisión → Réplica**.
- 🔥 🧠 **Escala a cero** solo con `min=0` y reglas **HTTP/TCP/KEDA**; **CPU y memoria no escalan a cero**.
- 🔥 🧠 Rango por defecto **0-10** réplicas; configurable hasta **1000**.
- 🧠 Ingress **externo** (público) vs **interno** (dentro del entorno/VNet).
- 🧠 Revisiones **inmutables**; modo Multiple permite **división de tráfico**.
- 💻 `az containerapp env create`, `az containerapp create/update --min-replicas/--max-replicas`, reglas de escalado, división de tráfico.
- 📌 Container Apps (escala a cero, ingress, revisiones) vs ACI (contenedor suelto sin escalado).

## Errores comunes

- Configurar una regla de CPU con `min=0` y esperar que escale a cero.
- Crear apps en entornos distintos y esperar comunicación interna por nombre.
- Confundir revisión (versión) con réplica (instancia).

## Preguntas que podrían aparecer

**1.** Tu Container App debe dejar de consumir recursos cuando no recibe peticiones. ¿Qué configuración lo permite?
- A) min replicas = 1 con regla de CPU · B) min replicas = 0 con una regla de escalado HTTP · C) Ingress interno · D) Modo de revisión Single

<details><summary>Respuesta</summary>

**B.** El escalado a cero requiere mínimo 0 réplicas y una regla basada en eventos, como la HTTP.
</details>

**2.** Quieres enviar el 10 % del tráfico a la nueva versión de una aplicación antes de promoverla. ¿Qué configuras?
- A) Deployment slots · B) Modo de revisión Multiple con división de tráfico · C) Dos entornos · D) Una regla KEDA

<details><summary>Respuesta</summary>

**B.** Las revisiones múltiples con pesos de tráfico permiten despliegues canary. Los slots son de App Service.
</details>

**3.** ¿Qué recurso comparten las aplicaciones que están en el mismo entorno de Container Apps?
- A) La imagen de contenedor · B) La red virtual y el área de trabajo de Log Analytics · C) Las réplicas · D) El registro de contenedor

<details><summary>Respuesta</summary>

**B.** El entorno es el límite que comparte red, logs y certificados entre sus aplicaciones.
</details>

## Relacionado

- [[13 - Azure Container Registry]]
- [[14 - Azure Container Instances]]
- [[16 - Comparación de servicios de contenedores]]
- [[17 - App Service Plan (niveles y escalado)]]
- [[00 - Índice - Cómputo]]
