---
tags: [ai-901, azure, ia, responsible-ai]
módulo: Responsible AI
peso_examen: Alto
objetivo_oficial: "1.1.2 Describe considerations for reliability and safety in an AI solution"
---

# Reliability and Safety (Fiabilidad y seguridad)

## ¿Qué significa?

Un sistema de IA debe **funcionar de forma consistente y predecible** (fiabilidad) y **no causar daño** a personas ni bienes, incluso en situaciones no previstas (seguridad, *safety*).

## Riesgo que intenta evitar

- Predicciones erróneas con consecuencias graves (diagnóstico, vehículos, control industrial).
- Comportamientos inesperados ante datos distintos a los de entrenamiento.
- En IA generativa: **alucinaciones** (información inventada presentada como cierta), contenido **dañino** (violencia, autolesión, odio, sexual), instrucciones peligrosas.

## Consideraciones en una solución

- **Pruebas rigurosas** antes de desplegar y **monitorización** después.
- **Umbrales de confianza**: si el modelo no está seguro, derivar a un humano.
- **Humano en el bucle** para decisiones de alto impacto.
- **Grounding** de las respuestas generativas en fuentes fiables (RAG) y detección de *groundedness*.
- **Filtros de contenido / guardrails** para bloquear salidas dañinas.
- **Degradación segura**: si algo falla, el sistema debe fallar de forma controlada.

## Ejemplo

Un sistema de visión detecta defectos en piezas de aviación. Se exige una tasa de error mínima, se prueba con miles de imágenes reales, y toda pieza con confianza baja pasa a inspección manual.

## Ejemplo con IA generativa

Un chatbot de farmacia sugiere dosis. Se conecta a una base de conocimiento oficial (grounding), se activan los filtros de autolesión y violencia, y se evalúa periódicamente la *groundedness* de sus respuestas.

## 📌 Safety vs Security

| | Safety (en Reliability and safety) | Security (en Privacy and security) |
|---|---|---|
| De qué protege | Del **daño** que el sistema puede causar a personas | De **ataques** y accesos indebidos al sistema y sus datos |
| Ejemplo | Bloquear contenido de autolesión; coche que frena bien | Bloquear un prompt injection; cifrar datos |

## 🧠 Memorizar

> [!important]
> - Reliability = **consistente**. Safety = **sin daño**.
> - Alucinaciones y contenido dañino son problemas de **reliability and safety**.
> - Medidas: pruebas, umbrales de confianza, humano en el bucle, guardrails, groundedness.

## Tips para AI-901

> [!tip]
> - ⭐ Palabras clave: *comportamiento inesperado, condiciones no previstas, daño, alucinación, contenido dañino, pruebas exhaustivas, umbral de confianza*.
> - 🔥 "Filtros de contenido de Foundry que bloquean violencia o autolesión" → safety.
> - ⚠️ "Cifrar datos" o "impedir ataques" → security (otro principio).

## Preguntas que podrían aparecer

**1.** Un asistente generativo de soporte técnico inventa números de teléfono cuando no conoce la respuesta. ¿Qué principio se ve comprometido y qué medida ayuda?
- A) Fairness; reequilibrar los datos
- B) Reliability and safety; grounding en una base de conocimiento y detección de groundedness
- C) Accountability; nombrar un responsable
- D) Inclusiveness; añadir idiomas

<details><summary>Respuesta</summary>

**B.** Las alucinaciones son un fallo de fiabilidad; el grounding y la comprobación de groundedness lo mitigan.
</details>

**2.** ¿Qué práctica apoya directamente la fiabilidad y seguridad de un modelo de diagnóstico?
- A) Publicar sus limitaciones · B) Derivar a un médico los casos con confianza baja · C) Traducir la interfaz · D) Anonimizar los datos

<details><summary>Respuesta</summary>

**B.** Umbral de confianza con humano en el bucle. A es transparency, C inclusiveness, D privacy.
</details>

## Relacionado

- [[Responsible AI (principios de Microsoft)]]
- [[Azure AI Content Safety y guardrails]]
- [[Grounding, RAG y Foundry IQ]]
- [[Privacy and Security (Privacidad y seguridad)]]

← Volver al índice: [[00 - Índice - Responsible AI]]
