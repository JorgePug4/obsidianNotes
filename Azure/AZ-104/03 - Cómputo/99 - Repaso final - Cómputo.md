---
tags: [az-104, azure, computo, repaso]
modulo: Cómputo
---

# 🎯 Repaso final · Cómputo (20-25 %)

## 1. Los conceptos más importantes

1. **ARM**: secciones parameters/variables/resources/outputs; modo **Incremental** (por defecto) vs **Complete** (borra lo no incluido). ([[01 - Azure Resource Manager y plantillas ARM]])
2. **Bicep**: `param`, `var`, `resource`, `module`, `output`, `existing`, `@secure()`; dependencias implícitas; `build` / `decompile`. ([[02 - Bicep]])
3. **Desplegar/exportar**: `az deployment group create`; exportar desde RG (estado actual) o desde Implementaciones (plantilla usada). ([[03 - Implementar, exportar y convertir plantillas]])
4. **VM**: Stopped factura, **Stopped (deallocated)** no; disco temporal se pierde; nombre Windows ≤ 15. ([[04 - Máquinas virtuales - creación y configuración]])
5. **Tamaños**: "s" = Premium; resize reinicia y puede requerir desasignar. ([[05 - Tamaños de VM y redimensionamiento]])
6. **Discos**: cambiar tipo con VM desasignada; ampliar sí, reducir no; Ultra y Premium v2 solo datos; caché SO RW / datos RO / logs None. ([[06 - Discos administrados]])
7. **Cifrado**: SSE siempre; CMK con DES; **encryption at host** cubre temporal; **ADE** = BitLocker/dm-crypt con Key Vault en la **misma región**; no en series A/Basic ni Ultra. ([[07 - Azure Disk Encryption y cifrado de discos]])
8. **Mover**: RG/suscripción con ARM (mismo tenant, dependencias juntas); **región = Resource Mover**. ([[08 - Mover una VM (grupo de recursos, suscripción o región)]])
9. **HA**: FD 3 / UD 20; SLA 99,9 (VM Premium) / 99,95 (set) / 99,99 (zonas); set y zona se eligen **al crear**. ([[09 - Alta disponibilidad - Availability Sets y Availability Zones]])
10. **VMSS**: Uniform vs **Flexible** (mezcla tamaños y Spot); reglas de autoescalado con umbral, duración y cool-down; upgrade policy Manual/Automatic/**Rolling**. ([[10 - Virtual Machine Scale Sets]])
11. **Extensiones**: Custom Script, **AMA + DCR** (el agente MMA está retirado), VMAccess, Run Command. ([[11 - Extensiones de VM y automatización]])
12. **Imágenes**: generalizar (`sysprep` / `waagent -deprovision`); **Galería → Definición → Versión** con réplicas por región. ([[12 - Imágenes y Azure Compute Gallery]])
13. **ACR**: geo-replicación, private link y content trust solo **Premium**; **AcrPull/AcrPush**; admin user deshabilitado. ([[13 - Azure Container Registry]])
14. **ACI**: container group comparte red y volúmenes; restart policy **Always/OnFailure/Never**; **sin autoescalado**; Azure Files para persistencia. ([[14 - Azure Container Instances]])
15. **Container Apps**: Entorno → App → Revisión → Réplica; **escala a cero** con min=0 y reglas HTTP/KEDA (CPU no); división de tráfico. ([[15 - Azure Container Apps]])
16. **App Service Plan**: slots Standard 5 / Premium 20; autoescalado desde **Standard**; Basic máx. 3 instancias manuales. ([[17 - App Service Plan (niveles y escalado)]])
17. **App Service**: app settings = variables; **slot settings no se intercambian**; Always On; identidad administrada + Key Vault. ([[18 - Azure App Service - creación y configuración]])
18. **Dominios/TLS**: **CNAME** subdominio, **A** apex, **TXT asuid** para validar; certificado gratuito administrado; SNI vs IP-based. ([[19 - App Service - certificados, TLS y dominios personalizados]])
19. **Backup de App Service**: Basic+ (solo producción en Basic); **10 GB / 4 GB**; automáticas cada hora 30 días vs personalizadas configurables. ([[20 - App Service - copias de seguridad]])
20. **Redes de App Service**: **VNet integration = salida**, **private endpoint = entrada**, restricciones de acceso por IP/etiqueta (y SCM aparte). ([[21 - App Service - redes]])
21. **Slots**: swap sin downtime, rollback con swap inverso, swap with preview, traffic routing. ([[22 - App Service - ranuras de implementación (deployment slots)]])

## 2. Tabla de decisión rápida

| Si el enunciado dice… | Respuesta |
|---|---|
| "no volver a pagar cómputo" | Deallocate |
| "los datos de D: desaparecieron" | Disco temporal |
| "disco Premium en una VM D2_v3" | Redimensionar a D2s_v3 |
| "cambiar el tipo de disco" | Desasignar primero |
| "cifrar también disco temporal sin agente" | Encryption at host |
| "BitLocker/dm-crypt" | Azure Disk Encryption |
| "mover a otra región" | Azure Resource Mover |
| "SLA 99,99 %" | 2+ VMs en 2+ zonas |
| "SLA 99,95 %" | Availability set |
| "añadir VM existente a un availability set" | No se puede: recrear |
| "mezclar Spot y tamaños distintos" | VMSS Flexible |
| "actualizar la imagen por lotes" | Rolling upgrade |
| "100 VMs idénticas en 3 regiones" | Compute Gallery con réplicas |
| "geo-replicación de imágenes de contenedor" | ACR Premium |
| "contenedor que ejecuta una tarea y termina" | ACI con restart policy Never/OnFailure |
| "contenedor sin IP pública" | ACI en subred delegada |
| "escala a cero", "revisiones", "KEDA" | Container Apps |
| "ranuras de implementación" | App Service Standard+ |
| "sin downtime y con rollback" | Swap de slots |
| "dominio personalizado con HTTPS gratis" | Certificado administrado de App Service |
| "la app debe llegar a una base de datos privada" | VNet integration |
| "la app solo accesible por IP privada" | Private endpoint |
| "retención de backups de 1 año en mi storage" | Copias personalizadas |

## 3. Números que debo memorizar

| Dato | Valor |
|---|---|
| Fault domains / update domains | 3 / 20 (por defecto 5) |
| SLA VM única Premium / set / zonas | 99,9 % / 99,95 % / 99,99 % |
| Máx. instancias VMSS | 1000 |
| Nombre de VM Windows / Linux | 15 / 64 caracteres |
| Tamaño máx. disco administrado | 32 TiB (Ultra 64 TiB) |
| Slots Standard / Premium / Isolated | 5 / 20 / 20 |
| Instancias Basic / Standard / Premium | 3 / 10 / 30 |
| Backup App Service: tamaño máx. | 10 GB (4 GB por base de datos) |
| Backup automático App Service | cada hora, 30 días |
| Container Apps réplicas por defecto | 0-10 (máx. configurable 1000) |
| Regla HTTP de Container Apps | ~10 peticiones concurrentes por réplica |
| ACR Basic/Standard/Premium | 10 / 100 / 500 GiB incluidos |
| Despliegues guardados por RG | 800 |

## 4. Diferencias que más fácil puedo confundir

| Pareja | Diferencia |
|---|---|
| Incremental vs Complete | No borra vs borra lo que falta en la plantilla |
| ARM vs Bicep | JSON verboso vs sintaxis simple (mismo motor) |
| Stopped vs Deallocated | Factura vs no factura cómputo |
| Disco temporal vs disco de datos | Efímero local vs administrado persistente |
| Availability set vs zona | Racks del mismo datacenter vs datacenters distintos |
| Escalado vertical vs horizontal | Resize vs más instancias |
| Uniform vs Flexible | Idénticas vs heterogéneas |
| Imagen vs snapshot | Plantilla generalizada vs copia de disco |
| Generalizada vs especializada | Pide credenciales nuevas vs clon exacto |
| ADE vs encryption at host vs SSE | Invitado vs host vs plataforma |
| ACI vs Container Apps | Sin escalado vs escala a cero |
| ACR AcrPull vs AcrPush | Descargar vs subir |
| Revisión vs réplica (Container Apps) | Versión vs instancia |
| Plan vs app (App Service) | Cómputo facturado vs aplicación |
| Scale up vs scale out (plan) | Nivel vs instancias |
| App setting vs slot setting | Viaja en el swap vs se queda en la ranura |
| VNet integration vs private endpoint | Salida vs entrada |
| Automáticas vs personalizadas (backup) | Cada hora/30 días fijos vs configurable en tu storage |
| SNI SSL vs IP-based SSL | Compartida vs IP dedicada |

## 5. Checklist de dominio

- [ ] Sé leer una plantilla ARM y decir qué despliega y qué parámetro cambiar.
- [ ] Sé los elementos de Bicep y cómo convertir entre ARM y Bicep.
- [ ] Sé desplegar con CLI/PowerShell y exportar plantillas.
- [ ] Sé crear una VM con todas sus opciones y los estados de energía.
- [ ] Sé elegir tamaño y redimensionar, con sus efectos.
- [ ] Sé los tipos de disco, cuándo usar cada uno y cómo ampliar/cambiar.
- [ ] Sé las cuatro formas de cifrado de discos y sus requisitos.
- [ ] Sé mover VMs entre RG, suscripción y región.
- [ ] Sé los SLA y cuándo usar set, zonas o ambos.
- [ ] Sé configurar un VMSS con reglas de autoescalado y upgrade policy.
- [ ] Sé qué extensiones existen y para qué sirve cada una.
- [ ] Sé generalizar y publicar imágenes en una galería.
- [ ] Sé los SKUs de ACR y cómo autenticar sin secretos.
- [ ] Sé crear ACI con red, volúmenes y restart policy.
- [ ] Sé configurar Container Apps con escalado a cero y revisiones.
- [ ] Sé qué incluye cada nivel de plan de App Service.
- [ ] Sé configurar app settings, Always On, identidad y diagnóstico.
- [ ] Sé añadir dominio personalizado con sus registros DNS y certificado.
- [ ] Sé configurar backups y sus límites.
- [ ] Sé distinguir VNet integration de private endpoint y aplicar restricciones.
- [ ] Sé usar slots, swap, swap with preview y traffic routing.

## 6. Preguntas de repaso

**1.** Despliegas una plantilla en modo Complete en un RG con recursos no incluidos. ¿Qué pasa?
- A) Nada · B) Se eliminan · C) Se detienen · D) Error

<details><summary>Respuesta</summary>**B.**</details>

---

**2.** Una VM apagada desde el sistema operativo sigue facturando cómputo. ¿Qué debes hacer?
- A) Eliminarla · B) Desasignarla desde el portal o CLI · C) Reducir su tamaño · D) Quitar la IP pública

<details><summary>Respuesta</summary>**B.**</details>

---

**3.** ¿Qué configuración alcanza un SLA del 99,99 % para VMs?
- A) VM única con Ultra Disk · B) 3 VMs en un availability set · C) 2 VMs en 2 zonas · D) VMSS Uniform en una zona

<details><summary>Respuesta</summary>**C.**</details>

---

**4.** Necesitas ejecutar un contenedor por lotes que termine y no se reinicie si acaba bien. ¿Qué servicio y configuración?
- A) Container Apps min=1 · B) ACI con restart policy OnFailure · C) AKS · D) App Service

<details><summary>Respuesta</summary>**B.**</details>

---

**5.** Una API en contenedor debe no costar nada de noche y escalar de día. ¿Qué usas?
- A) ACI · B) Container Apps con min=0 y regla HTTP · C) VMSS · D) App Service Basic

<details><summary>Respuesta</summary>**B.**</details>

---

**6.** Necesitas 10 ranuras de implementación. ¿Qué nivel de plan es el mínimo?
- A) Basic · B) Standard · C) Premium · D) Isolated

<details><summary>Respuesta</summary>**C.** Standard solo llega a 5.</details>

---

**7.** Tras un swap, producción apunta a la base de datos de staging. ¿Qué faltó?
- A) Always On · B) Marcar la cadena de conexión como slot setting · C) Reiniciar · D) Health check

<details><summary>Respuesta</summary>**B.**</details>

---

**8.** Una Web App necesita llegar a una base de datos con private endpoint. ¿Qué configuras?
- A) Private endpoint en la app · B) VNet integration · C) Restricciones de acceso · D) Service endpoint en la app

<details><summary>Respuesta</summary>**B.**</details>

---

**9.** Debes cambiar el tipo del disco de datos de Standard HDD a Premium SSD. ¿Qué requisito hay?
- A) Snapshot previo · B) VM desasignada y tamaño con "s" · C) Zona de disponibilidad · D) Encryption at host

<details><summary>Respuesta</summary>**B.**</details>

---

**10.** Quieres desplegar 200 VMs idénticas con la aplicación preinstalada en dos regiones. ¿Qué usas?
- A) Snapshot del disco · B) Imagen administrada en una región · C) Azure Compute Gallery con la versión replicada en ambas regiones · D) Custom Script Extension solamente

<details><summary>Respuesta</summary>**C.**</details>

---

**11.** ¿Qué SKU de Azure Container Registry permite geo-replicación y private link?
- A) Basic · B) Standard · C) Premium · D) Todos

<details><summary>Respuesta</summary>**C.**</details>

---

**12.** Configuras Azure Disk Encryption y falla. La VM es `Standard_A2_v2`. ¿Cuál es el problema?
- A) Falta Key Vault · B) ADE no es compatible con las series A y Basic · C) El disco es Premium · D) Falta identidad administrada

<details><summary>Respuesta</summary>**B.**</details>

> [!tip] Última pasada
> **Deallocate ≠ Stop**, **SLA 99,9/99,95/99,99**, **slots Standard 5 / Premium 20**, **slot settings no se intercambian**, **VNet integration = salida**, **Container Apps escala a cero (CPU no)**, **ACI sin autoescalado**, **"s" = Premium**.

Volver: [[00 - Índice - Cómputo]] · [[00 - AZ-104 Índice general (MOC)]]
