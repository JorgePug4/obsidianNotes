---
tags: [az-104, azure, gobernanza, tags]
modulo: Identidades y gobernanza
peso_examen: Medio-Alto
---

# Etiquetas (Tags)

## ¿Qué es?

Una **etiqueta** es un par **nombre: valor** que se adjunta a recursos, grupos de recursos y suscripciones para **organizarlos lógicamente** (propietario, entorno, centro de coste, proyecto). Son metadatos: no cambian el comportamiento del recurso.

## ¿Para qué sirve?

- **Facturación**: Cost Management agrupa y filtra costes por etiqueta.
- **Automatización**: scripts que apagan las VMs con `Env=Dev` por la noche.
- **Inventario y responsabilidad**: saber quién es dueño de cada recurso.
- **Gobernanza**: exigir etiquetas con Azure Policy.

## Conceptos clave

- **Límites** 🧠: máximo **50 etiquetas** por recurso/RG/suscripción; nombre hasta **512 caracteres** (**128 para cuentas de almacenamiento**); valor hasta **256 caracteres**.
- **No se heredan**: las etiquetas del RG o la suscripción **no** pasan automáticamente a los recursos. Para heredarlas → Azure Policy (*Inherit a tag from the resource group*, efecto Modify).
- Nombres **no distinguen mayúsculas**; los valores sí (a efectos de comparación).
- Caracteres no permitidos en el nombre: `< > % & \ ? /`.
- No todos los tipos de recurso admiten etiquetas (por ejemplo, algunos recursos clásicos). Los recursos creados por otros (como los del RG administrado de AKS) pueden no aceptarlas.
- Etiquetas y **facturación**: las etiquetas solo aparecen en el informe de costes de recursos que **generan coste** y que las soportan; hay que estar etiquetando para que aparezcan en el histórico (no se aplican retroactivamente).
- Se pueden aplicar con portal, CLI, PowerShell, ARM/Bicep y **Policy**.
- Al **mover** un recurso, sus etiquetas viajan con él.

## Cómo funciona

```bash
# Añadir etiquetas (sustituye TODAS las existentes)
az resource tag --tags Env=Prod Owner=ana --ids <resourceId>
# Añadir sin borrar las existentes
az tag update --resource-id <resourceId> --operation Merge --tags CostCenter=1234
az group update --name rg-web --tags Env=Prod
# Consultar recursos por etiqueta
az resource list --tag Env=Prod -o table
```

```powershell
Set-AzResourceGroup -Name rg-web -Tag @{Env="Prod"; Owner="ana"}
$r = Get-AzResource -ResourceGroupName rg-web -Name vm01
Update-AzTag -ResourceId $r.ResourceId -Tag @{CostCenter="1234"} -Operation Merge
Get-AzResource -TagName Env -TagValue Prod
```

> [!warning] `az resource tag` y `Set-AzResource -Tag` **reemplazan** el conjunto de etiquetas. Para añadir sin perder las existentes usa `az tag update --operation Merge` / `Update-AzTag -Operation Merge`.

## Componentes / configuración relevante para el examen

| Necesidad | Solución |
|---|---|
| Exigir una etiqueta al crear recursos | Policy *Require a tag and its value on resources* (Deny) |
| Añadir etiqueta si falta | Policy *Add a tag to resources* (Modify) |
| Heredar del RG | Policy *Inherit a tag from the resource group* (Modify + remediación) |
| Exigir etiqueta en RGs | Policy *Require a tag on resource groups* (modo **All**) |
| Ver costes por etiqueta | Cost Management → Análisis de costes → agrupar por etiqueta |
| Etiquetar en masa | Portal → Etiquetas (hoja global) → asignar a varios recursos; scripts |

Rol necesario: quien pueda **escribir** en el recurso (Contributor) o el rol **Tag Contributor** (solo etiquetas, sin modificar el recurso).

## Ejemplo

Finanzas quiere el coste mensual por departamento. Se define la etiqueta `CostCenter` obligatoria mediante Policy en la suscripción (Deny para nuevos recursos, Modify con remediación heredando del RG para los existentes). A partir de ese mes, Cost Management agrupa por `CostCenter`.

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Etiquetas** | Metadatos por recurso | Flexibles, base para costes | Siempre; estándar de nomenclatura + etiquetado |
| **Grupos de recursos** | Agrupar ciclo de vida | Borrado conjunto, RBAC | Recursos que viven y mueren juntos |
| **Grupos de administración** | Agrupar suscripciones | Policy/RBAC a escala | Organización de suscripciones |
| **Convención de nombres** | Identificar por nombre | No cambia | Complementa a las etiquetas (los nombres no se pueden cambiar) |

## AZ-104 Exam Tips

- 🔥 🧠 **50 tags** por recurso; nombre 512 (128 en storage); valor 256.
- 🔥 ⚠️ **No se heredan**; para heredar se usa **Azure Policy (Modify)**.
- 🧠 Rol de solo etiquetas: **Tag Contributor**.
- 💻 `az tag update --operation Merge` y `Update-AzTag -Operation Merge` para no perder etiquetas.
- 📌 Etiquetas para costes vs Policy para exigirlas: van juntas.
- ⚠️ Las etiquetas no son retroactivas en la facturación.

## Errores comunes

- Etiquetar el RG y esperar que las VMs de dentro aparezcan en el informe de costes con esa etiqueta.
- Reemplazar sin querer todas las etiquetas con `az resource tag`.
- Poner en la etiqueta información sensible: las etiquetas se ven en el registro de actividad y en facturación.

## Preguntas que podrían aparecer

**1.** Aplicas la etiqueta `Department=Finance` al grupo de recursos RG-Fin. Una semana después, el informe de costes no muestra ninguna VM con esa etiqueta. ¿Por qué?
- A) Las etiquetas tardan 30 días · B) Las etiquetas de RG no se heredan a los recursos · C) Las VMs no admiten etiquetas · D) Falta el rol Tag Contributor

<details><summary>Respuesta</summary>

**B.** Hay que etiquetar los recursos individualmente o usar una Policy de herencia con remediación.
</details>

**2.** ¿Cuál es el número máximo de etiquetas que puede tener un recurso de Azure?
- A) 15 · B) 25 · C) 50 · D) 100

<details><summary>Respuesta</summary>

**C.** 50 pares nombre-valor por recurso, grupo de recursos o suscripción.
</details>

## Relacionado

- [[11 - Azure Policy]]
- [[14 - Grupos de recursos y movimiento de recursos]]
- [[16 - Administración de costes (presupuestos, alertas y Advisor)]]
- [[00 - Índice - Identidades y gobernanza]]
