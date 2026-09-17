---
tags: [az-104, azure, identidad, gobernanza, repaso]
modulo: Identidades y gobernanza
---

# 🎯 Repaso final · Identidades y gobernanza (20-25 %)

Úsalo el día antes del examen. Si algo falla, vuelve a la nota enlazada.

## 1. Los conceptos más importantes

1. **Tenant** = organización; una suscripción confía en un solo tenant. Roles de **Entra** administran el directorio; **RBAC** administra recursos. ([[01 - Microsoft Entra ID para administradores]])
2. **Usuarios**: UPN con dominio verificado; **usage location** para licencias; eliminados recuperables **30 días**; creación masiva por **CSV**. ([[02 - Usuarios de Microsoft Entra ID]])
3. **Grupos**: Security vs M365; Assigned vs **Dynamic (P1)**; asignar roles y licencias a grupos. ([[03 - Grupos de Microsoft Entra ID]])
4. **Licencias por grupo** = P1, solo miembros directos. ([[04 - Licencias en Microsoft Entra ID]])
5. **Invitados B2B** usan sus credenciales; rol **Guest Inviter**; restricciones de dominio. ([[05 - Usuarios externos e invitados (B2B)]])
6. **SSPR**: None/Selected (**un grupo**)/All; 1 o 2 métodos; **writeback = P1 + Entra Connect**. ([[06 - Self-Service Password Reset (SSPR)]])
7. **RBAC** = entidad + rol + ámbito; herencia hacia abajo; permisos **acumulativos**; **Contributor no asigna roles ni gestiona locks**. ([[08 - Azure RBAC - roles integrados y ámbitos]])
8. **Roles personalizados**: Actions − NotActions; **NotActions no deniega**; AssignableScopes. ([[09 - Roles personalizados de Azure RBAC]])
9. **Policy**: definición → iniciativa → asignación; efectos **Deny/Audit/Append/Modify/AuditIfNotExists/DeployIfNotExists**; no borra existentes; remediación con identidad administrada. ([[11 - Azure Policy]])
10. **Locks**: **CanNotDelete / ReadOnly**; vencen a Owner; solo Owner/UAA los gestionan. ([[12 - Bloqueos de recursos (Locks)]])
11. **Tags**: 50 por recurso; **no se heredan** (Policy Modify para heredar). ([[13 - Etiquetas (Tags)]])
12. **RG**: no se anida ni renombra; región = metadatos; mover recursos = mismo tenant, dependencias juntas, RGs bloqueados durante el movimiento. ([[14 - Grupos de recursos y movimiento de recursos]])
13. **MG**: 6 niveles, 10 000 por tenant, un padre; suscripción = facturación + cuotas. ([[15 - Suscripciones y grupos de administración]])
14. **Costes**: presupuestos **no bloquean**; alertas de presupuesto/crédito/cuota; Advisor con 5 categorías. ([[16 - Administración de costes (presupuestos, alertas y Advisor)]])

## 2. Tabla de "rol mínimo para…"

| Tarea | Rol mínimo |
|---|---|
| Crear/borrar usuarios | User Administrator (Entra) |
| Restablecer contraseñas de usuarios | Password Administrator (Entra) |
| Crear grupos | Groups Administrator (Entra) |
| Asignar licencias | License Administrator (Entra) |
| Invitar externos | Guest Inviter (Entra) |
| Gestionar recursos sin asignar roles | Contributor (RBAC) |
| Asignar roles sin gestionar recursos | User Access Administrator (RBAC) |
| Crear/quitar locks | Owner o User Access Administrator |
| Asignar Azure Policy | Resource Policy Contributor |
| Crear presupuestos | Cost Management Contributor |
| Solo ver costes | Cost Management Reader / Billing Reader |
| Solo etiquetas | Tag Contributor |
| Administrar VMs sin tocar la red | Virtual Machine Contributor |
| Leer blobs con Entra ID | Storage Blob Data Reader |

## 3. Números que debo memorizar

| Dato | Valor |
|---|---|
| Usuario eliminado recuperable | 30 días |
| Invitación B2B válida | 90 días |
| Reconfirmación de registro SSPR | 180 días (por defecto) |
| Métodos SSPR requeridos | 1 o 2 |
| Dispositivos por usuario | 50 |
| Grupos asignables a roles | 500 |
| Asignaciones RBAC por suscripción | 4000 |
| Asignaciones RBAC por MG | 500 |
| Roles personalizados por tenant | 5000 |
| Tags por recurso | 50 (nombre 512 / 128 storage; valor 256) |
| RGs por suscripción | 980 |
| Niveles de MG | 6 |
| MGs por tenant | 10 000 |
| Evaluación de Policy | cada 24 h (y ~30 min tras asignar) |
| Suscripción deshabilitada conserva recursos | 90 días |
| Sincronización de Entra Connect | 30 min |

## 4. Diferencias que más fácil puedo confundir

| Pareja | La diferencia en una línea |
|---|---|
| Roles de Entra vs Azure RBAC | Directorio vs recursos; Global Admin no ve suscripciones sin elevar |
| Owner vs Contributor | Contributor no asigna roles ni gestiona locks |
| UAA vs RBAC Administrator | UAA gestiona todo el acceso; RBAC Admin solo asigna roles (con condiciones) |
| NotActions vs Deny | NotActions recorta el rol; no deniega frente a otros roles |
| Deny (Policy) vs Lock | Policy bloquea creaciones/cambios por regla; Lock bloquea borrado/modificación a todos |
| Append vs Modify | Append solo al crear; Modify también existentes con remediación |
| AuditIfNotExists vs DeployIfNotExists | Informa vs despliega (necesita identidad) |
| Exclusión vs Exención | Al asignar vs después, con caducidad |
| Security group vs M365 group | Permisos vs colaboración (Teams/SharePoint) |
| Assigned vs Dynamic | Manual vs regla (P1) |
| Member vs Guest | Interno vs externo B2B |
| Cambio vs restablecimiento de contraseña | Free vs licencia (P1 o M365 Business Standard+) |
| Mover a otro RG vs mover región | ARM move vs Resource Mover |
| Suscripción vs MG | Facturación/cuotas vs gobernanza |
| Presupuesto vs Policy de coste | Presupuesto alerta; no bloquea |
| Advisor vs Monitor vs Service Health | Recomendaciones vs telemetría vs incidencias de Azure |

## 5. Checklist de dominio

- [ ] Sé crear usuarios (individual, CSV, CLI/Graph) y restaurarlos.
- [ ] Sé qué es la ubicación de uso y por qué falla una licencia sin ella.
- [ ] Sé escribir una regla de grupo dinámico y sé que requiere P1.
- [ ] Sé configurar licencias por grupo y sus limitaciones (anidamiento).
- [ ] Sé invitar externos, restringir dominios y cuál es el rol Guest Inviter.
- [ ] Sé configurar SSPR (ámbito, métodos, registro, writeback y licencias).
- [ ] Sé qué son AUs y los tres estados de dispositivos.
- [ ] Sé los roles integrados clave y qué no puede hacer Contributor.
- [ ] Sé leer/crear un rol personalizado JSON y qué significa NotActions.
- [ ] Sé calcular permisos efectivos con herencia, grupos, locks y Policy.
- [ ] Sé todos los efectos de Policy y cuándo hace falta identidad administrada.
- [ ] Sé qué pasa con los recursos existentes al asignar Deny.
- [ ] Sé los dos tipos de lock y sus efectos secundarios (storage keys, VM start).
- [ ] Sé los límites de tags y cómo heredarlas.
- [ ] Sé las reglas para mover recursos (tenant, dependencias, bloqueo, RBAC).
- [ ] Sé la jerarquía de MGs, sus límites y qué se hereda.
- [ ] Sé crear presupuestos con alertas y action groups, y qué hace Advisor.

## 6. Preguntas de repaso

**1.** Un usuario pertenece al grupo "IT" que tiene Owner en la suscripción, y tiene una asignación directa de Reader en RG1. ¿Puede eliminar RG1?
- A) No, Reader lo impide · B) Sí, Owner heredado se suma · C) Solo si es Global Administrator · D) No, Owner no incluye eliminación de RGs

<details><summary>Respuesta</summary>**B.** Permisos acumulativos; salvo que exista un lock.</details>

---

**2.** Necesitas impedir que se creen máquinas virtuales de la serie M en todas las suscripciones del grupo de administración "Corp". ¿Qué usas?
- A) Lock ReadOnly en el MG · B) Azure Policy "Allowed virtual machine size SKUs" con Deny asignada al MG · C) RBAC Reader · D) Presupuesto

<details><summary>Respuesta</summary>**B.** Policy con Deny en el MG se hereda a todas las suscripciones.</details>

---

**3.** Asignas un bloqueo ReadOnly a una cuenta de almacenamiento. Los usuarios informan de que el portal ya no muestra los contenedores. ¿Por qué?
- A) ReadOnly cifra los datos · B) Listar las claves de acceso es una operación POST bloqueada por ReadOnly · C) Se perdió el rol Reader · D) Es un error de Azure

<details><summary>Respuesta</summary>**B.** El portal usa las claves para mostrar datos; ReadOnly impide listarlas.</details>

---

**4.** ¿Qué edición de Entra ID necesitas como mínimo para grupos dinámicos y SSPR con writeback?
- A) Free · B) P1 · C) P2 · D) Governance

<details><summary>Respuesta</summary>**B.** P1.</details>

---

**5.** Un rol personalizado tiene `Actions: ["Microsoft.Compute/*"]` y `NotActions: ["Microsoft.Compute/virtualMachines/delete"]`. Se asigna a un usuario que no tiene ningún otro rol. ¿Puede borrar VMs?
- A) Sí · B) No · C) Solo con lock · D) Solo VMs sin discos

<details><summary>Respuesta</summary>**B.** Sin otro rol que lo conceda, NotActions lo excluye.</details>

---

**6.** Necesitas que la etiqueta `Env` del grupo de recursos se aplique a todos los recursos existentes y nuevos del RG. ¿Qué efecto de Policy usas?
- A) Deny · B) Audit · C) Append · D) Modify con remediación

<details><summary>Respuesta</summary>**D.**</details>

---

**7.** ¿Qué ocurre al transferir una suscripción a otro tenant de Entra ID?
- A) Nada cambia · B) Se eliminan las asignaciones RBAC y las identidades administradas dejan de funcionar · C) Solo se pierden las etiquetas · D) Se eliminan los recursos

<details><summary>Respuesta</summary>**B.**</details>

---

**8.** Tu presupuesto mensual alcanza el 100 % pero las VMs siguen ejecutándose. ¿Por qué?
- A) La alerta tarda 30 días · B) Los presupuestos solo alertan; para actuar hay que enlazar un grupo de acciones con automatización · C) Falta el rol Owner · D) Advisor no está habilitado

<details><summary>Respuesta</summary>**B.**</details>

---

**9.** Debes mover una VM, su NIC y su disco a otro grupo de recursos. Durante el movimiento, ¿qué ocurre con la VM?
- A) Se apaga · B) Sigue funcionando, pero los RGs origen y destino quedan bloqueados para escritura · C) Cambia de región · D) Pierde la IP

<details><summary>Respuesta</summary>**B.**</details>

---

**10.** El helpdesk debe poder restablecer contraseñas únicamente de los usuarios de la unidad administrativa "Sucursal-Norte". ¿Qué rol y ámbito?
- A) Global Administrator en el tenant · B) Password Administrator con ámbito en la AU · C) User Access Administrator en la suscripción · D) Contributor en el RG

<details><summary>Respuesta</summary>**B.**</details>

> [!tip] Última pasada
> Si solo puedes repasar cuatro cosas: **Contributor no asigna roles ni gestiona locks**, **Policy no borra existentes**, **locks vencen a Owner**, **MG 6 niveles / tags 50 / RBAC 4000**.

Volver: [[00 - Índice - Identidades y gobernanza]] · [[00 - AZ-104 Índice general (MOC)]]
