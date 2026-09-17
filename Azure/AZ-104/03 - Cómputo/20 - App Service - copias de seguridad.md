---
tags: [az-104, azure, computo, app-service, backup]
modulo: Cómputo
peso_examen: Medio-Alto
---

# App Service · copias de seguridad

## ¿Qué es?

App Service puede crear **copias de seguridad** de la aplicación (contenido del sistema de archivos, configuración y, en los casos soportados, bases de datos vinculadas) y **restaurarlas** sobre la misma app, sobre un slot o sobre una app nueva.

Hay **dos modelos** 🧠:

| | **Copias automáticas (automatic backups)** | **Copias personalizadas (custom backups)** |
|---|---|---|
| Configuración | Ninguna (se activan en niveles soportados) | Requiere configurar una **cuenta de almacenamiento** y un contenedor |
| Frecuencia | **Cada hora**, no configurable | Configurable, **mínimo cada hora**; hasta ~12 al día contando manuales y programadas |
| Retención | **30 días**, no configurable | Configurable (incluida retención indefinida) |
| Almacenamiento | Gestionado por Azure (no accesible) | **Tu** cuenta de almacenamiento (contenedor de blobs) |
| Bases de datos | No incluye | Podía incluir bases vinculadas (en retirada, ver aviso) |
| Niveles | Standard, Premium, Isolated | **Basic**, Standard, Premium, Isolated |

## Conceptos clave

- **Niveles soportados** 🧠: **Basic o superior** para copias personalizadas; en **Basic solo el slot de producción**. Free y Shared **no** admiten copias.
- **Límite de tamaño** 🧠: la app (más las bases de datos vinculadas) no puede superar **10 GB**; una base de datos individual no puede superar **4 GB**. Si se supera, la copia falla.
- **Qué incluye**: configuración de la app, contenido de archivos y, opcionalmente, bases de datos vinculadas (SQL Database, MySQL in-App, PostgreSQL).
- ⚠️ **Aviso 2026**: Microsoft está retirando la inclusión de **bases de datos vinculadas** en las copias personalizadas nuevas; las existentes siguen funcionando durante un período de transición. Para bases de datos, usar la copia nativa del servicio de datos.
- **Excluir archivos**: mediante el archivo `_backup.filter` en `D:\home\site\wwwroot`.
- **Restauración**: sobre la **misma app**, sobre un **slot** o sobre una **app nueva**; se puede elegir restaurar solo el contenido o también la configuración y las bases de datos. La restauración **sobrescribe** y **detiene** temporalmente la app.
- Las copias **parciales** permiten excluir carpetas grandes.
- Los **slots** tienen sus propias copias de seguridad.
- La cuenta de almacenamiento debe estar en la **misma suscripción**; si tiene firewall, hay que permitir el acceso.

## Cómo funciona

```bash
# Copia manual (custom) a una cuenta de almacenamiento con SAS
az webapp config backup create --resource-group rg-web --webapp-name app-contoso \
  --backup-name backup-manual --container-url "https://st001.blob.core.windows.net/backups?<SAS>"
# Programación: cada 24 horas, retención 30 días
az webapp config backup update --resource-group rg-web --webapp-name app-contoso \
  --container-url "https://st001.blob.core.windows.net/backups?<SAS>" \
  --frequency 24h --retain-one true --retention 30
# Listar y restaurar
az webapp config backup list --resource-group rg-web --webapp-name app-contoso -o table
az webapp config backup restore --resource-group rg-web --webapp-name app-contoso \
  --backup-name backup-manual --container-url "https://st001.blob.core.windows.net/backups?<SAS>" --overwrite
az webapp config backup show --resource-group rg-web --webapp-name app-contoso
```

Portal: Web App → **Copias de seguridad** → Configurar (cuenta de almacenamiento, contenedor, programación, retención, bases de datos) → **Copia de seguridad ahora** / **Restaurar**.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Copias sin configurar nada, cada hora, 30 días | **Copias automáticas** (Standard+) |
| Guardar las copias en mi propia cuenta de almacenamiento | **Copias personalizadas** |
| Retención de 1 año | Copias **personalizadas** con retención configurada |
| Plan Basic con copia del slot de staging | **No es posible**: Basic solo copia producción |
| La copia falla por tamaño | La app supera **10 GB** → excluir con `_backup.filter` |
| Restaurar en un entorno de pruebas | Restaurar sobre una **app nueva o slot** |
| Free/Shared necesitan backup | **Escalar** a Basic o superior |

## Ejemplo

Una web en plan S1 de 3 GB debe conservarse un año. Las copias automáticas solo guardan 30 días, así que se configuran **copias personalizadas**: cuenta de almacenamiento `stbackups`, contenedor `appbackups`, programación diaria a las 02:00 y retención de 365 días. Antes de una actualización mayor se lanza una copia manual y, si algo falla, se restaura sobre el slot `staging` para validar antes de sobrescribir producción.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **Copias automáticas** | Sin configuración | Cada hora, 30 días, gratis con el nivel | Protección básica |
| **Copias personalizadas** | Control total | Programación, retención, tu storage | Requisitos de retención y ubicación |
| **Control de código fuente (Git/GitHub)** | Código | El código siempre recuperable | Complemento imprescindible |
| **Slots** | Volver a la versión anterior | Swap inmediato | Rollback rápido de despliegues |
| **Copia nativa de la base de datos** | Datos | PITR de SQL/MySQL | Siempre para datos |

## AZ-104 Exam Tips

- 🔥 🧠 **Backup requiere Basic o superior**; en **Basic solo el slot de producción**.
- 🔥 🧠 Límite **10 GB** (app + bases vinculadas) y **4 GB** por base de datos.
- 🧠 Automáticas: **cada hora, 30 días, no configurables**. Personalizadas: programación y retención propias, en **tu** cuenta de almacenamiento.
- 🧠 La restauración puede ir a la **misma app, un slot o una app nueva**.
- 💻 Configurar backup en el portal y con `az webapp config backup create/update/restore`.
- ⚠️ La inclusión de bases de datos vinculadas está en retirada; usa la copia nativa de la base de datos.

## Errores comunes

- Intentar configurar copias en un plan Free/Shared.
- Superar los 10 GB y no entender por qué falla la copia.
- Confiar solo en las copias automáticas cuando se exige retención larga.

## Preguntas que podrían aparecer

**1.** Una Web App en plan Basic necesita copias de seguridad del slot de staging. ¿Qué debes hacer?
- A) Nada, ya están incluidas · B) Escalar a Standard o superior · C) Usar la cuenta de almacenamiento del slot · D) Habilitar Always On

<details><summary>Respuesta</summary>

**B.** En Basic solo se puede copiar el slot de producción; los slots adicionales requieren Standard o superior.
</details>

**2.** ¿Cuál es el tamaño máximo admitido para una copia de seguridad de App Service (aplicación más bases de datos vinculadas)?
- A) 1 GB · B) 4 GB · C) 10 GB · D) 100 GB

<details><summary>Respuesta</summary>

**C.** 10 GB en total, con un máximo de 4 GB por base de datos.
</details>

## Relacionado

- [[17 - App Service Plan (niveles y escalado)]]
- [[18 - Azure App Service - creación y configuración]]
- [[22 - App Service - ranuras de implementación (deployment slots)]]
- [[10 - Operaciones de copia de seguridad y restauración]]
- [[00 - Índice - Cómputo]]
