---
tags: [az-104, azure, identidad, entra-id, licencias]
modulo: Identidades y gobernanza
peso_examen: Medio
---

# Licencias en Microsoft Entra ID

## ¿Qué es?

Las **licencias** determinan qué servicios y funciones de Microsoft (Microsoft 365, Entra ID P1/P2, EMS, Dynamics, etc.) puede usar cada usuario. Se compran como **suscripciones de producto** con un número de unidades y se **asignan** a usuarios directamente o a través de grupos.

## ¿Para qué sirve?

- Habilitar Exchange, Teams, Office, Intune, Entra ID P1/P2 por usuario.
- Controlar coste: cada licencia asignada consume una unidad.
- Delegar la asignación con grupos para que sea automática.

## Conceptos clave

- **Producto (SKU)** y **planes de servicio**: un producto (por ejemplo Microsoft 365 E5) contiene varios planes (Exchange Online, Teams, Entra ID P2…). Al asignar se pueden **deshabilitar planes concretos**.
- **Ubicación de uso (usage location)**: propiedad del usuario, obligatoria por restricciones legales de exportación. Sin ella la asignación falla.
- **Asignación directa** vs **asignación basada en grupos** (group-based licensing).
- **Licencias heredadas vs directas**: si un usuario tiene la misma licencia por grupo y directa, al quitar la directa mantiene la de grupo.
- **Errores de licenciamiento**: "No hay licencias suficientes", "conflicto de planes de servicio", "falta ubicación de uso". Se ven en el grupo (Licencias → errores) o en el usuario.
- **Licencias de Entra ID**: Free, **P1**, **P2** (ver [[01 - Microsoft Entra ID para administradores]]). Las funciones premium exigen licencia para cada usuario que las use.

## Cómo funciona (licencias por grupo)

1. Requisitos: tener **Entra ID P1** (o M365 E3/A3/G3, Business Premium…) para usar licencias por grupo.
2. Crear un grupo de seguridad (asignado o dinámico).
3. Grupo → **Licencias** → Asignaciones → elegir producto y planes → Guardar.
4. Entra procesa la asignación a todos los miembros; los nuevos miembros reciben la licencia automáticamente y quien sale del grupo la pierde.
5. El anidamiento **no** se propaga: solo miembros directos.

```powershell
# Microsoft Graph PowerShell: asignar licencia directa
Set-MgUserLicense -UserId ana@contoso.com -AddLicenses @{SkuId = "<skuId>"} -RemoveLicenses @()
Get-MgSubscribedSku | Select-Object SkuPartNumber, ConsumedUnits, @{n="Total";e={$_.PrepaidUnits.Enabled}}
```

## Componentes

| Elemento | Detalle |
|---|---|
| Productos comprados | Portal → Entra ID → Licencias → Todos los productos |
| Unidades asignadas / disponibles | Se ve por producto |
| Asignación a usuario | Usuario → Licencias |
| Asignación a grupo | Grupo → Licencias |
| Planes de servicio | Se activan/desactivan por asignación |
| Rol mínimo | **License Administrator** (o User Administrator / Global Administrator) |

## Configuración relevante para el examen

- Rol mínimo para asignar licencias: **License Administrator**.
- Licencias por grupo → requiere **P1** para los administradores/usuarios y solo funciona con **grupos de seguridad** (o M365) con miembros **directos**.
- Si un usuario tiene una licencia por grupo y necesita que no se le aplique un plan de servicio, se ajusta en la asignación del grupo, no del usuario.
- **Convertir directa → grupo**: asignar por grupo, esperar procesamiento y luego quitar la directa (no al revés, para evitar interrupciones).
- Los **usuarios invitados** no necesitan licencia para funciones básicas; para funciones P1/P2 se factura por **usuarios activos mensuales (MAU)** del tenant.

## Ejemplo

Contoso tiene 500 usuarios con licencia directa E3. Migran a licencias por grupo: crean el grupo dinámico "Todos los empleados" (`user.userType -eq "Member"`), le asignan E3, verifican que no hay errores y luego quitan las asignaciones directas con PowerShell. Resultado: los nuevos empleados quedan licenciados automáticamente.

## Comparaciones

| Método | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Asignación directa** | Usuario a usuario | Sin requisitos de licencia extra | Pocos usuarios, excepciones |
| **Licencias por grupo** | A todos los miembros | Automático, escalable, sin scripts | Organizaciones con P1; combinado con grupos dinámicos |
| **PowerShell / Graph** | Scripts masivos | Control total, informes | Migraciones, auditoría |
| **Centro de administración de Microsoft 365** | Interfaz alternativa | Vista comercial de licencias | Administradores de M365 |

## AZ-104 Exam Tips

- 🔥 🧠 **Usage location obligatoria** antes de asignar licencias.
- 🧠 Licencias por grupo = **P1** (u otra licencia que lo incluya) + **solo miembros directos**.
- 🧠 Rol mínimo: **License Administrator**.
- 📌 Al quitar una licencia directa que también viene por grupo, el usuario **la conserva** (heredada).
- ⚠️ Deshabilitar un plan de servicio no libera una unidad de licencia.
- ⚠️ Un usuario puede tener errores de licencia por **conflicto de planes** (por ejemplo dos productos con Exchange Online).

## Errores comunes

- Asignar licencias a grupos anidados y esperar que lleguen a los miembros indirectos.
- Quitar primero la licencia directa antes de que el grupo haya procesado (el usuario pierde acceso temporalmente).
- Olvidar revisar los errores de licencia del grupo tras la asignación.

## Preguntas que podrían aparecer

**1.** Tienes un grupo de seguridad "Marketing" con licencia Microsoft 365 E3 asignada. El grupo "Becarios-Marketing" está anidado dentro. Los becarios no reciben la licencia. ¿Cuál es la causa?
- A) Los becarios no tienen ubicación de uso · B) Las licencias por grupo no se aplican a grupos anidados · C) Falta el rol License Administrator · D) El grupo debe ser de tipo Microsoft 365

<details><summary>Respuesta</summary>

**B.** Solo los miembros directos reciben la licencia. Solución: asignar la licencia también a "Becarios-Marketing" o convertir la pertenencia en directa.
</details>

**2.** ¿Qué edición mínima de Microsoft Entra ID necesitas para usar la asignación de licencias basada en grupos?
- A) Free · B) P1 · C) P2 · D) Governance

<details><summary>Respuesta</summary>

**B.** P1 (o una licencia de Microsoft 365 que lo incluya) habilita licencias por grupo. P2 también sirve pero no es el mínimo.
</details>

## Relacionado

- [[02 - Usuarios de Microsoft Entra ID]]
- [[03 - Grupos de Microsoft Entra ID]]
- [[01 - Microsoft Entra ID para administradores]]
- [[00 - Índice - Identidades y gobernanza]]
