---
tags: [ai-901, azure, ia, voz, foundry-tools, servicio]
módulo: Texto y voz
peso_examen: Muy alto
objetivo_oficial: "2.2.2 Respond to spoken prompts by using a deployed multimodal model · 2.2.3 Build a lightweight application by using Azure Speech in Foundry Tools"
aliases: [Azure Speech, Azure Speech in Foundry Tools, Voice Live, Speech service]
---

# Azure AI Speech

## ¿Qué es?

**Azure Speech in Foundry Tools** (antes *Azure AI Speech*) es el servicio de Azure para todo lo relacionado con la voz: **reconocimiento** (voz a texto), **síntesis** (texto a voz), **traducción de voz** y, desde 2025-2026, **Voice Live**: una API única para **agentes de voz conversacionales en tiempo real**.

## ¿Para qué sirve?

Construir aplicaciones y agentes que escuchan y hablan: asistentes de voz, transcripción de reuniones y llamadas, subtitulado, locución automática, accesibilidad y centros de contacto.

## Capacidades

| Capacidad | Qué hace | Uso típico |
|---|---|---|
| **Speech to text** | Transcribe audio en tiempo real o por lotes (batch, fast transcription) | Actas, subtítulos, comandos de voz |
| **Text to speech** | Genera audio con **voces neuronales** y control **SSML**; estilos y avatares | Locuciones, lectura en voz alta |
| **Speech translation** | Traduce voz a texto o a voz en otro idioma | Reuniones multilingües |
| **Voice Live** | API **bidireccional en tiempo real** (WebSocket) que integra STT, TTS, detección de turnos, manejo de **interrupciones**, supresión de ruido, avatares e integración con **Foundry Agent Service** | Agentes de voz conversacionales |
| **Custom speech / custom neural voice / personal voice** | Personalizar reconocimiento y voces | Vocabulario propio, voz de marca (acceso limitado) |
| **Speaker recognition** | Identificar o verificar al hablante | Autenticación por voz (acceso limitado) |

## Voice Live: lo que evalúa el examen

Los objetivos 2.2.2 y 2.2.3 se practican en el laboratorio oficial activando **Voice mode** en un agente de Foundry:

1. Crear un agente (`speech-agent`) con un modelo y unas instrucciones.
2. Activar **Voice mode** bajo la lista de modelos; se abre el panel **Configuration** con la **voz** de entrada y salida, que puedes previsualizar.
3. **Save** y pulsar **Start session**: el agente pasa por los estados **Listening… → Processing… → Speaking…**.
4. El botón **cc** muestra la transcripción; **X** termina la sesión y muestra el transcript.
5. **Call agent** genera el código cliente, que gestiona la conexión al proyecto, el **streaming de audio** de entrada y salida y los dispositivos (micrófono y altavoz).

Frente al encadenado clásico (STT → modelo → TTS), Voice Live ofrece **conversación multi-turno en tiempo real, con interrupciones y supresión de ruido de fondo**, y admite modelos como GPT-Realtime además de voces de Azure y OpenAI.

## Responder a prompts hablados con un modelo multimodal (2.2.2)

Un **modelo multimodal de audio** acepta directamente voz como entrada y puede responder con voz, sin dos servicios separados. Es la alternativa "modelo" frente a la alternativa "servicio" (Azure Speech). En el examen, ambas rutas son válidas según el escenario:

| Ruta | Cuándo |
|---|---|
| **Modelo multimodal / Voice Live** | Conversación natural en tiempo real, interrupciones, agentes de voz |
| **Azure Speech STT/TTS por separado** | Transcripción masiva, subtítulos, locución de textos, control fino con SSML |

## Ejemplo

Un concesionario despliega un agente telefónico: Voice Live escucha al cliente, el agente consulta el inventario con una herramienta y responde hablando con una voz neuronal; si el cliente interrumpe, el agente se detiene y escucha.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **Azure Speech** vs **Azure Translator** | Audio ↔ texto (y traducción **de voz**) / traducción **de texto y documentos** |
| **Voice Live** vs **STT + TTS encadenados** | Flujo único en tiempo real con interrupciones / tres pasos separados |
| **Batch transcription** vs **tiempo real** | Archivos, asíncrono / streaming |
| **Custom neural voice** vs **voz neuronal estándar** | Voz propia entrenada (acceso limitado) / catálogo de voces |
| **Azure Speech** vs **Content Understanding (audio)** | Transcribir y sintetizar / **extraer campos y datos** de audio y vídeo |

## 🧠 Memorizar

> [!important]
> - Nombre actual: **Azure Speech in Foundry Tools**.
> - **Voice Live** = agentes de voz en tiempo real, interrupciones, avatares, integración con agentes de Foundry.
> - En el portal: interruptor **Voice mode** del agente + panel **Configuration** para elegir voz.
> - Estados de la sesión: **Listening → Processing → Speaking**.
> - **SSML** para controlar la voz; **diarización** para separar hablantes.

## Tips para AI-901

> [!tip]
> - ⭐ "Agente que conversa por voz con el usuario" → **Voice Live** (Voice mode en el agente).
> - ⭐ "Transcribir archivos de audio" → speech to text (batch).
> - 🔥 "El cliente puede interrumpir al asistente mientras habla" → Voice Live, no STT+TTS encadenados.
> - 🔥 "Extraer el motivo de la llamada y el importe reclamado de una grabación" → eso ya es **Content Understanding** (extracción), no solo Speech.
> - ⚠️ Un recurso de **Azure Speech** debe estar conectado al proyecto para los laboratorios de voz.

## ⚠️ Errores comunes

- Usar Translator para audio.
- Pensar que Voice Live es solo TTS: incluye entrada, salida y gestión de la conversación.

## 💡 Escenario

Una aerolínea quiere sustituir su IVR por un asistente telefónico que entienda lenguaje natural, consulte el estado del vuelo y responda hablando, permitiendo que el pasajero interrumpa. ¿Qué usas?

<details><summary>Respuesta</summary>

Un **agente de Foundry** con una herramienta que consulte el estado del vuelo, y **Azure Speech Voice Live** activado (Voice mode) para la conversación en tiempo real con manejo de interrupciones. La integración telefónica se haría con Azure Communication Services.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué capacidad de Azure Speech permite crear agentes de voz con conversación en tiempo real e interrupciones?
- A) Batch transcription · B) Voice Live · C) Custom speech · D) SSML

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** En el playground de un agente de Foundry, ¿qué opción habilita la interacción por voz?
- A) Tools → Web search · B) Voice mode · C) Knowledge · D) Guardrails

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** Verdadero o falso: para traducir un audio del español al inglés necesitas Azure Translator.

<details><summary>Respuesta</summary>

**Falso.** La **traducción de voz** es una capacidad de Azure Speech (Translator traduce texto y documentos).
</details>

## Relacionado

- [[Reconocimiento y síntesis de voz]] · [[Azure AI Translator]]
- [[Agentes de IA (Foundry Agent Service)]] · [[Catálogo de modelos de Foundry]]
- [[Azure Content Understanding]] (extraer datos de audio y vídeo)
- [[Foundry Tools (servicios de IA de Azure)]]
- [[Lab 04 - Agente de voz con Voice Live]]

← Volver al índice: [[00 - Índice - Texto y voz]]
