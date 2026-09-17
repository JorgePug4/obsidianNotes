---
tags: [az-104, azure, identidad, entra-id, administrative-units, dispositivos]
modulo: Identidades y gobernanza
peso_examen: Bajo-Medio
---

# Unidades administrativas y dispositivos ➕

> [!info] Estas dos áreas aparecieron en versiones anteriores del temario ("crear unidades administrativas", "administrar configuración de dispositivos"). En el temario vigente ya no se nombran de forma explícita, pero siguen apareciendo como opciones de respuesta y en escenarios de delegación. Nivel: reconocer y saber cuándo usarlas.

## Unidades administrativas (Administrative Units)

### ¿Qué es?

Una **unidad administrativa (AU)** es un contenedor de usuarios, grupos y dispositivos que permite **restringir el ámbito de un rol de Entra** a ese subconjunto. Es el equivalente en Entra ID a las **unidades organizativas (OU)** de AD DS, pero solo para delegar administración (no para directivas).

### ¿Para qué sirve?

- Delegar a un helpdesk regional el restablecimiento de contraseñas **solo** de los usuarios de su región.
- Separar la administración por facultades, países o filiales dentro de un mismo tenant.

### Conceptos clave

- Miembros: usuarios, grupos, dispositivos. Pertenencia asignada o **dinámica** (regla por atributos, requiere P1).
- **Roles con ámbito de AU**: User Administrator, Groups Administrator, Password Administrator, Helpdesk Administrator, Authentication Administrator, License Administrator, entre otros.
- **AU restringida (restricted management AU)** ➕: los objetos dentro solo pueden ser gestionados por administradores con ámbito en la AU, ni siquiera por administradores globales de tenant sin ámbito explícito.
- Licencia: **P1** para cada administrador con rol de ámbito AU.
- Una AU **no** puede contener otras AUs (no hay anidamiento).

### Ejemplo

Universidad con dos campus. Se crean las AUs "Campus Norte" y "Campus Sur". Al helpdesk de cada campus se le asigna *Password Administrator* con ámbito en su AU. Ninguno puede restablecer contraseñas del otro campus.

### Comparación

| Herramienta | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Unidad administrativa** | Delegar roles de Entra a un subconjunto | Privilegio mínimo dentro del directorio | Filiales, regiones, campus |
| **Rol de Entra sin ámbito** | Administrar todo el tenant | Simple | Organizaciones pequeñas |
| **Azure RBAC en un ámbito** | Delegar acceso a recursos | Herencia | Recursos de Azure, no directorio |
| **Tenants separados** | Aislamiento total | Independencia | Empresas realmente distintas |

## Dispositivos en Microsoft Entra ID

### ¿Qué es?

Los **dispositivos** se representan como objetos en Entra ID para poder aplicar acceso condicional ("solo desde dispositivos conformes"), SSO y administración con Intune.

### Conceptos clave

- **Entra registered** (registrado): dispositivos personales (BYOD), Windows/iOS/Android/macOS. El usuario inicia sesión con cuenta local o personal y añade la cuenta de trabajo.
- **Entra joined** (unido): dispositivos corporativos, propiedad de la organización, Windows 10/11. Inicio de sesión con la identidad de Entra. Sin AD local.
- **Hybrid Entra joined**: unido al AD DS local **y** registrado en Entra (mediante Entra Connect). Para organizaciones con AD y GPO.
- **Configuración de dispositivos** (Entra ID → Dispositivos → Configuración del dispositivo):
  - Quién puede **unir** dispositivos a Entra (todos / seleccionados / nadie).
  - Quién puede **registrar** dispositivos.
  - **Número máximo de dispositivos por usuario** (por defecto 50).
  - Exigir MFA para unir/registrar.
  - **Administradores locales** adicionales en dispositivos unidos (Global Admin y Entra Joined Device Local Administrator lo son por defecto).
- **Enterprise State Roaming** (P1): sincroniza configuración de usuario entre dispositivos Windows.
- Acciones: habilitar/deshabilitar, eliminar, ver BitLocker keys (si están escrowed), administrar con Intune.

### Comparación

| Estado | Propiedad | Inicio de sesión | Cuándo utilizarlo |
|---|---|---|---|
| **Registered** | Personal (BYOD) | Cuenta local/personal + cuenta de trabajo | Acceso desde dispositivos personales |
| **Joined** | Corporativa | Identidad de Entra | Organización cloud-only |
| **Hybrid joined** | Corporativa | AD DS local, registrado en Entra | Organización con AD DS y GPO |

## AZ-104 Exam Tips

- 🧠 AU = **OU para delegar roles de Entra**; sin anidamiento; roles de ámbito AU necesitan **P1**.
- 🧠 Dispositivos por usuario: **50** por defecto.
- 📌 Registered (BYOD) vs Joined (corporativo cloud) vs Hybrid joined (corporativo con AD local).
- 💻 Saber dónde se limita quién puede unir dispositivos y cuántos.
- ⚠️ Unir un **dispositivo** a Entra ID no es unir una **VM** a un dominio (eso es Entra Domain Services / AD DS). Sí existe la extensión de VM "Microsoft Entra login" para iniciar sesión en VMs con credenciales de Entra, que es otra cosa.

## Preguntas que podrían aparecer

**1.** Necesitas que el helpdesk de la filial de Portugal pueda restablecer contraseñas solo de los usuarios de esa filial. ¿Qué implementas?
- A) Un grupo de seguridad con el rol Password Administrator · B) Una unidad administrativa con los usuarios de Portugal y Password Administrator con ámbito en ella · C) Un tenant separado · D) Azure RBAC en un grupo de recursos

<details><summary>Respuesta</summary>

**B.** Las unidades administrativas permiten limitar el ámbito de los roles de Entra. A daría permisos sobre todo el tenant; C es excesivo; D no afecta al directorio.
</details>

**2.** Los usuarios informan que no pueden unir un sexto portátil a Entra ID. ¿Qué configuración revisas?
- A) Acceso condicional · B) Número máximo de dispositivos por usuario en configuración de dispositivos · C) Licencias P1 · D) Restricciones de colaboración externa

<details><summary>Respuesta</summary>

**B.** El límite de dispositivos por usuario se define en la configuración de dispositivos (50 por defecto, la organización puede haberlo reducido a 5).
</details>

## Relacionado

- [[01 - Microsoft Entra ID para administradores]]
- [[02 - Usuarios de Microsoft Entra ID]]
- [[03 - Grupos de Microsoft Entra ID]]
- [[17 - Identidad híbrida - Microsoft Entra Connect]]
- [[00 - Índice - Identidades y gobernanza]]
