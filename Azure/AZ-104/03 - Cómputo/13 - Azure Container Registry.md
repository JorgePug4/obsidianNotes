---
tags: [az-104, azure, computo, contenedores, acr]
modulo: Cómputo
peso_examen: Alto
---

# Azure Container Registry (ACR)

## ¿Qué es?

**Azure Container Registry** es un registro **privado** de imágenes de contenedor (compatible con Docker Registry v2 y OCI) y otros artefactos (gráficos de Helm, imágenes OCI). Es el origen desde el que ACI, Container Apps, AKS y App Service descargan imágenes.

## ¿Para qué sirve?

- Guardar imágenes propias con control de acceso.
- Construir imágenes en la nube (ACR Tasks).
- Replicar imágenes cerca de donde se ejecutan (geo-replicación).

## Conceptos clave

### SKUs 🧠

| SKU | Almacenamiento incluido | Webhooks | Geo-replicación | Private Link / firewall | Content trust | Cuándo |
|---|---|---|---|---|---|---|
| **Basic** | 10 GiB | 2 | No | No | No | Desarrollo, pruebas |
| **Standard** | 100 GiB | 10 | No | No | No | Producción normal |
| **Premium** | 500 GiB | 500 | **Sí** | **Sí** | **Sí** | Multi-región, requisitos de red y confianza |

- El SKU se puede **cambiar** en caliente (upgrade/downgrade).
- **Nombre**: 5-50 caracteres alfanuméricos, **único global**; el endpoint es `<nombre>.azurecr.io`.

### Autenticación y autorización 🧠

| Método | Uso | Nota |
|---|---|---|
| **Identidad de Entra ID** (`az acr login`) | Personas y automatización | Recomendado; usa RBAC |
| **Identidad administrada** | VMs, ACI, Container Apps, AKS | Sin secretos; rol **AcrPull** |
| **Service principal** | CI/CD externo | appId + secreto |
| **Usuario administrador (admin user)** | Compatibilidad | **Deshabilitado por defecto**; una sola cuenta compartida; no recomendado |
| **Tokens con ámbito (scope maps)** ➕ | Acceso por repositorio | Premium |

Roles clave: **AcrPull** (descargar), **AcrPush** (subir + descargar), **AcrDelete**, **AcrImageSigner**, Owner/Contributor (incluyen gestión del recurso).

### Otras características
- **ACR Tasks**: `az acr build` compila la imagen en Azure (sin Docker local); tareas automáticas al hacer commit o cuando cambia la imagen base (`az acr task create`).
- **Geo-replicación** (Premium): un mismo nombre/endpoint sirve la imagen desde la región más cercana.
- **Retención y limpieza**: directivas de retención de manifiestos sin etiquetar (Premium), `az acr repository delete`.
- **Importar imágenes**: `az acr import` copia imágenes de Docker Hub u otro ACR sin descargarlas localmente.
- **Zone redundancy** (Premium) y **cifrado con CMK** (Premium).
- **Webhooks**: notificar a un endpoint cuando se sube una imagen (por ejemplo, para desplegar).

## Cómo funciona

```bash
az acr create --resource-group rg-apps --name acrcontoso --sku Standard --admin-enabled false
az acr login --name acrcontoso                      # usa tu identidad de Entra ID
docker tag miapp:1.0 acrcontoso.azurecr.io/miapp:1.0
docker push acrcontoso.azurecr.io/miapp:1.0
az acr repository list --name acrcontoso -o table
az acr repository show-tags --name acrcontoso --repository miapp -o table
# Construir en la nube (sin Docker local)
az acr build --registry acrcontoso --image miapp:2.0 .
# Importar desde Docker Hub
az acr import --name acrcontoso --source docker.io/library/nginx:latest --image nginx:latest
# Dar permiso a una identidad administrada
az role assignment create --assignee <principalId> --role AcrPull --scope $(az acr show -n acrcontoso --query id -o tsv)
# Geo-replicación (Premium)
az acr replication create --registry acrcontoso --location northeurope
```

```powershell
New-AzContainerRegistry -ResourceGroupName rg-apps -Name acrcontoso -Sku Standard
Connect-AzContainerRegistry -Name acrcontoso
Get-AzContainerRegistryRepository -RegistryName acrcontoso
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Imágenes accesibles con baja latencia en 3 regiones | SKU **Premium** + geo-replicación |
| Acceso solo desde una VNet | SKU **Premium** + **private endpoint** / firewall |
| ACI o Container Apps deben descargar imágenes sin credenciales | **Identidad administrada** con rol **AcrPull** |
| Pipeline de CI/CD que sube imágenes | Service principal con **AcrPush** |
| Compilar una imagen sin Docker instalado | **`az acr build`** (ACR Tasks) |
| Evitar cuentas compartidas | Mantener el **usuario administrador deshabilitado** |
| Reducir almacenamiento | Directiva de retención + borrar manifiestos sin etiqueta |
| Copiar una imagen pública al registro privado | **`az acr import`** |

## Ejemplo

Contoso publica su API en contenedores. Crean `acrcontoso` (Premium, por geo-replicación a North Europe y East US y private endpoint). El pipeline usa un service principal con **AcrPush** para `az acr build`. Las Container Apps de cada región usan una **identidad administrada** con **AcrPull**. El usuario administrador permanece deshabilitado.

## Comparaciones

| Registro | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **ACR** | Registro privado en Azure | RBAC de Entra, private link, geo-replicación, tasks | Cargas en Azure |
| **Docker Hub** | Registro público | Imágenes públicas | Imágenes base |
| **GitHub Container Registry** ➕ | Registro ligado a repos | Integración con Actions | Proyectos en GitHub |
| **Registro en AKS (interno)** | — | — | No recomendado; usar ACR |

## 💻 Laboratorio: ACR

1. Crear `acr<iniciales>lab` (Basic, admin deshabilitado).
2. `az acr build --registry <acr> --image web:1.0 .` con un Dockerfile sencillo (nginx + index.html).
3. Listar repositorios y etiquetas.
4. Crear una identidad administrada, asignarle **AcrPull** sobre el registro y usarla desde ACI ([[14 - Azure Container Instances]]).
5. Importar `nginx:latest` desde Docker Hub con `az acr import`.

## AZ-104 Exam Tips

- 🔥 🧠 **Geo-replicación, private link y content trust = solo Premium.**
- 🔥 🧠 Rol **AcrPull** para descargar, **AcrPush** para subir.
- 🧠 El **usuario administrador está deshabilitado por defecto** y no es la respuesta recomendada.
- 🧠 Endpoint `<nombre>.azurecr.io`; nombre único global.
- 💻 `az acr create/login/build/import/repository list`, asignar AcrPull a identidades administradas.
- 📌 ACR (registro, almacena) vs ACI/Container Apps (ejecutan).

## Errores comunes

- Habilitar el usuario administrador en producción en lugar de usar identidades.
- Elegir Basic y luego necesitar geo-replicación.
- Dar Contributor cuando basta con AcrPull.

## Preguntas que podrían aparecer

**1.** Una Azure Container Instance no puede descargar una imagen de tu ACR y no quieres usar contraseñas. ¿Qué configuras?
- A) Habilitar el usuario administrador · B) Una identidad administrada con el rol AcrPull · C) Un service principal con Owner · D) Acceso anónimo

<details><summary>Respuesta</summary>

**B.** La identidad administrada con AcrPull permite la descarga sin credenciales almacenadas.
</details>

**2.** Necesitas que las imágenes de tu registro se sirvan desde tres regiones con el mismo nombre de host. ¿Qué SKU necesitas?
- A) Basic · B) Standard · C) Premium · D) Cualquiera

<details><summary>Respuesta</summary>

**C.** La geo-replicación es exclusiva del SKU Premium.
</details>

## Relacionado

- [[14 - Azure Container Instances]]
- [[15 - Azure Container Apps]]
- [[16 - Comparación de servicios de contenedores]]
- [[18 - Identidades administradas y entidades de servicio]]
- [[00 - Índice - Cómputo]]
