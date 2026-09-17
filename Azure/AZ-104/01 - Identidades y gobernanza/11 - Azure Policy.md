---
tags: [az-104, azure, gobernanza, policy]
modulo: Identidades y gobernanza
peso_examen: Muy alto
---

# Azure Policy

## ¿Qué es?

**Azure Policy** es el servicio de gobernanza que **evalúa los recursos contra reglas** y actúa: deniega la creación, audita, añade propiedades, modifica o despliega recursos complementarios. Responde a "¿qué se puede crear y cómo debe estar configurado?". En AZ-900 ([[Azure Policy]]) bastaba con el concepto; en AZ-104 hay que elegir el **efecto**, el **ámbito**, entender **iniciativas**, **exenciones** y **remediación**.

## ¿Para qué sirve?

- Limitar regiones permitidas o tamaños de VM.
- Exigir etiquetas (o heredarlas del RG).
- Auditar recursos sin cifrado, sin backup, con IP pública.
- Desplegar automáticamente agentes o configuraciones (DeployIfNotExists).
- Cumplimiento normativo (ISO, NIST) con iniciativas integradas.

## Conceptos clave

- **Definición de directiva (policy definition)**: JSON con condición (`if`) y efecto (`then`). Hay cientos integradas.
- **Iniciativa (initiative / policy set)**: conjunto de definiciones que se asignan juntas (por ejemplo "Microsoft cloud security benchmark").
- **Asignación (assignment)**: aplica una definición o iniciativa a un **ámbito** (MG, suscripción, RG). Puede tener **exclusiones** (ámbitos hijos que quedan fuera) y **parámetros**.
- **Exención (exemption)**: excepción formal con fecha de caducidad y motivo (Waiver / Mitigated) para un ámbito dentro de la asignación.
- **Efectos** 🧠:

| Efecto | Qué hace | Cuándo |
|---|---|---|
| **Deny** | Rechaza la creación/actualización | Impedir regiones/SKU no permitidas |
| **Audit** | Permite pero marca como no conforme (entrada en el registro de actividad) | Ver el impacto antes de denegar |
| **Append** | Añade campos al recurso al crearlo (por ejemplo una tag, una regla de IP) | Añadir propiedades sin bloquear |
| **Modify** | Añade, reemplaza o elimina propiedades/tags, también en existentes vía remediación | Heredar tags del RG |
| **AuditIfNotExists** | Audita si falta un recurso relacionado (por ejemplo VM sin extensión de antimalware) | Detectar carencias |
| **DeployIfNotExists** | Despliega el recurso relacionado si falta (requiere identidad administrada) | Instalar agentes, diagnósticos |
| **DenyAction** ➕ | Bloquea acciones concretas, como DELETE | Evitar borrados |
| **Manual** ➕ | Atestación manual de cumplimiento | Controles no automatizables |
| **Disabled** | No evalúa | Desactivar sin borrar |

- **Orden de evaluación** 🧠: Disabled → Append/Modify → Deny → Audit → (tras crear) AuditIfNotExists / DeployIfNotExists.
- **Cumplimiento**: los recursos existentes **no se eliminan**; se marcan **no conformes**. Deny solo actúa en creaciones/actualizaciones nuevas.
- **Evaluación**: al asignar (~30 min), cada **24 h**, al crear/modificar recursos, y bajo demanda (`az policy state trigger-scan`).
- **Remediación**: tarea que aplica DeployIfNotExists/Modify a recursos **existentes**. Necesita que la asignación tenga una **identidad administrada** con el rol adecuado.
- **Modo de cumplimiento**: `Default` (aplica) o `DoNotEnforce` (solo evalúa, "modo auditoría" de la asignación).
- **Herencia**: la asignación en un MG aplica a todas las suscripciones hijas. Con varias asignaciones, **la más restrictiva gana** (cualquier Deny bloquea).

## Cómo funciona

```
Definición  ──►  (opcional) Iniciativa  ──►  Asignación en un ámbito (+parámetros, +exclusiones, +identidad)
                                                     │
                       ┌─────────────────────────────┴────────────────────────────┐
                Creación/actualización de recurso                     Evaluación periódica (24 h)
                       │                                                            │
                Deny → rechaza                                          Marca conforme / no conforme
                Audit/Append/Modify → permite (y actúa)                 Remediación para existentes
```

Estructura JSON básica:

```json
{
  "properties": {
    "displayName": "Regiones permitidas",
    "mode": "Indexed",
    "parameters": {
      "listOfAllowedLocations": { "type": "Array", "metadata": { "displayName": "Regiones" } }
    },
    "policyRule": {
      "if": {
        "not": { "field": "location", "in": "[parameters('listOfAllowedLocations')]" }
      },
      "then": { "effect": "deny" }
    }
  }
}
```

- `mode`: **Indexed** (solo tipos que soportan tags y location) o **All** (incluye RGs y suscripciones). Para directivas de **tags sobre RGs** usar `All`.

## Componentes

| Componente | Dónde | Nota |
|---|---|---|
| Definiciones | Policy → Definiciones | Integradas o personalizadas; la definición se guarda en un MG o suscripción |
| Iniciativas | Policy → Definiciones (tipo iniciativa) | Agrupan definiciones |
| Asignaciones | Policy → Asignaciones | Ámbito + exclusiones + parámetros + mensaje de no cumplimiento |
| Cumplimiento | Policy → Cumplimiento | % y lista de recursos no conformes |
| Remediación | Policy → Remediación | Tareas para DeployIfNotExists/Modify |
| Exenciones | Policy → Exenciones | Con fecha de expiración |
| Rol mínimo | **Resource Policy Contributor** | Owner también; Contributor **no** puede crear asignaciones |

## Configuración relevante para el examen

- **Directivas integradas típicas**: *Allowed locations*, *Allowed virtual machine size SKUs*, *Require a tag and its value on resources*, *Inherit a tag from the resource group*, *Add a tag to resources*, *Not allowed resource types*, *Audit VMs that do not use managed disks*, *Deploy Log Analytics agent…*.
- **Asignar** (portal): Policy → Asignaciones → Asignar directiva → ámbito → exclusiones → definición → parámetros → remediación (crear identidad administrada) → mensaje de no cumplimiento.
- **Denegar y ya existen recursos**: siguen existiendo y aparecen como **no conformes**; no se pueden modificar de forma que sigan incumpliendo.
- **Policy vs RBAC**: Policy no da ni quita permisos; aunque seas Owner, un Deny te bloquea.

```bash
az policy definition create --name allowed-locations --rules rules.json --params params.json --mode Indexed
az policy assignment create --name "loc-eu" --policy allowed-locations --scope /subscriptions/<id> --params '{"listOfAllowedLocations":{"value":["westeurope","northeurope"]}}'
az policy state list --resource-group rg-web --filter "complianceState eq 'NonCompliant'"
az policy remediation create --name fix-tags --policy-assignment <assignmentId> --resource-group rg-web
```

```powershell
New-AzPolicyAssignment -Name "loc-eu" -PolicyDefinition (Get-AzPolicyDefinition -BuiltIn | ? {$_.Properties.DisplayName -eq "Allowed locations"}) -Scope "/subscriptions/<id>" -PolicyParameterObject @{listOfAllowedLocations=@("westeurope")}
Get-AzPolicyState -ResourceGroupName rg-web -Filter "ComplianceState eq 'NonCompliant'"
Start-AzPolicyRemediation -Name fix-tags -PolicyAssignmentId <id>
```

## Ejemplo

Contoso quiere que **todos** los recursos de la suscripción de producción tengan la etiqueta `CostCenter`, incluidos los ya existentes, y que se herede del grupo de recursos cuando falte. Solución: asignar la directiva integrada *Inherit a tag from the resource group if missing* (efecto **Modify**) con identidad administrada, y crear una **tarea de remediación** para los recursos existentes. Para bloquear creaciones sin etiqueta en RGs nuevos: *Require a tag on resource groups* (Deny, modo All).

## Comparaciones

| Servicio | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Azure Policy** | Qué se puede crear y cómo | Escala, cumplimiento continuo, remediación | Estándares organizativos |
| **Azure RBAC** | Quién puede hacer qué | Identidad y ámbito | Permisos |
| **Locks** | Proteger de borrado/cambio | Simple, absoluto | Recursos críticos |
| **Azure Blueprints** (en retirada) | Empaquetar Policy + RBAC + plantillas | Entornos repetibles | Sustituido por **Template Specs + Deployment Stacks** |
| **Defender for Cloud** | Postura de seguridad | Recomendaciones | Usa Policy por debajo (Microsoft cloud security benchmark) |

| Efecto | Uso | Ventaja | Cuándo utilizarlo |
|---|---|---|---|
| **Deny** | Bloquear | Preventivo | Regiones, SKUs, tipos de recurso |
| **Audit** | Informar | No rompe nada | Fase inicial, ver impacto |
| **Modify** | Corregir tags/propiedades | Actúa también en existentes con remediación | Tags, propiedades simples |
| **DeployIfNotExists** | Desplegar dependencias | Automatiza agentes y diagnósticos | Log Analytics, Defender, diagnósticos |

## 💻 Laboratorio: Allowed locations + tag heredada

1. Policy → Asignaciones → Asignar → ámbito `rg-lab` → definición *Allowed locations* → parámetro `westeurope`.
2. Intentar crear una cuenta de almacenamiento en `eastus` en `rg-lab`: debe fallar con "RequestDisallowedByPolicy".
3. Crear la etiqueta `Env=Lab` en `rg-lab`. Asignar *Inherit a tag from the resource group* (Modify) con identidad administrada asignada por el sistema.
4. Crear una tarea de remediación y comprobar que los recursos existentes reciben la tag.
5. Consultar Policy → Cumplimiento y esperar el ciclo (o forzar con `az policy state trigger-scan`).

## AZ-104 Exam Tips

- ⭐ **Definición → (iniciativa) → asignación en un ámbito**; herencia hacia abajo; el efecto más restrictivo gana.
- 🔥 🧠 **Deny no elimina** recursos existentes: los marca **no conformes**.
- 🔥 📌 **Append vs Modify**: Append solo en creación y no soporta remediación; Modify sí (tags en existentes).
- 🔥 📌 **AuditIfNotExists vs DeployIfNotExists**: informar vs desplegar. DeployIfNotExists y Modify necesitan **identidad administrada** para la remediación.
- 🧠 Rol mínimo: **Resource Policy Contributor**. Contributor no basta.
- 🧠 Evaluación cada **24 h**; tras asignar, ~30 min.
- 🧠 Tags en **grupos de recursos** → modo **All**.
- 💻 Asignar directiva con parámetros, crear exclusión, crear tarea de remediación, ver cumplimiento.
- ⚠️ **Exclusión** (al asignar) vs **exención** (después, con caducidad y motivo).
- ⚠️ Policy no sustituye a RBAC: no da permisos.

## Errores comunes

- Esperar que una Policy Deny borre las VMs no conformes.
- Asignar Modify sin identidad administrada y no entender por qué la remediación falla.
- Usar Policy para impedir borrados cuando lo correcto es un lock (aunque DenyAction también puede).
- Confundir "no conforme" con "bloqueado".

## Preguntas que podrían aparecer

**1.** Asignas una directiva *Allowed locations* = West Europe con efecto Deny a la suscripción. Ya existen 20 VMs en East US. ¿Qué ocurre con ellas?
- A) Se eliminan · B) Se mueven a West Europe · C) Siguen funcionando y aparecen como no conformes · D) Se detienen

<details><summary>Respuesta</summary>

**C.** Policy no toca los recursos existentes; solo los marca como no conformes e impide nuevas creaciones fuera de la región.
</details>

**2.** Necesitas que todos los recursos existentes y futuros de un grupo de recursos tengan la etiqueta `Owner` con el valor de la etiqueta del RG. ¿Qué efecto usas?
- A) Append · B) Deny · C) Modify con tarea de remediación · D) Audit

<details><summary>Respuesta</summary>

**C.** Modify puede añadir tags a recursos existentes mediante remediación; Append solo actúa en creaciones nuevas.
</details>

**3.** Un usuario con Contributor en la suscripción intenta asignar una directiva y no puede. ¿Qué rol mínimo necesita?
- A) Owner · B) Resource Policy Contributor · C) User Access Administrator · D) Security Admin

<details><summary>Respuesta</summary>

**B.** Resource Policy Contributor es el rol integrado mínimo para crear y asignar directivas.
</details>

## Relacionado

- [[12 - Bloqueos de recursos (Locks)]]
- [[13 - Etiquetas (Tags)]]
- [[15 - Suscripciones y grupos de administración]]
- [[18 - Identidades administradas y entidades de servicio]]
- [[Azure Policy]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
