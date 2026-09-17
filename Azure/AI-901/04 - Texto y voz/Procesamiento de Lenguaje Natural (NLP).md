---
tags: [ai-901, azure, ia, nlp]
módulo: Texto y voz
peso_examen: Alto
aliases: [NLP, Natural Language Processing, Procesamiento de lenguaje natural]
---

# Procesamiento de Lenguaje Natural (NLP)

## ¿Qué es?

El **Procesamiento de Lenguaje Natural (NLP)** es el área de la IA que permite a las máquinas **entender, interpretar y generar lenguaje humano**, escrito o hablado.

## ¿Para qué sirve?

Resuelve todo lo que tiene que ver con texto y conversación: clasificar correos, detectar el idioma, extraer entidades, medir opiniones, resumir documentos, traducir, responder preguntas y mantener conversaciones.

## Conceptos clave

- **Análisis de texto**: extraer información estructurada de texto existente. Ver [[Técnicas de análisis de texto]].
- **Comprensión del lenguaje**: interpretar la **intención** del usuario y sus **entidades** ("reserva un vuelo **a Madrid** **mañana**").
- **Generación de lenguaje**: producir texto nuevo (lo hacen los [[Large Language Models]]).
- **Corpus, tokenización, normalización, lematización**: preprocesado clásico (🟡 contexto).
- **Dos enfoques hoy**: servicios **especializados y deterministas** ([[Azure AI Language]]) o **modelos generativos** con prompts.

## Cómo funciona: los dos caminos de AI-901

El laboratorio oficial de análisis de texto lo plantea así: *"Foundry ofrece dos enfoques para el análisis de texto: modelos de IA de propósito general que resuelven un abanico amplio de tareas mediante prompts en lenguaje natural, y herramientas de lenguaje específicas que devuelven resultados estructurados y deterministas para tareas concretas"*.

| | Modelo generativo (LLM) | Azure Language in Foundry Tools |
|---|---|---|
| Cómo se pide | Prompt en lenguaje natural | Llamada a la API con parámetros |
| Salida | Texto libre (puede variar) | JSON estructurado con confianza |
| Fuerte en | Resumen flexible, redacción, tareas abiertas, varios idiomas | PII, idioma, entidades; volumen y consistencia |
| Cuándo | Necesitas flexibilidad y lenguaje natural | Necesitas resultados predecibles y auditables en un pipeline |

## Ejemplo

Un centro de soporte recibe 10 000 correos al día. Usa **Azure Language** para detectar idioma y redactar PII (determinista, obligatorio por cumplimiento) y un **LLM** para resumir el caso y proponer una respuesta (flexible).

## 🧠 Memorizar

> [!important]
> - NLP = entender y generar lenguaje humano.
> - Los **LLM son NLP**: nacen de esta disciplina.
> - Dos caminos en Foundry: **modelo generativo** (flexible) o **Azure Language** (determinista).
> - Si el escenario incluye **audio**, entra [[Azure AI Speech]]; el texto resultante ya es NLP.

## Tips para AI-901

> [!tip]
> - ⭐ "Resultados consistentes, alto volumen, cumplimiento" → Azure Language.
> - ⭐ "Tarea abierta descrita en lenguaje natural" → modelo generativo.
> - ⚠️ El examen ya no pregunta por LUIS ni por el pipeline clásico de NLP.

## Preguntas que podrían aparecer

**1.** ¿Cuál de estas afirmaciones sobre NLP y LLM es correcta?
- A) Los LLM no forman parte del NLP
- B) Los LLM son modelos de NLP capaces de entender y generar lenguaje
- C) El NLP solo trabaja con voz
- D) El NLP requiere siempre datos etiquetados

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Técnicas de análisis de texto]] · [[Azure AI Language]]
- [[Reconocimiento y síntesis de voz]] · [[Azure AI Speech]] · [[Azure AI Translator]]
- [[Large Language Models]] · [[Cargas de trabajo de IA]]

← Volver al índice: [[00 - Índice - Texto y voz]]
