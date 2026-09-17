---
tags: [az-104, azure, identidad, managed-identity, service-principal]
modulo: Identidades y gobernanza
peso_examen: Medio (transversal)
---

# Identidades administradas y entidades de servicio ➕

> [!info] No es un objetivo literal, pero aparece en muchos escenarios: Policy con remediación, cifrado con claves propias (CMK), App Service accediendo a Key Vault, VMs leyendo Storage sin claves. Debes reconocer cuándo se necesita una y cuál elegir.

## ¿Qué es?

- **Entidad de servicio (service principal, SP)**: la identidad de una **aplicación** en un tenant. Se crea al registrar una app y tiene credenciales (secreto o certificado) que hay que custodiar y rotar.
- **Identidad administrada (managed identity)**: una entidad de servicio **gestionada por Azure** para un recurso (VM, App Service, Function, Container App, Policy assignment…). **Sin credenciales que administrar**: Azure las rota.

## ¿Para qué sirve?

- Que un recurso de Azure se autentique en otro servicio (Key Vault, Storage, SQL, ARM) **sin secretos en el código**.
- Que un pipeline o script externo se autentique en Azure (service principal).

## Conceptos clave

- **Identidad administrada asignada por el sistema (system-assigned)**: se crea con el recurso y **muere con él**; un recurso, una identidad.
- **Asignada por el usuario (user-assigned)**: recurso independiente que se **asigna a uno o varios** recursos; sobrevive a ellos; permite preconfigurar permisos antes de crear el recurso.
- Ambas obtienen tokens del **endpoint de metadatos de instancia (IMDS)** (`169.254.169.254`) o del entorno de App Service.
- Se les asignan **roles RBAC** como a cualquier entidad (por ejemplo *Storage Blob Data Reader* en una cuenta) y **access policies / roles** en Key Vault.
- **Service principal**: se crea con `az ad sp create-for-rbac`; genera appId, tenant y secreto. Los secretos caducan (por defecto 1 año en portal, configurable).
- Roles para administrar identidades administradas: **Managed Identity Contributor** (crear/eliminar user-assigned) y **Managed Identity Operator** (asignar a recursos).

## Cómo funciona

```
VM con identidad administrada ──► IMDS (169.254.169.254) ──► token de Entra ID
                                                                   │
                                                                   ▼
                                       Key Vault / Storage / ARM valida el token y aplica RBAC
```

```bash
# System-assigned en una VM
az vm identity assign --name vm01 --resource-group rg-web
# User-assigned
az identity create --name id-web --resource-group rg-web
az vm identity assign --name vm01 --resource-group rg-web --identities /subscriptions/.../userAssignedIdentities/id-web
# Dar permiso a la identidad
az role assignment create --assignee <principalId> --role "Storage Blob Data Reader" --scope <storageAccountId>
# Service principal para automatización
az ad sp create-for-rbac --name sp-pipeline --role Contributor --scopes /subscriptions/<id>/resourceGroups/rg-web
```

## Dónde aparece en el examen

| Escenario | Identidad necesaria |
|---|---|
| Azure Policy con **DeployIfNotExists / Modify** y remediación | Identidad administrada en la asignación |
| Cuenta de almacenamiento o disco con **claves administradas por el cliente (CMK)** | Identidad (system o user-assigned) con acceso a Key Vault |
| App Service leyendo secretos de Key Vault (referencias) | Identidad administrada del App Service |
| VM que ejecuta scripts contra Azure sin credenciales | Identidad administrada de la VM |
| Pipeline de GitHub/Azure DevOps que despliega | Service principal (o federación de identidades de carga de trabajo) |
| Azure Site Recovery / Backup usando Key Vault | Identidad administrada del vault |
| Container Apps / ACI extrayendo imágenes de ACR | Identidad administrada con **AcrPull** |

## Comparaciones

| Tipo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **System-assigned MI** | Un recurso, ciclo de vida acoplado | Cero gestión; se borra con el recurso | Recurso único que accede a otros servicios |
| **User-assigned MI** | Varios recursos comparten identidad | Permisos preconfigurados; sobrevive al recurso | Scale sets, muchas VMs iguales, permisos previos al despliegue |
| **Service principal con secreto/cert** | Apps y herramientas fuera de Azure | Funciona en cualquier sitio | Pipelines, apps on-premises |
| **Workload identity federation** ➕ | Pipelines sin secretos | Sin credenciales almacenadas | GitHub Actions, Kubernetes |

## AZ-104 Exam Tips

- ⭐ "Sin almacenar credenciales" / "sin secretos en el código" → **identidad administrada**.
- 📌 **System-assigned** (muere con el recurso) vs **user-assigned** (independiente, reutilizable).
- 🧠 Policy con remediación, CMK en Storage/discos y Key Vault references necesitan identidad administrada.
- 💻 Habilitar identidad en VM/App Service y asignarle un rol RBAC.
- ⚠️ La identidad administrada solo sirve **desde recursos de Azure**; un script en tu portátil necesita un service principal o tu usuario.

## Preguntas que podrían aparecer

**1.** Una aplicación en App Service debe leer secretos de Azure Key Vault sin almacenar ninguna credencial en la configuración. ¿Qué haces?
- A) Guardar la clave de la cuenta en App Settings · B) Habilitar la identidad administrada del App Service y darle acceso al Key Vault · C) Crear un service principal con secreto · D) Usar una SAS

<details><summary>Respuesta</summary>

**B.** La identidad administrada permite autenticarse en Key Vault sin credenciales; después se concede un rol o access policy.
</details>

**2.** Tienes 50 VMs de un scale set que deben acceder a la misma cuenta de almacenamiento con los mismos permisos, configurados antes de crear las VMs. ¿Qué tipo de identidad es más adecuada?
- A) System-assigned en cada VM · B) User-assigned compartida · C) Service principal · D) Claves de cuenta

<details><summary>Respuesta</summary>

**B.** Una identidad asignada por el usuario se crea una vez, recibe los roles y se asigna a todas las instancias.
</details>

## Relacionado

- [[08 - Azure RBAC - roles integrados y ámbitos]]
- [[11 - Azure Policy]]
- [[06 - Cifrado de cuentas de almacenamiento]]
- [[13 - Azure Container Registry]]
- [[Azure Key Vault]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
