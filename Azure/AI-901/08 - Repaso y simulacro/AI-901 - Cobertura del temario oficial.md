---
tags: [ai-901, azure, ia, meta, cobertura]
certificación: Azure AI Fundamentals (AI-901)
módulo: Repaso final
---

# ✅ AI-901 · Cobertura del temario oficial

Matriz de verificación entre las **27 habilidades medidas** de la guía oficial de AI-901 (versión de habilidades medidas a **15 de abril de 2026**) y las notas de esta carpeta. Sirve para comprobar que no falta nada antes del examen.

## Dominio 1 · Identify AI concepts and responsibilities (40–45 %)

### 1.1 Describe principles of responsible AI

| # | Habilidad oficial | Nota |
|---|---|---|
| 1.1.1 | Consideraciones de **fairness** | [[Fairness (Equidad)]] |
| 1.1.2 | Consideraciones de **reliability and safety** | [[Reliability and Safety (Fiabilidad y seguridad)]] |
| 1.1.3 | Consideraciones de **privacy and security** | [[Privacy and Security (Privacidad y seguridad)]] |
| 1.1.4 | Consideraciones de **inclusiveness** | [[Inclusiveness (Inclusión)]] |
| 1.1.5 | Consideraciones de **transparency** | [[Transparency (Transparencia)]] |
| 1.1.6 | Consideraciones de **accountability** | [[Accountability (Responsabilidad)]] |

Apoyo transversal: [[Responsible AI (principios de Microsoft)]] · [[Azure AI Content Safety y guardrails]]

### 1.2 Identify AI model components and configurations

| # | Habilidad oficial | Nota |
|---|---|---|
| 1.2.1 | Cómo funcionan los modelos generativos | [[Large Language Models]] · [[Tokens y Embeddings]] · [[Generative AI]] |
| 1.2.2 | Elegir un modelo apropiado según capacidades | [[Catálogo de modelos de Foundry]] |
| 1.2.3 | Opciones de despliegue y parámetros de configuración | [[Despliegue y configuración de modelos]] |

### 1.3 Identify AI workloads

| # | Habilidad oficial | Nota |
|---|---|---|
| 1.3.1 | Escenarios de cargas de trabajo (GenAI y agentes, texto, voz, visión, extracción) | [[Cargas de trabajo de IA]] · [[Inteligencia Artificial]] |
| 1.3.2 | Técnicas de análisis de texto (frases clave, entidades, sentimiento, resumen) | [[Técnicas de análisis de texto]] |
| 1.3.3 | Reconocimiento y síntesis de voz | [[Reconocimiento y síntesis de voz]] |
| 1.3.4 | Modelos de visión y de generación de imágenes | [[Computer Vision]] · [[Generación de imágenes y vídeo]] |
| 1.3.5 | Técnicas de extracción de información (texto, imágenes, audio, vídeo) | [[Extracción de información]] |

## Dominio 2 · Implement AI solutions by using Microsoft Foundry (55–60 %)

### 2.1 Implement generative AI apps and agents by using Foundry

| # | Habilidad oficial | Nota | Lab |
|---|---|---|---|
| 2.1.1 | Crear prompts de sistema y de usuario eficaces | [[Prompts y Prompt Engineering]] | [[Lab 02 - Desplegar un modelo y crear un agente]] |
| 2.1.2 | Desplegar un modelo e interactuar con él en el portal | [[Despliegue y configuración de modelos]] | [[Lab 02 - Desplegar un modelo y crear un agente]] |
| 2.1.3 | Cliente de chat ligero con el Foundry SDK | [[Foundry SDK y cliente de chat]] | [[Lab 08 - Cliente Python con el Foundry SDK]] |
| 2.1.4 | Crear y probar una solución de agente único en el portal | [[Agentes de IA (Foundry Agent Service)]] | [[Lab 02 - Desplegar un modelo y crear un agente]] · [[Lab 07 - Foundry IQ (conocimiento para agentes)]] |
| 2.1.5 | Cliente ligero para un agente | [[Agentes de IA (Foundry Agent Service)]] · [[Foundry SDK y cliente de chat]] | [[Lab 08 - Cliente Python con el Foundry SDK]] |

### 2.2 Implement AI solutions for text and speech by using Foundry

| # | Habilidad oficial | Nota | Lab |
|---|---|---|---|
| 2.2.1 | Aplicación ligera con análisis de texto | [[Azure AI Language]] · [[Técnicas de análisis de texto]] | [[Lab 03 - Análisis de texto en Foundry]] |
| 2.2.2 | Responder a prompts hablados con un modelo multimodal | [[Azure AI Speech]] · [[Reconocimiento y síntesis de voz]] | [[Lab 04 - Agente de voz con Voice Live]] |
| 2.2.3 | Aplicación ligera con Azure Speech in Foundry Tools | [[Azure AI Speech]] | [[Lab 04 - Agente de voz con Voice Live]] |

### 2.3 Implement AI solutions with computer vision and image-generation capabilities

| # | Habilidad oficial | Nota | Lab |
|---|---|---|---|
| 2.3.1 | Interpretar entrada visual en prompts con un modelo multimodal | [[Modelos multimodales (visión en prompts)]] | [[Lab 05 - Visión y generación de imágenes]] |
| 2.3.2 | Crear salidas visuales nuevas con modelos generativos | [[Generación de imágenes y vídeo]] | [[Lab 05 - Visión y generación de imágenes]] |
| 2.3.3 | Aplicación ligera con capacidades de visión | [[Modelos multimodales (visión en prompts)]] · [[Azure AI Vision]] | [[Lab 05 - Visión y generación de imágenes]] |

### 2.4 Implement AI solutions for information extraction by using Foundry

| # | Habilidad oficial | Nota | Lab |
|---|---|---|---|
| 2.4.1 | Extraer de documentos y formularios con Content Understanding | [[Azure Content Understanding]] · [[Azure AI Document Intelligence]] | [[Lab 06 - Content Understanding (extracción de información)]] |
| 2.4.2 | Extraer de imágenes con Content Understanding | [[Azure Content Understanding]] · [[OCR (Reconocimiento óptico de caracteres)]] | [[Lab 06 - Content Understanding (extracción de información)]] |
| 2.4.3 | Extraer de audio y vídeo con Content Understanding | [[Azure Content Understanding]] | [[Lab 06 - Content Understanding (extracción de información)]] |
| 2.4.4 | Aplicación ligera de extracción con Content Understanding | [[Azure Content Understanding]] | [[Lab 06 - Content Understanding (extracción de información)]] |

## Notas complementarias (no son objetivos, pero ayudan)

| Nota | Por qué está |
|---|---|
| [[Microsoft Foundry]] · [[Foundry Tools (servicios de IA de Azure)]] | Plataforma sobre la que se apoya todo el dominio 2 |
| [[Grounding, RAG y Foundry IQ]] | Sustenta 2.1.4 y la fiabilidad de las respuestas |
| [[Procesamiento de Lenguaje Natural (NLP)]] | Contexto de 1.3.2 |
| [[Azure AI Translator]] | Aparece en escenarios de selección de servicio |
| [[Azure AI Vision]] · [[Reconocimiento facial (Face)]] | Distractores y alternativas en escenarios de visión |
| [[Machine Learning (fundamentos)]] · [[Tipos de Machine Learning]] · [[Evaluación de modelos de ML]] · [[Azure Machine Learning]] | 🟡 Contexto heredado de AI-900; útil para descartar distractores |
| [[Copilot y asistentes de Microsoft]] | 🟡 Contexto; distinguir Copilot Studio de Foundry |
| [[AI-900 vs AI-901 - Qué cambió]] | Evita estudiar temario retirado |

## Verificación

- **27 de 27** habilidades oficiales cubiertas con al menos una nota.
- **8 laboratorios** que cubren las cuatro secciones del dominio 2.
- **6 repasos finales** de módulo + [[AI-901 - Guía de repaso final]] + [[AI-901 - Escenarios rápidos (Problema → Tecnología)]].
- **50 preguntas** de simulacro con respuestas separadas.
- Sin contenido de administración de Azure (AZ-104) ni dominios retirados de AI-900 salvo lo marcado 🟡 contexto.

> [!warning] Antes del examen, verifica el temario vigente
> Microsoft actualiza las habilidades medidas periódicamente. Estas notas siguen la versión de **15 de abril de 2026**. Comprueba la fecha de "skills measured as of" en la guía oficial: `learn.microsoft.com/credentials/certifications/resources/study-guides/ai-901`.

---

Volver a: [[AI-901 - Guía de repaso final]] · [[00 - AI-901 Índice general]]
