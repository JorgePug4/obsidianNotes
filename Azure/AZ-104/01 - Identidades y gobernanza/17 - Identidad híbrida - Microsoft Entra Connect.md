---
tags: [az-104, azure, identidad, entra-connect, hibrido]
modulo: Identidades y gobernanza
peso_examen: Medio (contexto)
---

# Identidad híbrida · Microsoft Entra Connect ➕

> [!info] No es un objetivo literal del temario vigente, pero muchos escenarios de AZ-104 (usuarios "sincronizados", SSPR con writeback, Azure Files con AD DS) lo dan por sabido. Repaso conceptual en [[Microsoft Entra Connect]] (AZ-900).

## ¿Qué es?

**Microsoft Entra Connect** (Sync) es la aplicación que se instala en un servidor local para **sincronizar** usuarios, grupos y dispositivos de **Active Directory Domain Services** con Microsoft Entra ID, y para elegir cómo se autentican los usuarios híbridos. **Microsoft Entra Cloud Sync** es la alternativa ligera basada en un agente de aprovisionamiento gestionado desde la nube.

## ¿Para qué sirve?

- Que los empleados usen la **misma cuenta** en Windows local, Microsoft 365 y Azure.
- Habilitar SSPR con writeback, Azure Files con AD DS, dispositivos híbridos.

## Conceptos clave

- **Métodos de autenticación** 🧠:
  - **Password Hash Synchronization (PHS)**: sincroniza un hash del hash de la contraseña a Entra; la autenticación ocurre en la nube. Recomendado; funciona aunque el AD local esté caído; habilita Identity Protection de credenciales filtradas.
  - **Pass-through Authentication (PTA)**: la contraseña se valida contra el AD local mediante agentes; nada de hashes en la nube; requiere agentes disponibles.
  - **Federación (AD FS)**: Entra delega la autenticación a AD FS; más complejo; para requisitos como tarjetas inteligentes o MFA de terceros locales.
- **Seamless SSO**: inicio de sesión automático desde equipos unidos al dominio (con PHS o PTA).
- **Writeback**: de contraseñas (SSPR), de dispositivos, de grupos (Cloud Sync).
- **Filtrado**: por OU, dominio, grupo o atributos, para no sincronizar todo.
- **Atributo de origen (source anchor)**: `ms-DS-ConsistencyGuid` por defecto.
- **Entra Connect Health**: monitoriza la sincronización, AD FS y AD DS desde el portal.
- **Staging mode**: servidor secundario pasivo para alta disponibilidad (solo uno activo exporta).
- **Intervalo de sincronización**: cada **30 minutos** por defecto. Forzar: `Start-ADSyncSyncCycle -PolicyType Delta` (o `Initial`).
- **Requisitos**: Windows Server (2016+), SQL Server Express incluido (LocalDB) o SQL completo para >100 000 objetos, cuenta Global Administrator/Hybrid Identity Administrator para la instalación, cuenta de AD con permisos.
- **Cloud Sync vs Connect Sync**: Cloud Sync es más simple, multi-bosque desconectado, no soporta writeback de dispositivos ni algunos escenarios (Exchange híbrido complejo, filtrado por atributo).

## Cómo funciona

```
AD DS local  ──(Entra Connect Sync / Cloud Sync agent)──►  Microsoft Entra ID
   usuarios, grupos, contactos, dispositivos              usuarios "Windows Server AD" (source)
   ◄──── writeback: contraseñas (SSPR), grupos, dispositivos ────
Autenticación: PHS (nube) | PTA (agentes locales) | Federación (AD FS)
```

## Configuración relevante para el examen

| Escenario | Qué necesitas |
|---|---|
| SSPR para usuarios sincronizados | Password writeback habilitado + P1 |
| Editar el departamento de un usuario sincronizado | Cambiarlo en **AD DS**; Entra no permite editarlo |
| Los cambios locales tardan en verse | Esperar el ciclo de 30 min o forzar `Start-ADSyncSyncCycle -PolicyType Delta` |
| Solo sincronizar dos OUs | Filtrado por OU en el asistente |
| Servidor de sincronización caído | Segundo servidor en **staging mode** y promover |
| Usuarios sin AD local caído siguen entrando | **PHS** |
| No permitir hashes en la nube | **PTA** o federación |
| Azure Files con permisos NTFS de AD | Identidades sincronizadas + habilitar AD DS auth en la cuenta ([[14 - Acceso basado en identidad para Azure Files]]) |

## Comparaciones

| Método | Dónde se valida la contraseña | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **PHS** | Nube | Sencillo, resiliente, protección de credenciales | Por defecto |
| **PTA** | AD local (agentes) | No hay hashes en la nube; políticas locales inmediatas | Requisito de no almacenar hashes |
| **Federación (AD FS)** | AD FS local | Requisitos avanzados (smart card, MFA de terceros) | Solo si es imprescindible |
| **Cloud Sync** | Igual que Connect (PHS) | Agente ligero, gestión desde la nube, multi-bosque | Entornos simples o desconectados |

## AZ-104 Exam Tips

- 🧠 Sincronización cada **30 minutos**; forzar con `Start-ADSyncSyncCycle -PolicyType Delta`.
- 🧠 Usuario sincronizado: atributos se editan **en AD local**.
- 📌 PHS (nube, resiliente) vs PTA (validación local, sin hashes) vs AD FS (federación).
- 📌 Connect Sync (completo) vs Cloud Sync (ligero, agente).
- ⚠️ Writeback de contraseñas = **P1** + opción habilitada en Entra Connect.
- ⚠️ Solo **un** servidor de Entra Connect activo por tenant (los demás en staging).

## Preguntas que podrían aparecer

**1.** Un administrador cambia el número de teléfono de un usuario sincronizado en el portal de Entra y la opción está deshabilitada. ¿Qué debe hacer?
- A) Asignar P1 · B) Cambiarlo en Active Directory local y esperar la sincronización · C) Eliminar el usuario y recrearlo en la nube · D) Habilitar writeback

<details><summary>Respuesta</summary>

**B.** Los atributos de los usuarios sincronizados se administran en AD DS; Entra Connect los replica en el siguiente ciclo.
</details>

**2.** La empresa exige que los hashes de contraseña nunca salgan de su centro de datos, pero quiere SSO en Microsoft 365 sin desplegar AD FS. ¿Qué método eliges?
- A) Password Hash Synchronization · B) Pass-through Authentication · C) Federación · D) Cloud Sync con PHS

<details><summary>Respuesta</summary>

**B.** PTA valida la contraseña en el AD local mediante agentes, sin sincronizar hashes, y es más simple que AD FS.
</details>

## Relacionado

- [[06 - Self-Service Password Reset (SSPR)]]
- [[02 - Usuarios de Microsoft Entra ID]]
- [[14 - Acceso basado en identidad para Azure Files]]
- [[Microsoft Entra Connect]] (AZ-900)
- [[00 - Índice - Identidades y gobernanza]]
