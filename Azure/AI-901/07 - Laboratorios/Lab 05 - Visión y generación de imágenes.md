---
tags: [ai-901, azure, ia, laboratorio, vision, generative-ai]
módulo: Laboratorios
duración: ~30 min
---

# Lab 05 · Visión y generación de imágenes

## Objetivo

Practicar las **dos direcciones** de la visión generativa: enviar una **imagen dentro del prompt** a un modelo multimodal y **generar** imágenes (y vídeo) a partir de texto.

## Pasos

### Parte 1: interpretar imágenes (entrada visual)

1. Descarga el archivo de imágenes de ejemplo del laboratorio y descomprímelo.
2. Despliega `gpt-5-mini` desde **Discover → Models** y abre su playground.
3. Fija las **Instructions**: `You are an AI assistant that helps people identify vintage computer hardware.`
4. Pulsa **Upload image**, elige una imagen y escribe `What can you tell me about this?`. El prompt contiene **imagen + texto**.
5. Repite con otras imágenes (`What is this?`, `Tell me about this.`).
6. **Código.** En la pestaña **Call model**, selecciona Python y observa cómo el parámetro `input` se amplía con una lista de contenidos `input_text` e `input_image`.

### Parte 2: generar imágenes (salida visual)

7. Vuelve a **Models → Deploy a base model**. Filtra **Collections: Direct from Azure** e **Inference tasks: Text to image**.
8. Despliega un modelo disponible, por ejemplo `gpt-image-1-mini` o `FLUX.2-pro`. Se abre el **image playground**.
9. Escribe un prompt como `A vintage PC with a CRT monitor.` y revisa la imagen generada.
10. **Código.** En **View code**, observa `client.images.generate(model=…, prompt=…, n=1, size="1024x1024")` y que la imagen llega en **base64**.

### Parte 3 (opcional): generar vídeo

11. Filtra el catálogo por **Video generation** y despliega `Sora-2` si está disponible en tu suscripción. En el **video playground**, prueba un prompt como `A retro computer game.` El código de ejemplo usa la **API REST** con trabajos de generación.

12. **Limpieza.** Elimina el grupo de recursos.

## Qué está ocurriendo

Un modelo **multimodal** convierte la imagen en tokens igual que el texto y razona sobre ambos a la vez: por eso puede describir, identificar y responder preguntas sobre lo que ve. Los modelos **text to image** hacen lo contrario: a partir de la descripción generan píxeles nuevos. Son modelos distintos, con tareas de inferencia distintas en el catálogo.

## Qué debo aprender para AI-901

- Objetivo **1.3.4**: capacidades de los modelos de visión y de generación de imágenes.
- Objetivo **2.3.1**: interpretar entrada visual en prompts.
- Objetivo **2.3.2**: crear salidas visuales con modelos generativos.
- Objetivo **2.3.3**: app ligera con capacidades de visión.
- Reconocer `input_image` (entrada) frente a `images.generate` (salida).
- Que el catálogo se filtra por **inference task** y que la disponibilidad depende de región y cuota.

## Relacionado

- [[Modelos multimodales (visión en prompts)]] · [[Generación de imágenes y vídeo]]
- [[Computer Vision]] · [[Catálogo de modelos de Foundry]]

← Volver al índice: [[00 - Índice - Laboratorios]]
