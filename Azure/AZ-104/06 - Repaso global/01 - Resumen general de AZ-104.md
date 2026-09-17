---
tags: [az-104, azure, repaso, resumen]
modulo: Repaso global
---

# 📚 Resumen general de AZ-104

Una página por dominio con lo esencial. Si algo no te suena, vuelve a la nota enlazada.

## Dominio 1 · Identidades y gobernanza (20-25 %)

**Entra ID**: tenant = organización; una suscripción confía en **un** tenant. Roles de **Entra** (directorio) ≠ roles de **Azure RBAC** (recursos). Ediciones Free/P1/P2 (**P1** para grupos dinámicos, licencias por grupo, SSPR con writeback, unidades administrativas).

**Usuarios y grupos**: UPN con dominio verificado; **usage location** obligatoria para licencias; eliminados recuperables **30 días**; creación masiva con **CSV**. Grupos **Security** vs **Microsoft 365**; **Assigned** vs **Dynamic (P1)**; licencias por grupo solo a **miembros directos**.

**Externos**: invitados **B2B** con sus credenciales; rol **Guest Inviter**; restricciones por dominio; invitación válida **90 días**.

**SSPR**: ámbito None/**Selected (un grupo)**/All; 1 o 2 métodos; **writeback = P1 + Entra Connect**.

**RBAC**: asignación = **entidad + rol + ámbito**; herencia hacia abajo; permisos **acumulativos**. **Contributor no asigna roles ni gestiona locks**; **User Access Administrator** gestiona acceso. Roles de **datos** de Storage ≠ roles de control. Roles personalizados: **Actions − NotActions**, `AssignableScopes`; **NotActions no deniega**.

**Gobernanza**: **Policy** (definición → iniciativa → asignación; efectos **Deny, Audit, Append, Modify, AuditIfNotExists, DeployIfNotExists**; no borra lo existente; remediación con identidad administrada). **Locks** (**CanNotDelete / ReadOnly**) vencen a Owner. **Tags** (50, no se heredan). **RG** no se anida ni renombra; mover exige mismo tenant y dependencias juntas. **MG**: 6 niveles, un padre. **Costes**: presupuestos **solo alertan**; Advisor con 5 categorías.

## Dominio 2 · Almacenamiento (15-20 %)

**Cuenta**: GPv2 por defecto; **Premium solo LRS/ZRS**; nombre 3-24 minúsculas. **Redundancia**: LRS/ZRS/GRS/GZRS + variantes **RA** (lectura del secundario); failover del cliente deja **LRS**.

**Acceso en 4 capas**: **red** (firewall, service/private endpoint) → **autenticación** (clave, SAS, Entra) → **autorización** (permisos SAS o rol de datos) → **protección** (soft delete, versionado, inmutabilidad).

**SAS**: account / service / **user delegation** (Entra, solo Blob, ≤ 7 días). Revocar ad hoc = **regenerar clave**; con **directiva almacenada** (máx. 5) basta con borrarla.

**Cifrado**: SSE siempre; **CMK** con Key Vault (soft delete + purge protection) e identidad administrada; **cifrado de infraestructura solo al crear**.

**Blob**: block/append/page; acceso anónimo Private/Blob/Container; niveles **Hot / Cool (30) / Cold (90) / Archive (180, offline)**; rehidratación **Standard ≤ 15 h / High < 1 h**; **ciclo de vida** diario; **versionado** automático, **snapshots** manuales, **soft delete** 1-365 días.

**Files**: SMB (**445**) y **NFS solo Premium**; 100 TiB; identidad con **AD DS / Entra DS / Entra Kerberos** (uno por cuenta) + **RBAC de recurso compartido ∩ ACL NTFS**; snapshots (200) y soft delete; **File Sync** con cloud tiering.

**Object replication**: block blobs, **versioning en ambas + change feed en origen**.

## Dominio 3 · Cómputo (20-25 %)

**IaC**: ARM (JSON, secciones parameters/variables/resources/outputs; modo **Incremental** vs **Complete**) y **Bicep** (`param`, `var`, `resource`, `module`, `output`, `existing`; `build`/`decompile`). Exportar desde RG o desde el historial de implementaciones.

**VMs**: **Stopped factura**, **Deallocated no**; **disco temporal se pierde**; nombre Windows ≤ 15; tamaños con **"s" soportan Premium**; resize reinicia y puede requerir desasignar.

**Discos**: Standard HDD/SSD, Premium SSD, **Premium SSD v2 y Ultra solo datos**; cambiar tipo con VM desasignada; ampliar sí, reducir no; caché SO **ReadWrite**, datos **ReadOnly**, logs **None**.

**Cifrado de discos**: SSE (PMK/CMK con disk encryption set), **encryption at host** (cubre temporal), **ADE** (BitLocker/dm-crypt, Key Vault en la **misma región**, no en series A/Basic ni Ultra).

**HA**: FD **3** / UD **20**; SLA **99,9 / 99,95 / 99,99**; set y zona se eligen **al crear**. **VMSS**: **Flexible** (mezcla tamaños y Spot) vs Uniform; autoescalado con umbral, duración y **cool-down**; upgrade **Rolling** con sonda.

**Contenedores**: **ACR** (geo-replicación y private link = **Premium**; **AcrPull/AcrPush**); **ACI** (container group, restart policy **Always/OnFailure/Never**, **sin autoescalado**, Azure Files para persistencia); **Container Apps** (Entorno → App → Revisión → Réplica; **escala a cero** con min=0 y reglas HTTP/KEDA; división de tráfico).

**App Service**: el **plan** se factura; **slots Standard 5 / Premium 20** (Basic ninguno); autoescalado desde **Standard**; **slot settings no se intercambian**; dominio con **CNAME/A + TXT asuid**; certificado gratuito administrado; **backup Basic+ con límite de 10 GB**; **VNet integration = salida**, **private endpoint = entrada**.

## Dominio 4 · Redes (15-20 %)

**VNet**: una región y suscripción; **5 IPs reservadas**; subredes reservadas (**AzureBastionSubnet /26**, GatewaySubnet, AzureFirewallSubnet). **IP pública Standard**: estática, **cerrada por defecto**, zonal. **NAT Gateway** para salida fija.

**Peering**: ambos lados, **no transitivo**, sin solapamientos; **gateway transit** + **use remote gateways**.

**UDR**: prioridad **UDR > BGP > sistema**; next hop **Virtual appliance** requiere **IP forwarding**; **None** descarta.

**NSG**: prioridad 100-4096, menor gana, primera coincidencia; subred y/o NIC, **ambos deben permitir**; stateful; etiquetas de servicio. **ASG** en la configuración IP de la NIC. **IP flow verify** dice qué regla bloquea.

**Bastion**: RDP/SSH por **443** sin IP pública; **Standard** = cliente nativo y shareable links; **Premium** = grabación.

**Endpoints**: **service endpoint** (subred, IP pública del servicio, gratis, **no on-premises**) vs **private endpoint** (IP privada, por **subrecurso**, **DNS privatelink obligatorio**, sí on-premises).

**DNS**: zona pública (hospeda, **delegación NS**, **alias** para el apex) y **zona privada** (el DNS de Azure **no cruza VNets**; **un vínculo con autorregistro**; Private Resolver para híbrido).

**Load Balancer**: capa 4; frontend + backend pool + **sonda** + regla; sondas desde **168.63.129.16**; distribución 5 tuplas o **Client IP**; NAT rules. Capa 7 regional = **Application Gateway (WAF)**; global = **Front Door**; DNS = **Traffic Manager**.

## Dominio 5 · Monitorización y mantenimiento (10-15 %)

**Monitor**: métricas automáticas (**93 días**) vs logs (requieren **configuración de diagnóstico**: Log Analytics / Storage / Event Hub). **Activity log 90 días**. Agente actual **AMA + DCR**. **KQL**: `Tabla | where | summarize | render`.

**Alertas**: **métrica**, **búsqueda de registros**, **registro de actividad**; **grupo de acciones** (correo, SMS, webhook, runbook); **reglas de procesamiento** para suprimir en mantenimientos. **Insights** de VM (memoria y dependencias), Storage y Network. **Connection Monitor** para vigilancia continua; **VNet flow logs**.

**Backup**: **RSV** (VMs, Files, SQL/SAP en VM, MARS, **ASR**) vs **Backup vault** (discos, blobs, PostgreSQL, AKS); redundancia fija tras el primer elemento; **soft delete 14 días**; **CRR** con GRS. Políticas **Standard** (diaria, snapshot 1-5 d) vs **Enhanced** (cada 4 h, snapshot 1-30 d). Restaurar: crear VM, **reemplazar discos**, restaurar discos, **archivos individuales**.

**ASR**: crash-consistent cada **5 min**, retención **24 h** por defecto, **test failover** aislado + **limpieza**, **plan de recuperación**, flujo **failover → commit → reprotect → failback**.

## Cómo encajan los dominios

```
Identidad y gobernanza  →  quién puede hacer qué y bajo qué reglas
        ↓
Cómputo + Almacenamiento + Redes  →  qué se despliega y cómo se conecta
        ↓
Monitorización y backup  →  cómo se vigila y se recupera
```

## Relacionado

- [[02 - Guía de memorización (números, límites y nombres)]]
- [[03 - Tabla comparativa de servicios]]
- [[04 - Errores y confusiones frecuentes]]
- [[05 - Checklist final antes del examen]]
- [[00 - AZ-104 Índice general (MOC)]]
