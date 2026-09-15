---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Microsoft Entra Domain Services

## Concepto

**Microsoft Entra Domain Services** (antes Azure AD Domain Services) es un servicio **administrado por Microsoft** que ofrece **servicios de dominio tradicionales** en Azure: **unión a dominio, directivas de grupo (GPO), LDAP y autenticación Kerberos/NTLM**, sin que tengas que desplegar ni parchear controladores de dominio.

Problema que resuelve: muchas aplicaciones antiguas (**legacy**) requieren un dominio de Active Directory clásico para funcionar. [[Microsoft Entra ID]] no habla Kerberos ni LDAP. Si quieres mover esas apps a VMs de Azure, necesitas un dominio "a la antigua" en la nube. Entra Domain Services lo proporciona como servicio.

Para qué se usa: ejecutar en Azure cargas de trabajo que dependen de AD clásico, sin mantener servidores de dominio.

## Características principales

- Microsoft despliega y administra **dos controladores de dominio** en un **conjunto de réplicas** dentro de tu red virtual. Tú no tienes acceso administrativo a ellos ni necesitas parchearlos.
- **Sincronización unidireccional**: la información fluye de **Entra ID → Entra Domain Services**. Los cambios en el dominio administrado no vuelven a Entra ID.
- Cada tenant de Entra ID puede tener asociado un dominio administrado.
- En un escenario híbrido, la cadena es: **AD local → (Entra Connect) → Entra ID → Entra Domain Services**.
- Soporta unión de VMs, GPO, LDAP (incluido LDAP seguro) y Kerberos/NTLM.
- **No** soporta extensión de esquema ni confianzas de bosque completas del AD tradicional. Detalle de AZ-104; para AZ-900 basta saber que es una versión "administrada y algo limitada" del AD clásico.

> [!warning] Confusión frecuente
> - **Entra Domain Services ≠ Entra ID**. Entra ID es identidad web moderna (OAuth, SAML). Domain Services es AD clásico (Kerberos, LDAP, GPO) como servicio.
> - **Entra Domain Services ≠ AD DS en una VM de Azure**. Puedes instalar Windows Server con AD DS en una VM y administrarlo tú (IaaS). Domain Services es PaaS: Microsoft lo administra.
> - **Entra Domain Services ≠ Entra Connect**. Connect sincroniza local → nube. Domain Services entrega un dominio en la nube.

## Casos de uso

- Migrar una aplicación de 2008 que requiere autenticación Kerberos y unión a dominio a VMs de Azure → Entra Domain Services.
- Aplicar directivas de grupo a VMs de Azure sin instalar controladores de dominio.
- Organización "solo nube" (sin AD local) que necesita LDAP para una app antigua.

## Comparaciones

| | AD DS local | AD DS en VM de Azure (IaaS) | Microsoft Entra Domain Services (PaaS) | Microsoft Entra ID |
|---|---|---|---|---|
| Quién administra los DC | Tú | Tú | Microsoft | No hay DC |
| Kerberos / NTLM / LDAP | Sí | Sí | Sí | No |
| GPO | Sí | Sí | Sí | No |
| Unión a dominio de VMs | Sí | Sí | Sí | No (registro moderno) |
| Parcheo y backup | Tú | Tú | Microsoft | Microsoft |
| Extensión de esquema | Sí | Sí | No | No aplica |
| Uso típico | Red corporativa | Lift-and-shift con control total | Apps legacy en Azure sin gestionar DC | Nube, M365, SaaS, apps modernas |

## Conceptos que debo memorizar

> [!important]
> - Entra Domain Services = **dominio administrado**: unión a dominio, **GPO, LDAP, Kerberos/NTLM**.
> - Microsoft **administra los controladores de dominio**; tú no los tocas.
> - Sincronización **unidireccional: Entra ID → Domain Services**.
> - Para **aplicaciones legacy** que requieren AD clásico en Azure.
> - Es **PaaS**, no IaaS.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *domain join*, *group policy*, *LDAP*, *Kerberos*, *NTLM*, *legacy applications*, *without deploying domain controllers*.
> - Si la pregunta menciona cualquiera de esas palabras y añade "sin administrar/mantener controladores de dominio" → Entra Domain Services.
> - Si dice "queremos control total sobre el dominio, extender el esquema" → AD DS en una VM (IaaS), no Domain Services.
> - Trampa: "Entra Domain Services sincroniza de vuelta a Entra ID". Falso, es unidireccional.
> - Trampa: "Entra ID soporta directivas de grupo". Falso. Solo Domain Services (o AD DS).

## Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa migra a Azure una aplicación antigua que requiere autenticación Kerberos y unión a dominio. No quiere administrar controladores de dominio. ¿Qué servicio debe usar?
- A) Microsoft Entra ID
- B) Microsoft Entra Domain Services
- C) Microsoft Entra Connect
- D) Microsoft Entra External ID

**Respuesta: B.** Domain Services proporciona Kerberos y unión a dominio como servicio administrado. A no soporta Kerberos. C sincroniza AD local hacia la nube. D es para usuarios externos.

**Pregunta 2.** ¿Cuál de las siguientes características ofrece Microsoft Entra Domain Services pero NO Microsoft Entra ID?
- A) Autenticación multifactor
- B) Inicio de sesión único para aplicaciones SaaS
- C) Directivas de grupo (GPO)
- D) Acceso condicional

**Respuesta: C.** Las GPO son una función del dominio clásico que solo ofrece Domain Services. A, B y D son funciones de Entra ID.

**Pregunta 3.** ¿En qué dirección fluye la sincronización entre Microsoft Entra ID y Microsoft Entra Domain Services?
- A) Bidireccional
- B) De Domain Services hacia Entra ID
- C) De Entra ID hacia Domain Services
- D) No existe sincronización entre ambos

**Respuesta: C.** El dominio administrado recibe usuarios y grupos desde Entra ID; los cambios no regresan. A y B son incorrectas por la unidireccionalidad. D es falsa: sí existe sincronización.

## 🧠 Resumen para el examen

1. Entra Domain Services = AD clásico como servicio administrado en Azure.
2. Ofrece unión a dominio, GPO, LDAP, Kerberos y NTLM.
3. Microsoft administra los controladores de dominio (PaaS).
4. Sincronización unidireccional: Entra ID → Domain Services.
5. Para apps legacy que no funcionan con identidad moderna.
6. Entra ID no tiene GPO ni Kerberos; Domain Services sí.
7. Si necesitas control total (esquema, confianzas), usa AD DS en VM (IaaS).
8. Cadena híbrida: AD local → Connect → Entra ID → Domain Services.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
