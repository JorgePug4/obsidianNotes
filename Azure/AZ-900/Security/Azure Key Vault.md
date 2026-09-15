---
tags: [AZ-900, Azure, Seguridad, KeyVault]
up: "[[00 - Índice - Seguridad]]"
---

# Azure Key Vault

> [!info] Relevancia: **media**. Salió del temario explícito, pero sigue apareciendo como opción de respuesta en preguntas del tipo "¿dónde guardo un secreto?".

## 1. Concepto

**Azure Key Vault** es un servicio gestionado para **almacenar y controlar el acceso a secretos, claves de cifrado y certificados** de forma centralizada.

- **Problema que resuelve**: contraseñas y cadenas de conexión escritas en el código o en ficheros de configuración.
- **Para qué se usa**: que las aplicaciones pidan el secreto a Key Vault en tiempo de ejecución en lugar de llevarlo dentro.

## 2. Características principales

Guarda tres tipos de objetos:

| Tipo | Qué es | Ejemplo |
|---|---|---|
| **Secretos** (*Secrets*) | Texto sensible | Contraseñas, cadenas de conexión, tokens de API |
| **Claves** (*Keys*) | Claves criptográficas | Clave para cifrar discos o bases de datos |
| **Certificados** (*Certificates*) | Certificados SSL/TLS | Certificado HTTPS de tu web, con renovación automática |

Puntos clave:

- Las claves pueden respaldarse en **HSM** (módulos de seguridad hardware) en el nivel *Premium*.
- Registra **quién accede** a cada secreto (auditoría).
- Se integra con otros servicios: VMs, App Service, Storage, Azure Disk Encryption.
- Es la pieza de la capa **Aplicación** en la defensa en profundidad.

> [!warning] Confusión frecuente
> Key Vault **no gestiona identidades ni permisos de usuarios** (eso es Microsoft Entra ID / RBAC). Key Vault guarda *cosas secretas*; Entra ID decide *quién es quién*.

## 3. Casos de uso

- Una app web necesita la contraseña de la base de datos → la lee de Key Vault, no está en el código.
- Renovar automáticamente el certificado TLS del sitio web.
- Guardar la clave con la que se cifran los discos de las VMs.

## 4. Comparaciones

| Necesidad | Servicio |
|---|---|
| Guardar contraseña de una app | **Key Vault** |
| Gestionar usuarios y contraseñas de personas | Microsoft Entra ID |
| Dar permisos sobre recursos | RBAC |
| Detectar configuraciones inseguras | Defender for Cloud |

## 5. Conceptos que debo memorizar

> [!important]
> - Key Vault almacena **secretos, claves y certificados**.
> - Su objetivo es que los secretos **no estén en el código**.
> - Ofrece **auditoría** de accesos y soporte **HSM** (Premium).

## 6. Tips para AZ-900

> [!tip]
> - Palabras clave: **"secreto"**, **"cadena de conexión"**, **"certificado"**, **"clave de cifrado"**, **"centralizar"** → Key Vault.
> - Pregunta trampa: "¿qué servicio guarda las contraseñas de los usuarios de la organización?" → **Entra ID**, no Key Vault. Key Vault es para secretos de *aplicaciones*.
> - Si preguntan por "proteger la aplicación evitando credenciales en el código", la capa es **Aplicación** y el servicio es Key Vault.

## 7. Ejemplo de pregunta de examen

**Pregunta 1.** Tu equipo de desarrollo guarda la cadena de conexión a la base de datos dentro del código fuente. ¿Qué servicio de Azure debes recomendar para almacenarla de forma segura?

- A) Microsoft Entra ID
- B) Azure Key Vault ✅
- C) Azure Policy
- D) Microsoft Sentinel

*Correcta: B.* Key Vault existe para sacar secretos del código. A gestiona identidades; C define reglas de cumplimiento; D es un SIEM para detectar amenazas.

**Pregunta 2.** ¿Cuál de los siguientes NO es un tipo de objeto que se almacena en Azure Key Vault?

- A) Secretos
- B) Certificados
- C) Cuentas de usuario ✅
- D) Claves de cifrado

*Correcta: C.* Los tres restantes son los tipos que gestiona Key Vault. Las cuentas de usuario viven en Microsoft Entra ID.

**Pregunta 3.** Una empresa exige que sus claves de cifrado estén protegidas por hardware certificado. ¿Qué característica de Key Vault lo permite?

- A) Nivel Standard
- B) Soporte de HSM en el nivel Premium ✅
- C) Integración con Azure Monitor
- D) Soft delete

*Correcta: B.* Los HSM son módulos de seguridad hardware. A no incluye HSM; C es registro y alertas; D protege contra borrados accidentales, no contra extracción de claves.

## 🧠 Resumen para el examen

1. Key Vault = almacén centralizado de **secretos, claves y certificados**.
2. Evita credenciales en el código.
3. Nivel Premium añade protección por **HSM**.
4. Audita los accesos a cada secreto.
5. No confundir con Entra ID (identidades) ni RBAC (permisos).
6. Encaja en la capa **Aplicación** de la defensa en profundidad.

---
⬅️ [[00 - Índice - Seguridad|Volver al índice de Seguridad]]
