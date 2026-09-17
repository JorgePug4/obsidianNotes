---
tags: [ai-901, azure, ia, responsible-ai]
módulo: Responsible AI
peso_examen: Alto
objetivo_oficial: "1.1.5 Describe considerations for transparency in an AI solution"
---

# Transparency (Transparencia)

## ¿Qué significa?

Las personas deben **entender** qué hace el sistema de IA, **cómo** toma decisiones, **con qué datos** trabaja y **cuáles son sus limitaciones**. Incluye informar a los usuarios de que están interactuando con una IA y hacer que las decisiones sean **explicables**.

## Riesgo que intenta evitar

- Confianza excesiva en respuestas que pueden ser erróneas.
- Decisiones inexplicables (una "caja negra" que rechaza una solicitud sin motivo).
- Usuarios que creen hablar con una persona.
- No saber qué fuentes usó una respuesta generativa.

## Consideraciones en una solución

- **Documentar** propósito, capacidades, limitaciones y datos del sistema (Microsoft publica *transparency notes* de sus modelos y servicios).
- **Explicabilidad / interpretabilidad**: mostrar los factores que influyeron en una predicción.
- **Citas y fuentes** en respuestas RAG (Foundry IQ devuelve citas).
- **Avisos**: "Este contenido ha sido generado por IA y puede contener errores".
- **Registro y trazabilidad** (tracing en Foundry) para poder auditar respuestas.

## Ejemplo

Un agente de soporte muestra al final de cada respuesta las secciones de la documentación en las que se basó y un aviso de que es un asistente de IA. Un cliente que no está de acuerdo puede pedir a una persona que revise el caso.

## 📌 Transparency vs Accountability

| Transparency | Accountability |
|---|---|
| Que se **entienda** el sistema | Que alguien **responda** por él |
| Documentación, explicaciones, citas, avisos | Gobernanza, responsables, auditorías, cumplimiento |

## 🧠 Memorizar

> [!important]
> - Transparency = **entender + informar + explicar + citar**.
> - Herramientas: *transparency notes*, citas en RAG, tracing, avisos de IA.

## Tips para AI-901

> [!tip]
> - ⭐ Palabras clave: *explicar, informar, documentar limitaciones, fuentes, cita, interpretable, el usuario sabe que es una IA*.
> - ⚠️ "Designar un responsable" o "cumplir la ley" → accountability.

## Preguntas que podrían aparecer

**1.** Un sistema de préstamos rechaza solicitudes sin indicar los factores que llevaron a la decisión. ¿Qué principio se incumple?
- A) Transparency · B) Privacy and security · C) Inclusiveness · D) Reliability and safety

<details><summary>Respuesta</summary>

**A.** Las decisiones deben ser explicables.
</details>

**2.** ¿Qué práctica en un agente RAG apoya la transparencia?
- A) Cifrar el índice de búsqueda · B) Mostrar citas de los documentos usados en cada respuesta · C) Reducir la temperatura · D) Limitar el número de tokens

<details><summary>Respuesta</summary>

**B.** Las citas permiten al usuario verificar el origen de la información.
</details>

## Relacionado

- [[Responsible AI (principios de Microsoft)]]
- [[Accountability (Responsabilidad)]]
- [[Grounding, RAG y Foundry IQ]]

← Volver al índice: [[00 - Índice - Responsible AI]]
