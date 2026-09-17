---
tags: [az-104, azure, computo, arm, iac, plantillas]
modulo: Cómputo
peso_examen: Alto
---

# Azure Resource Manager y plantillas ARM

## ¿Qué es?

**Azure Resource Manager (ARM)** es la capa de implementación y administración de Azure. Una **plantilla ARM** es un archivo **JSON declarativo** que describe la infraestructura a desplegar (qué recursos, con qué propiedades y dependencias). ARM la lee, resuelve dependencias y crea los recursos **en paralelo** cuando puede, de forma **idempotente**.

## ¿Para qué sirve?

- Desplegar entornos completos de forma repetible (dev/test/prod idénticos).
- Versionar la infraestructura en Git.
- Integrar en pipelines CI/CD.
- Documentar la infraestructura existente (exportar).

## Conceptos clave

- **Declarativo**: describes el estado final, no los pasos.
- **Idempotente**: repetir el despliegue no duplica recursos.
- **Secciones de una plantilla** 🧠:

| Sección | Obligatoria | Qué contiene |
|---|---|---|
| `$schema` | Sí | URL del esquema (versión del lenguaje) |
| `contentVersion` | Sí | Versión de la plantilla (por ejemplo `1.0.0.0`) |
| `apiProfile` | No | Versión de API por defecto para los recursos |
| `parameters` | No | Valores que se pasan al desplegar (tipo, valores permitidos, valor por defecto, `secureString`) |
| `variables` | No | Valores calculados reutilizables |
| `functions` | No | Funciones definidas por el usuario |
| `resources` | Sí | Los recursos a desplegar (tipo, apiVersion, name, location, properties, dependsOn, tags, sku…) |
| `outputs` | No | Valores devueltos tras el despliegue |

- **Funciones de plantilla**: `parameters()`, `variables()`, `resourceGroup().location`, `resourceId()`, `concat()`, `uniqueString()`, `reference()`, `copyIndex()`, `if()`, `equals()`, `toLower()`, `subscription()`, `deployment()`.
- **`dependsOn`**: orden explícito; `reference()` y `resourceId()` crean dependencias implícitas.
- **`copy`**: crear N instancias de un recurso o propiedad (bucle).
- **Plantillas vinculadas y anidadas (linked / nested)**: modularizar.
- **Archivo de parámetros** (`*.parameters.json`) separado de la plantilla.
- **Modos de implementación** 🧠:
  - **Incremental** (por defecto): añade/actualiza los recursos de la plantilla; **no toca** los que existen en el RG y no están en la plantilla.
  - **Complete**: **elimina** los recursos del RG que **no** estén en la plantilla. Solo para despliegues a nivel de grupo de recursos.
- **Ámbitos de implementación**: grupo de recursos (más común), suscripción, grupo de administración, tenant.
- **Template specs** ➕: almacenar plantillas como recurso de Azure con versiones y RBAC.
- **Deployment stacks** ➕: gestionar un conjunto de recursos como unidad (sustituye a Blueprints), con deny settings.
- **What-if**: previsualizar cambios sin aplicar.
- Historial: RG → **Implementaciones** (deployments) muestra cada despliegue, sus entradas, salidas y plantilla.

## Cómo funciona

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageName": { "type": "string", "minLength": 3, "maxLength": 24 },
    "sku": { "type": "string", "defaultValue": "Standard_LRS", "allowedValues": ["Standard_LRS", "Standard_GRS", "Standard_ZRS"] },
    "location": { "type": "string", "defaultValue": "[resourceGroup().location]" }
  },
  "variables": {
    "accountName": "[toLower(parameters('storageName'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-05-01",
      "name": "[variables('accountName')]",
      "location": "[parameters('location')]",
      "sku": { "name": "[parameters('sku')]" },
      "kind": "StorageV2",
      "properties": { "supportsHttpsTrafficOnly": true, "minimumTlsVersion": "TLS1_2" }
    }
  ],
  "outputs": {
    "blobEndpoint": { "type": "string", "value": "[reference(variables('accountName')).primaryEndpoints.blob]" }
  }
}
```

Puntos para **interpretar** una plantilla en el examen:
- `"[...]"` = expresión evaluada; sin corchetes = literal.
- `parameters('x')` viene del archivo de parámetros o del comando; `variables('y')` se calcula.
- `dependsOn: ["[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"]` = espera a que exista la VNet.
- `copy: { "name": "vmLoop", "count": 3 }` + `copyIndex()` = tres recursos.
- `"condition": "[equals(parameters('env'),'prod')]"` = recurso solo si se cumple.
- Tipo de parámetro `securestring`/`secureObject` para contraseñas (no se registran).

## Modificar una plantilla existente (tareas típicas)

| Necesidad | Cambio |
|---|---|
| Hacer configurable el tamaño de la VM | Añadir parámetro `vmSize` con `allowedValues` y usar `[parameters('vmSize')]` en `hardwareProfile` |
| Añadir un disco de datos | Añadir objeto en `storageProfile.dataDisks` (lun, diskSizeGB, createOption) |
| Añadir una etiqueta a todos los recursos | Propiedad `tags` en cada recurso (o parámetro de objeto) |
| Desplegar 3 NICs iguales | `copy` con `count` y `copyIndex()` en el nombre |
| Evitar que la contraseña aparezca en logs | Tipo `securestring` |
| Recurso que depende de otro | `dependsOn` o usar `reference()`/`resourceId()` |
| Devolver la IP pública creada | Sección `outputs` con `reference(...)` |

## Componentes / configuración relevante para el examen

- Portal: **Implementar una plantilla personalizada** (Crear recurso → buscar "Template deployment") → crear su propia plantilla en el editor / cargar archivo / Quickstart (repositorio de plantillas de inicio rápido de Azure).
- Rol necesario: Contributor en el ámbito (y permisos para cada tipo de recurso). Para desplegar a nivel de suscripción, permisos en la suscripción.
- Errores comunes: `InvalidTemplate` (sintaxis), `ResourceNotFound` (dependencia mal), `InvalidTemplateDeployment` (Policy lo rechaza), cuota excedida.

## Ejemplo

Un equipo despliega semanalmente un entorno de pruebas con una VNet, dos VMs y una cuenta de almacenamiento. Guardan la plantilla en Git con un archivo de parámetros por entorno. Para "limpiar" recursos que ya no están en la plantilla usan modo **Complete** en el RG de pruebas (nunca en producción).

## Comparaciones

| Herramienta | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Plantilla ARM (JSON)** | IaC nativo de Azure | Soporte total, exportable desde el portal | Base de todo; cuando te dan JSON |
| **Bicep** | IaC nativo, sintaxis simple | Menos verboso, tipado, módulos, se compila a ARM | Nuevo código IaC en Azure |
| **Terraform** ➕ | IaC multi-nube | Estado, proveedores | Multi-nube |
| **Azure CLI / PowerShell (imperativo)** | Scripts | Flexibilidad | Tareas puntuales, no estado deseado |
| **Portal** | Manual | Sin código | Prototipos, luego exportar |

## AZ-104 Exam Tips

- ⭐ Secciones: **parameters, variables, resources, outputs** (+ $schema, contentVersion, functions).
- 🔥 🧠 **Incremental** (por defecto, no borra) vs **Complete** (borra lo que no está en la plantilla).
- 🧠 `dependsOn` para el orden; `copy`/`copyIndex()` para bucles; `securestring` para secretos.
- 🧠 `[resourceGroup().location]` = ubicación del RG; `uniqueString(resourceGroup().id)` = nombres únicos.
- 💻 Leer una plantilla y decir qué despliega, qué parámetro cambiar, cómo añadir un recurso.
- 📌 ARM JSON vs Bicep: mismo motor, distinta sintaxis; Bicep se transpila a JSON.
- ⚠️ El modo Complete solo existe para despliegues a **grupo de recursos**.

## Errores comunes

- Confundir parámetros (entrada) con variables (calculadas).
- Olvidar `dependsOn` y obtener errores de "recurso no encontrado".
- Usar modo Complete en producción y borrar recursos no incluidos.

## Preguntas que podrían aparecer

**1.** Despliegas una plantilla ARM en modo Complete en un grupo de recursos que contiene una VM que no aparece en la plantilla. ¿Qué ocurre con la VM?
- A) Se conserva · B) Se elimina · C) Se detiene · D) Se mueve a otro RG

<details><summary>Respuesta</summary>

**B.** El modo Complete elimina los recursos del RG que no están definidos en la plantilla.
</details>

**2.** En una plantilla ARM, ¿qué expresión devuelve la ubicación del grupo de recursos de destino?
- A) `[parameters('location')]` · B) `[resourceGroup().location]` · C) `[variables('location')]` · D) `[deployment().location]`

<details><summary>Respuesta</summary>

**B.** `resourceGroup()` devuelve el objeto del RG de destino, con su propiedad `location`.
</details>

**3.** Necesitas que una plantilla cree cinco cuentas de almacenamiento con nombres `st0` a `st4`. ¿Qué elemento usas?
- A) `dependsOn` · B) `copy` con `copyIndex()` · C) Cinco plantillas vinculadas · D) `outputs`

<details><summary>Respuesta</summary>

**B.** El bucle `copy` con `count: 5` y `copyIndex()` en el nombre genera las cinco instancias.
</details>

## Relacionado

- [[02 - Bicep]]
- [[03 - Implementar, exportar y convertir plantillas]]
- [[02 - Herramientas del administrador (Portal, CLI, PowerShell, Cloud Shell, ARM)]]
- [[14 - Grupos de recursos y movimiento de recursos]]
- [[00 - Índice - Cómputo]]
