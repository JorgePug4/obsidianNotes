---
tags: [ai-901, azure, ia, vision, generative-ai, foundry]
módulo: Computer Vision
peso_examen: Muy alto
objetivo_oficial: "2.3.1 Interpret visual input in prompts by using a deployed multimodal model · 2.3.3 Build a lightweight application that includes vision capabilities"
aliases: [Multimodal, Visión en prompts, Image input, Vision model]
---

# Modelos multimodales (visión en prompts)

## ¿Qué es?

Un **modelo multimodal** acepta en el mismo prompt **texto e imágenes** (y, según el modelo, audio o vídeo) y responde en lenguaje natural. Permite "conversar sobre una imagen": describirla, identificar objetos, leer su texto, razonar sobre ella o compararla con otra.

## ¿Para qué sirve?

Es la forma que evalúa AI-901 de **interpretar entrada visual en un prompt** (objetivo 2.3.1) y de **construir una app ligera con capacidades de visión** (2.3.3), sin usar un servicio especializado de visión.

## Cómo funciona en el portal (lab oficial)

1. Desplegar un modelo con capacidad de visión (`gpt-5-mini` en el laboratorio).
2. En el playground, fijar las **Instructions**: *"You are an AI assistant that helps people identify vintage computer hardware."*
3. Pulsar **Upload image** y adjuntar una foto: aparece una miniatura en el área del prompt.
4. Escribir el texto del prompt: *"What can you tell me about this?"* y enviar. El prompt contiene **imagen + texto**.
5. Revisar la respuesta y repetir con otras imágenes.

## Cómo se ve en código (Responses API)

```python
from openai import OpenAI

client = OpenAI(base_url=endpoint, api_key=api_key)

response = client.responses.create(
    model="gpt-5-mini",
    input=[{
        "role": "user",
        "content": [
            {"type": "input_text",  "text": "what's in this image?"},
            {"type": "input_image", "image_url": "https://an-online-image.jpg"},
        ],
    }],
)
print(response.output_text)
```

La clave que hay que reconocer: el `content` del mensaje es una **lista** con elementos `input_text` e `input_image`. La imagen puede ir como **URL** o codificada en base64.

## Qué puede hacer un modelo multimodal con una imagen

| Uso | Ejemplo de prompt |
|---|---|
| Describir | "¿Qué aparece en esta foto?" |
| Identificar | "¿Qué modelo de ordenador es este?" |
| Leer texto | "Transcribe la etiqueta" |
| Razonar | "¿Por qué esta pieza parece defectuosa?" |
| Comparar | "¿Qué diferencias hay entre estas dos imágenes?" |
| Estructurar | "Devuelve en JSON los objetos que ves" |
| Accesibilidad | "Describe la imagen para una persona ciega" |

## 📌 Modelo multimodal vs Azure Vision vs Content Understanding

| | Modelo multimodal | [[Azure AI Vision]] | [[Azure Content Understanding]] |
|---|---|---|---|
| Se pide con | Prompt en lenguaje natural | Parámetros de API | **Esquema de campos** (analizador) |
| Salida | Texto libre (o JSON si lo pides) | JSON fijo con cajas y confianza | JSON con **campos definidos** y grounding |
| Razonamiento | ✅ Alto | Bajo | Medio |
| Coordenadas | ❌ No garantizadas | ✅ Sí | ✅ En el resultado |
| Conversación | ✅ | ❌ | ❌ |
| Volumen masivo barato | ❌ | ✅ | ✅ |
| Caso típico | Chat con fotos, razonar sobre una escena | Etiquetado y detección a escala | Facturas, recibos, formularios, vídeo |

## 🧠 Memorizar

> [!important]
> - Multimodal = **imagen en el prompt**, respuesta en texto.
> - En el portal: **Upload image** en el playground.
> - En código: `content` con `input_text` + `input_image` (URL o base64).
> - No sustituye a Vision cuando hacen falta **coordenadas** ni a Content Understanding cuando hacen falta **campos**.

## Tips para AI-901

> [!tip]
> - ⭐ "El usuario envía una foto al chat y pregunta" → modelo multimodal.
> - ⭐ Fragmento de código con `input_image` → visión en el prompt.
> - 🔥 "Necesito que el asistente razone sobre lo que ve y lo explique" → multimodal.
> - ⚠️ El modelo debe **tener capacidad de visión**: no todos los modelos del catálogo la tienen.
> - ⚠️ Si el escenario pide extraer campos concretos y repetibles, no es multimodal: es Content Understanding.

## 💡 Escenario

Un técnico de campo fotografía una avería y quiere preguntar al asistente qué pieza es y qué comprobar a continuación. ¿Qué solución?

<details><summary>Respuesta</summary>

Un **modelo multimodal** desplegado en Foundry, con la imagen y la pregunta en el mismo prompt, e instrucciones que fijen el rol de asistente técnico. Podría encapsularse en un **agente** con conocimiento del manual (RAG) para citar procedimientos.
</details>

## Preguntas que podrían aparecer

**1.** En el siguiente fragmento, ¿qué está haciendo la aplicación?
```python
input=[{"role": "user", "content": [
    {"type": "input_text", "text": "¿Qué hay en esta imagen?"},
    {"type": "input_image", "image_url": "https://…/foto.jpg"}]}]
```
- A) Generando una imagen · B) Enviando una imagen y una pregunta a un modelo multimodal · C) Transcribiendo audio · D) Creando un índice vectorial

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Qué requisito debe cumplir el modelo desplegado para interpretar imágenes en el prompt?
- A) Ser un modelo de embeddings · B) Tener capacidad multimodal (entrada de imagen) · C) Estar desplegado como Batch · D) Tener temperature 0

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Computer Vision]] · [[Generación de imágenes y vídeo]]
- [[Catálogo de modelos de Foundry]] · [[Foundry SDK y cliente de chat]]
- [[Azure AI Vision]] · [[Azure Content Understanding]]
- [[Lab 05 - Visión y generación de imágenes]]

← Volver al índice: [[00 - Índice - Computer Vision]]
