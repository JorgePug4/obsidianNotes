---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Acceso de usuarios externos (Microsoft Entra External ID)

## Concepto

**Microsoft Entra External ID** es la familia de capacidades de [[Microsoft Entra ID]] para dar acceso a personas **que no son empleados** de la organización: socios, proveedores, clientes. Antes se llamaban **Azure AD B2B** y **Azure AD B2C**.

Problema que resuelve: colaborar con externos sin crear cuentas corporativas para ellos (más costo, más riesgo, más administración). El externo usa **su propia identidad** (su cuenta de trabajo, de Microsoft, de Google, etc.).

Para qué se usa:
- **Colaboración B2B**: invitar a un socio o proveedor a tu Teams, SharePoint o app interna como **usuario invitado (guest)**.
- **Escenarios de clientes (antes B2C)**: que los clientes de tu app se registren e inicien sesión con cuentas sociales o correo, en un tenant separado orientado al consumidor.

## Características principales

- **Invitación B2B**: el administrador (o un usuario autorizado) envía una invitación por correo. Se crea una cuenta de tipo **Guest** en tu tenant, sin contraseña propia: el invitado se autentica en **su** proveedor de identidad y luego entra a tus recursos.
- Los invitados tienen **permisos restringidos por defecto** y se pueden gobernar con [[Acceso condicional]] (exigirles MFA) y revisiones de acceso.
- Métodos de identidad del invitado: cuenta de Entra ID de otra organización, cuenta Microsoft, Google, o **código de acceso de un solo uso por correo** (email OTP) si no tiene ninguna.
- **Tipo de usuario**: *Member* (empleado) vs *Guest* (externo).
- Para AZ-900 basta reconocer: **B2B = socios (invitados en tu tenant)**; **B2C / clientes = consumidores (tenant externo separado)**. La configuración detallada corresponde a SC-300.

> [!warning] Confusión frecuente
> - **B2B ≠ B2C**. B2B es colaboración con **organizaciones socias** dentro de tu tenant de trabajo. B2C (ahora "External ID para clientes") es para **consumidores** de tus aplicaciones públicas, en un tenant separado.
> - **Usuario invitado ≠ usuario sincronizado**. El invitado viene de otra organización o proveedor; el sincronizado viene de tu AD local vía [[Microsoft Entra Connect]].
> - Invitar a un guest **no** le da acceso a todo: hay que asignarle explícitamente apps, grupos o recursos.

## Casos de uso

- Consultora externa que debe editar documentos en tu SharePoint durante un proyecto → invitación B2B como guest.
- Proveedor que revisa facturas en una app interna con su propia cuenta corporativa → B2B.
- Tienda online cuyos clientes se registran con Google o Facebook → External ID para clientes (antes B2C).

## Comparaciones

| | B2B (colaboración) | External ID para clientes (antes B2C) |
|---|---|---|
| Quién accede | Socios, proveedores, contratistas | Consumidores / público general |
| Dónde viven las cuentas | Como *Guest* en tu tenant de trabajo | En un tenant externo separado |
| Identidad usada | Su cuenta corporativa, Microsoft, Google, OTP por correo | Correo, cuentas sociales, credenciales locales |
| Recursos típicos | Teams, SharePoint, apps internas | Tus apps públicas (web, móvil) |
| Ejemplo | Contratista en tu Teams | Cliente que compra en tu e-commerce |

## Conceptos que debo memorizar

> [!important]
> - **External ID** = nombre actual de Azure AD B2B + B2C.
> - **B2B** → **usuario invitado (Guest)** que usa **su propia identidad**.
> - Tú **no gestionas la contraseña** del invitado.
> - **Member** = empleado; **Guest** = externo.
> - A los invitados se les puede aplicar **MFA y Acceso condicional**.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *partner*, *vendor*, *external organization*, *guest user*, *invite*, *their own credentials* → B2B / External ID.
> - Palabras clave: *customers*, *consumers*, *social accounts*, *public-facing application* → External ID para clientes (B2C).
> - Trampa: "Para que un proveedor acceda a SharePoint debo crear un usuario de empleado con contraseña". Incorrecto: se invita como guest.
> - Trampa: confundir External ID con Entra Connect. Connect es para **tus** usuarios locales; External ID es para usuarios de **otras** organizaciones.

## Ejemplo de pregunta de examen

**Pregunta 1.** Contoso necesita que un consultor de otra empresa acceda a un sitio de SharePoint usando la cuenta corporativa del consultor. ¿Qué característica debe usar?
- A) Microsoft Entra Connect
- B) Colaboración B2B de Microsoft Entra External ID
- C) Microsoft Entra Domain Services
- D) Azure RBAC

**Respuesta: B.** La colaboración B2B invita usuarios externos con su propia identidad. A sincroniza AD local. C provee dominio administrado. D asigna permisos, pero no crea identidades externas.

**Pregunta 2.** ¿Cómo se autentica un usuario invitado (guest) de B2B al acceder a recursos de tu tenant?
- A) Con una contraseña que tú creas y le envías
- B) Con las credenciales de su propio proveedor de identidad
- C) Con un certificado emitido por tu Active Directory local
- D) Sin autenticación, porque es invitado

**Respuesta: B.** El invitado se autentica en su organización (o con Microsoft, Google, OTP) y luego accede a tu tenant. A contradice el modelo (no gestionas su contraseña). C no aplica. D es falsa: siempre hay autenticación.

**Pregunta 3.** Una empresa desarrolla una app móvil para el público y quiere que los clientes se registren con cuentas de Google o Facebook. ¿Qué solución es la adecuada?
- A) Colaboración B2B
- B) Microsoft Entra External ID para clientes (antes Azure AD B2C)
- C) Microsoft Entra Connect
- D) Valores predeterminados de seguridad

**Respuesta: B.** Los escenarios de consumidores con identidades sociales corresponden a External ID para clientes. A es para socios empresariales. C y D no tienen relación con clientes externos.

## 🧠 Resumen para el examen

1. External ID = B2B + B2C con nombre nuevo.
2. B2B: invitas socios como Guest; usan su propia identidad.
3. Clientes (B2C): consumidores con cuentas sociales en tenant separado.
4. Tú no administras la contraseña del invitado.
5. Member = empleado, Guest = externo.
6. Los invitados pueden estar sujetos a MFA y Acceso condicional.
7. Palabras clave: partner/vendor → B2B; customer/consumer → clientes.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
