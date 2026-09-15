---
tags: [AZ-900, Azure, Seguridad, SIEM]
up: "[[00 - Índice - Seguridad]]"
---

# Microsoft Sentinel

> [!info] Relevancia: **baja**. Salió del temario en 2022. Aparece casi siempre como *distractor* frente a Defender for Cloud. Antiguo nombre: **Azure Sentinel**.

## 1. Concepto

**Microsoft Sentinel** es el **SIEM** (*Security Information and Event Management*) y **SOAR** (*Security Orchestration, Automation and Response*) nativo de la nube de Microsoft.

- **SIEM**: recoge registros de seguridad de muchas fuentes, los correlaciona y busca patrones de ataque.
- **SOAR**: responde de forma automática (por ejemplo, bloquear una IP o deshabilitar un usuario) mediante *playbooks* basados en Logic Apps.

- **Problema que resuelve**: los datos de seguridad están dispersos (Azure, Microsoft 365, firewalls, AWS, on-premises) y nadie los ve en conjunto.
- **Para qué se usa**: centros de operaciones de seguridad (SOC) que necesitan detectar, investigar y responder a escala.

## 2. Características principales

- **Conectores de datos** para Azure, Microsoft 365, AWS, firewalls de terceros, etc.
- Almacena los datos en un **Log Analytics workspace** (Azure Monitor).
- Reglas de **análisis** e **IA** para detectar amenazas y reducir falsos positivos.
- **Investigación** visual de incidentes.
- **Automatización** con playbooks.
- Escala sin instalar servidores propios (a diferencia de un SIEM tradicional on-premises).

## 3. Casos de uso

- Un SOC quiere ver en un mismo lugar los intentos de acceso sospechosos en Microsoft 365, las alertas de Defender for Cloud y los logs del firewall de la oficina.
- Cuando se detecta un usuario comprometido, un playbook lo deshabilita automáticamente y abre un ticket.

## 4. Comparaciones

| | Microsoft Sentinel | Microsoft Defender for Cloud | Azure Monitor |
|---|---|---|---|
| Tipo | SIEM + SOAR | Postura de seguridad + protección de cargas | Observabilidad (métricas y logs) |
| Enfoque | Seguridad de **toda la organización** | Seguridad de **recursos concretos** | Rendimiento y disponibilidad |
| Fuente de datos | Muchas (Azure, M365, terceros) | Recursos de Azure/multinube | Recursos de Azure |
| Respuesta automática | Sí (playbooks) | Parcial | Alertas y acciones básicas |

## 5. Conceptos que debo memorizar

> [!important]
> - Sentinel = **SIEM + SOAR** en la nube.
> - Recoge datos de **múltiples fuentes**, incluidas otras nubes y on-premises.
> - Usa **playbooks** para responder automáticamente.
> - Se apoya en **Log Analytics**.

## 6. Tips para AZ-900

> [!tip]
> - Palabras clave **"SIEM"**, **"SOAR"**, **"correlacionar eventos de seguridad de toda la empresa"**, **"investigar incidentes"** → Sentinel.
> - Si la pregunta habla de **recomendaciones** o **Secure Score**, la respuesta es Defender for Cloud, aunque Sentinel esté entre las opciones.
> - Si habla de **rendimiento, métricas o disponibilidad**, es Azure Monitor, no Sentinel.

## 7. Ejemplo de pregunta de examen

**Pregunta 1.** Tu equipo de seguridad necesita recopilar y correlacionar registros de Azure, Microsoft 365 y firewalls de terceros para detectar ataques en toda la organización. ¿Qué servicio debes usar?

- A) Microsoft Defender for Cloud
- B) Azure Monitor
- C) Microsoft Sentinel ✅
- D) Azure Key Vault

*Correcta: C.* Correlacionar registros de múltiples fuentes es la definición de SIEM. A evalúa recursos concretos; B es observabilidad; D guarda secretos.

**Pregunta 2.** ¿Qué tipo de solución es Microsoft Sentinel?

- A) Firewall gestionado
- B) SIEM y SOAR ✅
- C) Gestor de identidades
- D) Servicio de copias de seguridad

*Correcta: B.* A sería Azure Firewall; C, Microsoft Entra ID; D, Azure Backup.

## 🧠 Resumen para el examen

1. Sentinel = SIEM + SOAR nativo de la nube (antes *Azure Sentinel*).
2. Agrega registros de Azure, M365, terceros, on-premises y otras nubes.
3. Detecta con reglas e IA, investiga incidentes y responde con playbooks.
4. Almacena en Log Analytics.
5. Distractor habitual de Defender for Cloud: Sentinel correlaciona, Defender recomienda y protege.

---
⬅️ [[00 - Índice - Seguridad|Volver al índice de Seguridad]]
