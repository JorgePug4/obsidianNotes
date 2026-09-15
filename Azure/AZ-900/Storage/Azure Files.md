---
tags: [az-900, azure, almacenamiento, files, smb]
---

# Azure Files

Clases 84 y 85 del curso (Azure File Storage).

## 1. Concepto

**Azure Files** ofrece **recursos compartidos de archivos totalmente administrados en la nube**, accesibles con los mismos protocolos estándar que un servidor de archivos de toda la vida: **SMB** y **NFS**.

**Qué problema resuelve:** sustituir o ampliar los servidores de archivos locales sin cambiar la forma de trabajar. Varias personas o máquinas acceden a la misma carpeta simultáneamente, con carpetas y subcarpetas reales.

**Para qué se usa:** unidades de red compartidas, aplicaciones heredadas que esperan una ruta UNC (`\\servidor\carpeta`), archivos de configuración compartidos entre VM y herramientas de diagnóstico.

## 2. Características principales

- Se **monta como unidad de red** en Windows, Linux y macOS, tanto desde Azure como desde el equipo local.
- Protocolos: **SMB** (usa el **puerto 445**) y **NFS**.
- **Acceso simultáneo** desde muchos clientes, a diferencia de un disco administrado.
- Estructura **jerárquica real**: carpetas y subcarpetas.
- Se accede también por **HTTPS y REST** desde aplicaciones.
- Integración con **Microsoft Entra ID** y con Active Directory local para permisos NTFS.
- Admite **instantáneas** del recurso compartido.
- Niveles de servicio: **Estándar** (HDD) y **Premium** (SSD). Ver [[Almacenamiento Premium]].

### Azure File Sync

Servicio que **sincroniza** un servidor de archivos Windows local con un recurso compartido de Azure Files.

- Convierte el servidor local en una caché rápida del recurso en la nube.
- **Organización en niveles en la nube (cloud tiering)**: los archivos poco usados se quedan solo en Azure y liberan disco local.
- Escenario típico de examen: "quiero conservar el servidor local pero centralizar los archivos en Azure" → **Azure File Sync**.

> [!warning] Confusión frecuente
> **Azure Files ≠ Azure File Sync ≠ OneDrive.** Files es el recurso compartido en la nube. File Sync mantiene sincronizado un servidor Windows local con ese recurso. OneDrive y SharePoint son productos de Microsoft 365 para archivos de usuario final, no forman parte de Azure Storage.

## 3. Casos de uso

- Una empresa apaga su viejo servidor de archivos y mapea la unidad `Z:` de todos los empleados a Azure Files.
- Una aplicación antigua que solo sabe escribir en `\\servidor\datos` se migra a Azure sin tocar el código.
- Varias VM de un conjunto de escalado leen la misma configuración desde un recurso compartido.
- Una sucursal mantiene su servidor local con **File Sync** y el cloud tiering para no llenar el disco.

## 4. Comparaciones

| | **Azure Files** | **Azure Blob Storage** | **Azure Disk Storage** |
|---|---|---|---|
| Modelo | Sistema de archivos compartido | Objetos | Disco de bloque |
| Protocolo | **SMB / NFS** + REST | HTTP/HTTPS, REST | Adjunto a la VM |
| Se monta como unidad | **Sí** | No de forma nativa | Sí, pero en una sola VM |
| Varios clientes a la vez | **Sí** | Sí, por HTTP | No es lo habitual |
| Niveles Hot/Cool/Archive | No | **Sí** | No |
| Elígelo cuando | Necesitas una carpeta compartida o una ruta UNC | Sirves archivos a apps o web | Necesitas el disco de una VM |

## 5. Conceptos que debo memorizar

> [!important] Para el examen
> - Azure Files = **recurso compartido de archivos administrado** con **SMB y NFS**.
> - **SMB usa el puerto 445**. Si la empresa lo tiene bloqueado, el montaje falla.
> - Permite **acceso simultáneo** desde la nube y desde local al mismo tiempo.
> - **Azure File Sync** sincroniza un servidor Windows local con Azure Files e incluye **cloud tiering**.
> - Azure Files **no** usa niveles Hot/Cool/Archive.
> - Existe en nivel **Estándar (HDD)** y **Premium (SSD)**.

## 6. Tips para AZ-900

> [!tip] Palabras clave
> - "SMB", "NFS", "unidad de red", "ruta UNC", "\\\\servidor\\carpeta", "mapear unidad" → **Azure Files**
> - "sustituir el servidor de archivos local" → **Azure Files**
> - "mantener el servidor local y sincronizar con la nube" → **Azure File Sync**
> - "liberar espacio en el servidor local dejando los archivos fríos en la nube" → **cloud tiering de File Sync**
> - "aplicación heredada que no se puede modificar" y menciona archivos → **Azure Files**

> [!warning] Trampas habituales
> - El examen puede describir "archivos" para referirse a imágenes que consume una web: eso es **Blob**, no Files. El diferenciador es **cómo se accede** (HTTP → Blob; unidad montada → Files).
> - Preguntar por niveles de acceso Archive en Azure Files: no aplican.
> - Confundir File Sync con una copia de seguridad. File Sync **sincroniza**, no es Azure Backup.

## 7. Ejemplo de preguntas de examen

**Pregunta 1.** Una empresa quiere retirar su servidor de archivos local, pero necesita que los empleados sigan accediendo a los documentos mediante una unidad de red asignada. ¿Qué servicio cumple el requisito?

- A) Azure Blob Storage
- B) Azure Files
- C) Azure Disk Storage
- D) Azure Table Storage

**Respuesta correcta: B.** Azure Files se monta como unidad de red mediante SMB, que es exactamente el comportamiento que se pide. A no se monta como unidad de forma nativa. C solo se adjunta a una VM. D almacena datos NoSQL tabulares.

---

**Pregunta 2.** ¿Qué protocolo y puerto utiliza Azure Files para montar un recurso compartido en Windows?

- A) FTP en el puerto 21
- B) SMB en el puerto 445
- C) HTTP en el puerto 80
- D) RDP en el puerto 3389

**Respuesta correcta: B.** Azure Files usa SMB por el puerto 445, motivo por el que muchos proveedores de Internet y firewalls corporativos bloquean el acceso. A no es el protocolo de Azure Files. C se usa para Blob, no para montar unidades. D es escritorio remoto, nada que ver.

---

**Pregunta 3.** Una sucursal quiere conservar su servidor de archivos Windows local para tener acceso rápido, pero centralizar todos los datos en Azure y liberar espacio en disco. ¿Qué debe implementar?

- A) Azure File Sync
- B) Azure Data Box
- C) Azure Backup
- D) AzCopy

**Respuesta correcta: A.** Azure File Sync sincroniza el servidor local con Azure Files y, con cloud tiering, deja en local solo los archivos usados con frecuencia. B es un dispositivo para migraciones masivas puntuales. C hace copias de seguridad, no sincronización continua. D copia datos una vez, no mantiene la sincronización.

## 🧠 Resumen para el examen

1. Azure Files = **recursos compartidos administrados** accesibles por **SMB y NFS**.
2. **Puerto 445** para SMB, dato que suele aparecer como trampa.
3. Permite **acceso simultáneo** desde la nube y desde local.
4. Reemplaza servidores de archivos y da soporte a apps con **rutas UNC**.
5. **Azure File Sync** = servidor local + nube sincronizados, con **cloud tiering**.
6. **No** tiene niveles Hot/Cool/Archive.
7. Niveles **Estándar (HDD)** y **Premium (SSD)**.
8. Si se accede por **URL/HTTP** el servicio correcto es Blob; si se **monta como unidad**, es Files.
