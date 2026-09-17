---
tags: [az-900, azure, herramientas, portal, cloud-shell, cli, powershell]
modulo: Costes y herramientas
peso_examen: Medio
---

# Portal, Cloud Shell, CLI, PowerShell y app móvil

## Concepto

Todas las operaciones sobre Azure pasan por **Azure Resource Manager (ARM)**, y hay varias **herramientas** para hablar con él. Las gráficas son el **portal de Azure** y la **app móvil**; las de línea de comandos son la **CLI de Azure** y **Azure PowerShell**; y **Azure Cloud Shell** es un terminal en el navegador que ya trae las dos instaladas.

**Problema que resuelve:** el portal es cómodo para explorar y para tareas puntuales, pero repetir 50 veces la misma acción a mano es lento y propenso a errores. Las herramientas de línea de comandos permiten **automatizar** y **repetir** con exactitud.

**Para qué se utiliza:** administrar recursos, automatizar tareas, consultar estado y desplegar soluciones, desde la herramienta que mejor encaje con cada persona y tarea.

## Características principales

### Portal de Azure
- **Interfaz web gráfica** (portal.azure.com) para crear, configurar y monitorizar cualquier recurso.
- Paneles personalizables, asistentes paso a paso, acceso a Cloud Shell integrado.
- Ideal para **aprender, explorar y tareas puntuales**. No apto para automatizar.

### Azure Cloud Shell
- **Terminal en el navegador**, accesible desde el portal (icono `>_`) o en shell.azure.com.
- Ofrece **dos experiencias**: **Bash** (con la CLI de Azure) y **PowerShell** (con Azure PowerShell). Ambas están **preinstaladas y autenticadas** con tu sesión.
- Incluye editores, Git, Terraform, kubectl y más. **Persiste tus archivos** en una cuenta de almacenamiento (Azure Files) que se crea la primera vez.
- No requiere instalar nada en el equipo; funciona desde cualquier navegador y desde la app móvil.

### CLI de Azure (Azure CLI)
- Herramienta de **línea de comandos multiplataforma** (Windows, macOS, Linux) con comandos que empiezan por `az`.
- Sintaxis sencilla, cercana a Bash; ideal para scripts en shell y para quien viene de Linux.

```bash
az group create --name rg-demo --location westeurope
az vm create --resource-group rg-demo --name vm1 --image Ubuntu2204
```

### Azure PowerShell
- **Módulo de PowerShell** (`Az`) con cmdlets del tipo *Verbo-AzSustantivo*. Multiplataforma (PowerShell 7).
- Ideal para administradores de Windows y para scripts que encadenan objetos.

```powershell
New-AzResourceGroup -Name rg-demo -Location westeurope
New-AzVM -ResourceGroupName rg-demo -Name vm1 -Image Ubuntu2204
```

### App móvil de Azure
- Aplicación para **iOS y Android**: ver estado y alertas, reiniciar VMs, ejecutar comandos en Cloud Shell, consultar Service Health.
- Pensada para **supervisión y acciones rápidas** desde el móvil.

### Otras vías
- **API REST de ARM** y **SDKs** (.NET, Python, Java, Go…) para integrar Azure en aplicaciones.
- **Azure Arc** extiende estas mismas herramientas a servidores fuera de Azure. Ver [[Azure Arc]].

### Comparativa

| Herramienta | Tipo | Dónde se ejecuta | Instalación | Para quién / qué |
|---|---|---|---|---|
| **Portal** | Gráfica | Navegador | Ninguna | Explorar, tareas puntuales, aprender |
| **App móvil** | Gráfica | iOS / Android | App | Supervisar y actuar desde el móvil |
| **Cloud Shell** | Terminal | Navegador | Ninguna | Comandos sin instalar nada; CLI y PowerShell listos |
| **CLI de Azure** | Comandos `az` | Local o Cloud Shell | Sí (local) | Scripts tipo Bash; multiplataforma |
| **Azure PowerShell** | Cmdlets `*-Az*` | Local o Cloud Shell | Sí (local) | Scripts PowerShell; administradores Windows |

## Casos de uso

- Crear tu primera VM y ver sus métricas: **portal**.
- Crear 30 grupos de recursos con nombres secuenciales: script con **CLI** o **PowerShell**.
- Estás en un ordenador ajeno sin nada instalado y necesitas ejecutar comandos: **Cloud Shell**.
- Reiniciar una VM desde el tren: **app móvil**.

## Comparaciones

| Necesidad | Herramienta | No confundir con |
|---|---|---|
| Interfaz gráfica completa | Portal | App móvil (funciones limitadas) |
| Ejecutar CLI o PowerShell sin instalar nada | Cloud Shell | Instalar CLI localmente |
| Scripts con sintaxis `az` | CLI de Azure | PowerShell (`New-Az…`) |
| Scripts con cmdlets `Verbo-Az` | Azure PowerShell | CLI |
| Desplegar la misma infraestructura repetidamente | [[05 - Infraestructura como código - ARM y Bicep|Plantillas ARM / Bicep]] | Scripts imperativos (funcionan, pero no son idempotentes) |

## Conceptos que debo memorizar

> [!important]
> - **Portal** = gráfico, navegador. **App móvil** = supervisión rápida.
> - **Cloud Shell** = terminal en el navegador con **Bash (CLI)** y **PowerShell** preinstalados y autenticados; guarda archivos en Azure Files.
> - **CLI** = comandos `az`, multiplataforma. **PowerShell** = módulo `Az`, cmdlets `Verbo-AzNombre`, multiplataforma.
> - Todas hablan con **Azure Resource Manager**; el resultado es el mismo con cualquiera.

## Tips para AZ-900

> [!tip]
> - Si ves `az vm create` → **CLI**. Si ves `New-AzVM` → **PowerShell**.
> - "Sin instalar nada", "desde el navegador", "desde cualquier equipo" → **Cloud Shell**.
> - "Interfaz gráfica" → **portal**. "Desde el teléfono" → **app móvil**.
> - Trampa: "La CLI de Azure solo funciona en Linux". **Falso**, es multiplataforma. Igual con PowerShell (versión 7).
> - Trampa: "Cloud Shell requiere instalar la CLI". **Falso**, ya viene incluida.
> - Trampa: "Solo se puede administrar Azure desde el portal". **Falso**.

## Ejemplo de pregunta de examen

**Pregunta 1.** Un administrador necesita ejecutar comandos de la CLI de Azure desde un equipo prestado en el que no puede instalar software. ¿Qué debe usar?

- A) La app móvil de Azure
- B) Azure Cloud Shell
- C) Azure PowerShell instalado localmente
- D) Azure Arc

**Respuesta: B.** Cloud Shell ofrece la CLI en el navegador sin instalar nada.
- A) La app móvil permite Cloud Shell, pero la pregunta describe un equipo, no un teléfono; además no es la respuesta más directa.
- C) Requiere instalar software.
- D) Extiende la gestión a recursos externos; no es una consola.

**Pregunta 2.** ¿Cuál de los siguientes comandos corresponde a Azure PowerShell?

- A) `az group create --name rg1 --location westeurope`
- B) `New-AzResourceGroup -Name rg1 -Location westeurope`
- C) `azure group create rg1 westeurope`
- D) `create-resource-group rg1`

**Respuesta: B.** Los cmdlets de Azure PowerShell siguen el patrón *Verbo-AzSustantivo*.
- A) Es la CLI de Azure.
- C y D) No corresponden a ninguna herramienta actual.

## 🧠 Resumen para el examen

1. Portal = gráfico en navegador; app móvil = supervisión desde el teléfono.
2. Cloud Shell = terminal en navegador con CLI (Bash) y PowerShell preinstalados y autenticados.
3. CLI = `az …`, PowerShell = `Verbo-Az…`; ambos multiplataforma.
4. Todas las herramientas pasan por Azure Resource Manager.
5. Para automatizar y repetir, línea de comandos o plantillas; el portal es para tareas puntuales.

---
