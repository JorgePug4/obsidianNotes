---
tags: [ai-901, azure, ia, voz, speech]
módulo: Texto y voz
peso_examen: Muy alto
objetivo_oficial: "1.3.3 Identify features and capabilities of speech recognition and speech synthesis"
aliases: [Speech to text, Text to speech, STT, TTS, Reconocimiento de voz, Síntesis de voz]
---

# Reconocimiento y síntesis de voz

## ¿Qué es?

- **Reconocimiento de voz (speech recognition, speech-to-text, STT)**: convertir **audio hablado en texto**.
- **Síntesis de voz (speech synthesis, text-to-speech, TTS)**: convertir **texto en audio hablado**.

Son las dos direcciones de la carga de trabajo de voz y el objetivo 1.3.3 pide identificar sus características y capacidades.

## ¿Para qué sirve?

Transcribir reuniones, llamadas y dictados; subtitular vídeo; crear asistentes que hablan; accesibilidad (leer contenido en voz alta); centros de contacto automatizados.

## Reconocimiento de voz (STT)

| Característica | Detalle |
|---|---|
| **Entrada** | Audio: micrófono en tiempo real o archivo |
| **Salida** | Texto transcrito, con marcas de tiempo y confianza |
| **Modos** | **Tiempo real** (streaming, mientras se habla) · **Por lotes / batch transcription** (archivos, asíncrono) · **Fast transcription** (archivo, resultado rápido) |
| **Capacidades** | Múltiples idiomas, **identificación de idioma**, **diarización** (quién habla), puntuación automática, palabras clave, filtrado de improperios |
| **Personalización** | *Custom speech*: adaptar el modelo a vocabulario propio, acentos o ruido |
| **Traducción de voz** | Audio en un idioma → texto (o voz) en otro |

## Síntesis de voz (TTS)

| Característica | Detalle |
|---|---|
| **Entrada** | Texto (o SSML) |
| **Salida** | Audio |
| **Voces neuronales** | Voces con sonido natural, cientos de voces en muchos idiomas |
| **Control** | **SSML** (Speech Synthesis Markup Language) para ajustar velocidad, tono, pausas, pronunciación y énfasis; **estilos** (alegre, empático, de noticias) |
| **Personalización** | *Custom neural voice* (voz de marca, acceso limitado) y *personal voice* |
| **Avatares** | Avatares de texto a voz que hablan en vídeo |

## Cómo funciona

```
STT:  audio ─▶ modelo acústico/lingüístico ─▶ texto  ("¿Cómo funciona el reconocimiento de voz?")
TTS:  texto ─▶ modelo de voz neuronal ─▶ audio       (voz seleccionada, SSML)
```

En un asistente de voz se encadenan: **STT → modelo/agente → TTS**. Los modelos **multimodales de audio** y **Voice Live** hacen ese recorrido en un único flujo en tiempo real, con detección de turnos e interrupciones. Ver [[Azure AI Speech]].

## Ejemplo (laboratorios oficiales)

En el lab conceptual se activa *voice mode* en un chat playground: se habla al micrófono (STT), el modelo procesa el texto y la respuesta se vocaliza con la voz elegida (TTS); el botón **[CC]** muestra la transcripción. En el lab de Foundry se hace lo mismo sobre un **agente** con **Voice Live**, que además soporta conversación continua e interrupciones.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **STT** vs **TTS** | Voz → texto / texto → voz |
| **Tiempo real** vs **batch** | Mientras se habla / archivos ya grabados, asíncrono |
| **Traducción de voz** vs **traducción de texto** | La entrada es audio ([[Azure AI Speech]]) / la entrada es texto ([[Azure AI Translator]]) |
| **Diarización** vs **identificación de idioma** | Quién habla / en qué idioma habla |
| **Voz neuronal estándar** vs **custom neural voice** | Voces del catálogo / voz propia entrenada (acceso limitado) |
| **STT + LLM + TTS encadenados** vs **modelo multimodal / Voice Live** | Tres pasos con latencia / flujo único en tiempo real con interrupciones |

## 🧠 Memorizar

> [!important]
> - **STT = voz → texto**; **TTS = texto → voz**.
> - **SSML** controla pronunciación, velocidad, tono y pausas en TTS.
> - **Diarización** = separar hablantes. **Batch transcription** = archivos.
> - Un agente de voz = STT + modelo + TTS, o directamente **Voice Live**.

## Tips para AI-901

> [!tip]
> - ⭐ "Transcribir grabaciones de reuniones" → STT (batch).
> - ⭐ "Leer las respuestas en voz alta" → TTS.
> - 🔥 "Conversación natural en tiempo real con interrupciones" → **Voice Live**.
> - 🔥 "Saber qué participante dijo cada frase" → diarización.
> - ⚠️ "Traducir un audio a otro idioma" es **traducción de voz** (Speech), no Translator.
> - ⚠️ TTS **no** es lo mismo que generar audio musical o efectos.

## ⚠️ Errores comunes

- Invertir STT y TTS en preguntas rápidas.
- Elegir Translator para audio.

## 💡 Escenario

Un ayuntamiento quiere publicar las actas de los plenos con subtítulos y una versión audible para personas con baja visión, indicando quién interviene en cada momento. ¿Qué capacidades necesitas?

<details><summary>Respuesta</summary>

**Speech-to-text con diarización** (transcripción por hablante, para las actas y los subtítulos) y **text-to-speech** con voz neuronal (versión audible). Apoya además el principio de [[Inclusiveness (Inclusión)]].
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué capacidad convierte texto escrito en audio con voces naturales?
- A) Speech-to-text · B) Text-to-speech · C) Diarización · D) Traducción de texto

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Qué lenguaje de marcado permite ajustar el tono, la velocidad y las pausas de la voz sintetizada?
- A) HTML · B) SSML · C) JSON · D) YAML

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** Una empresa quiere transcribir 5 000 llamadas grabadas anoche, sin necesidad de resultados inmediatos. ¿Qué modo de reconocimiento de voz?
- A) Tiempo real (streaming) · B) Batch transcription · C) Voice Live · D) Custom neural voice

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Azure AI Speech]] · [[Azure AI Translator]]
- [[Procesamiento de Lenguaje Natural (NLP)]]
- [[Inclusiveness (Inclusión)]]
- [[Lab 04 - Agente de voz con Voice Live]]

← Volver al índice: [[00 - Índice - Texto y voz]]
