---
tags: [az-104, azure, computo, app-service, webapp]
modulo: Cómputo
peso_examen: Muy alto
---

# Azure App Service · creación y configuración

## ¿Qué es?

**Azure App Service** es la plataforma PaaS para alojar **aplicaciones web, APIs REST y back-ends móviles**, en Windows o Linux, con código (.NET, Java, Node.js, Python, PHP) o **contenedor**. Azure gestiona el sistema operativo, el parcheo, el balanceo y el escalado; tú gestionas la aplicación.

## ¿Para qué sirve?

- Publicar sitios y APIs sin administrar servidores.
- Integrar CI/CD, slots, certificados, autenticación y escalado.

## Conceptos clave

- Se despliega **dentro de un plan** ([[17 - App Service Plan (niveles y escalado)]]).
- **Nombre único global**: `https://<nombre>.azurewebsites.net`.
- **Pila de ejecución (runtime stack)**: versión de .NET, Java, Node, Python, PHP, o **contenedor** (imagen de ACR/Docker Hub) o **código**.
- **Configuración** 🧠:
  - **Configuración de la aplicación (App settings)**: variables de entorno; se pueden marcar como **"configuración de ranura" (slot setting)** para que **no se intercambien** al hacer swap; admiten **referencias a Key Vault** (`@Microsoft.KeyVault(SecretUri=...)`).
  - **Cadenas de conexión**: como app settings pero con tipo (SQLAzure, MySQL…); también pueden ser slot settings.
  - **Configuración general**: versión de pila, plataforma de 32/64 bits, **Always On**, **ARR affinity** (sesiones pegajosas), HTTP/2, protocolo FTP, **versión mínima de TLS**, comando de inicio.
  - **Documentos predeterminados**, **rutas virtuales**, **reglas de reescritura** (Windows, `web.config`).
- **Identidad administrada** (system/user-assigned) para acceder a Key Vault, Storage, SQL sin secretos.
- **Autenticación (Easy Auth)**: proveedores (Entra ID, Google, Facebook, Apple, GitHub) sin escribir código.
- **Implementación**: Centro de implementación (GitHub Actions, Azure Repos, Bitbucket, contenedor, FTP), **ZIP deploy** (`az webapp deploy`), **Run from package**, Git local.
- **Diagnóstico**: registros de aplicación, del servidor web, errores detallados, **Log stream**, **Kudu/SCM** (`https://<app>.scm.azurewebsites.net`), **App Service Diagnostics** (Diagnose and solve problems), **Application Insights**.
- **WebJobs** (Windows): tareas en segundo plano, continuas (requieren Always On) o programadas.
- **Cuotas de la app** y **health check** (ruta que Azure sondea para retirar instancias no saludables).
- **Restricciones de acceso**, **VNet integration** y **private endpoints**: ver [[21 - App Service - redes]].
- **Copias de seguridad**: ver [[20 - App Service - copias de seguridad]].

## Cómo funciona

```bash
# Crear plan y app (código)
az appservice plan create -g rg-web -n plan-web --sku S1
az webapp create -g rg-web -p plan-web -n app-contoso-web --runtime "DOTNETCORE:8.0"
# App en contenedor
az webapp create -g rg-web -p plan-linux -n app-contoso-api --deployment-container-image-name acrcontoso.azurecr.io/api:1.0
# Configuración
az webapp config appsettings set -g rg-web -n app-contoso-web --settings ENV=prod "KV_SECRET=@Microsoft.KeyVault(SecretUri=https://kv.vault.azure.net/secrets/db/)"
az webapp config connection-string set -g rg-web -n app-contoso-web --connection-string-type SQLAzure --settings Db="Server=..."
az webapp config set -g rg-web -n app-contoso-web --always-on true --min-tls-version 1.2 --http20-enabled true
# Identidad administrada
az webapp identity assign -g rg-web -n app-contoso-web
# Despliegue
az webapp deploy -g rg-web -n app-contoso-web --src-path ./app.zip --type zip
# Diagnóstico
az webapp log config -g rg-web -n app-contoso-web --application-logging filesystem --level information
az webapp log tail -g rg-web -n app-contoso-web
az webapp restart -g rg-web -n app-contoso-web
```

```powershell
New-AzWebApp -ResourceGroupName rg-web -Name app-contoso-web -Location westeurope -AppServicePlan plan-web
Set-AzWebApp -ResourceGroupName rg-web -Name app-contoso-web -AppSettings @{"ENV"="prod"}
Restart-AzWebApp -ResourceGroupName rg-web -Name app-contoso-web
```

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| La app "se duerme" y la primera petición tarda | **Always On** (Basic o superior) |
| Secretos fuera del código | **Referencias a Key Vault** + identidad administrada |
| Una app settings no debe cambiar al hacer swap | Marcarla como **slot setting** (configuración de ranura) |
| Sesiones que deben ir siempre a la misma instancia | **ARR affinity** activado (desactivarlo para apps sin estado) |
| Forzar HTTPS | **HTTPS Only** + versión mínima de TLS 1.2 |
| Login con cuentas corporativas sin código | **Authentication (Easy Auth)** con Entra ID |
| Ver por qué falla la app | **Log stream**, **Diagnose and solve problems**, Application Insights |
| Retirar instancias no saludables | **Health check** con una ruta |
| Desplegar desde GitHub en cada push | **Centro de implementación** con GitHub Actions |
| Ejecutar una tarea en segundo plano | **WebJob** continuo con Always On |

## Ejemplo

Una API .NET se publica en `app-contoso-api` sobre un plan S1. Se habilita la identidad administrada, se le da acceso al Key Vault y la cadena de conexión se guarda como referencia a Key Vault. Se activa **Always On**, **HTTPS Only**, TLS 1.2 y un **health check** en `/health`. El despliegue se hace desde GitHub Actions al slot `staging` y luego se intercambia.

## Comparaciones

| Servicio | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **App Service** | Web/API PaaS | Slots, dominios, certificados, auth, backup | Aplicaciones web tradicionales |
| **Container Apps** | Microservicios | Escala a cero, KEDA, revisiones | Microservicios con tráfico variable |
| **Azure Functions** | Funciones | Pago por ejecución | Eventos, tareas cortas |
| **Static Web Apps** ➕ | SPA + API | CDN global, gratis para estáticos | Front-ends estáticos |
| **VM / VMSS** | IaaS | Control del SO | Cuando se necesita el sistema operativo |

## 💻 Laboratorio: Web App

1. Crear `app-<iniciales>-lab` en un plan S1 (Windows, .NET 8) y desplegar el contenido de ejemplo.
2. Añadir una app setting `ENV=lab` y comprobarla desde la app o Kudu.
3. Activar **Always On**, **HTTPS Only** y TLS mínimo 1.2.
4. Habilitar el registro de aplicación y ver **Log stream** mientras se recarga la página.
5. Asignar identidad administrada y crear una referencia a un secreto de Key Vault.

## AZ-104 Exam Tips

- ⭐ La app vive en un **plan**; el plan define nivel, SO, región y escalado.
- 🔥 🧠 **App settings = variables de entorno**; las marcadas como **slot setting no se intercambian** en un swap.
- 🔥 🧠 **Always On** evita la descarga por inactividad (Basic+), necesario para WebJobs continuos.
- 🧠 Nombre **único global** → `<nombre>.azurewebsites.net`.
- 🧠 **Kudu/SCM** para archivos, consola y logs.
- 💻 `az webapp create`, `az webapp config appsettings set`, `az webapp deploy`, `az webapp log tail`.
- 📌 App settings vs cadenas de conexión: ambas son variables; las cadenas tienen tipo.
- ⚠️ Cambiar la mayoría de valores de configuración **reinicia** la aplicación.

## Errores comunes

- Guardar secretos en app settings en texto plano en vez de Key Vault.
- Olvidar marcar como slot setting la cadena de conexión de producción.
- Dejar ARR affinity activado en una app sin estado que escala horizontalmente.

## Preguntas que podrían aparecer

**1.** Necesitas que una configuración de la aplicación permanezca siempre asociada al slot de producción tras un intercambio. ¿Qué haces?
- A) Guardarla en Key Vault · B) Marcarla como configuración de ranura (slot setting) · C) Usar una cadena de conexión · D) Reiniciar la app

<details><summary>Respuesta</summary>

**B.** Las slot settings no se mueven con el swap y permanecen en su ranura.
</details>

**2.** Una aplicación en un plan Standard tarda varios segundos en responder tras periodos de inactividad. ¿Qué configuras?
- A) ARR affinity · B) Always On · C) HTTP/2 · D) Health check

<details><summary>Respuesta</summary>

**B.** Always On mantiene la aplicación cargada y evita el arranque en frío.
</details>

**3.** ¿Cómo accede una Web App a un secreto de Key Vault sin almacenar credenciales?
- A) Clave de acceso en app settings · B) Identidad administrada con acceso al Key Vault y referencia `@Microsoft.KeyVault(...)` · C) SAS · D) Certificado autofirmado

<details><summary>Respuesta</summary>

**B.** La identidad administrada más la referencia a Key Vault evitan almacenar secretos.
</details>

## Relacionado

- [[17 - App Service Plan (niveles y escalado)]]
- [[19 - App Service - certificados, TLS y dominios personalizados]]
- [[21 - App Service - redes]]
- [[22 - App Service - ranuras de implementación (deployment slots)]]
- [[18 - Identidades administradas y entidades de servicio]]
- [[00 - Índice - Cómputo]]
