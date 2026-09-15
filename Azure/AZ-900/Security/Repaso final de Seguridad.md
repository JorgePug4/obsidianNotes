---
tags: [AZ-900, Azure, Seguridad, Repaso]
up: "[[00 - Índice - Seguridad]]"
---

# 🎯 Repaso final de Seguridad

## Conceptos más importantes

- **Defensa en profundidad**: 7 capas, datos en el centro, cada capa cubre el fallo de la anterior.
- **Zero Trust**: verificar explícitamente, privilegio mínimo, asumir la brecha.
- **CIA**: Confidencialidad, Integridad, Disponibilidad.
- **Microsoft Defender for Cloud**: postura de seguridad (Secure Score, recomendaciones) + protección contra amenazas, híbrido y multinube.
- **Microsoft Sentinel**: SIEM + SOAR.
- **Azure Key Vault**: secretos, claves y certificados fuera del código.
- **Private Endpoint**: IP privada para un servicio PaaS (Private Link); **Service Endpoint**: ruta troncal, IP pública se mantiene.

## Tabla de servicios y su propósito

| Servicio | Propósito en una frase | Capa de defensa |
|---|---|---|
| Microsoft Entra ID + MFA + Conditional Access | Quién eres y bajo qué condiciones entras | Identidad |
| RBAC | Qué puedes hacer con cada recurso | Identidad |
| Azure DDoS Protection | Frenar ataques masivos desde Internet | Perímetro |
| Azure Firewall | Firewall gestionado para toda la VNet | Perímetro / Red |
| Network Security Group (NSG) | Reglas de tráfico por subred o NIC | Red |
| Service / Private Endpoint | Llegar a PaaS sin pasar por Internet | Red |
| Microsoft Defender for Cloud | Recomendaciones, Secure Score, alertas de amenazas | Cómputo (y transversal) |
| Azure Key Vault | Guardar secretos, claves y certificados | Aplicación |
| Cifrado en reposo / Storage | Proteger los datos en sí | Datos |
| Microsoft Sentinel | SIEM/SOAR para toda la organización | Transversal |

## Diferencias que más fácil confundo

| Confusión | Cómo distinguirlo |
|---|---|
| Defensa en profundidad vs Zero Trust | **Capas** vs **no confiar por defecto** |
| Defender for Cloud vs Sentinel | **Recomendaciones y Secure Score** vs **SIEM que correlaciona registros** |
| Defender for Cloud vs Azure Policy | **Seguridad** vs **cumplimiento de reglas de la organización** |
| Defender for Cloud vs Azure Advisor | Advisor da recomendaciones generales (coste, rendimiento...); Defender **protege** además de recomendar |
| Key Vault vs Entra ID | **Secretos de aplicaciones** vs **identidades de personas** |
| Private Endpoint vs Service Endpoint | **IP privada** vs **IP pública con ruta troncal** |
| NSG vs Azure Firewall | Reglas básicas por subred/NIC vs firewall gestionado avanzado con inteligencia de amenazas |
| Capa Perímetro vs capa Red | DDoS/Firewall vs NSG/segmentación |

## 10 tips de examen

1. Lee la palabra clave del enunciado: **"capas"**, **"Secure Score"**, **"SIEM"**, **"secreto"**, **"IP privada"**. Cada una apunta a un servicio.
2. Si ves **"Azure Security Center"** o **"Azure Defender"**, piensa Defender for Cloud.
3. Si ves **"Azure Sentinel"**, piensa Microsoft Sentinel.
4. Los **datos** son siempre el centro de la defensa en profundidad.
5. **MFA** vive en la capa Identidad; **DDoS** en Perímetro; **NSG** en Red.
6. Zero Trust tiene **tres** principios; si una opción incluye "confiar en la red corporativa", es falsa.
7. Defender for Cloud **tiene nivel gratuito** y cubre **AWS y GCP**; ambas afirmaciones aparecen en preguntas de verdadero/falso.
8. Key Vault guarda **tres** tipos de objetos: secretos, claves, certificados. "Usuarios" nunca es uno de ellos.
9. Cuando la pregunta pide "un único panel para evaluar la seguridad de los recursos", la respuesta es Defender for Cloud, aunque Sentinel o Monitor estén en las opciones.
10. Responsabilidad compartida: Microsoft cubre la seguridad **física**; el cliente cubre **datos, identidades y accesos** en cualquier modelo (IaaS, PaaS, SaaS).

## 10 preguntas de repaso tipo AZ-900

**1.** ¿Cuál es la capa más interna del modelo de defensa en profundidad?
- A) Aplicación · B) Red · C) Datos ✅ · D) Identidad
*Los datos son lo que el atacante busca; todas las capas los rodean.*

**2.** ¿Qué principio de Zero Trust implica diseñar la seguridad como si el atacante ya estuviera dentro?
- A) Verificar explícitamente · B) Asumir la brecha ✅ · C) Privilegio mínimo · D) Defensa en profundidad

**3.** Una empresa quiere ver un porcentaje que refleje cuánto ha mejorado la seguridad de sus recursos tras aplicar recomendaciones. ¿Qué característica busca?
- A) Compliance Score de Azure Policy · B) Secure Score de Defender for Cloud ✅ · C) Health de Azure Monitor · D) Score de Azure Advisor

**4.** ¿Qué servicio deberías usar para correlacionar alertas de seguridad de Azure, Microsoft 365 y un firewall de terceros?
- A) Defender for Cloud · B) Azure Monitor · C) Microsoft Sentinel ✅ · D) Azure Policy

**5.** Un desarrollador debe eliminar una clave de API del código fuente. ¿Dónde debe almacenarla?
- A) Azure Key Vault ✅ · B) Microsoft Entra ID · C) Azure Storage · D) Azure DevOps

**6.** ¿Qué componente crea una interfaz de red con IP privada de tu VNet para acceder a un servicio PaaS?
- A) Service Endpoint · B) Private Endpoint ✅ · C) NSG · D) VNet peering

**7.** ¿Verdadero o falso? Microsoft Defender for Cloud solo puede proteger recursos que se ejecutan en Azure.
- Falso ✅. Cubre on-premises (Azure Arc), AWS y GCP.

**8.** ¿En qué capa de la defensa en profundidad se sitúa la protección contra DDoS?
- A) Red · B) Perímetro ✅ · C) Física · D) Cómputo

**9.** ¿Cuál de los siguientes describe correctamente a Microsoft Sentinel?
- A) Herramienta de gestión de costes · B) SIEM y SOAR en la nube ✅ · C) Servicio de cifrado de discos · D) Gestor de certificados

**10.** ¿Qué funcionalidad de Defender for Cloud abre los puertos de administración de una VM solo cuando se solicitan y por tiempo limitado?
- A) Conditional Access · B) Azure Bastion · C) Just-in-time VM access ✅ · D) Service Endpoint
*Bastion también protege el acceso RDP/SSH, pero lo hace mediante un host intermedio en el navegador, no abriendo puertos temporalmente.*

---
⬅️ [[00 - Índice - Seguridad|Volver al índice de Seguridad]]
