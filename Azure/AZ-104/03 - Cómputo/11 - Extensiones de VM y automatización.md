---
tags: [az-104, azure, computo, vm, extensiones, automation]
modulo: Cómputo
peso_examen: Medio
---

# Extensiones de VM y automatización ➕

> [!info] No es un objetivo literal del temario vigente, pero las extensiones aparecen constantemente: ADE, Azure Monitor Agent, Custom Script, Entra login, Application Health en VMSS.

## ¿Qué es?

Una **extensión de VM** es un pequeño paquete de software que el **agente de Azure (waagent / Windows Guest Agent)** instala y ejecuta dentro de la VM para tareas de configuración, supervisión, seguridad o recuperación. Se gestionan desde ARM, portal, CLI o PowerShell.

## ¿Para qué sirve?

- Ejecutar scripts post-despliegue.
- Instalar agentes (monitorización, antimalware, cifrado).
- Recuperar acceso (restablecer contraseña, reparar RDP).

## Extensiones clave 🧠

| Extensión | Para qué |
|---|---|
| **Custom Script Extension** | Descargar y ejecutar un script (PowerShell/Bash), típicamente desde Blob Storage |
| **Azure Monitor Agent (AMA)** | Recopilar logs y métricas con **Data Collection Rules** (sustituye al Log Analytics Agent, retirado) |
| **Azure Disk Encryption (AzureDiskEncryption / AzureDiskEncryptionForLinux)** | Cifrado BitLocker/dm-crypt |
| **VMAccess** | Restablecer contraseña, clave SSH, usuario administrador; reparar configuración |
| **Application Health** | Informar del estado de la app en VMSS (rolling upgrades, reparación automática) |
| **Microsoft Entra login (AADSSHLoginForLinux / AADLoginForWindows)** | Iniciar sesión con credenciales de Entra ID |
| **DSC / Azure Automation State Configuration** | Configuración deseada |
| **Antimalware (IaaSAntimalware)** | Protección antimalware en Windows |
| **NetworkWatcherAgent** | Captura de paquetes y diagnóstico de conexión |
| **Dependency Agent** | Mapa de dependencias en VM Insights |

## Servicios de automatización relacionados

- **Azure Automation**: cuenta con **runbooks** (PowerShell, Python), **programaciones**, **hybrid runbook workers**, **variables y credenciales**. Usos típicos: apagar VMs por la noche, responder a alertas.
- **Azure Update Manager**: evaluación e instalación de actualizaciones del SO (sustituye a Update Management de Automation), con programaciones y mantenimiento.
- **Change Tracking and Inventory**: cambios en archivos, registro y software.
- **Run Command**: ejecutar comandos sin RDP/SSH, desde el portal o CLI (`az vm run-command invoke`).
- **Apagado automático (auto-shutdown)**: función de la propia VM (DevTest Labs), con notificación previa.
- **Azure Policy DeployIfNotExists**: desplegar extensiones automáticamente en VMs nuevas.

## Cómo funciona

```bash
az vm extension set --resource-group rg-web --vm-name vm-web01 \
  --name CustomScriptExtension --publisher Microsoft.Compute \
  --settings '{"fileUris":["https://st001.blob.core.windows.net/scripts/config.ps1"],"commandToExecute":"powershell -ExecutionPolicy Unrestricted -File config.ps1"}'
az vm extension list --resource-group rg-web --vm-name vm-web01 -o table
az vm extension delete --resource-group rg-web --vm-name vm-web01 --name CustomScriptExtension
az vm run-command invoke -g rg-web -n vm-web01 --command-id RunShellScript --scripts "systemctl restart nginx"
az vm user update -g rg-web -n vm-web01 --username azureadmin --password '<nuevaPwd>'   # VMAccess
```

```powershell
Set-AzVMExtension -ResourceGroupName rg-web -VMName vm-web01 -Name CustomScript -Publisher Microsoft.Compute `
  -ExtensionType CustomScriptExtension -TypeHandlerVersion 1.10 -Settings @{fileUris=@("https://...");commandToExecute="powershell -File config.ps1"}
Get-AzVMExtension -ResourceGroupName rg-web -VMName vm-web01
Set-AzVMAccessExtension -ResourceGroupName rg-web -VMName vm-web01 -UserName azureadmin -Password '<pwd>'
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Instalar software en 50 VMs nuevas automáticamente | **Custom Script Extension** desplegada por plantilla o Azure Policy (DeployIfNotExists) |
| Recopilar logs del SO en Log Analytics | **Azure Monitor Agent** + **Data Collection Rule** |
| Olvidé la contraseña del administrador | **Restablecer contraseña** (VMAccess) |
| Apagar las VMs de dev a las 20:00 | **Auto-shutdown** de la VM o **runbook** de Automation programado |
| Parchear el SO mensualmente | **Azure Update Manager** con programación de mantenimiento |
| Ejecutar un comando sin abrir RDP | **Run Command** |
| Comprobar el estado de la app en VMSS | **Application Health Extension** |
| Iniciar sesión en Linux con cuentas de Entra ID | Extensión **AADSSHLoginForLinux** + roles de VM Login |

## Ejemplo

Tras crear un VMSS, cada instancia nueva debe instalar nginx y descargar la web. Se añade la **Custom Script Extension** al modelo del scale set apuntando a un script en Blob Storage con SAS de lectura. Para el parcheo mensual, se define una programación en **Azure Update Manager** que aplica actualizaciones críticas el segundo martes a las 02:00 con ventana de 3 horas.

## Comparaciones

| Herramienta | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Custom Script Extension** | Ejecutar script al desplegar | Simple, en la plantilla | Configuración inicial |
| **Run Command** | Comando puntual | Sin RDP/SSH, sin extensión permanente | Diagnóstico y arreglos |
| **cloud-init** (Linux) | Configuración en el primer arranque | Nativo Linux | VMs Linux nuevas |
| **DSC / State Configuration** | Estado deseado continuo | Corrige desviaciones | Configuración a largo plazo |
| **Azure Automation runbooks** | Orquestación programada | Múltiples recursos, webhooks | Apagados, respuestas a alertas |
| **Update Manager** | Parches | Evaluación y programaciones | Cumplimiento de actualizaciones |

## AZ-104 Exam Tips

- ⭐ Las extensiones las ejecuta el **agente de Azure** dentro de la VM; si el agente falla, la extensión falla.
- 🧠 **Custom Script Extension** para scripts; **Run Command** para comandos puntuales.
- 🔥 🧠 El **Log Analytics Agent (MMA) está retirado**: usar **Azure Monitor Agent con Data Collection Rules**.
- 🧠 **VMAccess** restablece contraseñas y claves SSH.
- 💻 `az vm extension set`, `az vm run-command invoke`, restablecer contraseña en el portal.
- 📌 Azure Automation (runbooks) vs Update Manager (parches) vs extensiones (software en la VM).

## Errores comunes

- Instalar el agente de Log Analytics antiguo en vez de AMA.
- Ejecutar un script con Custom Script Extension sin dar acceso (SAS) al blob.
- Esperar que las extensiones funcionen con el agente de Azure detenido.

## Preguntas que podrían aparecer

**1.** Necesitas ejecutar un script de configuración en 30 VMs existentes sin conectarte a ninguna. ¿Qué usas?
- A) Azure Bastion · B) Custom Script Extension o Run Command · C) Boot diagnostics · D) Azure Policy Deny

<details><summary>Respuesta</summary>

**B.** Ambas ejecutan código dentro de la VM sin sesión interactiva.
</details>

**2.** Quieres recopilar los registros de eventos de Windows de tus VMs en un área de trabajo de Log Analytics. ¿Qué agente y qué configuración necesitas?
- A) Log Analytics Agent y una solución · B) Azure Monitor Agent y una Data Collection Rule · C) Dependency Agent · D) Network Watcher Agent

<details><summary>Respuesta</summary>

**B.** El agente actual es Azure Monitor Agent, que requiere reglas de recopilación de datos (DCR).
</details>

## Relacionado

- [[04 - Máquinas virtuales - creación y configuración]]
- [[10 - Virtual Machine Scale Sets]]
- [[03 - Logs - Log Analytics y configuración de diagnóstico]]
- [[07 - Azure Disk Encryption y cifrado de discos]]
- [[00 - Índice - Cómputo]]
