---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Microsoft Entra Connect

## Concepto

**Microsoft Entra Connect** es una herramienta que se instala en un servidor local y **sincroniza** las identidades de **Active Directory local (AD DS)** hacia **[[Microsoft Entra ID]]**. Permite una **identidad híbrida**: el mismo usuario y contraseña funcionan en la red corporativa y en la nube.

Problema que resuelve: una empresa con años de historia tiene miles de usuarios en su AD local. Recrearlos a mano en la nube sería lento y provocaría contraseñas duplicadas. Entra Connect copia y mantiene sincronizados esos usuarios de forma automática.

Para qué se usa: dar a los empleados una única identidad para todo (escritorio local, Microsoft 365, Azure), sin duplicar administración.

## Características principales

- Sincroniza **usuarios, grupos y hashes de contraseña** desde AD local hacia Entra ID.
- Dirección principal: **local → nube** (con writeback opcional de algunos atributos, fuera del alcance de AZ-900).
- Métodos de autenticación híbrida (para AZ-900basta con reconocerlos):
  - **Sincronización de hash de contraseña (PHS)**: el más sencillo; Entra ID valida la contraseña en la nube. Recomendado por Microsoft.
  - **Autenticación de paso a través (PTA)**: la contraseña se valida contra el AD local en tiempo real.
  - **Federación (AD FS)**: un servidor de federación local hace la autenticación.
- Existe una alternativa ligera: **Microsoft Entra Cloud Sync** (agente pequeño, configuración desde la nube). Para AZ-900 basta saber que existe.

> [!warning] Confusión frecuente
> Entra Connect **no crea un dominio en la nube** ni ofrece Kerberos/LDAP. Solo copia identidades de AD local a Entra ID. Si la pregunta habla de "unir VMs de Azure a un dominio" la respuesta es [[Microsoft Entra Domain Services]], no Entra Connect.

## Casos de uso

- Empresa con AD local que adopta Microsoft 365: instala Entra Connect para que todos entren al correo con su cuenta de siempre.
- Migración gradual a la nube: los usuarios siguen creándose en el AD local y aparecen automáticamente en Entra ID.
- Sucursal que necesita SSO a Azure sin recrear cuentas.

## Comparaciones

| | Microsoft Entra Connect | Microsoft Entra Domain Services |
|---|---|---|
| Qué hace | Sincroniza identidades AD local → Entra ID | Provee dominio administrado (Kerberos, LDAP, GPO) en Azure |
| Dónde se instala | En un servidor Windows local | No se instala; es un servicio de Azure |
| Requiere AD local | Sí | No (puede alimentarse desde Entra ID) |
| Objetivo | Identidad híbrida, SSO a la nube | Apps legacy y VMs unidas a dominio en Azure |

## Conceptos que debo memorizar

> [!important]
> - **Entra Connect = puente AD local ↔ Entra ID** (identidad híbrida).
> - Se instala **on-premises**.
> - Sincroniza **usuarios, grupos y hashes de contraseña**.
> - Permite que la **misma cuenta** sirva para local y nube.
> - Métodos: **PHS**, **PTA**, **Federación**.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *hybrid identity*, *synchronize on-premises users*, *same credentials on-premises and cloud*.
> - Si la pregunta menciona "ya tenemos Active Directory local" y quieren usar esas cuentas en la nube → **Entra Connect**.
> - Trampa: Entra Connect no elimina el AD local; ambos conviven.
> - Trampa: Entra Connect no es un servicio de dominio ni ofrece GPO.

## Ejemplo de pregunta de examen

**Pregunta 1.** Contoso tiene Active Directory local con 5,000 usuarios y quiere que esos mismos usuarios inicien sesión en Microsoft 365 sin crear cuentas nuevas. ¿Qué debe implementar?
- A) Microsoft Entra Domain Services
- B) Microsoft Entra Connect
- C) Azure Virtual Network
- D) Microsoft Entra External ID

**Respuesta: B.** Entra Connect sincroniza las cuentas locales con Entra ID. A crea un dominio administrado en Azure, no sincroniza hacia M365. C es red. D es para usuarios externos, no empleados.

**Pregunta 2.** ¿Dónde se instala Microsoft Entra Connect?
- A) En una máquina virtual de Azure administrada por Microsoft
- B) En un servidor de la red local de la organización
- C) En el portal de Azure como extensión
- D) En cada dispositivo de usuario final

**Respuesta: B.** Es software que corre en un servidor on-premises con acceso al AD local. Las demás opciones no describen su modelo de despliegue.

## 🧠 Resumen para el examen

1. Entra Connect sincroniza AD local → Entra ID.
2. Habilita identidad híbrida (misma cuenta local y en la nube).
3. Se instala en un servidor local.
4. Sincroniza usuarios, grupos y hashes de contraseña.
5. Métodos: PHS (recomendado), PTA, Federación.
6. No es un dominio en la nube: para eso, Entra Domain Services.
7. Entra Cloud Sync es la alternativa ligera basada en agente.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
