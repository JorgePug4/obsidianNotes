---
tags: [az-104, azure, repaso, comparativa]
modulo: Repaso global
---

# 📊 Tabla comparativa de servicios AZ-104

Todas las comparaciones que el examen suele pedir, reunidas. Cada fila responde a "¿cuál elijo?".

## 1. Identidad y control de acceso

| Servicio | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Roles de Microsoft Entra** | Administrar el **directorio** | Granularidad por tarea (usuarios, grupos, licencias) | Gestión de identidades |
| **Azure RBAC** | Administrar **recursos** | Herencia por ámbito, roles integrados y personalizados | Permisos sobre VMs, redes, storage |
| **Roles de datos (Storage, Key Vault)** | **Plano de datos** | Acceso a blobs, secretos, archivos con Entra ID | Aplicaciones e identidades administradas |
| **Unidades administrativas** | Delegar roles de Entra por subconjunto | Privilegio mínimo en el directorio | Filiales, campus (P1) |
| **Identidad administrada** | Identidad de un recurso | Sin secretos, rotación automática | VMs, App Service, Policy, CMK |
| **Entidad de servicio** | Identidad de una app externa | Funciona fuera de Azure | Pipelines, apps on-premises |

## 2. Gobernanza

| Mecanismo | Pregunta que responde | Ámbito | Cuándo utilizarlo |
|---|---|---|---|
| **Azure RBAC** | ¿Quién puede hacer qué? | MG/Sub/RG/Recurso | Permisos |
| **Azure Policy** | ¿Qué se puede crear y cómo? | MG/Sub/RG | Estándares (regiones, SKUs, tags, diagnóstico) |
| **Locks** | ¿Se puede borrar o modificar? | Sub/RG/Recurso | Proteger de errores |
| **Tags** | ¿De quién es y a qué proyecto pertenece? | Recurso | Costes e inventario |
| **Grupos de administración** | ¿Cómo aplico lo mismo a muchas suscripciones? | Tenant | Organización a escala |
| **Presupuestos** | ¿Cuánto llevo gastado? | Sub/RG/MG | Control de coste (solo alerta) |
| **Azure Advisor** | ¿Qué debería mejorar? | Sub/RG | Optimización |

## 3. Almacenamiento

| Servicio | Tipo de dato | Acceso | Cuándo utilizarlo |
|---|---|---|---|
| **Blob Storage** | Objetos | HTTPS/REST | Web, backups, data lake, archivado |
| **Azure Files** | Archivos compartidos | **SMB / NFS** | Servidores de archivos, perfiles, volúmenes compartidos |
| **Discos administrados** | Bloque | Adjunto a una VM | Disco de SO y datos |
| **Queue Storage** | Mensajes | REST | Desacoplar componentes |
| **Table Storage** | NoSQL clave-valor | REST | Datos simples y baratos |
| **Azure NetApp Files** ➕ | NAS de alto rendimiento | SMB/NFS | SAP, HPC |

| Control de acceso a Storage | Ámbito | Revocación | Cuándo |
|---|---|---|---|
| **Clave de cuenta** | Todo | Regenerar | Solo administración |
| **SAS ad hoc** | Recurso | Regenerar clave | Enlaces temporales |
| **SAS con directiva almacenada** | Recurso | **Borrar la directiva** | Partners, accesos largos |
| **User delegation SAS** | Blob | Revocar clave de delegación | Apps modernas (≤ 7 días) |
| **RBAC de datos** | Cuenta/contenedor | Quitar el rol | Opción recomendada |
| **Acceso anónimo** | Contenedor | Cambiar nivel | Contenido público |

| Protección de datos | Blob | Files |
|---|---|---|
| Automática por escritura | **Versionado** | — |
| Manual puntual | **Snapshot** | **Snapshot** (200) |
| Papelera | **Soft delete** blobs y contenedores | **Soft delete** del recurso compartido |
| Punto en el tiempo | **Point-in-time restore** | Azure Backup |
| WORM | **Inmutabilidad** | — |

## 4. Cómputo

| Servicio | Control | Escalado | Cuándo utilizarlo |
|---|---|---|---|
| **VM** | Total del SO | Manual (resize) | Lift and shift, software específico |
| **VMSS** | Total, muchas instancias | **Autoescalado** | Web/worker escalable |
| **Azure Container Instances** | Contenedor suelto | Manual | Tareas, jobs, pruebas |
| **Azure Container Apps** | Contenedor serverless | **Automático, escala a cero** | Microservicios, APIs |
| **AKS** ➕ | Kubernetes completo | HPA/cluster autoscaler | Plataformas complejas |
| **App Service** | Aplicación web | Manual y automático | Webs y APIs con slots y dominios |
| **Azure Functions** ➕ | Función | Por eventos | Código corto y esporádico |

| Alta disponibilidad | Protege de | SLA | Cuándo |
|---|---|---|---|
| **VM única con Premium SSD** | Fallo de disco | 99,9 % | No críticos |
| **Availability set** | Rack y mantenimiento | 99,95 % | Regiones sin zonas |
| **Availability zones** | Centro de datos | **99,99 %** | Producción |
| **VMSS + zonas** | Igual + escalado | 99,99 % | Cargas elásticas |
| **Site Recovery / multi-región** | Caída de región | Según diseño | DR |

| Cifrado de discos | Cubre | Clave | Cuándo |
|---|---|---|---|
| **SSE (PMK)** | Discos administrados | Microsoft | Por defecto |
| **SSE + CMK** | Discos administrados | Tuya (Key Vault) | Control de claves |
| **Encryption at host** | + temporal y caché | PMK o CMK | Recomendado hoy |
| **ADE** | + temporal, en el invitado | Tuya (Key Vault) | Requisito de BitLocker/dm-crypt |

## 5. Redes

| Servicio | Capa | Ámbito | Cuándo utilizarlo |
|---|---|---|---|
| **Load Balancer** | 4 | Regional | TCP/UDP, alto rendimiento |
| **Application Gateway** | 7 | Regional | HTTP con URL, TLS, **WAF** |
| **Front Door** | 7 | **Global** | Web global con caché y WAF |
| **Traffic Manager** | DNS | Global | Elegir región por DNS |
| **NAT Gateway** | — | Subred | Salida a Internet con IP fija |

| Seguridad de red | Filtra por | Ámbito | Cuándo |
|---|---|---|---|
| **NSG** | IP, puerto, protocolo, etiquetas, ASG | Subred / NIC | Segmentación básica |
| **ASG** | Agrupación de NICs | Usado en NSG | Microsegmentación |
| **Azure Firewall** ➕ | FQDN, amenazas, L3-L7 | VNet / hub | Perímetro centralizado |
| **WAF** | OWASP | App Gateway / Front Door | Aplicaciones web |
| **Bastion** | Acceso administrativo | VNet | RDP/SSH sin IP pública |

| Conectividad privada a PaaS | IP privada | On-premises | Coste |
|---|---|---|---|
| **Service endpoint** | No | **No** | Gratis |
| **Private endpoint** | **Sí** | **Sí** | De pago |
| **Firewall por IP** | No | Sí (IP pública) | Gratis |

| Conectividad híbrida | Medio | Cifrado | Cuándo |
|---|---|---|---|
| **VPN Site-to-Site** | Internet | Sí | Sucursales, respaldo |
| **VPN Point-to-Site** | Internet | Sí | Equipos individuales |
| **ExpressRoute** | Circuito privado | **No** por defecto | Producción crítica |
| **Peering** | Backbone | N/A | VNet ↔ VNet |

| DNS | Resuelve | Cuándo |
|---|---|---|
| **Azure DNS público** | Dominios de Internet | Hospedar el dominio corporativo |
| **Azure Private DNS** | Nombres internos en VNets vinculadas | Private endpoints, nombres propios |
| **DNS de Azure (168.63.129.16)** | VMs de la misma VNet | Por defecto |
| **DNS personalizado / Private Resolver** | Lo que configures / híbrido | Integración con AD DS |

## 6. Monitorización y protección de datos

| Servicio | Responde a | Cuándo utilizarlo |
|---|---|---|
| **Azure Monitor** | ¿Cómo va mi recurso? | Métricas, logs, alertas |
| **Log Analytics** | ¿Qué pasó exactamente? | Consultas KQL |
| **Application Insights** | ¿Cómo va mi aplicación? | APM |
| **Network Watcher** | ¿Por qué no conecta? | Diagnóstico de red |
| **Connection Monitor** | ¿Sigue conectando? | Vigilancia continua |
| **Service Health** | ¿Hay una incidencia de Azure? | Caídas del proveedor |
| **Resource Health** | ¿Está sano este recurso? | Diagnóstico puntual |
| **Advisor** | ¿Qué mejoro? | Recomendaciones |
| **Defender for Cloud** ➕ | ¿Estoy seguro? | Postura de seguridad |

| Tipo de alerta | Señal | Coste | Cuándo |
|---|---|---|---|
| **Métrica** | Números | Por serie | Umbrales de rendimiento |
| **Búsqueda de registros** | KQL | Por regla | Patrones y texto |
| **Registro de actividad** | Operaciones | **Gratis** | Auditoría, Service Health |

| Protección de datos | Objetivo | RPO/RTO | Cuándo |
|---|---|---|---|
| **Azure Backup (RSV)** | VMs, Files, SQL/SAP, on-premises | Horas/días | Copias con retención |
| **Backup vault** | Discos, blobs, PostgreSQL, AKS | Horas | Cargas nuevas |
| **Azure Site Recovery** | Continuidad del servicio | **Minutos** | Desastre regional |
| **Snapshots** | Un disco o recurso compartido | Puntual | Antes de un cambio |
| **Redundancia (GRS/ZRS)** | Durabilidad | N/A | No es backup |

## Relacionado

- [[01 - Resumen general de AZ-104]]
- [[02 - Guía de memorización (números, límites y nombres)]]
- [[04 - Errores y confusiones frecuentes]]
- [[00 - AZ-104 Índice general (MOC)]]
