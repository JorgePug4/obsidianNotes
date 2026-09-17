---
tags: [ai-901, azure, ia, nlp, voz, repaso]
módulo: Texto y voz
---

# 🎯 Repaso final · Texto y voz

## 1. Los conceptos más importantes

1. Las **cuatro técnicas oficiales** de análisis de texto: **frases clave, entidades, sentimiento y resumen**.
2. Los labs añaden **detección de idioma** y **PII redaction**, que son hoy las capacidades vigentes de Azure Language en Foundry.
3. **Sentimiento** → positivo, negativo, neutro o **mixto**, con puntuaciones.
4. **STT** = voz a texto; **TTS** = texto a voz; **SSML** controla la voz.
5. **Voice Live** integra STT, TTS, turnos e interrupciones para agentes de voz en tiempo real.
6. **Translator** traduce texto y documentos; **Speech** traduce voz.
7. Dos caminos para el análisis de texto: **servicio determinista** o **modelo generativo flexible**.

## 2. Tabla necesidad → servicio

| Necesidad | Servicio / capacidad |
|---|---|
| Convertir voz en texto | [[Azure AI Speech]] – speech to text |
| Convertir texto en voz | [[Azure AI Speech]] – text to speech |
| Conversación de voz en tiempo real con un agente | [[Azure AI Speech]] – **Voice Live** |
| Traducir texto o documentos | [[Azure AI Translator]] |
| Traducir voz | [[Azure AI Speech]] – speech translation |
| Analizar sentimiento | [[Azure AI Language]] o modelo generativo |
| Detectar entidades (personas, lugares, fechas) | [[Azure AI Language]] – NER |
| Extraer frases clave | [[Azure AI Language]] o modelo generativo |
| Resumir un documento | Modelo generativo (o resumen de Azure Language) |
| Detectar el idioma | [[Azure AI Language]] – language detection |
| Detectar y ocultar datos personales | [[Azure AI Language]] – PII redaction |
| Extraer campos de una grabación o un documento | [[Azure Content Understanding]] |

## 3. Diferencias que más se confunden

| Par confuso | Cómo distinguirlo |
|---|---|
| Frases clave vs entidades | Temas principales / elementos clasificados por categoría |
| Entidades vs PII | Cualquier entidad / datos personales, con redacción |
| Sentimiento vs clasificación | Polaridad de la opinión / categorías propias |
| Resumen extractivo vs abstractivo | Frases del original / frases nuevas |
| Detección de idioma vs traducción | Identificar / convertir |
| STT vs TTS | Voz→texto / texto→voz |
| Voice Live vs STT+TTS | Tiempo real con interrupciones / pasos encadenados |
| Speech vs Translator | Audio / texto |
| Speech vs Content Understanding | Transcribir / extraer campos |
| Azure Language vs LLM | Determinista y barato / flexible y generativo |

## 4. Tips de examen

> [!tip]
> 1. Primero mira la **entrada**: si hay audio, la respuesta pasa por Speech.
> 2. "Resultados consistentes y auditables" → servicio; "tarea abierta" → modelo generativo.
> 3. "Mixto" existe como resultado de sentimiento.
> 4. "Conservando el formato del documento" → document translation.
> 5. "Quién dijo qué" → diarización.
> 6. "Interrupciones y conversación natural" → Voice Live.
> 7. Si piden **campos estructurados** de un audio o documento, cambia de módulo: es Content Understanding.

## 5. Preguntas de repaso

**1.** ¿Qué técnica identifica los temas principales de un documento sin clasificarlos en categorías?

<details><summary>Respuesta</summary>

**Extracción de frases clave (key phrase extraction).**
</details>

**2.** Una empresa quiere subtitular vídeos y saber qué ponente habla en cada momento. ¿Qué capacidades?

<details><summary>Respuesta</summary>

**Speech to text** con **diarización**.
</details>

**3.** ¿Qué servicio traduce automáticamente los manuales PDF de una empresa a cinco idiomas conservando el formato?

<details><summary>Respuesta</summary>

**Azure Translator**, traducción de documentos.
</details>

**4.** ¿Qué capacidad de Azure Language es imprescindible para cumplir con normativas de protección de datos al archivar textos?

<details><summary>Respuesta</summary>

**PII detection / redaction.**
</details>

**5.** Verdadero o falso: para que un agente responda a prompts hablados es obligatorio implementar STT y TTS por separado.

<details><summary>Respuesta</summary>

**Falso.** Se puede usar un modelo multimodal de audio o activar **Voice Live** en el agente.
</details>

**6.** ¿Qué lenguaje permite ajustar pausas y pronunciación en la síntesis de voz?

<details><summary>Respuesta</summary>

**SSML.**
</details>

**7.** Un pipeline debe enrutar cada comentario al modelo del idioma correcto. ¿Qué capacidad va primero?

<details><summary>Respuesta</summary>

**Detección de idioma.**
</details>

**8.** ¿Qué diferencia hay entre analizar sentimiento con Azure Language y con un modelo generativo?

<details><summary>Respuesta</summary>

Azure Language devuelve etiquetas fijas y puntuaciones de forma determinista y barata; el modelo generativo es más flexible y matizado, pero menos predecible y más caro.
</details>

## Checklist

- [ ] Comprendo las cuatro técnicas oficiales de análisis de texto y sé reconocerlas en escenarios.
- [ ] Sé identificar las capacidades de reconocimiento y síntesis de voz.
- [ ] Sé diferenciar Azure Language, Azure Speech, Azure Translator y Content Understanding.
- [ ] Entiendo cuándo usar un servicio determinista y cuándo un modelo generativo.
- [ ] Sé qué es Voice Live y cómo se activa en un agente.
- [ ] Puedo responder las preguntas de práctica.

Volver al índice: [[00 - Índice - Texto y voz]] · [[00 - AI-901 Índice general]]
