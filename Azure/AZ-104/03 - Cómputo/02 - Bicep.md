---
tags: [az-104, azure, computo, bicep, iac]
modulo: Cómputo
peso_examen: Alto
---

# Bicep

## ¿Qué es?

**Bicep** es el lenguaje declarativo de Microsoft para describir infraestructura de Azure con una sintaxis **más simple que JSON**. Un archivo `.bicep` se **transpila** a una plantilla ARM JSON en el momento del despliegue (el motor es el mismo ARM). Soporta todos los tipos de recurso y versiones de API desde el primer día.

## ¿Para qué sirve?

- Escribir y mantener IaC con menos ruido (sin corchetes, sin `dependsOn` manual).
- Modularizar con **módulos** y reutilizar.
- Convertir plantillas ARM existentes (`decompile`).

## Conceptos clave

- **Elementos** 🧠: `targetScope`, `param`, `var`, `resource`, `module`, `output`, `existing`, decoradores (`@description`, `@allowed`, `@secure`, `@minLength`, `@maxLength`, `@minValue`, `@maxValue`).
- **Nombre simbólico**: `resource stg 'Microsoft.Storage/storageAccounts@2023-05-01' = { ... }` → `stg` se usa para referenciar propiedades (`stg.properties.primaryEndpoints.blob`) y crea **dependencias implícitas**.
- **Interpolación de cadenas**: `'${prefix}-vm'` en vez de `concat()`.
- **Bucles**: `[for i in range(0, 3): { ... }]`; **condicionales**: `if (env == 'prod')`.
- **Módulos**: `module net './network.bicep' = { name: 'netDeploy', params: { ... } }`; también desde registro de módulos (ACR) o template specs.
- **Ámbitos**: `targetScope = 'resourceGroup' | 'subscription' | 'managementGroup' | 'tenant'`.
- **Archivo de parámetros**: `.bicepparam` (nativo) o JSON.
- **Bicep CLI**: integrada en Azure CLI (`az bicep install/upgrade/build/decompile`) y disponible para PowerShell. VS Code tiene extensión con IntelliSense.
- `az bicep build` → JSON; `az bicep decompile` → de JSON a Bicep (puede requerir ajustes manuales).
- No hay estado (como Terraform): el estado es Azure.

## Cómo funciona

```bicep
targetScope = 'resourceGroup'

@description('Nombre base de los recursos')
@minLength(3)
@maxLength(10)
param prefix string

@allowed(['Standard_LRS', 'Standard_GRS', 'Standard_ZRS'])
param sku string = 'Standard_LRS'

param location string = resourceGroup().location

@secure()
param adminPassword string

var storageName = toLower('${prefix}${uniqueString(resourceGroup().id)}')

resource stg 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageName
  location: location
  sku: { name: sku }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
  }
}

resource vnet 'Microsoft.Network/virtualNetworks@2024-01-01' = {
  name: '${prefix}-vnet'
  location: location
  properties: {
    addressSpace: { addressPrefixes: ['10.0.0.0/16'] }
    subnets: [for i in range(0, 2): {
      name: 'subnet${i}'
      properties: { addressPrefix: '10.0.${i}.0/24' }
    }]
  }
}

resource existingKv 'Microsoft.KeyVault/vaults@2023-07-01' existing = {
  name: 'kv-shared'
}

module vm './vm.bicep' = if (sku != 'Standard_LRS') {
  name: 'vmDeploy'
  params: {
    name: '${prefix}-vm'
    subnetId: vnet.properties.subnets[0].id   // dependencia implícita
    adminPassword: adminPassword
  }
}

output blobEndpoint string = stg.properties.primaryEndpoints.blob
```

```bash
az bicep install
az bicep build --file main.bicep            # genera main.json
az bicep decompile --file main.json         # genera main.bicep
az deployment group create --resource-group rg-web --template-file main.bicep --parameters prefix=web sku=Standard_ZRS
az deployment group what-if --resource-group rg-web --template-file main.bicep --parameters main.bicepparam
```

```powershell
New-AzResourceGroupDeployment -ResourceGroupName rg-web -TemplateFile main.bicep -prefix web -sku Standard_ZRS
New-AzResourceGroupDeployment -ResourceGroupName rg-web -TemplateFile main.bicep -TemplateParameterFile main.bicepparam -WhatIf
```

## Modificar un archivo Bicep existente (tareas típicas)

| Necesidad | Cambio |
|---|---|
| Parametrizar el tamaño de VM | `param vmSize string = 'Standard_B2s'` y usarlo en `hardwareProfile.vmSize` |
| Restringir valores | `@allowed([...])` |
| Contraseña segura | `@secure() param adminPassword string` |
| Recurso ya existente (no crear) | `resource x '...' existing = { name: '...' }` |
| Desplegar solo en prod | `= if (env == 'prod')` |
| Varias instancias | `[for ... in ...: { }]` |
| Referenciar la subred de otra VNet | `vnet.properties.subnets[0].id` |
| Reutilizar un archivo | `module` |
| Devolver un valor | `output nombre tipo = expresión` |

## Componentes / comparación de sintaxis ARM ↔ Bicep

| Concepto | ARM JSON | Bicep |
|---|---|---|
| Parámetro | `"parameters": { "x": { "type": "string" } }` | `param x string` |
| Variable | `"variables": { "y": "..." }` | `var y = '...'` |
| Recurso | `{ "type": "...", "apiVersion": "...", ... }` | `resource sym 'tipo@apiVersion' = { }` |
| Referencia | `[reference(resourceId(...)).prop]` | `sym.properties.prop` |
| Dependencia | `"dependsOn": [...]` | Implícita por referencia simbólica |
| Concatenar | `[concat(a, '-', b)]` | `'${a}-${b}'` |
| Bucle | `"copy": { "count": n }` + `copyIndex()` | `[for i in range(0, n): { }]` |
| Condición | `"condition": "[...]"` | `= if (...)` |
| Salida | `"outputs": { }` | `output x string = ...` |
| Modularización | Plantillas vinculadas/anidadas | `module` |

## AZ-104 Exam Tips

- ⭐ Bicep = misma capacidad que ARM con **sintaxis simplificada**; se **transpila** a JSON.
- 🔥 🧠 `az bicep build` (Bicep → JSON) y `az bicep decompile` (JSON → Bicep).
- 🧠 `param`, `var`, `resource`, `module`, `output`, `existing`, `@secure()`, `@allowed()`.
- 🧠 Dependencias **implícitas** por nombre simbólico; `dependsOn` solo si es necesario.
- 💻 Desplegar con `az deployment group create --template-file main.bicep` o `New-AzResourceGroupDeployment -TemplateFile main.bicep`.
- 📌 `existing` referencia sin crear; `module` reutiliza.
- ⚠️ El portal **no** acepta Bicep directamente en "Implementar plantilla personalizada": hay que compilar a JSON o usar CLI/PowerShell.

## Errores comunes

- Intentar pegar Bicep en el editor de plantillas del portal.
- Poner `dependsOn` innecesario y crear dependencias circulares.
- Olvidar `@secure()` en contraseñas (quedan en el historial de despliegue).

## Preguntas que podrían aparecer

**1.** Tienes una plantilla ARM en JSON y quieres mantenerla en Bicep. ¿Qué comando usas?
- A) `az bicep build` · B) `az bicep decompile` · C) `az deployment group export` · D) `az bicep publish`

<details><summary>Respuesta</summary>

**B.** `decompile` convierte JSON a Bicep. `build` hace lo contrario.
</details>

**2.** En un archivo Bicep, ¿cómo evitas que una contraseña pasada como parámetro se registre en el historial de implementación?
- A) `param pwd string` · B) `@secure() param pwd string` · C) `var pwd = ''` · D) `output pwd string`

<details><summary>Respuesta</summary>

**B.** El decorador `@secure()` marca el parámetro como seguro (equivale a `securestring`).
</details>

**3.** Un archivo Bicep define `resource vnet ... = { }` y una NIC cuya subred es `vnet.properties.subnets[0].id`. ¿Es necesario añadir `dependsOn` en la NIC?
- A) Sí, siempre · B) No; la referencia simbólica crea la dependencia implícita · C) Solo en modo Complete · D) Solo si se despliega a suscripción

<details><summary>Respuesta</summary>

**B.** Bicep infiere la dependencia al referenciar el recurso por su nombre simbólico.
</details>

## Relacionado

- [[01 - Azure Resource Manager y plantillas ARM]]
- [[03 - Implementar, exportar y convertir plantillas]]
- [[00 - Índice - Cómputo]]
