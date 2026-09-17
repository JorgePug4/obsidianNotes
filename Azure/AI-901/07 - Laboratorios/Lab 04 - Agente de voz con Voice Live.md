---
tags: [ai-901, azure, ia, laboratorio, voz, agentes]
módulo: Laboratorios
duración: ~25 min
---

# Lab 04 · Agente de voz con Voice Live

## Objetivo

Añadir **voz** a un agente de Foundry con **Azure Speech Voice Live** y mantener una conversación hablada, entendiendo el ciclo escuchar → procesar → hablar.

## Pasos

1. **Crear el agente.** En **Home → Build an agent → Start building** (o **Build → Agents**), crea `speech-agent`. Selecciona un modelo desplegado y escribe las **Instructions**:
   ```
   You are an AI agent that provides information about AI and related topics. You answer questions concisely and precisely.
   ```
   Pulsa **Save** y prueba por escrito con `What can you help me with?`.
2. **Activar la voz.** Bajo la lista de modelos, activa **Voice mode**. Se abre el panel **Configuration** (o ábrelo con el icono de engranaje): revisa la configuración de entrada y salida de voz y **previsualiza** distintas voces hasta elegir una. Cierra el panel y pulsa **Save**.
3. **Conversar.** Pulsa **Start session** y permite el acceso al micrófono. Cuando el estado sea **Listening…**, di algo como *"How does speech recognition work?"*. Observa el paso por **Processing…** y **Speaking…**. Usa **cc** para ver la transcripción y continúa la conversación con otra pregunta.
4. **Terminar.** Pulsa **X** para cerrar la sesión y revisar el transcript.
5. **Código.** Pulsa **Call agent** y revisa el ejemplo: gestiona la conexión al proyecto, el **streaming de audio** de entrada y salida y los dispositivos de audio.
6. **Limpieza.** Elimina el grupo de recursos.

## Qué está ocurriendo

Activar *Voice mode* integra **Voice Live** en el agente: en lugar de encadenar manualmente reconocimiento de voz, modelo y síntesis, una sola API bidireccional gestiona el audio en tiempo real, detecta cuándo terminas de hablar, permite **interrupciones** y suprime ruido de fondo. Es la diferencia entre "hablar y esperar" y una conversación natural.

## Qué debo aprender para AI-901

- Objetivo **1.3.3**: características de reconocimiento (STT) y síntesis (TTS) de voz.
- Objetivo **2.2.2**: responder a prompts hablados con un modelo multimodal.
- Objetivo **2.2.3**: construir una app con Azure Speech in Foundry Tools.
- Los estados de la sesión: **Listening → Processing → Speaking**.
- Que las **voces** se eligen y previsualizan en la configuración.

## Relacionado

- [[Azure AI Speech]] · [[Reconocimiento y síntesis de voz]]
- [[Agentes de IA (Foundry Agent Service)]] · [[Inclusiveness (Inclusión)]]

← Volver al índice: [[00 - Índice - Laboratorios]]
