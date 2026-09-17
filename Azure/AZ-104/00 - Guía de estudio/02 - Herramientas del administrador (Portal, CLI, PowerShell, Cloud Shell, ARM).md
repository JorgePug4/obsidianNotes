---
tags: [az-104, azure, herramientas, cli, powershell, arm]
modulo: Guía de estudio
peso_examen: Transversal
---

# 🧰 Herramientas del administrador de Azure

## ¿Qué es?

El conjunto de interfaces con las que un administrador crea y gestiona recursos: **Azure Portal**, **Azure CLI**, **Azure PowerShell**, **Azure Cloud Shell**, plantillas **ARM/Bicep** y la **API REST**. Todas hablan con el mismo punto de entrada: **Azure Resource Manager (ARM)**, la capa de administración que autentica, autoriza (RBAC), valida (Policy) y ejecuta cada petición.

## ¿Para qué sirve?

- El portal para exploración y tareas puntuales.
- CLI/PowerShell para automatizar y para tareas que el portal no expone.
- ARM/Bicep para despliegues repetibles (infraestructura como código).
- Cloud Shell para tener CLI y PowerShell listos sin instalar nada.

## Conceptos clave

- **Azure Resource Manager**: capa de administración única; todo pasa por ella (plano de control).
- **Plano de control vs plano de datos**: crear una cuenta de almacenamiento es plano de control (ARM, RBAC de roles clásicos); leer un blob es plano de datos (claves, SAS o roles de datos).
- **Proveedor de recursos (resource provider)**: `Microsoft.Compute`, `Microsoft.Network`, `Microsoft.Storage`… Hay que **registrarlo** en la suscripción antes de usar un tipo de recurso por primera vez.
- **Contexto**: la suscripción activa en CLI (`az account set`) o PowerShell (`Set-AzContext`).
- **Idempotencia**: ARM y Bicep permiten repetir un despliegue sin duplicar recursos.

## Cómo funciona

```
Portal / CLI / PowerShell / SDK / REST
            │
            ▼
   Azure Resource Manager  ──► autenticación (Entra ID)
            │                 ──► autorización (RBAC)
            │                 ──► validación (Policy, locks)
            ▼
   Proveedores de recursos (Compute, Network, Storage…)
```

## Componentes y comandos que debes reconocer

### Azure CLI (multiplataforma, sintaxis `az <grupo> <acción>`)

```bash
az login
az account list --output table
az account set --subscription "Producción"
az group create --name rg-web --location westeurope
az vm create --resource-group rg-web --name vm01 --image Ubuntu2204 --size Standard_B2s --generate-ssh-keys
az vm list --query "[].{name:name, rg:resourceGroup}" --output table
az storage account create --name stweb001 --resource-group rg-web --sku Standard_LRS --kind StorageV2
az deployment group create --resource-group rg-web --template-file main.bicep
```

- `--query` usa **JMESPath**; `--output` acepta `json`, `table`, `tsv`, `yaml`.
- `az find` y `az <comando> --help` para descubrir opciones.

### Azure PowerShell (módulo **Az**, verbo-sustantivo)

```powershell
Connect-AzAccount
Get-AzSubscription
Set-AzContext -Subscription "Producción"
New-AzResourceGroup -Name rg-web -Location westeurope
New-AzVM -ResourceGroupName rg-web -Name vm01 -Image Ubuntu2204 -Size Standard_B2s
Get-AzVM | Select-Object Name, ResourceGroupName
New-AzStorageAccount -ResourceGroupName rg-web -Name stweb001 -SkuName Standard_LRS -Kind StorageV2 -Location westeurope
New-AzResourceGroupDeployment -ResourceGroupName rg-web -TemplateFile main.bicep
```

- El módulo antiguo **AzureRM** está retirado; la respuesta correcta siempre es **Az**.
- `Get-Command -Module Az.Compute` para descubrir cmdlets.

### Azure Cloud Shell

- Shell en el navegador (portal, Windows Terminal, VS Code) con **Bash o PowerShell**, Az CLI y módulo Az preinstalados.
- Requiere una **cuenta de almacenamiento con Azure Files** para persistir el directorio home (`clouddrive`). En 2024 se añadió el modo **efímero** sin storage.
- Incluye editor integrado (`code .`), Git, Terraform, kubectl, Bicep CLI.

### Otros

- **Azure Mobile App**: consulta estado, ejecuta Cloud Shell.
- **ARM templates (JSON)** y **Bicep**: ver [[01 - Azure Resource Manager y plantillas ARM]] y [[02 - Bicep]].
- **Azure Arc** ➕: extiende ARM a servidores y Kubernetes fuera de Azure para gestionarlos como recursos de Azure.

## Configuración relevante para el examen

| Necesidad | Herramienta / comando |
|---|---|
| Cambiar de suscripción | `az account set` / `Set-AzContext` |
| Registrar proveedor de recursos | `az provider register --namespace Microsoft.Xyz` / `Register-AzResourceProvider` |
| Ver cuotas y límites | Portal → Suscripción → Uso + cuotas |
| Despliegue de plantilla a RG | `az deployment group create` / `New-AzResourceGroupDeployment` |
| Despliegue a suscripción | `az deployment sub create` / `New-AzSubscriptionDeployment` |
| Exportar plantilla | Portal → RG → Exportar plantilla; `az group export` |

## Ejemplo

Un administrador debe crear 40 cuentas de almacenamiento con la misma configuración en tres suscripciones. Usar el portal 40 veces es lento y propenso a errores. Solución: un archivo Bicep con parámetros, ejecutado en bucle desde Cloud Shell con `az deployment group create` tras cambiar el contexto a cada suscripción.

## Comparaciones

| Herramienta | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Azure Portal** | Interfaz gráfica | Sin instalar, descubrimiento visual, asistentes | Tareas puntuales, exploración, diagnóstico |
| **Azure CLI** | Comandos `az` en Bash/PowerShell/cmd | Multiplataforma, sintaxis compacta, JMESPath | Scripts en Linux/macOS, pipelines |
| **Azure PowerShell** | Cmdlets `Verb-AzNoun` | Objetos .NET, integración con PowerShell existente | Administradores Windows, scripts complejos |
| **Cloud Shell** | Shell en navegador | Nada que instalar, ya autenticado, herramientas incluidas | Desde cualquier equipo, demostraciones, examen de laboratorio |
| **ARM / Bicep** | Infraestructura como código declarativa | Repetible, idempotente, control de versiones | Entornos completos, despliegues repetidos |
| **REST API / SDK** | Programático | Máximo control | Aplicaciones que gestionan Azure |

## AZ-104 Exam Tips

- ⭐ Todo pasa por **Azure Resource Manager**; si un recurso no aparece como desplegable, suele faltar **registrar el proveedor de recursos**.
- 🧠 CLI = `az`; PowerShell = módulo **Az** (no AzureRM). Ambos están en Cloud Shell.
- 🧠 Cloud Shell clásico necesita **una cuenta de almacenamiento con Azure Files**; el modo efímero no.
- 💻 Debes reconocer, no escribir de memoria, comandos como `az vm create`, `New-AzVM`, `az deployment group create`, `New-AzResourceGroupDeployment`.
- 📌 Si la pregunta pide "una solución que funcione desde macOS y Linux sin instalar nada" → **Cloud Shell**. "Automatizar con objetos .NET en Windows" → **PowerShell**.
- ⚠️ `--output table` y `--query` son de Azure CLI; `Select-Object` y `Where-Object` son de PowerShell. No mezclar.

## Errores comunes

- Confundir plano de control (ARM, roles como Contributor) con plano de datos (Storage Blob Data Contributor, claves, SAS).
- Creer que el portal expone todas las opciones. Algunas (por ejemplo ciertas propiedades de VMSS o de Policy) solo están en CLI/PowerShell/plantillas.
- Olvidar cambiar de contexto de suscripción y desplegar en la equivocada.

## Preguntas que podrían aparecer

**1.** Necesitas ejecutar comandos de Azure PowerShell desde un portátil con macOS sin instalar módulos. ¿Qué usas?
- A) Azure CLI en Homebrew · B) Azure Cloud Shell · C) Windows PowerShell 5.1 · D) Azure Mobile App

<details><summary>Respuesta</summary>

**B.** Cloud Shell trae PowerShell con el módulo Az en el navegador. A es CLI, no PowerShell. C no existe en macOS. D solo ofrece un shell limitado y no evita la necesidad de Cloud Shell.
</details>

**2.** Al intentar crear el primer recurso de tipo `Microsoft.ContainerInstance` en una suscripción nueva, el despliegue falla con "The subscription is not registered to use namespace". ¿Qué debes hacer?
- A) Asignar el rol Owner · B) Registrar el proveedor de recursos · C) Crear un grupo de administración · D) Quitar el bloqueo ReadOnly

<details><summary>Respuesta</summary>

**B.** El error indica que el proveedor de recursos no está registrado en la suscripción. Se resuelve con `az provider register` o desde Suscripción → Proveedores de recursos.
</details>

## Relacionado

- [[01 - Azure Resource Manager y plantillas ARM]]
- [[02 - Bicep]]
- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[15 - Suscripciones y grupos de administración]]
- [[00 - AZ-104 Índice general (MOC)]]
