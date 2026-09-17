---
tags: [az-104, azure, almacenamiento, repaso]
modulo: Almacenamiento
---

# 🎯 Repaso final · Almacenamiento (15-20 %)

## 1. Los conceptos más importantes

1. **Cuenta**: GPv2 por defecto; Premium (block blob / FileStorage) solo LRS/ZRS; nombre 3-24 minúsculas+números único. ([[01 - Cuentas de almacenamiento]])
2. **Redundancia**: LRS 11 nueves / ZRS 12 / GRS-GZRS 16; **RA** = lectura del secundario; failover del cliente deja LRS. ([[02 - Redundancia de almacenamiento]])
3. **Firewall**: All / Selected (subredes con service endpoint + IPs públicas) / Disabled (solo private endpoints); excepción de servicios de confianza; el portal deja de ver datos si tu IP no está permitida. ([[03 - Firewalls y redes virtuales de Azure Storage]])
4. **SAS**: account / service / **user delegation** (Entra, ≤ 7 días, solo Blob); revocar ad hoc = regenerar clave; **stored access policy** (máx. 5) permite revocar. ([[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]])
5. **Claves**: dos para rotar; roles de **datos** (Blob Data Reader/Contributor/Owner) ≠ Contributor; deshabilitar acceso por clave rompe SAS de cuenta/servicio. ([[05 - Claves de acceso y autorización con Microsoft Entra ID]])
6. **Cifrado**: siempre activo; **CMK** = Key Vault con soft delete + purge protection + identidad administrada; **cifrado de infraestructura solo al crear**. ([[06 - Cifrado de cuentas de almacenamiento]])
7. **Object replication**: block blobs, versioning en ambas + change feed en origen; 10 reglas, 2 destinos; asíncrona; cuentas/regiones/suscripciones distintas. ([[07 - Replicación de objetos (Object Replication)]])
8. **AzCopy** (copy/sync, Entra o SAS, server-to-server, reanudable) vs **Storage Explorer** (GUI). ([[08 - Azure Storage Explorer y AzCopy]])
9. **Blob**: block/append/page; acceso anónimo Private/Blob/Container; `$web`; inmutabilidad (time-based / legal hold). ([[09 - Azure Blob Storage]])
10. **Niveles**: Hot / Cool 30 / Cold 90 / Archive 180 días; Archive offline; rehidratar Standard ≤ 15 h, High < 1 h. ([[10 - Niveles de acceso de Blob (Hot, Cool, Cold, Archive)]])
11. **Ciclo de vida**: reglas por modificación/creación/último acceso; tierToCool/Cold/Archive/delete; diario; hasta 24 h. ([[11 - Administración del ciclo de vida de Blob]])
12. **Protección Blob**: versionado (auto) / snapshot (manual) / soft delete blobs y contenedores (1-365 días) / PITR. ([[12 - Versionado, instantáneas y eliminación temporal de Blob]])
13. **Files**: SMB 445 / NFS solo Premium; 100 TiB; niveles Transaction optimized/Hot/Cool; sin RA. ([[13 - Azure Files]])
14. **Identidad en Files**: AD DS / Entra DS / Entra Kerberos (uno por cuenta); RBAC share (Reader/Contributor/Elevated) ∩ ACL NTFS; permiso predeterminado. ([[14 - Acceso basado en identidad para Azure Files]])
15. **Files protección**: 200 snapshots; soft delete 1-365 días restaura el recurso con snapshots. ([[15 - Instantáneas y eliminación temporal en Azure Files]])
16. **File Sync**: Storage Sync Service → sync group → 1 cloud endpoint + N server endpoints; cloud tiering; agente WS2016+. ([[16 - Azure File Sync]])

## 2. Tabla de decisión rápida

| Si el enunciado dice… | Respuesta |
|---|---|
| "sin compartir la clave", "temporal", "caduca" | SAS |
| "revocar sin regenerar claves" | Stored access policy |
| "sin usar claves de cuenta", "con Entra ID" | User delegation SAS / rol de datos |
| "solo desde la subred X" | Firewall + service endpoint |
| "desde on-premises con IP privada", "nunca por Internet" | Private endpoint |
| "Backup/Monitor deben seguir accediendo con firewall" | Servicios de confianza |
| "claves propias", "rotación controlada" | CMK en Key Vault |
| "doble cifrado" | Cifrado de infraestructura (al crear) |
| "leer en la región secundaria durante incidencia" | RA-GRS / RA-GZRS |
| "sobrevivir a fallo de zona y de región" | GZRS |
| "copiar blobs a otra cuenta/región de mi elección" | Object replication |
| "línea de comandos, reanudable, sync" | AzCopy |
| "interfaz gráfica" | Storage Explorer |
| "puede tardar horas en recuperarse" | Archive |
| "menos de una hora desde Archive" | Rehidratación High |
| "mover automáticamente según antigüedad" | Ciclo de vida |
| "recuperar versión anterior automáticamente" | Versionado |
| "recuperar contenedor borrado" | Soft delete de contenedores |
| "no modificable ni borrable N años" | Inmutabilidad time-based |
| "NFS" | Premium FileStorage |
| "puerto 445 bloqueado" | VPN/ExpressRoute o File Sync |
| "permisos NTFS por usuario en Azure Files" | Identidad (AD DS / Entra DS / Entra Kerberos) |
| "sin línea de visión al DC" | Entra Kerberos |
| "caché local en sucursales" | File Sync con cloud tiering |

## 3. Números que debo memorizar

| Dato | Valor |
|---|---|
| Nombre de cuenta | 3-24 caracteres |
| Durabilidad LRS / ZRS / GRS | 11 / 12 / 16 nueves |
| Reglas de firewall | 200 IP + 200 VNet |
| Stored access policies por contenedor | 5 |
| User delegation SAS máx. | 7 días |
| Retención mínima Cool / Cold / Archive | 30 / 90 / 180 días |
| Rehidratación Standard / High | ≤ 15 h / < 1 h (< 10 GB) |
| Reglas de ciclo de vida | 100; ejecución diaria, hasta 24 h |
| Soft delete blobs / contenedores / Files | 1-365 días |
| Object replication | 10 reglas por directiva, 2 destinos por origen |
| Tamaño máx. recurso compartido | 100 TiB |
| Snapshots por recurso compartido | 200 |
| Puerto SMB | 445 |
| Cloud endpoints por sync group | 1 |
| Agente File Sync | Windows Server 2016+ |

## 4. Diferencias que más fácil puedo confundir

| Pareja | Diferencia |
|---|---|
| Service endpoint vs Private endpoint | Subred/IP pública del servicio/gratis vs IP privada/on-premises/de pago |
| Account SAS vs Service SAS | Varios servicios vs un recurso |
| SAS ad hoc vs con directiva | Revocar regenerando clave vs borrando directiva |
| Storage Account Contributor vs Blob Data Contributor | Control (lista claves) vs datos |
| GRS vs RA-GRS | Sin lectura vs con lectura del secundario |
| GRS vs Object replication | Toda la cuenta, región emparejada vs selectivo, destino elegido |
| Versionado vs Snapshot | Automático vs manual |
| Soft delete blob vs contenedor vs Files | Tres configuraciones distintas |
| Cool vs Cold | 30 vs 90 días; ambos online |
| Cold vs Archive | Online vs offline (rehidratar) |
| Set Blob Tier vs Copy Blob (Archive) | Mueve el blob vs conserva el original archivado |
| Azure Files vs File Sync | Servicio vs sincronización con servidores |
| AD DS vs Entra DS vs Entra Kerberos | DC local vs dominio administrado vs sin DC |
| SMB Share Contributor vs Elevated Contributor | Sin/con cambio de ACL NTFS |
| Cifrado SSE vs Azure Disk Encryption | Servicio de storage vs dentro del SO de la VM |

## 5. Checklist de dominio

- [ ] Sé elegir tipo de cuenta, rendimiento y redundancia según requisitos.
- [ ] Sé qué se puede cambiar tras crear la cuenta.
- [ ] Sé configurar el firewall (modos, subredes, IPs, excepciones) y sus efectos en el portal.
- [ ] Sé distinguir service endpoint y private endpoint para Storage.
- [ ] Sé crear los tres tipos de SAS y directivas de acceso; sé revocar.
- [ ] Sé rotar claves sin cortes y qué roles de datos hacen falta con Entra ID.
- [ ] Sé configurar CMK y cifrado de infraestructura.
- [ ] Sé los requisitos de object replication.
- [ ] Sé usar AzCopy (copy, sync, login) y Storage Explorer.
- [ ] Sé crear contenedores con el nivel de acceso correcto y el sitio estático.
- [ ] Sé los cuatro niveles, retenciones y rehidratación.
- [ ] Sé escribir una regla de ciclo de vida.
- [ ] Sé habilitar versionado, snapshots, soft delete y restaurar.
- [ ] Sé crear y montar recursos compartidos SMB/NFS y el tema del puerto 445.
- [ ] Sé configurar identidad en Files y sus dos niveles de permisos.
- [ ] Sé snapshots y soft delete de Files.
- [ ] Sé los componentes y el orden de despliegue de File Sync.

## 6. Preguntas de repaso

**1.** Necesitas que una aplicación en una VM de Azure y un servidor on-premises conectado por ExpressRoute accedan a una cuenta de almacenamiento que no debe tener exposición pública. ¿Qué configuras?
- A) Service endpoint · B) Private endpoint + acceso público deshabilitado · C) Regla IP con rango privado · D) SAS

<details><summary>Respuesta</summary>**B.**</details>

---

**2.** ¿Qué tipo de SAS se firma con credenciales de Microsoft Entra ID?
- A) Account SAS · B) Service SAS · C) User delegation SAS · D) Stored access policy

<details><summary>Respuesta</summary>**C.**</details>

---

**3.** Un usuario tiene Contributor en la suscripción y no puede leer blobs con `az storage blob list --auth-mode login`. ¿Qué le falta?
- A) Owner · B) Storage Blob Data Reader · C) Clave de acceso · D) Nada, es un bug

<details><summary>Respuesta</summary>**B.**</details>

---

**4.** ¿Qué redundancia elegirías para una cuenta Premium block blob que debe sobrevivir a la caída de un centro de datos?
- A) LRS · B) ZRS · C) GRS · D) GZRS

<details><summary>Respuesta</summary>**B.** Premium solo admite LRS y ZRS.</details>

---

**5.** Un blob se subió a Cold hace 20 días y se cambia a Hot. ¿Qué ocurre?
- A) Nada · B) Cargo por eliminación anticipada por los 70 días restantes · C) No se puede cambiar · D) Se rehidrata en 15 h

<details><summary>Respuesta</summary>**B.** Cold tiene 90 días de retención mínima.</details>

---

**6.** Necesitas que los blobs no accedidos en 60 días pasen a Cool automáticamente. ¿Qué dos cosas configuras?
- A) Versionado y soft delete · B) Access time tracking y una regla de ciclo de vida por último acceso · C) Change feed y object replication · D) Snapshot y lifecycle

<details><summary>Respuesta</summary>**B.**</details>

---

**7.** Configuras object replication y falla por falta de requisitos. La cuenta de destino tiene versioning. ¿Qué falta en el origen?
- A) Soft delete · B) Versioning y change feed · C) GRS · D) Private endpoint

<details><summary>Respuesta</summary>**B.**</details>

---

**8.** Debes copiar 10 TB entre dos cuentas de regiones distintas desde un script, con posibilidad de reanudar. ¿Qué herramienta?
- A) Storage Explorer · B) AzCopy · C) Portal · D) Robocopy

<details><summary>Respuesta</summary>**B.**</details>

---

**9.** ¿Qué origen de identidad para Azure Files permite acceder desde portátiles unidos a Entra ID sin conectividad con un controlador de dominio?
- A) AD DS · B) Entra Domain Services · C) Entra Kerberos · D) Clave de cuenta

<details><summary>Respuesta</summary>**C.**</details>

---

**10.** Un administrador eliminó un recurso compartido de Azure Files con 50 instantáneas. Soft delete está habilitado con 14 días. ¿Qué se puede recuperar?
- A) Solo el recurso compartido sin instantáneas · B) El recurso compartido con sus instantáneas · C) Nada · D) Solo las instantáneas

<details><summary>Respuesta</summary>**B.**</details>

---

**11.** Un requisito exige que los datos de una cuenta nueva estén cifrados con dos capas independientes. ¿Qué opción marcas al crearla?
- A) CMK · B) Cifrado de infraestructura · C) HTTPS obligatorio · D) TLS 1.2

<details><summary>Respuesta</summary>**B.**</details>

---

**12.** ¿Cuántos server endpoints puede tener un sync group de Azure File Sync?
- A) 1 · B) 2 · C) Varios · D) Ninguno

<details><summary>Respuesta</summary>**C.** Uno por servidor registrado; el cloud endpoint es único.</details>

> [!tip] Última pasada
> **Service vs Private endpoint**, **tres tipos de SAS y cómo se revocan**, **roles de datos ≠ Contributor**, **Cool 30 / Cold 90 / Archive 180**, **versioning + change feed para object replication**, **445 y NFS solo Premium**.

Volver: [[00 - Índice - Almacenamiento]] · [[00 - AZ-104 Índice general (MOC)]]
