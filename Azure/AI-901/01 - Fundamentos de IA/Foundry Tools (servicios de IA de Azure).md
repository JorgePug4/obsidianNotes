---
tags: [ai-901, azure, ia, foundry, servicios]
módulo: Fundamentos de IA
peso_examen: Muy alto
aliases: [Foundry Tools, Azure AI Services, Azure Cognitive Services, Servicios de IA de Azure]
---

# Foundry Tools (servicios de IA de Azure)

## ¿Qué es?

**Foundry Tools** es el nombre actual (2026) de la familia de **servicios de IA preconstruidos** de Azure, conocidos antes como *Azure AI Services* y, antes, *Azure Cognitive Services*. Son capacidades listas para usar (sin entrenar nada) que devuelven **resultados estructurados y deterministas** a través de una API o un SDK.

En el portal de Foundry se prueban en **Build → Services**.

## ¿Para qué sirve?

Para añadir a una aplicación o a un agente capacidades concretas de voz, lenguaje, traducción, visión y extracción de información con salida predecible, ideal para **pipelines automatizados** donde un modelo generativo sería demasiado variable.

## El catálogo que debes conocer

| Servicio (nombre actual) | Nombre anterior | Qué hace | Nota |
|---|---|---|---|
| **Azure Language in Foundry Tools** | Azure AI Language / Text Analytics | Detección de idioma, **PII**, entidades (NER), sentimiento, frases clave, resumen | Varias funciones clásicas están en retirada progresiva (hasta 2029); lo vigente en Foundry: idioma, PII, NER. Ver [[Azure AI Language]] |
| **Azure Speech in Foundry Tools** | Azure AI Speech | Voz → texto, texto → voz, traducción de voz, **Voice Live** para agentes de voz | Ver [[Azure AI Speech]] |
| **Azure Translator in Foundry Tools** | Azure AI Translator | Traducción de texto y documentos | Ver [[Azure AI Translator]] |
| **Azure Vision in Foundry Tools** | Azure AI Vision / Computer Vision | Análisis de imagen (descripción, etiquetas, objetos, personas, OCR), Face | Ver [[Azure AI Vision]] |
| **Azure Content Understanding in Foundry Tools** | (nuevo; integra Document Intelligence) | Extraer información de **documentos, imágenes, audio y vídeo** en JSON | Ver [[Azure Content Understanding]] |
| **Azure Document Intelligence in Foundry Tools** | Form Recognizer | Modelos preconstruidos de facturas, recibos, ID… | Ahora parte de Content Understanding. Ver [[Azure AI Document Intelligence]] |
| **Azure AI Content Safety** | – | Moderación de texto/imagen, prompt shields, groundedness | Ver [[Azure AI Content Safety y guardrails]] |
| **Azure AI Search / Foundry IQ** | Cognitive Search | Índices y bases de conocimiento para RAG | Ver [[Grounding, RAG y Foundry IQ]] |
| **Custom Vision** | – | Clasificación/detección de objetos con tus propias imágenes | 🟡 contexto |

## Cómo funciona

1. El recurso de Foundry ya incluye el acceso a los servicios (o creas un recurso específico).
2. Llamas al servicio por REST o SDK con **endpoint + clave** o identidad de Entra.
3. Recibes JSON con resultados y **puntuaciones de confianza**.

## Ejemplo

Un agente de atención al cliente recibe un correo en alemán: **Language** detecta el idioma → **Translator** lo traduce → **Language (PII)** redacta datos personales antes de guardarlo → el modelo generativo redacta la respuesta.

## 📌 Foundry Tools vs modelo generativo

| | Foundry Tools | Modelo generativo (LLM) |
|---|---|---|
| Resultado | Estructurado, **determinista**, con confianza | Texto libre, puede variar |
| Se guía con | Parámetros de API | Prompt |
| Coste/latencia | Bajos y predecibles | Mayores, según tokens |
| Cuándo | Pipelines, cumplimiento, volumen, tareas concretas (PII, OCR, transcripción) | Tareas abiertas, resumen flexible, conversación |
| Ejemplo oficial | Azure Language – Text PII Redaction | `gpt-5-mini` resumiendo una reseña |

Los laboratorios oficiales insisten en que **ambos enfoques son válidos** y que un agente puede usar una Foundry Tool como herramienta para obtener resultados más predecibles.

## Tabla rápida: necesidad → servicio

| Necesidad | Servicio / capacidad |
|---|---|
| Convertir voz en texto | Azure Speech (speech-to-text) |
| Convertir texto en voz | Azure Speech (text-to-speech) |
| Agente de voz en tiempo real | Azure Speech **Voice Live** |
| Traducir texto | Azure Translator |
| Detectar idioma / PII / entidades | Azure Language |
| Analizar sentimiento | Azure Language o modelo generativo |
| Describir/etiquetar imágenes | Azure Vision o modelo multimodal |
| Leer texto de imágenes (OCR) | Content Understanding (OCR/Read) o Azure Vision |
| Extraer campos de facturas/recibos | Content Understanding (prebuilt-invoice / receipt) |
| Extraer información de audio/vídeo | Content Understanding |
| Generar texto, imágenes | Modelos del catálogo de Foundry |
| Entrenar modelos ML propios | Azure Machine Learning |
| Moderar contenido, bloquear jailbreaks | Azure AI Content Safety / guardrails |

## 🧠 Memorizar

> [!important]
> - Foundry Tools = **Azure AI Services = Cognitive Services** (mismo producto, nuevo nombre).
> - Sus resultados son **deterministas y estructurados**; los de un LLM, no.
> - Sufijo oficial: "*Azure X in Foundry Tools*".
> - Content Understanding cubre **4 modalidades**: documento, imagen, audio, vídeo.

## Tips para AI-901

> [!tip]
> - ⭐ La pregunta clásica es "necesidad → servicio". Aprende la tabla de arriba.
> - 🔥 Si el enunciado exige "resultados consistentes", "cumplimiento normativo", "detectar y redactar PII" → Foundry Tools, no un prompt.
> - ⚠️ Si dice "el equipo quiere flexibilidad para pedir cosas distintas en lenguaje natural" → modelo generativo.

## ⚠️ Errores comunes

- Responder "Azure OpenAI" a todo. Muchos escenarios de AI-901 tienen una herramienta específica más adecuada.
- Confundir Azure Vision (analizar) con generación de imágenes (modelos generativos).

## 💡 Escenario

Una clínica debe eliminar automáticamente nombres, teléfonos y direcciones de miles de notas clínicas antes de almacenarlas, con un resultado auditable. ¿Qué usarías?

<details><summary>Respuesta</summary>

**Azure Language in Foundry Tools – Text PII Redaction**. Es una tarea concreta, con volumen y necesidad de resultado determinista y auditable; un prompt a un LLM sería menos fiable.
</details>

## Preguntas que podrían aparecer

**1.** ¿Cómo se llama actualmente la colección de servicios de IA preconstruidos de Azure que antes se conocía como Azure AI Services?
- A) Foundry Models · B) Foundry Tools · C) Foundry IQ · D) Foundry Agent Service

<details><summary>Respuesta</summary>

**B.** Foundry Models es el catálogo de modelos; Foundry IQ es conocimiento; Agent Service ejecuta agentes.
</details>

**2.** Una empresa necesita transcribir automáticamente grabaciones de reuniones y luego traducir el texto al inglés. ¿Qué dos servicios usarías?
- A) Azure Vision y Azure Language
- B) Azure Speech y Azure Translator
- C) Content Understanding y Azure Machine Learning
- D) Azure Translator y Azure Vision

<details><summary>Respuesta</summary>

**B.** Speech transcribe (speech-to-text) y Translator traduce el texto. También sería válido Content Understanding para audio, pero la combinación pedida es B.
</details>

## Relacionado

- [[Microsoft Foundry]]
- [[Azure AI Language]] · [[Azure AI Speech]] · [[Azure AI Translator]]
- [[Azure AI Vision]] · [[Azure Content Understanding]] · [[Azure AI Document Intelligence]]
- [[Azure AI Content Safety y guardrails]]
- [[AI-901 - Escenarios rápidos (Problema → Tecnología)]]

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
