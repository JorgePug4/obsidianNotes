---
tags: [az-104, azure, almacenamiento, blob, object-replication]
modulo: Almacenamiento
peso_examen: Medio
---

# Replicación de objetos (Object Replication)

## ¿Qué es?

La **replicación de objetos** copia **block blobs** de forma **asíncrona** desde un contenedor de origen a un contenedor de destino, que puede estar en **otra cuenta de almacenamiento**, en **otra región** e incluso en **otra suscripción o tenant**. A diferencia de la redundancia (GRS), aquí **tú eliges** origen, destino y qué blobs replicar.

## ¿Para qué sirve?

- Reducir latencia: tener los datos cerca de los usuarios de otra región.
- Distribuir cargas de trabajo: procesar en una región y analizar en otra.
- Optimizar coste: replicar a una cuenta con nivel Cool/Archive como copia.
- Cumplir residencia de datos con copias controladas.

## Conceptos clave

- **Requisitos** 🧠:
  - Origen y destino: **GPv2** o **Premium block blobs**.
  - **Blob versioning** habilitado en **origen y destino**.
  - **Change feed** habilitado en **origen**.
  - Solo **block blobs** (no append ni page blobs).
- **Directiva de replicación (policy)**: definida en el **destino** y referenciada en el origen; contiene hasta **10 reglas** 🧠. Cada regla: contenedor origen → contenedor destino, **prefijos** de filtro opcionales, y qué copiar: solo blobs nuevos, todo, o desde una fecha.
- Máximo **2 destinos** por cuenta origen (2 directivas de origen) 🧠. Una cuenta puede ser destino de varias.
- Asíncrona: sin SLA de tiempo; se puede consultar el **estado de replicación** de cada blob (Complete / Failed / Pending).
- Los blobs replicados en destino son **de solo lectura para la replicación**: se puede escribir en el destino, pero la replicación sobrescribirá. Los borrados **se replican**; las versiones anteriores no.
- No replica: metadatos de nivel de acceso? Sí replica el contenido y las propiedades; el nivel del blob en destino sigue la configuración del destino. No replica **snapshots** ni blobs en **Archive** (no se pueden leer).
- Rol: **Contributor** (o superior) en ambas cuentas para configurar; en cuentas de otro tenant se usa la modalidad "sin acceso al destino" con ID de directiva.
- Coste: transacciones + egreso entre regiones.

## Cómo funciona

```
Cuenta origen (versioning + change feed)                Cuenta destino (versioning)
  contenedor "fotos" ──regla (prefijo "2026/")──►  contenedor "fotos-copia"
  Storage lee el change feed periódicamente y copia las escrituras/borrados de forma asíncrona
```

```bash
# Habilitar requisitos
az storage account blob-service-properties update --account-name stsrc --resource-group rg --enable-versioning true --enable-change-feed true
az storage account blob-service-properties update --account-name stdst --resource-group rg --enable-versioning true
# Crear directiva (desde el destino) y aplicarla al origen
az storage account or-policy create --account-name stdst --resource-group rg \
  --source-account stsrc --destination-account stdst \
  --source-container fotos --destination-container fotos-copia --prefix-match "2026/" --min-creation-time "2026-01-01T00:00:00Z"
az storage account or-policy list --account-name stsrc --resource-group rg
```

```powershell
$rule = New-AzStorageObjectReplicationPolicyRule -SourceContainer fotos -DestinationContainer fotos-copia -PrefixMatch "2026/"
Set-AzStorageObjectReplicationPolicy -ResourceGroupName rg -StorageAccountName stdst -PolicyId default -SourceAccount stsrc -Rule $rule
```

Portal: cuenta origen → **Replicación de objetos** → Configurar reglas de replicación → destino, contenedores, filtros → Guardar (crea la directiva en ambas).

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| Copiar solo los blobs de un prefijo | Filtro de prefijo en la regla |
| Copiar también los blobs existentes | Opción "todo" o "desde fecha" al crear la regla |
| Replicar a otra suscripción/tenant | Soportado; en otro tenant hay que permitir "cross-tenant replication" en ambas cuentas |
| Falla la creación de la directiva | Revisar versioning en ambas y change feed en origen |
| Necesito replicar append blobs / archivos | No soportado → usar AzCopy o File Sync |

## Ejemplo

Una app en West Europe genera informes en el contenedor `informes`. El equipo de análisis en East US necesita leerlos con baja latencia y solo los del año actual. Se habilitan versioning y change feed en la cuenta de origen, versioning en la de East US, y se crea una regla `informes` → `informes` con prefijo `2026/` y copia de existentes. Las lecturas en East US son locales y no consumen egreso repetido.

## Comparaciones

| Mecanismo | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Object replication** | Copia selectiva de block blobs entre cuentas | Controlas destino, filtros, cuentas distintas | Latencia, distribución, copias con otro nivel |
| **GRS / GZRS** | Redundancia de toda la cuenta a la región emparejada | Automático, sin configurar | DR de la cuenta completa |
| **AzCopy (copy / sync)** | Copia puntual o programada | Cualquier tipo de blob y Files | Migraciones, cargas iniciales |
| **Azure File Sync** | Sincronizar servidores con Files | Bidireccional con caché | Archivos, no blobs |
| **Azure Data Factory** ➕ | Pipelines de datos | Transformaciones | ETL |

## AZ-104 Exam Tips

- 🔥 🧠 Requisitos: **versioning en origen y destino**, **change feed en origen**, solo **block blobs**, cuentas **GPv2 o Premium block blob**.
- 🧠 **10 reglas** por directiva; **2 destinos** por cuenta de origen.
- 📌 Object replication (selectiva, cuentas distintas, tú eliges región) vs GRS (toda la cuenta, región emparejada, sin elección).
- 🧠 Es **asíncrona**; los blobs en Archive no se replican.
- 💻 Configurar la regla desde el portal y con `az storage account or-policy`.

## Errores comunes

- Intentar configurarla en cuentas Premium FileStorage o con append blobs.
- Olvidar el change feed en el origen.
- Esperar que replique blobs históricos sin marcar la opción de copiar existentes.

## Preguntas que podrían aparecer

**1.** Intentas crear una directiva de replicación de objetos entre dos cuentas GPv2 y falla. Ambas tienen versioning habilitado. ¿Qué falta?
- A) Change feed en la cuenta de origen · B) GRS en la cuenta de destino · C) Un private endpoint · D) Soft delete

<details><summary>Respuesta</summary>

**A.** Object replication lee el change feed del origen para saber qué copiar.
</details>

**2.** Necesitas que los blobs del contenedor `logs` de una cuenta en Europa se copien automáticamente a una cuenta en Asia que pertenece a otra suscripción, replicando solo los que empiezan por `app1/`. ¿Qué usas?
- A) GRS · B) Object replication con filtro de prefijo · C) Azure File Sync · D) Redundancia RA-GZRS

<details><summary>Respuesta</summary>

**B.** Object replication permite elegir la cuenta destino (otra suscripción, otra región) y filtrar por prefijo. GRS no permite elegir región ni cuenta.
</details>

## Relacionado

- [[02 - Redundancia de almacenamiento]]
- [[12 - Versionado, instantáneas y eliminación temporal de Blob]]
- [[08 - Azure Storage Explorer y AzCopy]]
- [[09 - Azure Blob Storage]]
- [[00 - Índice - Almacenamiento]]
