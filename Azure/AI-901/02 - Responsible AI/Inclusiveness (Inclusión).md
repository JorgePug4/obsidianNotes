---
tags: [ai-901, azure, ia, responsible-ai]
módulo: Responsible AI
peso_examen: Alto
objetivo_oficial: "1.1.4 Describe considerations for inclusiveness in an AI solution"
---

# Inclusiveness (Inclusión)

## ¿Qué significa?

La IA debe **empoderar a todas las personas** y ser **accesible y útil** con independencia de sus capacidades físicas, idioma, cultura, edad o nivel de recursos. Nadie debería quedar fuera del beneficio de la tecnología.

## Riesgo que intenta evitar

- Excluir a personas con **discapacidad** (visual, auditiva, motora, cognitiva).
- Funcionar solo en un **idioma** o para un **acento** concreto.
- Interfaces que exigen un canal único (solo texto, solo voz).
- Modelos que no reconocen bien a determinados grupos (p. ej., reconocimiento de voz que falla con hablantes no nativos).

## Consideraciones en una solución

- Ofrecer **varios canales**: texto, voz (Azure Speech), subtítulos, lectura en voz alta.
- Soporte **multilingüe** (Azure Translator, modelos multilingües, detección de idioma).
- Cumplir pautas de **accesibilidad** (contraste, lectores de pantalla, descripciones de imagen generadas con visión).
- Probar con usuarios diversos.

## Ejemplo

Una app municipal genera **descripciones automáticas de imágenes** (Azure Vision o un modelo multimodal) para personas ciegas, ofrece **texto a voz** para quienes no pueden leer la pantalla y traduce el contenido a los idiomas más hablados en la ciudad.

## 📌 Inclusiveness vs Fairness

| Inclusiveness | Fairness |
|---|---|
| ¿Puede **usarlo** y beneficiarse todo el mundo? | ¿Son **justos los resultados** entre grupos? |
| Accesibilidad, idiomas, canales | Sesgo, discriminación, paridad |
| "Añadir subtítulos" | "Misma tasa de aprobación para perfiles equivalentes" |

## 🧠 Memorizar

> [!important]
> - Inclusiveness = **accesible y útil para todos**.
> - Herramientas típicas: **Speech (voz), Translator (idiomas), Vision (descripciones de imágenes)**.

## Tips para AI-901

> [!tip]
> - ⭐ Palabras clave: *discapacidad, accesibilidad, todos los usuarios, idiomas, subtítulos, lectores de pantalla, no dejar a nadie fuera*.
> - ⚠️ Si el enunciado habla de **resultados injustos** entre grupos, es fairness, no inclusiveness.

## Preguntas que podrían aparecer

**1.** Una empresa añade a su chatbot la opción de interactuar por voz y de recibir las respuestas en audio para usuarios con baja visión. ¿Qué principio aplica?
- A) Fairness · B) Inclusiveness · C) Transparency · D) Accountability

<details><summary>Respuesta</summary>

**B.** Se amplía el acceso a personas con discapacidad.
</details>

**2.** ¿Qué capacidad de Foundry Tools apoya de forma más directa la inclusión lingüística?
- A) Azure Translator · B) Content Understanding · C) Azure AI Content Safety · D) Azure Machine Learning

<details><summary>Respuesta</summary>

**A.** Traducir permite que usuarios de distintos idiomas usen la solución.
</details>

## Relacionado

- [[Responsible AI (principios de Microsoft)]]
- [[Fairness (Equidad)]]
- [[Azure AI Speech]] · [[Azure AI Translator]] · [[Azure AI Vision]]

← Volver al índice: [[00 - Índice - Responsible AI]]
