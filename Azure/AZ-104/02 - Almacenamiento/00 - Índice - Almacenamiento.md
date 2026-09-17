---
tags: [az-104, azure, almacenamiento, storage, MOC]
tipo: MOC
modulo: Almacenamiento
peso_examen: 15-20 %
---

# AZ-104 · Dominio 2 · Implementar y administrar almacenamiento

> [!important] Peso en el examen: **15-20 %**
> En AZ-900 bastaba con elegir Blob/Files/Disk y LRS/GRS. En AZ-104 te preguntan **cómo asegurar el acceso** (firewall, SAS, claves, identidad), **qué opción de cuenta** cumple un requisito y **qué característica de Blob/Files** resuelve un escenario (niveles, ciclo de vida, versionado, snapshots, soft delete, File Sync).

## Objetivos oficiales → notas

### 2.1 Configurar el acceso al almacenamiento

| Objetivo oficial | Nota |
|---|---|
| Configurar firewalls y redes virtuales de Azure Storage | [[03 - Firewalls y redes virtuales de Azure Storage]] |
| Crear y usar tokens de firma de acceso compartido (SAS) | [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]] |
| Configurar directivas de acceso almacenadas | [[04 - Firmas de acceso compartido (SAS) y directivas de acceso almacenadas]] |
| Administrar claves de acceso | [[05 - Claves de acceso y autorización con Microsoft Entra ID]] |
| Configurar el acceso basado en identidad para Azure Files | [[14 - Acceso basado en identidad para Azure Files]] |

### 2.2 Configurar y administrar cuentas de almacenamiento

| Objetivo oficial | Nota |
|---|---|
| Crear y configurar cuentas de almacenamiento | [[01 - Cuentas de almacenamiento]] |
| Configurar la redundancia de Azure Storage | [[02 - Redundancia de almacenamiento]] |
| Configurar la replicación de objetos | [[07 - Replicación de objetos (Object Replication)]] |
| Configurar el cifrado de la cuenta de almacenamiento | [[06 - Cifrado de cuentas de almacenamiento]] |
| Administrar datos con Azure Storage Explorer y AzCopy | [[08 - Azure Storage Explorer y AzCopy]] |

### 2.3 Configurar Azure Files y Azure Blob Storage

| Objetivo oficial | Nota |
|---|---|
| Crear y configurar un recurso compartido de archivos | [[13 - Azure Files]] |
| Crear y configurar un contenedor en Blob Storage | [[09 - Azure Blob Storage]] |
| Configurar los niveles de almacenamiento | [[10 - Niveles de acceso de Blob (Hot, Cool, Cold, Archive)]] |
| Configurar instantáneas y eliminación temporal para Azure Files | [[15 - Instantáneas y eliminación temporal en Azure Files]] |
| Configurar la administración del ciclo de vida de blobs | [[11 - Administración del ciclo de vida de Blob]] |
| Configurar el control de versiones de blobs | [[12 - Versionado, instantáneas y eliminación temporal de Blob]] |
| Contexto necesario | [[16 - Azure File Sync]] ➕ |

### Repaso
- [[99 - Repaso final - Almacenamiento]]

## Orden de estudio sugerido

1. Cuenta y redundancia (01, 02): la base.
2. Acceso (03, 04, 05): el bloque más preguntado.
3. Cifrado, replicación y herramientas (06, 07, 08).
4. Blob (09 → 12) y Files (13 → 16).
5. Repaso (99).

> [!tip] La idea central del dominio
> Cuatro capas de control de acceso a una cuenta, de fuera hacia dentro: **red** (firewall, service/private endpoint) → **autenticación** (clave, SAS, Entra ID) → **autorización** (permisos de la SAS o rol RBAC de datos) → **protección de datos** (soft delete, versionado, snapshots, inmutabilidad). El examen combina estas capas en un mismo escenario.

Volver: [[00 - AZ-104 Índice general (MOC)]] · Repaso de fundamentos: [[Almacenamiento en Azure (Índice)]] (AZ-900)
