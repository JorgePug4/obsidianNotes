---
tags: [az-104, azure, computo, compute, MOC]
tipo: MOC
modulo: Cómputo
peso_examen: 20-25 %
---

# AZ-104 · Dominio 3 · Implementar y administrar recursos de cómputo de Azure

> [!important] Peso en el examen: **20-25 %** (empatado con identidad como el dominio de mayor peso)
> Cuatro bloques: **infraestructura como código** (ARM y Bicep), **máquinas virtuales** (creación, discos, cifrado, tamaños, movimiento, alta disponibilidad, scale sets), **contenedores** (ACR, ACI, Container Apps) y **App Service** (plan, app, TLS, dominios, backup, redes, slots). Es el dominio con más preguntas de "qué configuración cumple el requisito con el mínimo coste/esfuerzo".

## Objetivos oficiales → notas

### 3.1 Automatizar la implementación de recursos con plantillas ARM o archivos Bicep

| Objetivo oficial | Nota |
|---|---|
| Interpretar una plantilla ARM o un archivo Bicep | [[01 - Azure Resource Manager y plantillas ARM]] · [[02 - Bicep]] |
| Modificar una plantilla ARM existente | [[01 - Azure Resource Manager y plantillas ARM]] |
| Modificar un archivo Bicep existente | [[02 - Bicep]] |
| Implementar recursos con una plantilla ARM o un archivo Bicep | [[03 - Implementar, exportar y convertir plantillas]] |
| Exportar una implementación como plantilla ARM o convertir ARM a Bicep | [[03 - Implementar, exportar y convertir plantillas]] |

### 3.2 Crear y configurar máquinas virtuales

| Objetivo oficial | Nota |
|---|---|
| Crear una máquina virtual | [[04 - Máquinas virtuales - creación y configuración]] |
| Configurar Azure Disk Encryption | [[07 - Azure Disk Encryption y cifrado de discos]] |
| Mover una VM a otro grupo de recursos, suscripción o región | [[08 - Mover una VM (grupo de recursos, suscripción o región)]] |
| Administrar tamaños de VM | [[05 - Tamaños de VM y redimensionamiento]] |
| Administrar discos de VM | [[06 - Discos administrados]] |
| Implementar VMs en zonas y conjuntos de disponibilidad | [[09 - Alta disponibilidad - Availability Sets y Availability Zones]] |
| Implementar y configurar Virtual Machine Scale Sets | [[10 - Virtual Machine Scale Sets]] |
| Contexto necesario | [[11 - Extensiones de VM y automatización]] ➕ · [[12 - Imágenes y Azure Compute Gallery]] ➕ |

### 3.3 Aprovisionar y administrar contenedores en Azure Portal

| Objetivo oficial | Nota |
|---|---|
| Crear y administrar un Azure Container Registry | [[13 - Azure Container Registry]] |
| Aprovisionar un contenedor con Azure Container Instances | [[14 - Azure Container Instances]] |
| Aprovisionar un contenedor con Azure Container Apps | [[15 - Azure Container Apps]] |
| Administrar el tamaño y el escalado de contenedores (ACI y Container Apps) | [[14 - Azure Container Instances]] · [[15 - Azure Container Apps]] · [[16 - Comparación de servicios de contenedores]] |

### 3.4 Crear y configurar Azure App Service

| Objetivo oficial | Nota |
|---|---|
| Aprovisionar un plan de App Service | [[17 - App Service Plan (niveles y escalado)]] |
| Configurar el escalado de un plan de App Service | [[17 - App Service Plan (niveles y escalado)]] |
| Crear un App Service | [[18 - Azure App Service - creación y configuración]] |
| Configurar certificados y TLS para un App Service | [[19 - App Service - certificados, TLS y dominios personalizados]] |
| Asignar un nombre DNS personalizado existente a un App Service | [[19 - App Service - certificados, TLS y dominios personalizados]] |
| Configurar la copia de seguridad de un App Service | [[20 - App Service - copias de seguridad]] |
| Configurar la red de un App Service | [[21 - App Service - redes]] |
| Configurar ranuras de implementación | [[22 - App Service - ranuras de implementación (deployment slots)]] |

### Repaso
- [[99 - Repaso final - Cómputo]]

## Orden de estudio sugerido

1. IaC (01-03): te servirá para leer todos los ejemplos de después.
2. VMs (04-12): el bloque más largo; haz los laboratorios.
3. Contenedores (13-16).
4. App Service (17-22).
5. Repaso (99).

> [!tip] La idea central del dominio
> Elegir el **servicio de cómputo** adecuado (VM, VMSS, ACI, Container Apps, App Service) y luego la **configuración concreta** que cumple SLA, coste y requisitos. Aprende las tablas de "qué nivel/SKU incluye qué".

Volver: [[00 - AZ-104 Índice general (MOC)]]
