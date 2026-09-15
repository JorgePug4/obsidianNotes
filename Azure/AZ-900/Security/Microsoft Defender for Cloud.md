---
tags: [AZ-900, Azure, Seguridad, Defender]
up: "[[00 - Índice - Seguridad]]"
---

# Microsoft Defender for Cloud

> [!info] Relevancia: **alta**. Es el único servicio de seguridad que el temario nombra explícitamente. Antes se llamaba **Azure Security Center** (y su parte de pago, **Azure Defender**).

## 1. Concepto

**Microsoft Defender for Cloud** es una herramienta de **gestión de la postura de seguridad (CSPM)** y **protección de cargas de trabajo (CWPP)**. Evalúa la configuración de tus recursos, te dice qué mejorar y detecta amenazas en ejecución.

- **Problema que resuelve**: no saber si tus cientos de recursos están bien configurados ni si alguien los está atacando.
- **Para qué se usa**: obtener un **panel único** con recomendaciones, un **Secure Score** y alertas de amenazas.

## 2. Características principales

- **Secure Score** (puntuación de seguridad): un porcentaje que mide cómo de bien cumples las recomendaciones. Sube al aplicarlas.
- **Recomendaciones**: acciones concretas ("activa MFA", "cifra este disco", "cierra el puerto 3389").
- **Protección contra amenazas**: alertas cuando detecta comportamiento malicioso (fuerza bruta a una VM, acceso anómalo a SQL, etc.).
- **Just-in-time VM access**: abre puertos de administración solo cuando se piden y por tiempo limitado.
- **Cobertura híbrida y multinube**: Azure, on-premises (vía Azure Arc), AWS y GCP.
- Dos niveles:
  - **Gratuito** (*Foundational CSPM*): recomendaciones y Secure Score para recursos de Azure.
  - **De pago** (*planes Defender*): protección avanzada por tipo de recurso (Defender for Servers, for Storage, for SQL, for Containers...).

> [!warning] Confusión frecuente: Defender for Cloud vs Microsoft Sentinel
> - **Defender for Cloud**: *"¿Cómo de segura está mi configuración y qué amenazas ve en mis recursos?"* → postura + protección.
> - **Sentinel**: *"Reúne los registros de todo (Azure, on-prem, otros proveedores) y analízalos"* → SIEM/SOAR.
> Defender for Cloud **envía** sus alertas a Sentinel; no son lo mismo.

## 3. Casos de uso

- Descubrir que 12 VMs tienen el puerto RDP abierto a Internet y arreglarlo desde una lista de recomendaciones.
- Recibir una alerta porque alguien intenta iniciar sesión por fuerza bruta en un servidor.
- Medir la mejora de seguridad de la empresa mes a mes con el Secure Score.
- Proteger también servidores on-premises y en AWS desde el mismo panel.

## 4. Comparaciones

| | Defender for Cloud | Microsoft Sentinel | Azure Policy |
|---|---|---|---|
| Función | Postura de seguridad + protección de cargas | SIEM + SOAR (correlación de registros y respuesta) | Cumplimiento de reglas organizativas |
| Pregunta que responde | ¿Estoy bien configurado? ¿Me atacan? | ¿Qué pasó en toda la organización? | ¿Cumplen los recursos mis normas? |
| Nivel gratuito | Sí | No | Sí |
| Multinube | Sí | Sí | Solo Azure (y Arc) |

## 5. Conceptos que debo memorizar

> [!important]
> - Antiguo nombre: **Azure Security Center** + **Azure Defender**.
> - **Secure Score** = medida de tu postura de seguridad.
> - Tiene **nivel gratuito** con recomendaciones.
> - Cubre **Azure, on-premises, AWS y GCP**.
> - Ofrece **recomendaciones**, **alertas de amenazas** y **acceso JIT** a VMs.

## 6. Tips para AZ-900

> [!tip]
> - Palabras clave: **"postura de seguridad"**, **"recomendaciones"**, **"Secure Score"**, **"evaluar la seguridad de los recursos"**, **"Security Center"** → Defender for Cloud.
> - Palabras clave: **"SIEM"**, **"correlacionar registros"**, **"respuesta automatizada"**, **"SOAR"** → Sentinel.
> - Pregunta trampa: "¿qué servicio muestra un porcentaje de cumplimiento de recomendaciones de seguridad?" → Defender for Cloud (Secure Score), **no** Azure Policy (que muestra cumplimiento de *tus* políticas).
> - Si el enunciado menciona "Azure Security Center", es el mismo servicio con su nombre antiguo.
> - "Solo abrir el puerto RDP cuando un administrador lo solicite" → **Just-in-time VM access** (función de Defender for Cloud).

## 7. Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas una herramienta que evalúe de forma continua la configuración de tus recursos de Azure y te proponga acciones para mejorar su seguridad. ¿Qué debes usar?

- A) Microsoft Sentinel
- B) Azure Monitor
- C) Microsoft Defender for Cloud ✅
- D) Azure Advisor

*Correcta: C.* Es su función principal (recomendaciones y Secure Score). A correlaciona registros; B recoge métricas y logs de rendimiento; D da recomendaciones generales (coste, rendimiento, fiabilidad, seguridad) pero su bloque de seguridad se alimenta de Defender for Cloud, y no protege contra amenazas.

**Pregunta 2.** ¿Qué indicador de Microsoft Defender for Cloud resume en un único número la postura de seguridad de una suscripción?

- A) Compliance Score
- B) Secure Score ✅
- C) Health Score
- D) Trust Score

*Correcta: B.* El resto no son indicadores de Defender for Cloud. "Compliance Score" existe en Microsoft Purview (cumplimiento normativo), no aquí.

**Pregunta 3.** Una empresa tiene servidores en Azure, en su datacenter local y en AWS. Quiere un único panel con recomendaciones de seguridad para todos. ¿Es posible con Microsoft Defender for Cloud?

- A) No, solo cubre recursos de Azure
- B) Sí, mediante Azure Arc y los conectores multinube ✅
- C) Solo si migra AWS a Azure
- D) Solo con Microsoft Sentinel

*Correcta: B.* Defender for Cloud es híbrido y multinube. A y C son falsas; D confunde con el SIEM, que sí agrega datos pero no es la única forma ni la que da recomendaciones de postura.

## 🧠 Resumen para el examen

1. Defender for Cloud = antiguo **Azure Security Center**.
2. Evalúa la **postura de seguridad** y da **recomendaciones**.
3. **Secure Score**: porcentaje que sube al aplicar recomendaciones.
4. Detecta y alerta sobre **amenazas** en tiempo real.
5. **Just-in-time VM access** limita los puertos de administración.
6. Tiene **nivel gratuito**; los planes Defender de pago añaden protección avanzada.
7. Cubre **Azure, on-premises (Arc), AWS y GCP**.
8. No es un SIEM: eso es **Sentinel**.
9. No es Azure Policy: Policy mide cumplimiento de *tus* normas, no la seguridad.

---
⬅️ [[00 - Índice - Seguridad|Volver al índice de Seguridad]]
