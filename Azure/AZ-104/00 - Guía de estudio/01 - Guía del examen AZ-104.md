---
tags: [az-104, azure, certificacion, guia]
modulo: Guía de estudio
---

# 📘 Guía del examen AZ-104: Microsoft Azure Administrator

## ¿Qué es?

**AZ-104** es el examen de la certificación **Microsoft Certified: Azure Administrator Associate**. Evalúa si sabes **implementar, administrar y supervisar** el entorno Azure de una organización: identidades, gobernanza, almacenamiento, cómputo y redes virtuales.

A diferencia de AZ-900 (fundamentos, "qué es"), AZ-104 es un examen de **rol**: te pone en la piel de un administrador que debe resolver escenarios concretos con la configuración correcta.

## ¿Para qué sirve?

- Validar que puedes administrar cargas de trabajo reales en Azure.
- Es requisito habitual en ofertas de administrador cloud, DevOps junior y soporte de infraestructura.
- Es la base para certificaciones superiores (AZ-305 Solutions Architect, AZ-500 Security, AZ-700 Networking).

## Formato del examen

> [!info] Datos prácticos (verificados en Microsoft Learn, con la reserva de que Microsoft puede ajustarlos)
> - **Duración**: 100 minutos de examen (unos 120 minutos de cita en total).
> - **Preguntas**: aproximadamente 40-60.
> - **Puntuación**: escala de 1 a 1000; **aprobado con 700**.
> - **Idiomas**: inglés y varios más, entre ellos español; la versión en inglés se actualiza primero y las demás unas 8 semanas después.
> - **Tipos de pregunta**: opción múltiple, respuesta múltiple, arrastrar y soltar, listas desplegables en escenarios, **casos prácticos (case studies)** con varias preguntas sobre el mismo escenario, preguntas de "sí/no" en serie que no permiten volver atrás, y **preguntas basadas en laboratorio** (pueden aparecer o no según la versión del examen).
> - **Prerrequisitos formales**: ninguno. Microsoft recomienda al menos 6 meses de experiencia práctica administrando Azure y conocimientos de redes, virtualización, identidad y PowerShell/CLI.
> - **Precio orientativo**: 165 USD (varía por país).
> - **Renovación**: la certificación caduca al año y se renueva **gratis** con una evaluación en línea en Microsoft Learn.

## Dominios y pesos (versión vigente)

| Dominio | Peso | Qué evalúa en una frase |
|---|---|---|
| Administrar identidades y gobernanza de Azure | 20-25 % | Entra ID (usuarios, grupos, licencias, invitados, SSPR), RBAC, Policy, locks, tags, suscripciones, grupos de administración, costes |
| Implementar y administrar almacenamiento | 15-20 % | Acceso seguro (firewalls, SAS, claves, identidad), cuentas, redundancia, replicación, cifrado, Blob y Files |
| Implementar y administrar recursos de cómputo | 20-25 % | ARM/Bicep, VMs (discos, cifrado, tamaños, HA, scale sets), contenedores (ACR, ACI, Container Apps), App Service |
| Implementar y administrar redes virtuales | 15-20 % | VNet, subredes, peering, IPs públicas, UDR, NSG/ASG, Bastion, endpoints, DNS, Load Balancer, troubleshooting |
| Supervisar y mantener recursos de Azure | 10-15 % | Azure Monitor (métricas, logs, KQL, alertas, Insights), Network Watcher, Azure Backup, Site Recovery |

> [!warning] Cambios recientes que debes conocer
> - **17 de abril de 2026**: última actualización oficial del documento "skills measured". Identidad y gobernanza subió a 20-25 % y redes bajó a 15-20 %. Microsoft califica el resto de cambios como menores.
> - Toda la terminología es **Microsoft Entra ID** (no "Azure AD"). Si ves el nombre antiguo en una pregunta, es sinónimo.
> - **Azure Container Apps** está explícitamente en el temario junto a Azure Container Instances (aprovisionar y escalar).
> - **Bicep** tiene el mismo peso que ARM JSON: interpretar, modificar, desplegar, exportar y **convertir ARM → Bicep**.
> - **Azure Backup vault** aparece junto a Recovery Services vault; hay que saber cuál usar para cada carga de trabajo.
> - **Alert processing rules** y **Connection Monitor** están nombrados en el temario de monitorización.
> - Servicios retirados que ya no son respuesta válida para nuevos despliegues: **Basic Load Balancer y Basic Public IP** (retirados el 30/09/2025), **Log Analytics agent (MMA)** (retirado en 2024, reemplazado por Azure Monitor Agent + Data Collection Rules), **Azure Blueprints** (en retirada), **NSG flow logs** (se retiran el 30/09/2027, sustituidos por VNet flow logs).
> - **Incertidumbre**: algunas fuentes de terceros afirman que la actualización de 2026 añadió "administración de servicios de IA" o "Copilot". No he podido confirmarlo en el documento oficial y otras fuentes lo desmienten. Estas notas se ciñen a los objetivos verificados; si Microsoft publica una nueva versión, revisa el study guide oficial.

## Cómo son las preguntas de AZ-104

| Tipo de pregunta | Ejemplo de enunciado | Cómo atacarla |
|---|---|---|
| Escenario + "¿qué debes hacer?" | "Necesitas que los usuarios accedan a un recurso compartido con credenciales de Entra ID sin controlador de dominio" | Identifica la **palabra clave** (aquí: Entra Kerberos) y descarta las opciones que violan un requisito |
| Requisito de **mínimo privilegio / mínimo coste / mínimo esfuerzo** | "Solución que minimice el esfuerzo administrativo" | Elige la opción **más restrictiva o más simple** que cumpla todo |
| Serie de afirmaciones Sí/No | "Para cada afirmación, selecciona Sí si es verdadera" | Suelen apoyarse en **límites y reglas** (NSG subred+NIC, herencia, peering no transitivo). Aquí caen los detalles memorizados |
| Case study | Empresa con varios requisitos técnicos y de negocio | Lee primero las preguntas, luego busca en el caso solo lo necesario |
| Laboratorio (si aparece) | "Crea una VM con estos parámetros" | Practica en el portal; no memorices rutas exactas, entiende dónde vive cada opción |
| Arrastrar/ordenar pasos | "Ordena los pasos para configurar Site Recovery" | Piensa en las **dependencias**: primero el vault, luego la política, luego habilitar |

## Estrategia de estudio recomendada

1. **Semana 1-2**: Identidades y gobernanza. Es el dominio más "de memoria" (roles, límites, herencia) y el de mayor peso junto con cómputo.
2. **Semana 3**: Almacenamiento. Muchas preguntas de "qué opción cumple X". Practica SAS, firewall y niveles de acceso en un laboratorio.
3. **Semana 4-5**: Cómputo. Haz al menos un despliegue con Bicep, una VM con disco de datos y cifrado, un scale set y una web app con slots.
4. **Semana 6**: Redes. Dibuja siempre el escenario (VNet, subredes, NSG, LB). Practica peering y private endpoints.
5. **Semana 7**: Monitorización y backup. Configura alertas con action groups y un backup de VM con restauración.
6. **Semana 8**: Repaso global, banco de preguntas y simulacro. Repite los temas con más fallos.

> [!tip] Regla de oro
> Cada vez que una nota diga 🧠, apúntalo en tu propia lista de memorización. El examen de AZ-104 se gana en los **detalles** (límites, nombres de subredes reservadas, qué SKU soporta qué), no en los conceptos generales.

## Fuentes oficiales que respaldan estas notas

- Microsoft Learn · Study guide for Exam AZ-104 (skills measured vigente).
- Microsoft Learn · Rutas de aprendizaje oficiales AZ-104 (5 rutas, una por dominio).
- Repositorio oficial MicrosoftLearning/AZ-104-MicrosoftAzureAdministrator (laboratorios 01-11, incluidos 09b Container Instances y 09c Container Apps).
- Documentación de producto de Azure (Storage, Virtual Machines, Networking, Monitor, Backup, Site Recovery, App Service, Container Apps, Entra ID).

## Relacionado

- [[00 - AZ-104 Índice general (MOC)]]
- [[02 - Herramientas del administrador (Portal, CLI, PowerShell, Cloud Shell, ARM)]]
- [[05 - Checklist final antes del examen]]
