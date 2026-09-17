---
tags: [ai-901, azure, ia, responsible-ai]
módulo: Responsible AI
peso_examen: Alto
objetivo_oficial: "1.1.6 Describe considerations for accountability in an AI solution"
---

# Accountability (Responsabilidad)

## ¿Qué significa?

**Las personas y organizaciones deben ser responsables de los sistemas de IA** que crean y operan. La IA no "decide sola": debe existir un marco de **gobernanza**, personas con autoridad para revisar y corregir, y cumplimiento de normas legales y éticas.

## Riesgo que intenta evitar

- Que nadie responda cuando el sistema causa un perjuicio.
- Despliegues sin revisión, sin auditoría y sin posibilidad de apelación.
- Incumplir regulación (protección de datos, normas sectoriales, leyes de IA).

## Consideraciones en una solución

- Definir **roles y responsables** (propietario del sistema, comité de revisión).
- Aplicar un **marco de gobierno** (p. ej., el *Responsible AI Standard* de Microsoft) con revisiones antes del despliegue.
- **Humano en el bucle** con capacidad de anular decisiones y proceso de **apelación**.
- **Auditoría**: registros, evaluaciones periódicas, trazabilidad (tracing y evaluaciones en Foundry).
- **Cumplimiento legal y normativo**.

## Ejemplo

Antes de lanzar un agente que responde consultas fiscales, la empresa nombra un responsable del producto, un comité revisa las evaluaciones de seguridad y calidad, se establece un canal para reclamar y se guarda el registro de todas las conversaciones para auditoría.

## 🧠 Memorizar

> [!important]
> - Accountability = **alguien responde**: gobernanza, responsables, auditoría, cumplimiento, apelación.
> - Es el principio "paraguas": garantiza que los otros cinco se apliquen de verdad.

## Tips para AI-901

> [!tip]
> - ⭐ Palabras clave: *responsable, gobernanza, marco, comité, auditoría, cumplimiento legal, revisión humana con autoridad, apelar una decisión*.
> - ⚠️ "Explicar la decisión al usuario" es transparency; "poder recurrir la decisión ante una persona responsable" es accountability.

## Preguntas que podrían aparecer

**1.** Una organización crea un comité que revisa cada sistema de IA antes de su despliegue y define quién responde de sus resultados. ¿Qué principio aplica?
- A) Fairness · B) Transparency · C) Accountability · D) Inclusiveness

<details><summary>Respuesta</summary>

**C.** Gobernanza y responsables definidos.
</details>

**2.** Verdadero o falso: según los principios de Microsoft, un sistema de IA suficientemente preciso puede tomar decisiones de alto impacto sin que ninguna persona sea responsable de ellas.

<details><summary>Respuesta</summary>

**Falso.** La responsabilidad siempre recae en las personas y organizaciones.
</details>

## Relacionado

- [[Responsible AI (principios de Microsoft)]]
- [[Transparency (Transparencia)]]
- [[Gobernanza en Azure]] (gobernanza de recursos en Azure, AZ-900)

← Volver al índice: [[00 - Índice - Responsible AI]]
