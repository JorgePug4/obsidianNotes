---
tags: [az-104, azure, almacenamiento, cifrado, cmk, key-vault]
modulo: Almacenamiento
peso_examen: Medio-Alto
---

# Cifrado de cuentas de almacenamiento

## ¿Qué es?

**Azure Storage Service Encryption (SSE)** cifra automáticamente todos los datos en reposo con **AES-256**, siempre, sin coste y sin poder desactivarse. Lo que sí configuras es **quién controla las claves** y si añades una **segunda capa** de cifrado.

## ¿Para qué sirve?

- Cumplir normativas que exigen **claves propias** (customer-managed keys, CMK) o rotación controlada.
- Aislar clientes con claves distintas por ámbito (encryption scopes).
- Doble cifrado (infraestructura) para requisitos de alta seguridad.

## Conceptos clave

- **Claves administradas por Microsoft (MMK / PMK)**: por defecto; Microsoft gestiona y rota.
- **Claves administradas por el cliente (CMK)**: la clave vive en **Azure Key Vault** o **Managed HSM**; la cuenta usa una **identidad administrada** (system o user-assigned) para acceder a ella. Aplica a **Blob y Files** (para Queue y Table hay que habilitarlo al crear la cuenta). Rotación automática cuando la versión de la clave no se fija. Requisitos del Key Vault: **soft delete** y **purge protection** habilitados.
- **Claves proporcionadas por el cliente (CPK)** ➕: la app envía la clave en cada petición REST a Blob; no se almacena.
- **Cifrado de infraestructura (double encryption)**: segunda capa a nivel de infraestructura con clave distinta; se decide **al crear la cuenta** (o mediante encryption scope). No se puede activar después.
- **Encryption scopes** ➕: ámbitos de cifrado por contenedor o blob con su propia clave (MMK o CMK) dentro de la misma cuenta.
- **Cifrado en tránsito**: HTTPS (Secure transfer required) y SMB 3.x con cifrado; TLS mínimo 1.2.
- Discos administrados tienen su propio modelo (ver [[07 - Azure Disk Encryption y cifrado de discos]]).

## Cómo funciona (CMK)

```
Cuenta de almacenamiento ──(identidad administrada)──► Key Vault (clave RSA)
       │                                                    ▲
       └── DEK cifrada con la KEK del Key Vault ────────────┘
Si la clave se deshabilita o se borra → la cuenta deja de poder descifrar → datos inaccesibles (no perdidos)
```

```bash
# Key Vault con requisitos
az keyvault create --name kv-st001 --resource-group rg --enable-purge-protection true --enable-rbac-authorization true
az keyvault key create --vault-name kv-st001 --name st-key --kty RSA --size 2048
# Identidad para la cuenta
az storage account update --name st001 --resource-group rg --assign-identity
# Rol para la identidad en el KV (RBAC): Key Vault Crypto Service Encryption User
az role assignment create --assignee <principalId> --role "Key Vault Crypto Service Encryption User" --scope <kvId>
# Activar CMK
az storage account update --name st001 --resource-group rg --encryption-key-source Microsoft.Keyvault \
  --encryption-key-vault https://kv-st001.vault.azure.net --encryption-key-name st-key
# Cifrado de infraestructura (solo al crear)
az storage account create --name st002 --resource-group rg --location westeurope --sku Standard_LRS --kind StorageV2 --require-infrastructure-encryption true
```

```powershell
Set-AzStorageAccount -ResourceGroupName rg -Name st001 -AssignIdentity
Set-AzStorageAccount -ResourceGroupName rg -Name st001 -KeyvaultEncryption -KeyName st-key -KeyVaultUri https://kv-st001.vault.azure.net
```

## Componentes / configuración relevante para el examen

| Configuración | Dónde | Notas |
|---|---|---|
| Tipo de clave (MMK/CMK) | Cuenta → Cifrado | Cambiable en cualquier momento (Blob/Files) |
| Key Vault | Debe tener soft delete + purge protection; misma región recomendada | Managed HSM también válido |
| Identidad | System-assigned o user-assigned | User-assigned permite configurarlo al crear la cuenta |
| Versión de clave | Fija o automática | Automática = rotación sin intervención |
| Cifrado de infraestructura | Solo al crear | Doble cifrado |
| Encryption scopes | Blob → Ámbitos de cifrado | Por contenedor/blob |
| Cifrado en tránsito | Configuración → Transferencia segura, TLS mínimo | HTTPS / SMB cifrado |

## Ejemplo

Un banco exige que Microsoft **no** pueda acceder a las claves de cifrado de sus documentos y que las claves roten cada 90 días. Solución: Key Vault con purge protection, clave RSA con directiva de rotación automática, identidad administrada asignada por el usuario para la cuenta con rol *Key Vault Crypto Service Encryption User*, y CMK en la cuenta sin fijar versión. Para un requisito adicional de "doble cifrado", crear la cuenta con cifrado de infraestructura.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **MMK** | Por defecto | Cero gestión | Sin requisitos especiales |
| **CMK (Key Vault)** | Control de claves | Rotación, revocación, auditoría | Cumplimiento, control del cliente |
| **CMK (Managed HSM)** | Claves en HSM dedicado | FIPS 140-2 nivel 3 | Requisitos regulatorios estrictos |
| **CPK** | Clave por petición | La clave nunca se guarda en Azure | Apps que gestionan sus claves |
| **Cifrado de infraestructura** | Segunda capa | Defensa en profundidad | Requisito de doble cifrado |
| **Encryption scope** | Claves por contenedor | Aislamiento por cliente | Multi-tenant |

## AZ-104 Exam Tips

- ⭐ El cifrado en reposo **siempre está activo**; lo que eliges es la **clave**.
- 🔥 🧠 CMK requiere **Key Vault con soft delete y purge protection** + **identidad administrada**.
- 🧠 **Cifrado de infraestructura** solo se activa **al crear** la cuenta.
- 🧠 Si revocas/borras la clave CMK, los datos quedan **inaccesibles** hasta restaurar la clave.
- 💻 Configurar CMK desde el portal (Cifrado → Claves administradas por el cliente) y CLI.
- 📌 Cifrado de Storage (SSE) vs Azure Disk Encryption (dentro del SO de la VM).
- ⚠️ CMK para Queue/Table solo si se habilita al crear la cuenta.

## Errores comunes

- Intentar habilitar cifrado de infraestructura en una cuenta existente.
- Crear el Key Vault sin purge protection y fallar al activar CMK.
- Olvidar dar permisos a la identidad de la cuenta en el Key Vault (get, wrapKey, unwrapKey o el rol RBAC equivalente).

## Preguntas que podrían aparecer

**1.** Quieres usar tus propias claves para cifrar una cuenta de almacenamiento existente. ¿Qué requisitos debe cumplir el Key Vault?
- A) Estar en otra región · B) Tener soft delete y purge protection habilitados · C) Ser de SKU Premium · D) Tener acceso público deshabilitado

<details><summary>Respuesta</summary>

**B.** CMK exige soft delete y purge protection para evitar la pérdida irreversible de la clave.
</details>

**2.** Un requisito exige doble cifrado en reposo para una cuenta de almacenamiento nueva. ¿Qué haces?
- A) Habilitar CMK · B) Crear la cuenta con cifrado de infraestructura habilitado · C) Activar HTTPS obligatorio · D) Usar Premium

<details><summary>Respuesta</summary>

**B.** El cifrado de infraestructura añade una segunda capa y solo puede habilitarse en la creación.
</details>

## Relacionado

- [[01 - Cuentas de almacenamiento]]
- [[18 - Identidades administradas y entidades de servicio]]
- [[07 - Azure Disk Encryption y cifrado de discos]]
- [[Azure Key Vault]] (AZ-900)
- [[00 - Índice - Almacenamiento]]
