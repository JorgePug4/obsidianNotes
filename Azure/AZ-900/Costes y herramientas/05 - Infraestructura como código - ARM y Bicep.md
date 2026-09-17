---
tags: [az-900, azure, herramientas, iac, arm, bicep, plantillas]
modulo: Costes y herramientas
peso_examen: Medio
---

# Infraestructura como código: plantillas ARM y Bicep

## Concepto

**Infraestructura como código** (IaC) es la práctica de **describir la infraestructura en archivos de texto** (plantillas) que se guardan en control de versiones y se despliegan de forma **automática y repetible**, en lugar de crear recursos a mano en el portal.

En Azure, las plantillas nativas son las **plantillas de Azure Resource Manager (ARM)**, escritas en **JSON**, y **Bicep**, un lenguaje más legible que se compila a ARM. Ambas las procesa [[02 - Recursos, grupos de recursos y suscripciones|Azure Resource Manager]].

**Problema que resuelve:** crear a mano un entorno con red, subredes, VMs, base de datos y reglas de seguridad lleva horas y cada vez sale ligeramente distinto. Con IaC, el entorno de pruebas y el de producción se crean desde el mismo archivo y son idénticos.

**Para qué se utiliza:**
- Desplegar entornos completos de forma consistente (dev, test, prod).
- Recrear la infraestructura tras un desastre.
- Revisar cambios de infraestructura como se revisa código (pull requests).
- Integrar el despliegue en pipelines de CI/CD.

## Características principales

### Declarativo vs imperativo

| Enfoque | Cómo funciona | Ejemplo |
|---|---|---|
| **Imperativo** | Dices **cómo** hacerlo, paso a paso | Script de CLI o PowerShell: "crea el grupo, luego la VNet, luego la VM" |
| **Declarativo** | Dices **qué** quieres al final; la plataforma calcula los pasos | Plantilla ARM / Bicep: "quiero este grupo con esta VNet y esta VM" |

Las plantillas ARM y Bicep son **declarativas**.

### Plantillas ARM (Azure Resource Manager templates)
- Archivo **JSON** que describe los recursos, sus propiedades y dependencias.
- **Idempotentes**: puedes desplegar la misma plantilla muchas veces y el resultado es el mismo; si el recurso ya existe con esa configuración, no cambia nada.
- Azure **orquesta** el orden y crea en **paralelo** lo que no depende entre sí.
- Admiten **parámetros** (valores que cambian entre entornos), **variables**, **salidas** y **plantillas vinculadas** (modularidad).
- Se **validan** antes de ejecutar: si hay un error, no se crea nada.
- Se pueden **exportar** desde recursos existentes en el portal.
- Se despliegan desde el portal, la CLI (`az deployment group create`), PowerShell (`New-AzResourceGroupDeployment`) o pipelines.

### Bicep
- **Lenguaje específico de dominio** (DSL) de Microsoft para escribir plantillas ARM con una sintaxis **más simple y corta** que el JSON.
- Se **transpila** a JSON de ARM automáticamente; tiene las mismas capacidades y soporte de día uno para todos los servicios.
- Mejor legibilidad, modularidad y herramientas (autocompletado en VS Code).
- Es la opción **recomendada** por Microsoft para nuevas plantillas.

```bicep
resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'stdemo001'
  location: 'westeurope'
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}
```

### Herramientas de terceros
- **Terraform** (HashiCorp): IaC declarativo **multinube**; usa su propio lenguaje (HCL). Disponible en Cloud Shell.
- **Ansible, Chef, Puppet**: más orientados a configurar el software dentro de las máquinas.
- El examen solo exige conocer **ARM y Bicep** como opciones nativas; Terraform puede aparecer como distractor.

### Relación con otras herramientas

| Herramienta | Enfoque | Cuándo |
|---|---|---|
| Portal | Manual | Explorar, tareas puntuales |
| CLI / PowerShell | Imperativo (scripts) | Automatizar acciones concretas |
| ARM / Bicep | Declarativo (plantillas) | Desplegar entornos completos, repetibles |
| [[Azure Blueprints]] (retirado) | Paquete de plantillas + Policy + RBAC | Ya no entra en el examen |

## Casos de uso

- Crear el entorno de pruebas cada mañana y destruirlo cada noche con el mismo resultado: **plantilla Bicep** en un pipeline.
- Auditar quién cambió qué en la infraestructura: plantillas en **Git** con historial.
- Estandarizar la creación de una VNet segura para todos los equipos: plantilla con **parámetros**.

## Comparaciones

| Concepto | Qué es | No confundir con |
|---|---|---|
| Plantilla ARM | Plantilla JSON declarativa nativa | Script de CLI (imperativo) |
| Bicep | Sintaxis simplificada que compila a ARM | Un servicio distinto (es el mismo motor) |
| Declarativo | Describes el estado final | Imperativo (describes los pasos) |
| Idempotente | Repetir da el mismo resultado | Script que falla si el recurso ya existe |
| Terraform | IaC de terceros multinube | Opción nativa de Azure |

## Conceptos que debo memorizar

> [!important]
> - **IaC** = infraestructura definida en archivos, versionada y desplegada automáticamente.
> - **Plantillas ARM** = **JSON**, **declarativas**, **idempotentes**, con parámetros; Azure orquesta dependencias y paraleliza.
> - **Bicep** = sintaxis más simple que se compila a ARM; recomendado.
> - **Declarativo** (qué) vs **imperativo** (cómo): plantillas vs scripts.
> - **Terraform** = alternativa de terceros, multinube.

## Tips para AZ-900

> [!tip]
> - "Desplegar la misma infraestructura de forma **repetible y consistente**" → **plantillas ARM / Bicep**.
> - "Archivo **JSON** que define recursos" → **plantilla ARM**.
> - "Sintaxis más sencilla que JSON para plantillas de Azure" → **Bicep**.
> - "Describir el estado final en lugar de los pasos" → **declarativo**.
> - Trampa: "Las plantillas ARM se ejecutan secuencialmente recurso a recurso". **Falso**, paralelizan lo independiente.
> - Trampa: "Bicep es un servicio de Azure diferente de ARM". **Falso**, se compila a ARM.
> - Trampa: "Un script de PowerShell es infraestructura declarativa". **Falso**, es imperativo.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu equipo necesita desplegar el mismo entorno (red, VMs y base de datos) en tres suscripciones distintas, garantizando que sea idéntico en todas. ¿Qué debes usar?

- A) El portal de Azure
- B) Una plantilla ARM o Bicep
- C) La app móvil de Azure
- D) Azure Advisor

**Respuesta: B.** Las plantillas declarativas garantizan despliegues repetibles e idénticos.
- A) Manual y propenso a diferencias.
- C) Solo supervisión y acciones simples.
- D) Recomienda mejoras; no despliega.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre las plantillas de Azure Resource Manager es correcta?

- A) Se escriben en YAML.
- B) Son imperativas: describen los pasos a ejecutar.
- C) Son declarativas y se pueden desplegar varias veces con el mismo resultado.
- D) Solo pueden desplegarse desde el portal.

**Respuesta: C.** Las plantillas ARM son declarativas e idempotentes.
- A) Son JSON (o Bicep, que compila a JSON).
- B) Describen el estado final, no los pasos.
- D) Se despliegan desde portal, CLI, PowerShell y pipelines.

## 🧠 Resumen para el examen

1. IaC = infraestructura en archivos de texto, versionada y desplegada automáticamente.
2. Plantillas ARM = JSON declarativo, idempotente, con parámetros; Azure orquesta y paraleliza.
3. Bicep = sintaxis simple que se compila a ARM; recomendado por Microsoft.
4. Declarativo (plantillas) vs imperativo (scripts CLI/PowerShell).
5. Terraform = alternativa de terceros multinube.

---
