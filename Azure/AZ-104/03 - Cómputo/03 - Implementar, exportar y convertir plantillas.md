---
tags: [az-104, azure, computo, arm, bicep, deployment]
modulo: Cómputo
peso_examen: Alto
---

# Implementar, exportar y convertir plantillas

## ¿Qué es?

Las operaciones prácticas de infraestructura como código: **desplegar** una plantilla ARM o Bicep en un ámbito, **exportar** una implementación o un grupo de recursos existente como plantilla ARM, y **convertir** ARM JSON a Bicep.

## ¿Para qué sirve?

- Reproducir un entorno creado a mano en el portal (exportar → parametrizar → desplegar).
- Documentar la infraestructura actual.
- Migrar plantillas antiguas a Bicep.

## Conceptos clave

### Desplegar
- **Ámbitos**: RG (`az deployment group`), suscripción (`az deployment sub`), MG (`az deployment mg`), tenant (`az deployment tenant`). PowerShell: `New-AzResourceGroupDeployment`, `New-AzSubscriptionDeployment` (alias `New-AzDeployment`), `New-AzManagementGroupDeployment`, `New-AzTenantDeployment`.
- **Origen de la plantilla**: archivo local (`--template-file`), URI (`--template-uri`, por ejemplo un blob con SAS o GitHub raw), **template spec** (`--template-spec`).
- **Parámetros**: en línea (`--parameters a=1 b=2`), archivo (`--parameters @params.json` o `.bicepparam`), o combinados (los en línea sobrescriben).
- **Modo**: `--mode Incremental|Complete` (solo RG).
- **What-if**: `az deployment group what-if` / `-WhatIf` para ver qué se crearía, modificaría o eliminaría.
- **Validación**: `az deployment group validate`.
- **Nombre del despliegue**: `--name`; si se omite, se usa el nombre del archivo. Los despliegues se guardan en el historial del RG (máximo **800** por RG; ARM borra automáticamente los más antiguos).
- Portal: **Implementar una plantilla personalizada** → "Compilar su propia plantilla en el editor" (JSON) o cargar archivo → parámetros → Revisar y crear. También desde **Quickstart templates** y desde cualquier hoja "Revisar y crear" → **Descargar una plantilla para la automatización** antes de crear.

### Exportar
- **Desde un grupo de recursos**: RG → **Exportar plantilla** → genera JSON con el estado actual de todos (o algunos) recursos. ⚠️ Algunos tipos no se exportan, los secretos no se incluyen, los valores están "hardcoded" (hay que **parametrizar**), y puede incluir propiedades de solo lectura que hay que limpiar.
- **Desde una implementación**: RG → Implementaciones → seleccionar → **Plantilla** / **Descargar**: es la plantilla **exacta** que se usó, con sus parámetros. Preferible si existe.
- **Desde un recurso**: recurso → Exportar plantilla (solo ese recurso y dependencias).
- **Antes de crear**: en el asistente, "Descargar una plantilla para la automatización".
- CLI: `az group export --name rg-web > template.json`; PowerShell: `Export-AzResourceGroup -ResourceGroupName rg-web`.
- Desde una implementación: `az deployment group export --resource-group rg-web --name <deployment>` / `Save-AzResourceGroupDeploymentTemplate`.

### Convertir
- **ARM → Bicep**: `az bicep decompile --file template.json` (o en VS Code: "Decompile into Bicep"). Puede requerir correcciones manuales (nombres simbólicos genéricos, advertencias).
- **Bicep → ARM**: `az bicep build --file main.bicep`.
- Portal → Exportar plantilla → pestaña **Bicep** ➕: algunos ámbitos ya permiten descargar directamente en Bicep.

## Cómo funciona

```bash
# Desplegar a RG con archivo de parámetros y what-if previo
az deployment group what-if --resource-group rg-web --template-file main.bicep --parameters @main.bicepparam
az deployment group create --resource-group rg-web --name deploy-web-01 --template-file main.bicep --parameters @main.bicepparam --mode Incremental
# Desplegar desde URI
az deployment group create --resource-group rg-web --template-uri "https://raw.githubusercontent.com/org/repo/main/azuredeploy.json" --parameters vmName=vm01
# Desplegar a nivel de suscripción (crea el RG)
az deployment sub create --location westeurope --template-file sub.bicep
# Exportar
az group export --name rg-web --resource-ids <id1> <id2> > exported.json
az deployment group export --resource-group rg-web --name deploy-web-01 > used-template.json
# Convertir
az bicep decompile --file exported.json
# Historial
az deployment group list --resource-group rg-web -o table
az deployment group show --resource-group rg-web --name deploy-web-01 --query properties.outputs
```

```powershell
New-AzResourceGroupDeployment -ResourceGroupName rg-web -Name deploy-web-01 -TemplateFile main.bicep -TemplateParameterFile main.bicepparam -Mode Incremental -WhatIf
New-AzResourceGroupDeployment -ResourceGroupName rg-web -TemplateUri "https://.../azuredeploy.json" -vmName vm01
New-AzSubscriptionDeployment -Location westeurope -TemplateFile sub.bicep
Export-AzResourceGroup -ResourceGroupName rg-web -Path .\exported.json
Save-AzResourceGroupDeploymentTemplate -ResourceGroupName rg-web -DeploymentName deploy-web-01
Get-AzResourceGroupDeployment -ResourceGroupName rg-web
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Reproducir en otro RG una VM creada desde el portal | Exportar plantilla del RG (o de la implementación) → parametrizar → desplegar |
| Ver qué cambiará un despliegue sin aplicarlo | `what-if` |
| Desplegar Bicep desde el portal | Compilar a JSON (`az bicep build`) y usar "Implementar una plantilla personalizada" |
| Plantilla con secretos | Parámetros `securestring` + referencia a Key Vault en el archivo de parámetros |
| Compartir una plantilla estándar con RBAC y versiones | **Template spec** |
| Plantilla exportada falla al desplegar | Limpiar propiedades de solo lectura, parametrizar nombres únicos, añadir secretos |
| Crear el RG y sus recursos en un solo despliegue | Despliegue a nivel de **suscripción** |
| Reintentar un despliegue fallido | Corregir y volver a ejecutar (idempotente); revisar "Detalles de la operación" |

## Ejemplo

Un administrador creó a mano en `rg-demo` una VNet, una VM y una cuenta de almacenamiento. Para replicarlo en tres regiones: exporta la plantilla del RG, convierte a Bicep con `decompile`, sustituye nombres fijos por `param` y `uniqueString`, añade `@secure()` a la contraseña, y despliega con `az deployment group create` tres veces con distintos parámetros de ubicación.

## Comparaciones

| Método de exportación | Uso | Ventajas | Limitaciones |
|---|---|---|---|
| **Exportar desde RG** | Recursos creados a mano | Captura el estado actual | Sin secretos, valores fijos, algunos tipos no soportados |
| **Exportar desde implementación** | Recursos creados con plantilla | Plantilla exacta con parámetros | Solo refleja ese despliegue |
| **Descargar antes de crear** | Asistente del portal | Limpia y lista para usar | Solo el recurso del asistente |
| **Template spec** | Compartir plantillas | Versiones, RBAC | Hay que publicarla |

## 💻 Laboratorio: exportar, convertir y redesplegar

1. Crear una cuenta de almacenamiento desde el portal; antes de "Crear", descargar la plantilla para la automatización.
2. RG → Exportar plantilla → descargar. Comparar ambos JSON.
3. `az bicep decompile --file template.json` y revisar el `.bicep` generado: parametrizar nombre y ubicación, añadir `@allowed` al SKU.
4. `az deployment group what-if` en un RG nuevo; luego `create`.
5. Ver el despliegue en RG → Implementaciones y consultar las salidas.

## AZ-104 Exam Tips

- 🔥 🧠 `az deployment group create` / `New-AzResourceGroupDeployment` con `--template-file` (local) o `--template-uri`.
- 🔥 🧠 Exportar: **RG → Exportar plantilla** (estado actual) o **Implementaciones → Plantilla** (la usada).
- 🧠 `az bicep decompile` (JSON → Bicep); `az bicep build` (Bicep → JSON).
- 🧠 Despliegue a suscripción (`az deployment sub create`) puede crear RGs.
- 💻 Usar el portal "Implementar una plantilla personalizada" y "Descargar una plantilla para la automatización".
- ⚠️ Las plantillas exportadas **no incluyen secretos** y necesitan parametrización.
- ⚠️ El portal solo acepta JSON en el editor de plantillas.

## Errores comunes

- Desplegar la plantilla exportada tal cual y fallar por nombres ya existentes o propiedades de solo lectura.
- Confundir `az deployment group` (RG) con `az deployment sub` (suscripción).
- Buscar la contraseña de administrador en la plantilla exportada.

## Preguntas que podrían aparecer

**1.** Creaste una VM desde el portal y necesitas la plantilla exacta con los parámetros que se usaron para desplegarla de nuevo en otro grupo de recursos. ¿Dónde la obtienes?
- A) Exportar plantilla del RG · B) Implementaciones del RG → seleccionar la implementación → Plantilla · C) Azure Advisor · D) Registro de actividad

<details><summary>Respuesta</summary>

**B.** El historial de implementaciones conserva la plantilla y los parámetros usados. Exportar desde el RG genera una plantilla del estado actual, no la original.
</details>

**2.** ¿Qué comando de Azure CLI despliega un archivo Bicep en un grupo de recursos?
- A) `az bicep build` · B) `az deployment group create --template-file main.bicep` · C) `az group deployment export` · D) `az resource create`

<details><summary>Respuesta</summary>

**B.** `az deployment group create` acepta archivos Bicep directamente y los compila.
</details>

## Relacionado

- [[01 - Azure Resource Manager y plantillas ARM]]
- [[02 - Bicep]]
- [[14 - Grupos de recursos y movimiento de recursos]]
- [[04 - Máquinas virtuales - creación y configuración]]
- [[00 - Índice - Cómputo]]
