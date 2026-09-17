---
tags: [az-104, azure, almacenamiento, seguridad, sas]
modulo: Almacenamiento
peso_examen: Muy alto
---

# Firmas de acceso compartido (SAS) y directivas de acceso almacenadas

## ¿Qué es?

Una **SAS (Shared Access Signature)** es un URI firmado que concede **acceso delegado, limitado y temporal** a recursos de una cuenta de almacenamiento sin compartir la clave de la cuenta. Define **qué** (servicios, recursos), **permisos** (leer, escribir, borrar, listar…), **cuándo** (inicio/expiración), **desde dónde** (IP) y **cómo** (HTTPS).

Una **directiva de acceso almacenada (stored access policy)** es una definición guardada **en el servidor** (en el contenedor, recurso compartido, cola o tabla) a la que se puede asociar una **SAS de servicio**, para poder **modificar o revocar** la SAS sin regenerar claves.

## ¿Para qué sirve?

- Dar a un cliente un enlace de descarga que caduca en 1 hora.
- Permitir que una app móvil suba fotos a un contenedor concreto durante un día.
- Revocar inmediatamente el acceso de un partner sin afectar a nadie más (con directiva almacenada).

## Conceptos clave

- **Tipos de SAS** 🧠:

| Tipo | Firmada con | Ámbito | Revocación |
|---|---|---|---|
| **SAS de cuenta (account SAS)** | Clave de la cuenta | Uno o varios servicios (blob, file, queue, table); operaciones de servicio | Regenerar la clave |
| **SAS de servicio (service SAS)** | Clave de la cuenta | Un recurso de un servicio (contenedor, blob, recurso compartido, cola, tabla) | Regenerar la clave o, si usa **directiva almacenada**, borrar/modificar la directiva |
| **SAS de delegación de usuario (user delegation SAS)** | **Credenciales de Entra ID** (clave de delegación de usuario) | Solo **Blob** (contenedor o blob) | Revocar la clave de delegación; caduca máx. 7 días. **Recomendada por Microsoft** |

- **Parámetros del URI**: `sv` (versión), `ss` (servicios: b, f, q, t), `srt` (tipos de recurso: s=servicio, c=contenedor, o=objeto), `sp` (permisos: r, w, d, l, a, c, u, p…), `st`/`se` (inicio/expiración, UTC), `sip` (IP), `spr` (protocolo https), `sr` (recurso: b blob, c contenedor, d directorio), `si` (identificador de directiva almacenada), `sig` (firma).
- **Ad hoc SAS** (todo va en el URI) vs **SAS con directiva almacenada** (inicio, expiración y permisos vienen de la directiva y se pueden cambiar).
- Máximo **5 directivas de acceso almacenadas** por contenedor / recurso compartido / cola / tabla 🧠.
- Una SAS **no se puede "listar" ni "invalidar"** una vez emitida (no hay registro central); la única forma de invalidar una SAS ad hoc es **regenerar la clave** con la que se firmó (afecta a todas las SAS de esa clave) o esperar a que caduque.
- La SAS no salta el **firewall** de la cuenta ni el **RBAC de datos** si se usa Entra ID: son capas independientes.
- Si `Allow storage account key access` está deshabilitado, las SAS de cuenta y de servicio **dejan de funcionar** (solo user delegation SAS).
- **Directiva de expiración de SAS** ➕: la cuenta puede definir un intervalo máximo recomendado y registrar/bloquear SAS que lo excedan.

## Cómo funciona

```
Cliente ──► GET https://st001.blob.core.windows.net/docs/informe.pdf?sv=2024-...&sr=b&sp=r&se=2026-10-01T18:00Z&spr=https&sig=XYZ
            Storage verifica: firma válida (clave o clave de delegación) → fecha vigente → permiso r → IP/protocolo → firewall → sirve el blob
```

```bash
# SAS de servicio para un blob (1 hora, solo lectura)
az storage blob generate-sas --account-name st001 --container-name docs --name informe.pdf \
  --permissions r --expiry 2026-10-01T18:00Z --https-only --full-uri --account-key <key>
# SAS de cuenta
az storage account generate-sas --account-name st001 --services b --resource-types sco --permissions rl --expiry 2026-10-01T18:00Z --https-only --account-key <key>
# User delegation SAS (con Entra ID; requiere rol de datos)
az storage blob generate-sas --account-name st001 --container-name docs --name informe.pdf --permissions r --expiry 2026-10-01T18:00Z --auth-mode login --as-user --full-uri
# Directiva de acceso almacenada
az storage container policy create --account-name st001 --container-name docs --name partner-read --permissions rl --expiry 2026-12-31T00:00Z
az storage blob generate-sas --account-name st001 --container-name docs --name informe.pdf --policy-name partner-read --full-uri --account-key <key>
az storage container policy delete --account-name st001 --container-name docs --name partner-read   # revoca todas las SAS asociadas
```

```powershell
$ctx = New-AzStorageContext -StorageAccountName st001 -StorageAccountKey <key>
New-AzStorageBlobSASToken -Container docs -Blob informe.pdf -Permission r -ExpiryTime (Get-Date).AddHours(1) -Protocol HttpsOnly -FullUri -Context $ctx
New-AzStorageAccountSASToken -Service Blob -ResourceType Service,Container,Object -Permission "rl" -ExpiryTime (Get-Date).AddDays(1) -Context $ctx
New-AzStorageContainerStoredAccessPolicy -Container docs -Policy partner-read -Permission rl -ExpiryTime (Get-Date).AddMonths(3) -Context $ctx
New-AzStorageBlobSASToken -Container docs -Blob informe.pdf -Policy partner-read -Context $ctx
Remove-AzStorageContainerStoredAccessPolicy -Container docs -Policy partner-read -Context $ctx
```

## Componentes / configuración relevante para el examen

| Necesidad | Solución |
|---|---|
| Acceso temporal a **varios servicios** (blob + files) | **Account SAS** |
| Acceso a **un contenedor** durante una semana | **Service SAS** |
| Poder **revocar** sin regenerar claves | **Service SAS con stored access policy** → borrar la directiva |
| Máxima seguridad, sin usar claves de cuenta | **User delegation SAS** (Entra ID) |
| Solo desde la IP del partner y por HTTPS | Parámetros `sip` y `spr=https` |
| Revocar una SAS ad hoc ya emitida | **Regenerar la clave** de la cuenta con la que se firmó |
| Limitar duración máxima de las SAS | Directiva de expiración de SAS en la cuenta |

Portal: Cuenta → **Firma de acceso compartido** (account SAS) · Contenedor/blob → **Generar SAS** (service o user delegation) · Contenedor → **Directiva de acceso** (stored access policies).

## Ejemplo

Un partner necesita **leer y listar** los blobs del contenedor `catalogo` durante 6 meses, y Contoso quiere poder cortar el acceso en cualquier momento sin afectar a otros clientes. Solución: crear la directiva almacenada `partner-6m` (rl, expiración +6 meses) en `catalogo`, generar una **service SAS** asociada a la directiva y entregarla. Para revocar: borrar la directiva. Una SAS ad hoc obligaría a regenerar la clave, rompiendo el resto de SAS.

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Account SAS** | Varios servicios/operaciones de cuenta | Amplio | Herramientas de administración, migraciones |
| **Service SAS (ad hoc)** | Un recurso, corto plazo | Simple | Enlaces de descarga temporales |
| **Service SAS + stored access policy** | Acceso revocable | Revocar/modificar sin tocar claves; máx. 5 directivas | Partners, accesos largos |
| **User delegation SAS** | Blob con Entra ID | Sin claves, auditable, máx. 7 días | Aplicaciones modernas, cumplimiento |
| **Clave de cuenta** | Acceso total | — | Nunca para clientes; solo administración |
| **RBAC de datos (Entra ID)** | Usuarios y apps con identidad | Sin secretos | Siempre que el cliente tenga identidad de Entra |

## 💻 Laboratorio: SAS revocable

1. Crear el contenedor `docs` y subir un archivo.
2. Generar una SAS ad hoc de solo lectura (1 hora) desde el portal y abrir la URL en el navegador.
3. Crear una directiva de acceso `revocable` (lectura + lista, 1 mes) en el contenedor.
4. Generar una SAS asociada a la directiva; probarla.
5. Eliminar la directiva y comprobar que la SAS deja de funcionar (403) mientras la ad hoc sigue viva.
6. Regenerar `key1` y comprobar que la ad hoc también muere.

## AZ-104 Exam Tips

- ⭐ SAS = acceso **delegado, limitado y temporal** sin compartir la clave.
- 🔥 📌 **Account SAS** (varios servicios) vs **Service SAS** (un recurso) vs **User delegation SAS** (Entra ID, solo Blob, ≤ 7 días).
- 🔥 🧠 Revocar una SAS ad hoc = **regenerar la clave**. Revocar una SAS con **directiva almacenada** = borrar/modificar la directiva.
- 🧠 Máximo **5 directivas** por contenedor.
- 🧠 Parámetros: `sp` permisos, `se` expiración, `sip` IP, `spr` protocolo, `si` directiva.
- 💻 Generar SAS en portal, CLI y PowerShell; crear directivas de acceso.
- ⚠️ La SAS no evita el firewall ni funciona si se deshabilitó el acceso por clave (salvo user delegation).
- ⚠️ La hora de la SAS es **UTC**; los relojes desincronizados provocan errores 403 (usar `st` unos minutos antes).

## Errores comunes

- Emitir SAS ad hoc de larga duración y luego no poder revocarlas sin regenerar claves.
- Creer que una directiva almacenada sirve para account SAS (solo para service SAS).
- Dar permisos de escritura/borrado por defecto.

## Preguntas que podrían aparecer

**1.** Emites una SAS de servicio ad hoc con expiración en 2027 a un proveedor y ahora debes revocarla de inmediato sin afectar al resto de clientes. ¿Qué habrías necesitado para hacerlo posible?
- A) Haber usado una account SAS · B) Haberla asociado a una directiva de acceso almacenada · C) Haber usado HTTPS · D) Haber puesto un lock

<details><summary>Respuesta</summary>

**B.** Solo las SAS asociadas a una directiva almacenada se revocan borrando la directiva. Sin ella, la única opción es regenerar la clave, que invalida todas las SAS firmadas con ella.
</details>

**2.** Una aplicación debe generar enlaces temporales a blobs sin usar nunca las claves de la cuenta de almacenamiento. ¿Qué tipo de SAS eliges?
- A) Account SAS · B) Service SAS · C) User delegation SAS · D) Stored access policy

<details><summary>Respuesta</summary>

**C.** La SAS de delegación de usuario se firma con credenciales de Entra ID en lugar de la clave de la cuenta.
</details>

**3.** ¿Cuántas directivas de acceso almacenadas se pueden definir en un contenedor?
- A) 1 · B) 5 · C) 10 · D) Ilimitadas

<details><summary>Respuesta</summary>

**B.** Cinco por contenedor, recurso compartido, cola o tabla.
</details>

## Relacionado

- [[05 - Claves de acceso y autorización con Microsoft Entra ID]]
- [[03 - Firewalls y redes virtuales de Azure Storage]]
- [[09 - Azure Blob Storage]]
- [[08 - Azure Storage Explorer y AzCopy]]
- [[00 - Índice - Almacenamiento]]
