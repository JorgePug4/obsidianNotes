---
tags: [az-104, azure, repaso, preguntas, practica]
modulo: Repaso global
---

# 🎯 Banco de preguntas de práctica AZ-104

60 preguntas por dominio con respuesta y explicación desplegable. Úsalo por bloques, no de una sentada. Para una prueba cronometrada completa, ve a [[07 - Simulacro final AZ-104]].

---

## Dominio 1 · Identidades y gobernanza (15 preguntas)

**1.** Necesitas que un usuario pueda crear y eliminar máquinas virtuales en `RG-Prod`, pero no debe poder conceder permisos a nadie. Siguiendo el privilegio mínimo, ¿qué rol asignas y en qué ámbito?
- A) Owner en RG-Prod · B) Contributor en la suscripción · C) Virtual Machine Contributor en RG-Prod · D) User Access Administrator en RG-Prod

<details><summary>Respuesta</summary>

**C.** Virtual Machine Contributor gestiona VMs sin permitir asignación de roles ni tocar otros tipos de recurso. Owner permitiría dar permisos; Contributor en la suscripción excede el ámbito; UAA gestiona acceso, no recursos.
</details>

**2.** Un administrador con rol Owner en la suscripción no consigue eliminar un grupo de recursos. ¿Cuál es la causa más probable?
- A) Le falta el rol User Access Administrator · B) Existe un bloqueo CanNotDelete o ReadOnly · C) Azure Policy deniega la lectura · D) Su cuenta es invitada

<details><summary>Respuesta</summary>

**B.** Los locks prevalecen sobre cualquier rol; hay que eliminarlos primero.
</details>

**3.** Debes aplicar la etiqueta `CostCenter` del grupo de recursos a todos los recursos que ya existen dentro de él. ¿Qué efecto de Azure Policy usas?
- A) Append · B) Deny · C) Modify con tarea de remediación · D) AuditIfNotExists

<details><summary>Respuesta</summary>

**C.** Modify puede actuar sobre recursos existentes mediante remediación; Append solo se aplica en la creación.
</details>

**4.** Tu organización tiene 22 suscripciones y necesitas que una directiva de regiones permitidas se aplique a todas ellas y a las futuras, con el mínimo esfuerzo. ¿Qué haces?
- A) Asignar la directiva en cada suscripción · B) Asignarla en el grupo de administración raíz o en un MG que las contenga · C) Crear un rol personalizado · D) Usar un bloqueo

<details><summary>Respuesta</summary>

**B.** Las asignaciones en un grupo de administración se heredan a todas las suscripciones hijas.
</details>

**5.** Un usuario tiene Reader en la suscripción, pertenece a un grupo con Contributor en `RG-Dev` y tiene una asignación directa de Virtual Machine Contributor en `VM1` (dentro de `RG-Dev`). ¿Puede crear una cuenta de almacenamiento en `RG-Dev`?
- A) No, Reader lo impide · B) Sí, por el Contributor heredado del grupo · C) Solo si es Owner · D) No, Virtual Machine Contributor lo limita

<details><summary>Respuesta</summary>

**B.** Los permisos son acumulativos: el Contributor del grupo aplica a todo el RG.
</details>

**6.** Necesitas que los usuarios sincronizados desde Active Directory puedan restablecer su contraseña y que el cambio llegue al dominio local. ¿Qué requisitos hay?
- A) Entra ID Free y SSPR habilitado · B) Entra ID P1 y password writeback en Entra Connect · C) Entra ID P2 y unidades administrativas · D) Un servidor AD FS

<details><summary>Respuesta</summary>

**B.** El writeback requiere P1 (o M365 Business Premium) y la opción habilitada en Entra Connect o Cloud Sync.
</details>

**7.** Quieres que todos los usuarios cuyo atributo `department` sea "Ventas" pertenezcan automáticamente a un grupo. ¿Qué configuras y qué licencia necesitas?
- A) Grupo asignado, Free · B) Grupo dinámico de usuarios, P1 · C) Grupo de Microsoft 365, Free · D) Unidad administrativa, P2

<details><summary>Respuesta</summary>

**B.** La pertenencia dinámica requiere Microsoft Entra ID P1.
</details>

**8.** ¿Qué ocurre con los recursos existentes cuando asignas una directiva con efecto Deny que prohíbe las cuentas de almacenamiento con acceso público?
- A) Se eliminan · B) Se modifican automáticamente · C) Siguen existiendo y aparecen como no conformes · D) Se bloquean para lectura

<details><summary>Respuesta</summary>

**C.** Deny solo bloquea creaciones y actualizaciones que incumplan; lo existente se marca no conforme.
</details>

**9.** Debes mover una máquina virtual y sus discos de la suscripción A a la suscripción B. Ambas pertenecen a tenants distintos de Microsoft Entra. ¿Qué haces primero?
- A) Nada especial · B) Transferir una suscripción al mismo tenant · C) Crear un peering · D) Exportar la VM como imagen

<details><summary>Respuesta</summary>

**B.** El movimiento entre suscripciones exige el mismo tenant.
</details>

**10.** ¿Qué rol es el mínimo para crear y asignar directivas de Azure Policy?
- A) Owner · B) Contributor · C) Resource Policy Contributor · D) Security Admin

<details><summary>Respuesta</summary>

**C.** Contributor no incluye las acciones de Microsoft.Authorization necesarias para Policy.
</details>

**11.** Necesitas que el equipo de soporte de la filial de Lisboa solo pueda restablecer contraseñas de los usuarios de esa filial. ¿Qué implementas?
- A) Un grupo de seguridad con Password Administrator · B) Una unidad administrativa con esos usuarios y el rol con ámbito en ella · C) Un tenant nuevo · D) RBAC en un grupo de recursos

<details><summary>Respuesta</summary>

**B.** Las unidades administrativas limitan el ámbito de los roles de Entra ID.
</details>

**12.** Configuras un presupuesto mensual de 5000 € en una suscripción con alerta al 100 %. Al superarse, los recursos siguen funcionando. ¿Es correcto?
- A) No, deberían detenerse · B) Sí: los presupuestos solo alertan; para actuar hay que enlazar un grupo de acciones con automatización · C) No, falta el rol Owner · D) Sí, pero solo en suscripciones EA

<details><summary>Respuesta</summary>

**B.**
</details>

**13.** ¿Cuál es el número máximo de etiquetas que admite un recurso de Azure?
- A) 15 · B) 25 · C) 50 · D) 100

<details><summary>Respuesta</summary>

**C.**
</details>

**14.** Un rol personalizado define `Actions: ["*"]` y `NotActions: ["Microsoft.Storage/storageAccounts/delete"]`. El usuario también tiene Contributor en el mismo ámbito. ¿Puede eliminar cuentas de almacenamiento?
- A) No · B) Sí, porque Contributor lo permite y los permisos se suman · C) Solo si es Owner · D) Depende de la prioridad

<details><summary>Respuesta</summary>

**B.** NotActions no es una denegación efectiva frente a otros roles.
</details>

**15.** Tras transferir una suscripción a otro tenant de Microsoft Entra, los administradores pierden el acceso. ¿Por qué?
- A) Se eliminó la suscripción · B) Las asignaciones de rol RBAC se eliminan en la transferencia · C) Faltan licencias · D) Hay un lock

<details><summary>Respuesta</summary>

**B.** También dejan de funcionar las identidades administradas y las directivas de acceso ligadas al tenant anterior.
</details>

---

## Dominio 2 · Almacenamiento (12 preguntas)

**16.** Necesitas que una cuenta de almacenamiento sobreviva a la pérdida de una zona y de la región, y poder **leer** los datos durante la interrupción. ¿Qué redundancia eliges?
- A) ZRS · B) GRS · C) GZRS · D) RA-GZRS

<details><summary>Respuesta</summary>

**D.** Solo las variantes RA permiten leer el punto de conexión secundario sin failover.
</details>

**17.** Un partner necesita acceso de lectura a un contenedor durante seis meses, y quieres poder revocarlo en cualquier momento sin afectar a otros clientes. ¿Qué generas?
- A) SAS de cuenta ad hoc · B) SAS de servicio asociada a una directiva de acceso almacenada · C) Clave de acceso · D) Acceso anónimo

<details><summary>Respuesta</summary>

**B.** Borrando la directiva se revocan todas las SAS asociadas sin tocar las claves.
</details>

**18.** Una aplicación con identidad administrada recibe 403 al leer blobs, aunque tiene el rol Storage Account Contributor. ¿Qué le falta?
- A) Owner · B) Un rol de plano de datos como Storage Blob Data Reader · C) Una SAS · D) Un private endpoint

<details><summary>Respuesta</summary>

**B.** Los roles de control no otorgan acceso a los datos vía Entra ID.
</details>

**19.** Quieres que los blobs no modificados en 90 días pasen a Archive y se eliminen a los 7 años. ¿Qué configuras?
- A) Object replication · B) Una regla de administración del ciclo de vida · C) Soft delete · D) Un snapshot programado

<details><summary>Respuesta</summary>

**B.** Las reglas de ciclo de vida permiten tierToArchive y delete por antigüedad.
</details>

**20.** Los servidores on-premises conectados por VPN deben acceder de forma privada a una cuenta de almacenamiento. ¿Qué implementas?
- A) Service endpoint en la subred del gateway · B) Private endpoint con zona DNS privada · C) Regla de firewall con el rango privado · D) Acceso anónimo

<details><summary>Respuesta</summary>

**B.** Los service endpoints no aplican a tráfico procedente de fuera de Azure.
</details>

**21.** ¿Qué requisitos debe cumplir la cuenta de origen para configurar la replicación de objetos?
- A) GRS habilitado · B) Versionado y change feed habilitados · C) Soft delete de 30 días · D) Nivel Premium

<details><summary>Respuesta</summary>

**B.** Y la cuenta de destino necesita versionado.
</details>

**22.** Necesitas un recurso compartido de archivos accesible por NFS 4.1 desde VMs Linux. ¿Qué tipo de cuenta creas?
- A) Standard GPv2 · B) Premium FileStorage · C) Premium block blobs · D) BlobStorage

<details><summary>Respuesta</summary>

**B.**
</details>

**23.** Los usuarios deben acceder a un recurso compartido SMB con sus credenciales corporativas desde portátiles unidos a Microsoft Entra, sin conectividad con un controlador de dominio. ¿Qué origen de identidad configuras?
- A) AD DS · B) Microsoft Entra Domain Services · C) Microsoft Entra Kerberos · D) Clave de la cuenta

<details><summary>Respuesta</summary>

**C.**
</details>

**24.** Un blob de 4 GB está en Archive y se necesita en menos de una hora. ¿Qué haces?
- A) Descargarlo con AzCopy · B) Cambiar su nivel con prioridad de rehidratación alta · C) Copiarlo con prioridad estándar · D) Cambiar la redundancia

<details><summary>Respuesta</summary>

**B.** La rehidratación de prioridad alta completa en menos de una hora para blobs de menos de 10 GB.
</details>

**25.** Tras habilitar el firewall de la cuenta para permitir solo una subred, el portal deja de mostrar los blobs desde tu equipo. ¿Qué ocurre?
- A) Se han borrado · B) Tu IP pública no está permitida en el firewall · C) Falta el rol Owner · D) La cuenta pasó a Archive

<details><summary>Respuesta</summary>

**B.** Hay que añadir tu IP o acceder desde la subred permitida.
</details>

**26.** ¿Qué herramienta usarías para copiar 5 TB entre dos cuentas de almacenamiento sin que los datos pasen por tu equipo y pudiendo reanudar?
- A) Storage Explorer · B) AzCopy con URLs de origen y destino · C) El portal · D) Azure File Sync

<details><summary>Respuesta</summary>

**B.**
</details>

**27.** Un administrador eliminó un contenedor completo hace 2 días. La cuenta tiene soft delete de blobs (14 días) pero no de contenedores. ¿Se puede recuperar?
- A) Sí, con Undelete por blob · B) No por esa vía: hace falta el soft delete de contenedores · C) Sí, con point-in-time restore siempre · D) Sí, con object replication

<details><summary>Respuesta</summary>

**B.** Son configuraciones independientes.
</details>

---

## Dominio 3 · Cómputo (15 preguntas)

**28.** Una VM apagada desde el sistema operativo sigue generando coste de cómputo. ¿Qué debe hacerse?
- A) Eliminar la IP pública · B) Desasignarla (Stop desde el portal o CLI) · C) Reducir su tamaño · D) Quitarle el disco temporal

<details><summary>Respuesta</summary>

**B.**
</details>

**29.** Necesitas un SLA del 99,99 % para las VMs de una aplicación web. ¿Qué despliegas?
- A) Dos VMs en un availability set · B) Una VM con discos Ultra · C) Dos o más VMs en al menos dos zonas de disponibilidad · D) Tres VMs en la misma zona

<details><summary>Respuesta</summary>

**C.**
</details>

**30.** Quieres añadir un disco Premium SSD a una VM `Standard_D4_v3` y la opción no está disponible. ¿Qué haces?
- A) Ampliar la cuota · B) Redimensionar a `Standard_D4s_v3` · C) Cambiar de región · D) Habilitar encryption at host

<details><summary>Respuesta</summary>

**B.** Solo los tamaños con "s" admiten almacenamiento Premium.
</details>

**31.** Despliegas una plantilla ARM en modo Complete sobre un grupo de recursos que contiene una cuenta de almacenamiento no incluida en la plantilla. ¿Qué ocurre?
- A) Se conserva · B) Se elimina · C) Se marca no conforme · D) Falla el despliegue

<details><summary>Respuesta</summary>

**B.**
</details>

**32.** Necesitas un conjunto de escalado que combine instancias Spot y regulares de distintos tamaños. ¿Qué modo de orquestación usas?
- A) Uniform · B) Flexible · C) Availability set · D) Proximity placement group

<details><summary>Respuesta</summary>

**B.**
</details>

**33.** Un contenedor debe ejecutarse una vez, procesar datos y finalizar, reintentando solo si falla. ¿Qué configuras en Azure Container Instances?
- A) Restart policy Always · B) Restart policy OnFailure · C) Restart policy Never con min replicas 1 · D) Una regla KEDA

<details><summary>Respuesta</summary>

**B.**
</details>

**34.** Una API en contenedor no debe generar coste cuando no recibe peticiones y debe escalar automáticamente con la demanda. ¿Qué servicio y configuración?
- A) ACI con varios grupos · B) Container Apps con min replicas 0 y regla de escalado HTTP · C) VMSS con autoescalado · D) App Service Basic

<details><summary>Respuesta</summary>

**B.** Las reglas de CPU no permiten escalar a cero; la regla HTTP sí.
</details>

**35.** Una aplicación en un plan Basic necesita ranuras de implementación y autoescalado. ¿Cuál es el cambio mínimo?
- A) Cambiar a Shared · B) Escalar verticalmente a Standard · C) Añadir instancias · D) Migrar a Isolated

<details><summary>Respuesta</summary>

**B.**
</details>

**36.** Tras un swap entre `staging` y producción, la aplicación de producción apunta a la base de datos de pruebas. ¿Qué faltó configurar?
- A) Always On · B) Marcar la cadena de conexión como configuración de ranura · C) Health check · D) ARR affinity

<details><summary>Respuesta</summary>

**B.**
</details>

**37.** Necesitas asignar `tienda.contoso.com` a una Web App. ¿Qué registros DNS creas?
- A) Solo A · B) CNAME hacia `<app>.azurewebsites.net` y TXT `asuid.tienda` · C) MX y TXT · D) NS

<details><summary>Respuesta</summary>

**B.**
</details>

**38.** Una Web App debe acceder a una base de datos que solo admite tráfico desde una red virtual. ¿Qué configuras?
- A) Private endpoint para la app · B) Integración con red virtual · C) Restricciones de acceso · D) Hybrid Connections

<details><summary>Respuesta</summary>

**B.** VNet integration controla el tráfico **saliente** de la app.
</details>

**39.** Debes cifrar los discos de una VM incluyendo el disco temporal y las cachés, sin instalar nada dentro del sistema operativo. ¿Qué eliges?
- A) SSE con PMK · B) Azure Disk Encryption · C) Encryption at host · D) Cifrado de infraestructura de Storage

<details><summary>Respuesta</summary>

**C.**
</details>

**40.** Quieres desplegar 150 VMs idénticas con el software ya instalado en dos regiones. ¿Qué usas?
- A) Snapshot del disco de SO · B) Imagen administrada en una región · C) Azure Compute Gallery con la versión replicada en ambas regiones · D) Custom Script Extension únicamente

<details><summary>Respuesta</summary>

**C.**
</details>

**41.** ¿Qué SKU de Azure Container Registry es necesario para geo-replicación y private link?
- A) Basic · B) Standard · C) Premium · D) Cualquiera

<details><summary>Respuesta</summary>

**C.**
</details>

**42.** Tienes que convertir una plantilla ARM en JSON a Bicep para mantenerla. ¿Qué comando usas?
- A) `az bicep build` · B) `az bicep decompile` · C) `az deployment group export` · D) `az group export`

<details><summary>Respuesta</summary>

**B.**
</details>

---

## Dominio 4 · Redes (12 preguntas)

**43.** ¿Cuántas direcciones IP pueden asignarse a recursos en la subred `10.4.8.0/26`?
- A) 64 · B) 62 · C) 59 · D) 60

<details><summary>Respuesta</summary>

**C.** 64 menos las 5 reservadas por Azure.
</details>

**44.** VNet1 está emparejada con VNet2, y VNet2 con VNet3. ¿Qué se necesita para que VNet1 alcance VNet3?
- A) Nada, el peering es transitivo · B) Un peering directo entre VNet1 y VNet3, o una NVA con rutas definidas por el usuario · C) Gateway transit · D) Un private endpoint

<details><summary>Respuesta</summary>

**B.**
</details>

**45.** El NSG de la subred permite el puerto 8080, pero la NIC tiene otro NSG sin reglas personalizadas. ¿Llega el tráfico?
- A) Sí · B) No: la regla predeterminada DenyAllInBound del NSG de la NIC lo bloquea · C) Solo desde la VNet · D) Depende de la prioridad más baja

<details><summary>Respuesta</summary>

**B.**
</details>

**46.** Una subred tiene una ruta del sistema, una ruta BGP y una UDR, todas para `0.0.0.0/0`. ¿Cuál se aplica?
- A) La del sistema · B) La BGP · C) La UDR · D) La más antigua

<details><summary>Respuesta</summary>

**C.** Prioridad: UDR > BGP > sistema.
</details>

**47.** Configuras una UDR con next hop hacia una VM que actúa como firewall, pero el tráfico se pierde. ¿Qué falta?
- A) Un NSG permisivo · B) Habilitar el reenvío de IP en la NIC de la NVA y en su sistema operativo · C) Un peering · D) Una IP pública

<details><summary>Respuesta</summary>

**B.**
</details>

**48.** Necesitas administrar VMs por RDP sin exponer el puerto 3389 y sin IP pública en las VMs, y además grabar las sesiones. ¿Qué despliegas?
- A) Bastion Basic · B) Bastion Standard · C) Bastion Premium · D) JIT VM Access

<details><summary>Respuesta</summary>

**C.** La grabación de sesión es una característica del SKU Premium.
</details>

**49.** Tras crear un private endpoint para una cuenta de almacenamiento, las VMs siguen resolviendo la IP pública. ¿Qué falta?
- A) Un NSG · B) Vincular la zona DNS privada `privatelink.blob.core.windows.net` a la VNet y crear el registro A · C) Un service endpoint · D) Reiniciar las VMs

<details><summary>Respuesta</summary>

**B.**
</details>

**50.** Dos redes virtuales emparejadas no resuelven los nombres de host de sus VMs. ¿Qué implementas?
- A) Servidor DNS en cada VNet · B) Una zona DNS privada vinculada a ambas redes · C) Azure DNS público · D) Un private endpoint

<details><summary>Respuesta</summary>

**B.**
</details>

**51.** Todas las instancias del backend de un Load Balancer aparecen como no saludables, aunque la aplicación responde en local. ¿Qué revisas primero?
- A) El SKU del balanceador · B) Que el NSG permita el origen AzureLoadBalancer y que la sonda reciba un 200 OK · C) El idle timeout · D) Las zonas

<details><summary>Respuesta</summary>

**B.**
</details>

**52.** Una aplicación web debe enrutar `/api` y `/web` a grupos distintos de servidores, terminar TLS y protegerse frente a ataques OWASP, dentro de una región. ¿Qué servicio?
- A) Load Balancer · B) Application Gateway con WAF · C) Traffic Manager · D) Front Door

<details><summary>Respuesta</summary>

**B.** Front Door también podría, pero es global; el enunciado limita a una región.
</details>

**53.** Necesitas que todas las VMs de una subred salgan a Internet siempre con la misma dirección IP. ¿Qué implementas?
- A) IP pública en cada NIC · B) NAT Gateway con IP pública estática · C) Service endpoint · D) Azure Firewall obligatoriamente

<details><summary>Respuesta</summary>

**B.** Azure Firewall también daría IP fija, pero NAT Gateway es la solución directa y más simple.
</details>

**54.** ¿Qué registro de Azure DNS permite apuntar el dominio raíz a un perfil de Front Door?
- A) CNAME · B) Registro alias · C) TXT · D) SRV

<details><summary>Respuesta</summary>

**B.**
</details>

---

## Dominio 5 · Monitorización y mantenimiento (6 preguntas)

**55.** Los registros de diagnóstico de un Key Vault no aparecen en Log Analytics. ¿Qué falta?
- A) Un agente en el Key Vault · B) Una configuración de diagnóstico que envíe los registros al área de trabajo · C) Una alerta · D) Application Insights

<details><summary>Respuesta</summary>

**B.**
</details>

**56.** Necesitas recibir un aviso cuando alguien elimine una máquina virtual en la suscripción. ¿Qué tipo de regla de alerta creas?
- A) De métrica · B) Del registro de actividad · C) De búsqueda de registros · D) Smart detection

<details><summary>Respuesta</summary>

**B.**
</details>

**57.** ¿Dónde se protegen los blobs y los discos administrados con Azure Backup?
- A) Recovery Services vault · B) Backup vault · C) Key Vault · D) Storage account

<details><summary>Respuesta</summary>

**B.**
</details>

**58.** Necesitas copias de seguridad de una VM cada 4 horas y soporte para Trusted Launch. ¿Qué directiva configuras?
- A) Standard · B) Enhanced · C) MARS · D) Operacional

<details><summary>Respuesta</summary>

**B.**
</details>

**59.** Una VM crítica debe recuperarse conservando su nombre, su IP privada y sus etiquetas. ¿Qué opción de restauración eliges?
- A) Crear una VM nueva · B) Reemplazar los discos existentes · C) Restaurar discos y crear otra VM · D) Restauración de archivos

<details><summary>Respuesta</summary>

**B.**
</details>

**60.** Tras ejecutar un test failover con Site Recovery y validar la aplicación, ¿qué debes hacer para no seguir pagando los recursos de prueba?
- A) Commit · B) Limpiar la conmutación por error de prueba · C) Reprotect · D) Failback

<details><summary>Respuesta</summary>

**B.**
</details>

---

## Cómo usar este banco

> [!tip] Método
> 1. Primera pasada: responde sin mirar y anota los fallos.
> 2. Vuelve a la nota enlazada de cada fallo (no a la explicación de la respuesta).
> 3. Segunda pasada 48 horas después: solo las falladas.
> 4. Cuando aciertes el 90 % dos veces seguidas, pasa al [[07 - Simulacro final AZ-104]].

## Relacionado

- [[07 - Simulacro final AZ-104]]
- [[04 - Errores y confusiones frecuentes]]
- [[05 - Checklist final antes del examen]]
- [[00 - AZ-104 Índice general (MOC)]]
