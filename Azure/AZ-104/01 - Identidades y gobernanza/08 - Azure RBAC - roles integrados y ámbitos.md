---
tags: [az-104, azure, identidad, rbac, gobernanza]
modulo: Identidades y gobernanza
peso_examen: Muy alto
---

# Azure RBAC · roles integrados y ámbitos

## ¿Qué es?

**Azure RBAC** (control de acceso basado en roles) es el sistema de autorización de Azure Resource Manager que define **quién** (entidad de seguridad) puede hacer **qué** (rol) **dónde** (ámbito). En AZ-900 se ve a nivel de concepto ([[Azure RBAC]]); en AZ-104 hay que elegir el rol mínimo correcto, el ámbito adecuado y predecir los permisos resultantes.

## ¿Para qué sirve?

- Aplicar **privilegio mínimo**: cada persona o aplicación tiene exactamente lo que necesita.
- Separar responsabilidades: red, cómputo, almacenamiento, facturación.
- Delegar administración por suscripción o grupo de recursos.

## Conceptos clave

- **Asignación de rol = entidad de seguridad + definición de rol + ámbito**.
- **Entidad de seguridad**: usuario, grupo, entidad de servicio, identidad administrada.
- **Ámbito**: grupo de administración → suscripción → grupo de recursos → recurso. **Herencia hacia abajo.**
- **Permisos acumulativos**: la suma de todas las asignaciones aplicables. RBAC es un modelo de **permitir**; no hay "denegar" en las asignaciones normales.
- **Deny assignments** ➕: existen, pero solo las crean Azure (Blueprints, aplicaciones administradas, deployment stacks); no se crean a mano.
- **Plano de control vs plano de datos**: *Actions* (gestionar recursos) vs *DataActions* (acceder a los datos: leer blobs, enviar mensajes a una cola).
- **Límites** 🧠: **4000 asignaciones por suscripción**, **500 por grupo de administración**, **5000 roles personalizados por tenant**.

## Roles integrados que debes dominar

### Roles generales

| Rol | Permite | No permite |
|---|---|---|
| **Owner** | Todo, incluida la asignación de roles | — |
| **Contributor** | Crear y gestionar todos los recursos | Asignar roles, gestionar bloqueos con alcance (sí puede crear locks en la práctica: `Microsoft.Authorization/locks/*` está incluido) |
| **Reader** | Ver todo | Cambiar nada |
| **User Access Administrator** | Gestionar acceso (asignar roles) | Gestionar recursos |
| **Role Based Access Control Administrator** | Asignar roles (con restricciones vía condiciones) | Gestionar recursos; menos amplio que UAA |

> [!warning] Aclaración sobre Contributor y bloqueos
> Contributor incluye `*` en Actions con NotActions solo para `Microsoft.Authorization/*/Delete`, `Microsoft.Authorization/*/Write`, `Microsoft.Authorization/elevateAccess/Action`, `Microsoft.Blueprint/blueprintAssignments/write|delete`, `Microsoft.Compute/galleries/share/action`, `Microsoft.Purview/consents/write|delete`, `Microsoft.Resources/deploymentStacks/manageDenySetting/action`. Como los locks viven bajo `Microsoft.Authorization/locks`, **Contributor NO puede crear ni borrar bloqueos**; hacen falta **Owner** o **User Access Administrator**.

### Roles específicos frecuentes en el examen

| Rol | Ámbito típico | Qué hace |
|---|---|---|
| **Virtual Machine Contributor** | RG | Gestionar VMs (no la red ni el storage que usan; no puede iniciar sesión en la VM) |
| **Virtual Machine Administrator Login / User Login** | VM | Iniciar sesión en la VM con Entra ID como admin / usuario |
| **Network Contributor** | RG / VNet | Gestionar redes |
| **Storage Account Contributor** | Cuenta | Gestionar la cuenta (plano de control), incluye listar claves |
| **Storage Blob Data Owner / Contributor / Reader** | Cuenta / contenedor | Plano de **datos** de blobs (necesario para acceso con Entra ID) |
| **Storage File Data SMB Share Contributor / Elevated Contributor / Reader** | Recurso compartido | Permisos de nivel de recurso compartido en Azure Files |
| **Backup Contributor / Operator / Reader** | Vault | Gestionar copias de seguridad |
| **Monitoring Contributor / Reader** | Cualquiera | Configurar / leer monitorización |
| **Log Analytics Contributor / Reader** | Workspace | Administrar / consultar logs |
| **Resource Policy Contributor** | MG / Sub | Crear y asignar Azure Policy |
| **Billing Reader / Cost Management Contributor** | Sub | Ver costes / gestionar presupuestos |
| **Key Vault Administrator / Secrets User / Crypto User** | Key Vault | Plano de datos de Key Vault (modelo RBAC) |
| **Security Admin / Security Reader** | Sub | Defender for Cloud |
| **Managed Identity Operator / Contributor** | Identidad | Asignar / crear identidades administradas |
| **Website Contributor** | App Service | Gestionar web apps (no planes) |
| **AcrPull / AcrPush** | ACR | Extraer / subir imágenes |

## Cómo funciona (evaluación de acceso)

```
1. Usuario hace una petición a ARM (por ejemplo, borrar una VM)
2. ARM recoge TODAS las asignaciones de rol del usuario y sus grupos
   en el recurso, el RG, la suscripción y los MGs por encima
3. Une los Actions y resta los NotActions de cada rol (por rol, no entre roles)
4. Comprueba si la acción está permitida por alguno de los roles
5. Comprueba deny assignments (raro)
6. Comprueba bloqueos (locks) y Azure Policy
7. Ejecuta o rechaza
```

## Configuración relevante para el examen

- **Asignar un rol** (portal): recurso/RG/suscripción → **Control de acceso (IAM)** → Agregar asignación de roles → rol → miembros → revisar y asignar.
- **Quién puede asignar roles**: Owner, User Access Administrator (o RBAC Administrator) en ese ámbito o superior.
- **Ver acceso**: IAM → **Comprobar acceso** (Check access) → ver los roles efectivos de un usuario en ese ámbito.
- **Asignar a un grupo** es la práctica recomendada.
- **Condiciones** (ABAC) ➕: se pueden añadir condiciones a asignaciones de roles de datos de storage (por ejemplo, solo blobs con cierta etiqueta).

```bash
az role assignment create --assignee ana@contoso.com --role "Virtual Machine Contributor" --scope /subscriptions/<id>/resourceGroups/rg-web
az role assignment list --assignee ana@contoso.com --all -o table
az role definition list --name "Contributor" --output json
```

```powershell
New-AzRoleAssignment -SignInName ana@contoso.com -RoleDefinitionName "Reader" -ResourceGroupName rg-web
Get-AzRoleAssignment -ResourceGroupName rg-web
Get-AzRoleDefinition "Virtual Machine Contributor" | Select-Object -ExpandProperty Actions
```

## Ejemplo

Un operador debe **reiniciar** VMs en RG-Prod y nada más. Opciones: *Contributor* (excesivo), *Virtual Machine Contributor* (puede también crear/borrar VMs, aún excesivo), un **rol personalizado** con `Microsoft.Compute/virtualMachines/restart/action` y `read` sobre RG-Prod (correcto, ver [[09 - Roles personalizados de Azure RBAC]]).

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Azure RBAC** | Quién puede hacer qué en recursos | Herencia, roles integrados | Siempre para recursos de Azure |
| **Roles de Entra ID** | Administrar el directorio | Granular por tarea | Usuarios, grupos, licencias, dominios |
| **Azure Policy** | Qué se puede crear y cómo | Cumplimiento a escala | Estándares (regiones, SKUs, tags) |
| **Locks** | Evitar borrado/modificación accidental | Se aplica a todos, incluso Owner | Recursos críticos |
| **Claves / SAS de Storage** | Acceso a datos sin identidad | Simple | Cuando no hay Entra ID disponible |

## 💻 Laboratorio: RBAC a distintos ámbitos

1. Crear el grupo "Lab-Readers" con un usuario y asignarle **Reader** en la **suscripción**.
2. Asignar al mismo usuario **Contributor** en el grupo de recursos `rg-lab`.
3. Iniciar sesión con ese usuario: verificar que ve toda la suscripción pero solo puede crear recursos en `rg-lab`.
4. Intentar asignar un rol a otro usuario desde `rg-lab`: debe fallar (Contributor no asigna roles).
5. Usar IAM → Comprobar acceso para ver los roles efectivos y con `az role assignment list --assignee` desde Cloud Shell.

## AZ-104 Exam Tips

- ⭐ **Entidad + Rol + Ámbito**; **herencia hacia abajo**; permisos **acumulativos**.
- 🔥 📌 **Contributor** hace todo menos **asignar roles** y **gestionar bloqueos**.
- 🔥 📌 **User Access Administrator** gestiona acceso, no recursos.
- 🔥 📌 Roles de **datos** de Storage (Blob Data Reader…) son distintos de los roles de la cuenta (Storage Account Contributor). Para leer blobs con Entra ID hace falta un rol de datos.
- 🧠 Límites: 4000 asignaciones por suscripción, 500 por MG, 5000 roles personalizados por tenant.
- 🧠 Virtual Machine Contributor **no** da acceso a iniciar sesión en la VM ni a la VNet.
- 💻 Asignar roles desde IAM y con `az role assignment create` / `New-AzRoleAssignment`; comprobar acceso.
- ⚠️ Un Owner de suscripción no puede borrar un recurso bloqueado hasta quitar el lock. No es un problema de RBAC.
- ⚠️ Los roles de Entra (Global Administrator) no dan permisos en suscripciones, salvo elevación de acceso.

## Errores comunes

- Elegir Owner "por si acaso": el examen siempre penaliza el exceso de privilegio.
- Asignar rol en el recurso cuando el escenario pide "todos los recursos actuales y futuros del RG" (hay que asignar en el RG).
- Pensar que dar *Reader* en la suscripción y *Contributor* en un RG crea un conflicto: se suman.

## Preguntas que podrían aparecer

**1.** Un usuario tiene Reader en la suscripción Sub1 y Contributor en el grupo de recursos RG1. ¿Puede crear una VM en RG1 y asignar el rol Reader a un compañero en RG1?
- A) Sí y sí · B) Sí y no · C) No y no · D) No y sí

<details><summary>Respuesta</summary>

**B.** Contributor en RG1 permite crear la VM. Asignar roles requiere Owner o User Access Administrator, que no tiene.
</details>

**2.** Debes permitir que un equipo administre todas las máquinas virtuales de un grupo de recursos, pero no debe poder modificar las redes virtuales del mismo grupo. ¿Qué rol y ámbito eliges?
- A) Contributor en el RG · B) Virtual Machine Contributor en el RG · C) Owner en cada VM · D) Network Contributor en el RG

<details><summary>Respuesta</summary>

**B.** Virtual Machine Contributor gestiona las VMs sin permitir cambios en VNets. A permitiría tocar la red; C es excesivo y no cubre VMs futuras; D es lo contrario.
</details>

**3.** Una aplicación necesita leer blobs de una cuenta de almacenamiento autenticándose con su identidad administrada. Se le asignó *Storage Account Contributor* pero recibe 403 al leer blobs. ¿Qué falta?
- A) Owner en la cuenta · B) Un rol de plano de datos como Storage Blob Data Reader · C) Una clave de acceso · D) Reader en la suscripción

<details><summary>Respuesta</summary>

**B.** Storage Account Contributor es plano de control. Para operar sobre los datos con Entra ID hace falta un rol de datos (Blob Data Reader/Contributor/Owner).
</details>

## Relacionado

- [[09 - Roles personalizados de Azure RBAC]]
- [[10 - Interpretar asignaciones de acceso]]
- [[12 - Bloqueos de recursos (Locks)]]
- [[05 - Claves de acceso y autorización con Microsoft Entra ID]]
- [[Azure RBAC]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
