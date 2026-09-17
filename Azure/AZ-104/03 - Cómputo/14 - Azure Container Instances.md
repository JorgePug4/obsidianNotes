---
tags: [az-104, azure, computo, contenedores, aci]
modulo: Cómputo
peso_examen: Alto
---

# Azure Container Instances (ACI)

## ¿Qué es?

**Azure Container Instances** ejecuta **contenedores sueltos** sin orquestador ni servidores que administrar. Se paga **por segundo** por vCPU y memoria asignadas. Es la forma más rápida de poner un contenedor en marcha en Azure.

## ¿Para qué sirve?

- Tareas por lotes, trabajos programados, procesamiento de eventos.
- Aplicaciones simples o APIs de un solo contenedor.
- Extender capacidad (nodos virtuales de AKS).
- Entornos de compilación y pruebas efímeros.

## Conceptos clave

- **Container group** 🧠: unidad de despliegue. Uno o varios contenedores que **comparten ciclo de vida, red (IP y puertos) y volúmenes**, programados en la misma máquina. Análogo a un *pod* de Kubernetes. Multi-contenedor solo con plantilla ARM/Bicep, YAML o CLI (`--file`), no en el asistente simple del portal.
- **Recursos**: vCPU y memoria por contenedor; el grupo suma. Límites típicos: hasta **4 vCPU / 16 GB** en muchas regiones (Linux), menos en Windows; con **Confidential** y GPU hay SKUs específicos.
- **SKU**: Standard, Dedicated, **Confidential** ➕.
- **Sistemas operativos**: Linux y Windows (Windows con menos características: sin multi-contenedor con varios contenedores? sí soporta, pero con limitaciones de volúmenes y sin grupos multi-contenedor en algunos casos).
- **Redes** 🧠:
  - **Pública**: IP pública + **etiqueta DNS** → `<label>.<region>.azurecontainer.io`.
  - **Privada**: desplegado **en una subred de una VNet** (delegada a `Microsoft.ContainerInstance/containerGroups`); sin IP pública.
  - No admite ambas a la vez.
- **Directiva de reinicio (restart policy)** 🧠: **Always** (servicios), **OnFailure** (tareas que deben completarse), **Never** (ejecutar una vez). Con OnFailure/Never el grupo termina en estado *Succeeded*/*Failed*.
- **Volúmenes**: **Azure Files** (montaje SMB con clave), emptyDir, secret, gitRepo (en desuso).
- **Variables de entorno** y **secretos** (secure environment variables).
- **Identidad administrada** (system o user-assigned) para acceder a ACR, Key Vault, Storage.
- **Logs y diagnóstico**: `az container logs`, `az container attach`, `az container exec`, integración con **Log Analytics**.
- **Escalado**: ACI **no escala automáticamente**; se despliegan más grupos o se usa otro servicio. El "tamaño" se ajusta en la definición (vCPU/memoria) y requiere recrear el grupo.
- **Reinicios y actualizaciones**: cambiar la imagen implica recrear el container group (`az container create` de nuevo).

## Cómo funciona

```bash
# Contenedor público con DNS
az container create --resource-group rg-apps --name aci-web --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu 1 --memory 1.5 --ports 80 --ip-address Public --dns-name-label contoso-aci-web --os-type Linux --restart-policy Always
az container show --resource-group rg-apps --name aci-web --query "{fqdn:ipAddress.fqdn,state:instanceView.state}" -o table
az container logs --resource-group rg-apps --name aci-web
az container exec --resource-group rg-apps --name aci-web --exec-command "/bin/sh"
# Desde ACR con identidad administrada
az container create -g rg-apps -n aci-api --image acrcontoso.azurecr.io/miapp:1.0 \
  --assign-identity <userAssignedId> --acr-identity <userAssignedId> --cpu 2 --memory 4 --ports 8080
# En una VNet privada
az container create -g rg-apps -n aci-priv --image nginx --vnet vnet-apps --subnet aci-subnet --cpu 1 --memory 1
# Tarea que se ejecuta una vez
az container create -g rg-apps -n aci-job --image mcr.microsoft.com/azure-cli --restart-policy OnFailure \
  --command-line "az --version" --cpu 1 --memory 1
# Montar Azure Files
az container create -g rg-apps -n aci-files --image nginx --azure-file-volume-share-name share1 \
  --azure-file-volume-account-name st001 --azure-file-volume-account-key <key> --azure-file-volume-mount-path /mnt/data
az container delete -g rg-apps -n aci-web --yes
```

```powershell
New-AzContainerGroup -ResourceGroupName rg-apps -Name aci-web -Image mcr.microsoft.com/azuredocs/aci-helloworld `
  -OsType Linux -DnsNameLabel contoso-aci-web -Cpu 1 -MemoryInGB 1.5 -Port 80 -RestartPolicy Always
Get-AzContainerInstanceLog -ResourceGroupName rg-apps -ContainerGroupName aci-web
```

Portal: **Instancias de contenedor** → Crear → origen de la imagen (Quickstart, ACR, Docker Hub u otro registro), tamaño (vCPU/memoria), red (pública/privada/ninguna), etiqueta DNS, **directiva de reinicio**, variables de entorno, comando de reemplazo.

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| Ejecutar un trabajo que termina y no debe reiniciarse | Restart policy **Never** (o OnFailure si debe reintentar) |
| Servicio web siempre disponible | Restart policy **Always** + IP pública con etiqueta DNS |
| El contenedor no debe tener IP pública | Desplegar en **subred de VNet** (delegada) |
| Persistir datos entre reinicios | Volumen de **Azure Files** |
| Varios contenedores que comparten red y volumen (por ejemplo, app + sidecar de logs) | **Container group** multi-contenedor (YAML/ARM) |
| Descargar imagen de ACR sin credenciales | **Identidad administrada** + AcrPull |
| Cambiar vCPU/memoria | **Recrear** el grupo con los nuevos valores |
| Ver por qué falla el contenedor | `az container logs` y eventos en `instanceView` |
| Necesito escalado automático por HTTP o eventos | **No es ACI** → Container Apps |

## Ejemplo

Un proceso nocturno convierte archivos. Se despliega un container group con la imagen del conversor, **restart policy OnFailure**, 2 vCPU y 4 GB, montando un recurso compartido de Azure Files con los archivos de entrada y salida. Un runbook de Automation lanza `az container create` cada noche y el grupo se elimina al terminar, pagando solo los minutos de ejecución.

## Comparaciones

| Servicio | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **ACI** | Contenedor suelto | Arranque en segundos, pago por segundo, sin orquestador | Tareas, jobs, apps simples |
| **Container Apps** | Microservicios serverless | Escalado a cero, ingress, revisiones, KEDA/Dapr | Aplicaciones con escalado automático |
| **AKS** | Kubernetes gestionado | Control total, ecosistema K8s | Plataformas complejas |
| **App Service (contenedor)** | Web app en contenedor | Slots, dominios, certificados | Webs y APIs |
| **VMSS** | VMs | Control del SO | Cargas no contenerizadas |

## 💻 Laboratorio: ACI

1. Desplegar `aci-hello` con la imagen `mcr.microsoft.com/azuredocs/aci-helloworld`, IP pública y etiqueta DNS; abrir el FQDN en el navegador.
2. Ver logs con `az container logs` y entrar con `az container exec`.
3. Crear un grupo con **restart policy Never** que ejecute `az --version` y comprobar el estado *Succeeded*.
4. Montar un recurso compartido de Azure Files y escribir un archivo desde el contenedor.
5. Desplegar un contenedor en una subred delegada y comprobar que solo responde desde dentro de la VNet.

## AZ-104 Exam Tips

- ⭐ **Container group** = contenedores que comparten **ciclo de vida, red y volúmenes**.
- 🔥 🧠 Restart policy: **Always / OnFailure / Never**.
- 🔥 🧠 ACI **no tiene autoescalado ni escala a cero por HTTP**; se paga por segundo mientras el grupo existe.
- 🧠 Red: IP pública con **etiqueta DNS** `<label>.<region>.azurecontainer.io` **o** subred de VNet delegada, no ambas.
- 🧠 Persistencia → **Azure Files**.
- 💻 `az container create/logs/exec/show/delete`, portal.
- 📌 Cambiar tamaño = **recrear** el grupo.

## Errores comunes

- Esperar escalado automático en ACI.
- Guardar datos en el sistema de archivos del contenedor (se pierden al reiniciar).
- Usar restart policy Always para un job que debe terminar (se reinicia en bucle).

## Preguntas que podrían aparecer

**1.** Necesitas ejecutar un contenedor que procese un archivo y finalice, reintentando solo si falla. ¿Qué directiva de reinicio configuras?
- A) Always · B) OnFailure · C) Never · D) Rolling

<details><summary>Respuesta</summary>

**B.** OnFailure reinicia solo si el proceso termina con error; si termina correctamente, el grupo pasa a Succeeded.
</details>

**2.** Un contenedor de ACI debe guardar resultados que sobrevivan al reinicio del contenedor. ¿Qué configuras?
- A) Un disco administrado · B) Un volumen de Azure Files · C) Un blob container montado como disco · D) Nada, ACI persiste por defecto

<details><summary>Respuesta</summary>

**B.** ACI monta recursos compartidos de Azure Files como volúmenes persistentes.
</details>

**3.** Necesitas que un contenedor solo sea accesible desde máquinas virtuales de una red virtual. ¿Qué haces?
- A) Asignar IP pública y NSG · B) Desplegar el container group en una subred delegada de la VNet · C) Usar etiqueta DNS privada · D) Usar Container Apps

<details><summary>Respuesta</summary>

**B.** ACI admite despliegue en una subred delegada, obteniendo una IP privada y sin IP pública.
</details>

## Relacionado

- [[13 - Azure Container Registry]]
- [[15 - Azure Container Apps]]
- [[16 - Comparación de servicios de contenedores]]
- [[13 - Azure Files]]
- [[00 - Índice - Cómputo]]
