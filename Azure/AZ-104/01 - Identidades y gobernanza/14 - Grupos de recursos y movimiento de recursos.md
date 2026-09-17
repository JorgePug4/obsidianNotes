---
tags: [az-104, azure, gobernanza, resource-groups, mover-recursos]
modulo: Identidades y gobernanza
peso_examen: Alto
---

# Grupos de recursos y movimiento de recursos

## ¿Qué es?

Un **grupo de recursos (RG)** es el contenedor lógico obligatorio de todo recurso de Azure. Agrupa recursos que comparten **ciclo de vida** (se despliegan, actualizan y eliminan juntos) y es un **ámbito** para RBAC, Policy, locks y etiquetas.

## ¿Para qué sirve?

- Organizar recursos por aplicación, entorno o equipo.
- Borrar todo un entorno de una vez.
- Delegar acceso a un equipo sobre "sus" recursos.
- Aplicar directivas y bloqueos a un conjunto.

## Conceptos clave

- Un recurso pertenece a **exactamente un** RG.
- El RG tiene una **región** (donde se guardan sus metadatos), pero **sus recursos pueden estar en otras regiones**. Si la región del RG no está disponible, no se pueden actualizar los metadatos, pero los recursos siguen funcionando.
- Los RGs **no se anidan**.
- **No se puede renombrar** un RG: hay que crear otro y mover los recursos.
- **Eliminar un RG elimina todos sus recursos** (a menos que haya locks).
- Un recurso puede **interactuar** con recursos de otro RG (por ejemplo, una VM en RG-A puede usar una VNet de RG-B), pero conviene mantener juntos los que dependen entre sí.
- Límite: **980 RGs por suscripción**; 800 recursos por tipo por RG (con excepciones).
- **Deployment stacks** ➕: sustituyen a Blueprints para gestionar recursos como una unidad con deny settings.

## Mover recursos (a otro RG o suscripción)

Reglas 🧠:
- El movimiento **no cambia la región** del recurso (para cambiar región: Azure Resource Mover, Site Recovery o recrear).
- Durante el movimiento, **el RG origen y destino quedan bloqueados** (no se puede crear/modificar/borrar en ellos, pero los recursos siguen funcionando).
- Al mover a **otra suscripción**, ambas deben estar en el **mismo tenant** de Entra ID y el destino debe tener registrado el **proveedor de recursos**.
- **Los recursos dependientes deben moverse juntos**: una VM con sus discos, NIC, IP pública y, según el caso, la VNet. Hay una lista oficial de soporte de movimiento por tipo de recurso ("Move operation support for resources").
- **No se pueden mover** (ejemplos): Azure Backup vault con datos protegidos (hay que detener la protección primero en algunos escenarios), VMs con **Azure Disk Encryption** entre suscripciones sin pasos adicionales, recursos con reservas, App Service Certificates a veces, VMs en un **availability set** deben moverse con todo el conjunto, discos de VMs con CMK requieren que la VM esté desasignada.
- **Las asignaciones de rol hechas en el recurso no se mueven** (ver [[10 - Interpretar asignaciones de acceso]]). Las etiquetas sí.
- Los **locks** en el recurso viajan con él; un lock ReadOnly en origen o destino **impide** el movimiento.
- Antes de mover conviene **validar** (`az resource invoke-action --action validateMoveResources` o la API *validateMoveResources*).
- Rol necesario: `Microsoft.Resources/subscriptions/resourceGroups/moveResources/action` en origen y **write** en destino (Contributor en ambos basta).
- Azure Policy: el movimiento se evalúa contra las directivas del destino.

```bash
az group create --name rg-new --location westeurope
az resource move --destination-group rg-new --ids <id1> <id2>
az resource move --destination-group rg-new --destination-subscription-id <subId> --ids <id>
az group delete --name rg-old --yes --no-wait
```

```powershell
New-AzResourceGroup -Name rg-new -Location westeurope
$res = Get-AzResource -ResourceGroupName rg-old -Name vm01
Move-AzResource -DestinationResourceGroupName rg-new -ResourceId $res.ResourceId
Remove-AzResourceGroup -Name rg-old -Force
```

Portal: RG → Recursos → seleccionar → **Mover** → a otro grupo de recursos / a otra suscripción / a otra región (abre Resource Mover).

## Configuración relevante para el examen

| Acción | Qué saber |
|---|---|
| Crear RG | Nombre único en la suscripción, región (metadatos) |
| Eliminar RG | Borra todo; los locks lo impiden; `az group delete` |
| Exportar plantilla | RG → Exportar plantilla (ver [[03 - Implementar, exportar y convertir plantillas]]) |
| Mover a otro RG | Mismo tenant; bloqueo temporal de ambos RGs; validar |
| Mover a otra suscripción | Mismo tenant; proveedor registrado; cuotas del destino |
| Mover a otra región | **Azure Resource Mover** (no es un "move" de ARM) |
| Listar recursos por RG | `az resource list -g rg-web` |

## Ejemplo

Contoso consolida el proyecto "Web" en la suscripción de producción. Los recursos (VM, NIC, disco, IP pública, NSG) están en `rg-web-dev` de la suscripción Dev. Ambas suscripciones están en el mismo tenant. Se valida el movimiento, se mueven todos juntos a `rg-web-prod`, se reasignan los roles que estaban en los recursos y se vuelve a configurar la Policy del destino. La VNet compartida se queda en su RG original: la VM sigue funcionando porque la NIC solo referencia la subred.

## Comparaciones

| Operación | Uso | Ventaja | Cuándo utilizarla |
|---|---|---|---|
| **Mover a otro RG** | Reorganizar | Sin recrear, sin cambiar IPs | Cambios de organización |
| **Mover a otra suscripción** | Cambiar facturación/propiedad | Sin recrear | Consolidación, separación de entornos |
| **Azure Resource Mover** | Cambiar región | Orquesta dependencias | Cumplimiento de residencia de datos, cercanía |
| **Azure Site Recovery** | Cambiar región de VMs con mínimo downtime | Replicación continua | Migraciones de VMs, DR |
| **Recrear con plantilla** | Cualquier cambio | Limpio | Cuando el movimiento no está soportado |

## AZ-104 Exam Tips

- ⭐ RG = contenedor con **ciclo de vida** común; **no se anida**; **no se renombra**.
- 🔥 🧠 La región del RG es solo **metadatos**; los recursos pueden estar en otras regiones.
- 🔥 🧠 Mover a otra suscripción exige **mismo tenant** y mover **dependencias juntas**.
- 🧠 Durante el movimiento, origen y destino quedan **bloqueados** para escritura.
- 🧠 **980** RGs por suscripción.
- 💻 `az resource move`, `Move-AzResource`, validar antes de mover, exportar plantilla.
- 📌 Mover RG/suscripción (ARM) vs mover región (**Resource Mover**).
- ⚠️ Las asignaciones RBAC del recurso **no** viajan; las etiquetas y locks **sí**.

## Errores comunes

- Intentar mover una VM sin sus discos o NIC.
- Creer que mover cambia la región.
- Olvidar registrar el proveedor de recursos en la suscripción destino.
- Eliminar un RG "vacío" que en realidad contenía recursos ocultos (siempre revisar con "Mostrar tipos ocultos").

## Preguntas que podrían aparecer

**1.** Necesitas mover una máquina virtual con su disco administrado, su NIC y su IP pública desde la suscripción A a la suscripción B. Ambas suscripciones pertenecen a tenants de Microsoft Entra distintos. ¿Qué debes hacer primero?
- A) Nada, mover directamente · B) Transferir una de las suscripciones al mismo tenant · C) Crear un peering · D) Exportar la VM como imagen

<details><summary>Respuesta</summary>

**B.** El movimiento entre suscripciones requiere que ambas estén en el mismo tenant. Después se mueven la VM y todas sus dependencias juntas.
</details>

**2.** Mientras mueves 30 recursos de RG1 a RG2, un compañero intenta crear una cuenta de almacenamiento en RG2 y recibe un error. ¿Por qué?
- A) RG2 tiene un lock permanente · B) Los grupos origen y destino se bloquean durante la operación de movimiento · C) Falta el rol Contributor · D) Los movimientos requieren RG vacío

<details><summary>Respuesta</summary>

**B.** Durante el movimiento ambos RGs quedan bloqueados para operaciones de escritura hasta que termina.
</details>

**3.** ¿Qué afirmación es correcta sobre un grupo de recursos creado en la región West Europe?
- A) Todos sus recursos deben estar en West Europe · B) Solo guarda metadatos en West Europe; los recursos pueden estar en cualquier región · C) Se puede renombrar después · D) Puede contener otros grupos de recursos

<details><summary>Respuesta</summary>

**B.** La región del RG solo determina dónde se almacenan sus metadatos.
</details>

## Relacionado

- [[10 - Interpretar asignaciones de acceso]]
- [[12 - Bloqueos de recursos (Locks)]]
- [[13 - Etiquetas (Tags)]]
- [[08 - Mover una VM (grupo de recursos, suscripción o región)]]
- [[03 - Implementar, exportar y convertir plantillas]]
- [[00 - Índice - Identidades y gobernanza]]
