---
tags: [az-104, azure, repaso, simulacro, examen]
modulo: Repaso global
---

# 🧪 Simulacro final AZ-104

> [!important] Instrucciones
> - **50 preguntas**, **75 minutos** (ritmo del examen real: ~1,5 min por pregunta).
> - Responde **sin consultar las notas**. Anota tus respuestas en papel o en una nota aparte.
> - Las respuestas están **todas al final**, no debajo de cada pregunta.
> - Corrección: **35/50 (70 %)** es el umbral de aprobado aproximado.
> - Al terminar, apunta el dominio de cada fallo y repasa esa nota.
>
> Distribución por dominio, como en el examen real: identidad 12, almacenamiento 9, cómputo 12, redes 10, monitorización 7.

---

## Preguntas

**1.** Un usuario necesita reiniciar máquinas virtuales de un grupo de recursos, sin poder crearlas ni eliminarlas y sin asignar permisos. Ningún rol integrado encaja exactamente. ¿Qué haces?
A) Asignar Contributor · B) Asignar Virtual Machine Contributor · C) Crear un rol personalizado con `Microsoft.Compute/virtualMachines/restart/action` y permisos de lectura · D) Asignar Owner en cada VM

**2.** ¿Qué afirmación sobre los bloqueos de recursos es correcta?
A) Un bloqueo ReadOnly permite iniciar y detener una VM · B) Contributor puede crear bloqueos · C) Un bloqueo CanNotDelete permite modificar el recurso pero no eliminarlo · D) Los bloqueos no se heredan

**3.** Una directiva de Azure Policy con efecto DeployIfNotExists no aplica los cambios en los recursos existentes. ¿Qué falta?
A) Cambiar el efecto a Deny · B) Crear una tarea de remediación y que la asignación tenga identidad administrada · C) Asignarla en un grupo de administración · D) Habilitar un bloqueo

**4.** ¿Cuántos niveles de grupos de administración pueden anidarse bajo el grupo raíz?
A) 3 · B) 6 · C) 8 · D) 10

**5.** Un usuario invitado debe poder invitar a otros colaboradores externos, sin más permisos en el directorio. ¿Qué rol le asignas?
A) User Administrator · B) Guest Inviter · C) Global Reader · D) Groups Administrator

**6.** ¿Qué configuración es obligatoria en un usuario antes de poder asignarle una licencia?
A) Un grupo de seguridad · B) La ubicación de uso (usage location) · C) Un método de autenticación · D) Un dominio personalizado

**7.** Necesitas que una directiva exija la etiqueta `Owner` en todos los **grupos de recursos** nuevos. ¿Qué modo debe tener la definición?
A) Indexed · B) All · C) NotSpecified · D) Extended

**8.** Un usuario pertenece a un grupo con Contributor en la suscripción y tiene Reader asignado directamente en un grupo de recursos. ¿Qué puede hacer en ese grupo de recursos?
A) Solo leer · B) Crear y gestionar recursos · C) Nada · D) Solo asignar roles

**9.** ¿Qué método de SSPR **no** está disponible para las cuentas de administrador?
A) Aplicación móvil · B) Correo electrónico · C) Preguntas de seguridad · D) SMS

**10.** ¿Qué ocurre con las asignaciones de rol definidas directamente sobre un recurso cuando se mueve a otro grupo de recursos?
A) Se conservan · B) Se eliminan y hay que recrearlas · C) Se convierten en heredadas · D) Bloquean el movimiento

**11.** Necesitas que las suscripciones de producción no permitan crear recursos fuera de Europa, incluidas las futuras. ¿Dónde asignas la directiva?
A) En cada grupo de recursos · B) En cada suscripción · C) En el grupo de administración que las contiene · D) En el tenant de Entra ID

**12.** ¿Qué rol permite ver los costes de una suscripción sin poder modificar nada?
A) Contributor · B) Cost Management Reader · C) Owner · D) Monitoring Contributor

**13.** Una cuenta de almacenamiento Premium block blob debe sobrevivir al fallo de un centro de datos dentro de la región. ¿Qué redundancia configuras?
A) LRS · B) ZRS · C) GRS · D) RA-GZRS

**14.** ¿Qué tipo de SAS se firma con credenciales de Microsoft Entra ID y caduca como máximo a los 7 días?
A) Account SAS · B) Service SAS · C) User delegation SAS · D) SAS con directiva almacenada

**15.** Necesitas mover automáticamente a Cool los blobs que no se han **leído** en 60 días. ¿Qué debes habilitar además de la regla de ciclo de vida?
A) Versionado · B) Change feed · C) Seguimiento del tiempo de último acceso · D) Soft delete

**16.** ¿Cuál es la retención mínima antes de que se apliquen cargos por eliminación anticipada en el nivel Archive?
A) 30 días · B) 90 días · C) 180 días · D) 365 días

**17.** Configuras el firewall de una cuenta para permitir solo una subred, pero las VMs de esa subred siguen sin acceder. ¿Qué falta?
A) Un private endpoint · B) Habilitar el service endpoint Microsoft.Storage en la subred · C) Un NSG · D) Peering

**18.** ¿Qué requisito tiene el Key Vault para poder usar claves administradas por el cliente en una cuenta de almacenamiento?
A) SKU Premium · B) Soft delete y purge protection habilitados · C) Estar en otra región · D) Acceso público deshabilitado

**19.** Un recurso compartido de Azure Files no puede montarse desde la oficina, pero sí desde una VM de Azure. ¿Cuál es la causa más probable?
A) La cuota es insuficiente · B) El ISP bloquea el puerto 445 · C) Falta soft delete · D) El recurso es NFS

**20.** ¿Cuántas directivas de acceso almacenadas admite un contenedor de blobs?
A) 1 · B) 3 · C) 5 · D) 10

**21.** ¿Qué componente de Azure File Sync define la ruta local del servidor que se sincroniza?
A) Cloud endpoint · B) Server endpoint · C) Sync group · D) Storage Sync Service

**22.** ¿Qué combinación permite restaurar un contenedor de blobs completo eliminado por error?
A) Versionado · B) Soft delete de contenedores · C) Snapshots · D) Object replication

**23.** Una VM debe conservar su dirección IP pública tras desasignarla. ¿Qué configuras?
A) IP pública Basic dinámica · B) IP pública Standard (siempre estática) · C) NAT Gateway · D) IP privada estática

**24.** Necesitas redimensionar una VM a un tamaño de otra familia que no aparece en la lista. ¿Qué haces?
A) Crear una VM nueva · B) Desasignar la VM y repetir el cambio · C) Cambiar la región · D) Añadir un disco

**25.** ¿Qué afirmación sobre el disco temporal de una VM es correcta?
A) Es un disco administrado · B) Se conserva al desasignar la VM · C) Se pierde al desasignar o redimensionar la VM · D) Se puede ampliar

**26.** ¿Qué SLA ofrece una única VM con todos sus discos Premium SSD?
A) 95 % · B) 99,9 % · C) 99,95 % · D) 99,99 %

**27.** Un VMSS debe aplicar una imagen nueva por lotes comprobando la salud de las instancias. ¿Qué directiva de actualización configuras?
A) Manual · B) Automatic · C) Rolling · D) Reimage

**28.** ¿Qué elemento de Azure Container Instances agrupa contenedores que comparten red y volúmenes?
A) Revisión · B) Container group · C) Entorno · D) Pod

**29.** Una Container App con una regla de escalado basada en CPU y `min replicas = 0` nunca baja a cero. ¿Por qué?
A) Falta el ingress · B) Las reglas de CPU y memoria no permiten escalar a cero · C) El entorno es Consumption · D) Falta una revisión

**30.** ¿Cuántas ranuras de implementación admite un plan Standard de App Service?
A) 0 · B) 5 · C) 10 · D) 20

**31.** ¿Qué configuración de App Service **no** se intercambia durante un swap?
A) Las app settings normales · B) La versión del runtime · C) Las app settings marcadas como slot setting · D) Los ajustes de WebJobs

**32.** ¿Qué requisito tiene Azure Disk Encryption respecto al Key Vault?
A) Debe estar en otra suscripción · B) Debe estar en la misma región y suscripción que la VM · C) Debe ser Premium · D) Debe tener private endpoint

**33.** Necesitas crear una imagen a partir de una VM Windows para desplegar VMs nuevas que pidan sus propias credenciales. ¿Qué ejecutas antes de capturar?
A) `waagent -deprovision+user` · B) `sysprep /generalize /oobe /shutdown` · C) Un snapshot · D) `az vm deallocate` únicamente

**34.** ¿Qué comando despliega un archivo Bicep en un grupo de recursos?
A) `az bicep build` · B) `az deployment group create --template-file main.bicep` · C) `az group export` · D) `az bicep decompile`

**35.** ¿Cuántas direcciones IP quedan disponibles para recursos en una subred `/27`?
A) 32 · B) 30 · C) 27 · D) 29

**36.** ¿Qué nombre debe tener obligatoriamente la subred donde se despliega Azure Bastion?
A) BastionSubnet · B) AzureBastionSubnet · C) GatewaySubnet · D) AzureFirewallSubnet

**37.** Un spoke debe usar el VPN Gateway desplegado en el hub. ¿Qué configuras en los emparejamientos?
A) Gateway transit en el spoke y use remote gateways en el hub · B) Gateway transit en el hub y use remote gateways en el spoke · C) Allow forwarded traffic en ambos · D) Un gateway en cada spoke

**38.** ¿Qué tipo de next hop en una tabla de rutas descarta el tráfico?
A) Internet · B) Virtual network · C) None · D) Virtual network gateway

**39.** ¿A qué se asocia un grupo de seguridad de aplicación (ASG)?
A) A una subred · B) A una máquina virtual · C) A la configuración IP de una interfaz de red · D) A un grupo de recursos

**40.** ¿Qué herramienta indica exactamente qué regla de NSG bloquea una conexión concreta?
A) Next hop · B) Verificación de flujo de IP · C) Topology · D) Connection Monitor

**41.** Una cuenta de almacenamiento debe ser accesible con una IP privada desde una red local conectada por ExpressRoute. ¿Qué implementas?
A) Service endpoint · B) Private endpoint con zona DNS privada · C) Regla de firewall con el rango privado · D) Peering

**42.** ¿Cuántos vínculos de red virtual con autorregistro admite una red virtual en zonas DNS privadas?
A) 1 · B) 2 · C) 5 · D) Ilimitados

**43.** Necesitas balancear tráfico UDP entre varias máquinas virtuales. ¿Qué servicio usas?
A) Application Gateway · B) Front Door · C) Azure Load Balancer · D) Traffic Manager

**44.** Una sonda HTTP de un balanceador marca todo el backend como no saludable aunque la aplicación funciona. ¿Cuál es una causa habitual?
A) El SKU es Standard · B) La ruta de la sonda devuelve una redirección en lugar de 200 OK · C) El idle timeout es de 4 minutos · D) Falta una regla NAT

**45.** ¿Durante cuánto tiempo se conservan las métricas de plataforma en Azure Monitor?
A) 30 días · B) 90 días · C) 93 días · D) 1 año

**46.** ¿Qué agente y configuración se usan hoy para recopilar registros del sistema operativo invitado?
A) Log Analytics Agent y una solución · B) Azure Monitor Agent con reglas de recopilación de datos · C) Dependency Agent · D) Extensión de diagnóstico WAD

**47.** Durante la ventana de mantenimiento no quieres recibir notificaciones de las alertas de una suscripción, sin desactivar las reglas. ¿Qué configuras?
A) Un grupo de acciones vacío · B) Una regla de procesamiento de alertas con supresión programada · C) Severidad 4 en todas las reglas · D) Umbrales dinámicos

**48.** ¿Qué tipo de almacén se necesita para proteger una base de datos Azure Database for PostgreSQL?
A) Recovery Services vault · B) Backup vault · C) Key Vault · D) Storage account

**49.** ¿Con qué frecuencia genera Azure Site Recovery puntos de recuperación coherentes con el bloqueo para VMs de Azure?
A) Cada minuto · B) Cada 5 minutos · C) Cada 15 minutos · D) Cada hora

**50.** Backup Reports aparece sin datos tras habilitarlo. ¿Cuál es la causa más probable?
A) Falta el rol Owner · B) No se ha creado la configuración de diagnóstico del vault hacia Log Analytics o no han pasado 24 horas · C) El vault usa GRS · D) Falta habilitar el soft delete

---

## Hoja de respuestas

> [!warning] No sigas leyendo hasta terminar las 50 preguntas.

<details><summary>Ver respuestas y explicaciones</summary>

| # | Resp. | Explicación breve | Repasar |
|---|---|---|---|
| 1 | **C** | Ningún rol integrado da solo reinicio: rol personalizado con la acción `restart/action` | [[09 - Roles personalizados de Azure RBAC]] |
| 2 | **C** | CanNotDelete permite leer y modificar; ReadOnly impide iniciar/detener; Contributor no gestiona locks; se heredan | [[12 - Bloqueos de recursos (Locks)]] |
| 3 | **B** | DeployIfNotExists y Modify requieren identidad administrada y tarea de remediación | [[11 - Azure Policy]] |
| 4 | **B** | Seis niveles bajo el raíz | [[15 - Suscripciones y grupos de administración]] |
| 5 | **B** | Guest Inviter es el rol mínimo para invitar | [[05 - Usuarios externos e invitados (B2B)]] |
| 6 | **B** | Sin usage location la asignación de licencia falla | [[04 - Licencias en Microsoft Entra ID]] |
| 7 | **B** | Las directivas sobre grupos de recursos requieren modo All | [[11 - Azure Policy]] |
| 8 | **B** | Los permisos se acumulan; Reader no reduce Contributor | [[10 - Interpretar asignaciones de acceso]] |
| 9 | **C** | Las preguntas de seguridad no están disponibles para administradores | [[06 - Self-Service Password Reset (SSPR)]] |
| 10 | **B** | Las asignaciones con ámbito de recurso no se mueven | [[14 - Grupos de recursos y movimiento de recursos]] |
| 11 | **C** | La asignación en un MG se hereda a suscripciones actuales y futuras | [[15 - Suscripciones y grupos de administración]] |
| 12 | **B** | Cost Management Reader (o Billing Reader) solo lee costes | [[16 - Administración de costes (presupuestos, alertas y Advisor)]] |
| 13 | **B** | Premium solo admite LRS y ZRS | [[02 - Redundancia de almacenamiento]] |
| 14 | **C** | La user delegation SAS se firma con Entra ID y dura como máximo 7 días | [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]] |
| 15 | **C** | La condición por último acceso requiere access time tracking | [[11 - Administración del ciclo de vida de Blob]] |
| 16 | **C** | Archive: 180 días (Cool 30, Cold 90) | [[10 - Niveles de acceso de Blob (Hot, Cool, Cold, Archive)]] |
| 17 | **B** | Sin el service endpoint el tráfico llega con IP pública | [[03 - Firewalls y redes virtuales de Azure Storage]] |
| 18 | **B** | CMK exige soft delete y purge protection | [[06 - Cifrado de cuentas de almacenamiento]] |
| 19 | **B** | SMB usa el puerto 445, bloqueado por muchos ISP | [[13 - Azure Files]] |
| 20 | **C** | Cinco por contenedor | [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]] |
| 21 | **B** | El server endpoint define la ruta local | [[16 - Azure File Sync]] |
| 22 | **B** | El soft delete de contenedores es independiente del de blobs | [[12 - Versionado, instantáneas y eliminación temporal de Blob]] |
| 23 | **B** | Las IP Standard son siempre estáticas | [[02 - Direcciones IP públicas y privadas]] |
| 24 | **B** | Desasignar mueve la VM a otro clúster con más tamaños | [[05 - Tamaños de VM y redimensionamiento]] |
| 25 | **C** | El disco temporal es efímero | [[04 - Máquinas virtuales - creación y configuración]] |
| 26 | **B** | 99,9 % con Premium SSD o Ultra en una sola VM | [[09 - Alta disponibilidad - Availability Sets y Availability Zones]] |
| 27 | **C** | Rolling actualiza por lotes con sonda de estado | [[10 - Virtual Machine Scale Sets]] |
| 28 | **B** | El container group comparte ciclo de vida, red y volúmenes | [[14 - Azure Container Instances]] |
| 29 | **B** | Solo las reglas basadas en eventos (HTTP, KEDA) escalan a cero | [[15 - Azure Container Apps]] |
| 30 | **B** | Standard 5, Premium 20 | [[17 - App Service Plan (niveles y escalado)]] |
| 31 | **C** | Las slot settings permanecen en su ranura | [[22 - App Service - ranuras de implementación (deployment slots)]] |
| 32 | **B** | ADE exige Key Vault en la misma región y suscripción | [[07 - Azure Disk Encryption y cifrado de discos]] |
| 33 | **B** | Sysprep generaliza la imagen de Windows | [[12 - Imágenes y Azure Compute Gallery]] |
| 34 | **B** | `az deployment group create` acepta Bicep directamente | [[03 - Implementar, exportar y convertir plantillas]] |
| 35 | **C** | 32 − 5 = 27 | [[01 - Azure Virtual Network y subredes]] |
| 36 | **B** | AzureBastionSubnet, mínimo /26 | [[08 - Azure Bastion]] |
| 37 | **B** | Gateway transit lo ofrece quien tiene el gateway | [[03 - Peering de redes virtuales]] |
| 38 | **C** | Next hop None descarta el tráfico | [[04 - Rutas definidas por el usuario (UDR) y NVA]] |
| 39 | **C** | El ASG se asigna a la configuración IP de la NIC | [[06 - Application Security Group (ASG)]] |
| 40 | **B** | IP flow verify devuelve la regla responsable | [[07 - Reglas de seguridad efectivas]] |
| 41 | **B** | Solo el private endpoint es alcanzable desde on-premises | [[10 - Private Endpoint y Private Link]] |
| 42 | **A** | Un único vínculo con autorregistro por VNet | [[12 - Azure Private DNS y resolución de nombres]] |
| 43 | **C** | Solo el Load Balancer opera en capa 4 con UDP | [[13 - Azure Load Balancer]] |
| 44 | **B** | La sonda HTTP solo acepta 200 OK | [[14 - Solución de problemas de balanceo de carga]] |
| 45 | **C** | 93 días | [[02 - Métricas en Azure Monitor]] |
| 46 | **B** | AMA con DCR; el agente antiguo está retirado | [[03 - Logs - Log Analytics y configuración de diagnóstico]] |
| 47 | **B** | Las reglas de procesamiento suprimen notificaciones | [[05 - Alertas, grupos de acciones y reglas de procesamiento]] |
| 48 | **B** | PostgreSQL, discos, blobs y AKS van a Backup vault | [[08 - Azure Backup - Recovery Services vault y Backup vault]] |
| 49 | **B** | Cada 5 minutos | [[11 - Azure Site Recovery]] |
| 50 | **B** | Backup Reports necesita el diagnóstico del vault y ~24 h | [[12 - Informes y alertas de copias de seguridad]] |

</details>

## Corrección y plan de acción

| Aciertos | Interpretación | Qué hacer |
|---|---|---|
| **45-50** | Listo para el examen | Repasar solo [[02 - Guía de memorización (números, límites y nombres)]] |
| **40-44** | Muy buen nivel | Revisar los dominios de los fallos y rehacer el simulacro en 3 días |
| **35-39** | Aprobado justo | Repasar los dos dominios más flojos completos y [[04 - Errores y confusiones frecuentes]] |
| **28-34** | Aún no | Volver a las notas de los dominios fallados y rehacer sus laboratorios |
| **< 28** | Falta base | Repasar los cinco repasos finales de dominio y el [[06 - Banco de preguntas de práctica]] |

### Registro de resultados

| Fecha | Aciertos | Dominios más flojos | Acción |
|---|---|---|---|
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

## Relacionado

- [[06 - Banco de preguntas de práctica]]
- [[05 - Checklist final antes del examen]]
- [[04 - Errores y confusiones frecuentes]]
- [[00 - AZ-104 Índice general (MOC)]]
