---
tags: [ai-901, azure, ia, generative-ai, foundry, modelos]
módulo: Generative AI y agentes
peso_examen: Muy alto
objetivo_oficial: "1.2.2 Identify an appropriate AI model, based on capabilities"
aliases: [Foundry Models, Model catalog, Catálogo de modelos, Azure OpenAI]
---

# Catálogo de modelos de Foundry

## ¿Qué es?

**Foundry Models** es el **catálogo de modelos** de Microsoft Foundry (portal: **Discover → Models**). Reúne miles de modelos de **OpenAI, Microsoft, Meta, Mistral, DeepSeek, xAI, Anthropic, Black Forest Labs, Cohere, Hugging Face** y otros, que puedes explorar, comparar, desplegar y usar con una API común.

## ¿Para qué sirve?

El objetivo 1.2.2 pide **elegir el modelo adecuado según sus capacidades**: tarea, modalidades, tamaño/coste, contexto, razonamiento, idioma, licencia y región.

## Conceptos clave

- **Colecciones**: **Direct from Azure / sold directly by Azure** (modelos de OpenAI, Microsoft MAI, Phi, DeepSeek, Llama, Mistral, Grok… alojados por Azure, con SLA de Azure, sin pasar por Marketplace) y **Partners & community** (publicados por terceros).
- **Azure OpenAI**: los modelos de OpenAI (gpt-5, gpt-5-mini, gpt-5-nano, gpt-4.1, gpt-image, gpt-realtime, Sora, text-embedding) dentro de Foundry. El nombre "Azure OpenAI Service" persiste en la documentación.
- **Filtros del catálogo**: *inference task* (chat completion, text to image, video generation, embeddings, speech…), modalidad, proveedor, licencia, fine-tuning.
- **Tarjeta del modelo** (*model card*): descripción, capacidades, límites, contexto, precios, regiones, *transparency note*.
- **Benchmarks y comparación**: el catálogo permite comparar métricas de calidad, coste y latencia.
- **Model router**: opción que elige automáticamente el modelo más adecuado por petición (🟡 contexto).
- **Foundry Local**: ejecutar modelos (sobre todo SLM) en el dispositivo (🟡 contexto).

## Cómo elegir el modelo (criterios)

| Necesidad | Capacidad a buscar | Ejemplo |
|---|---|---|
| Chat general, resumen, redacción | Chat completion, buen coste | gpt-5-mini |
| Máxima capacidad / razonamiento complejo | Modelo grande o de razonamiento | gpt-5, Claude Opus, modelos "reasoning" |
| Coste/latencia mínimos, dispositivos | SLM | Phi, gpt-5-nano |
| Entender imágenes en el prompt | **Multimodal (visión)** | gpt-5-mini, gpt-4.1, Gemma, Claude |
| Responder a voz en tiempo real | Multimodal de audio / realtime | gpt-realtime, Voice Live |
| Generar imágenes | Text to image | gpt-image-1-mini, gpt-image-2, FLUX.2-pro, MAI-Image-2 |
| Generar vídeo | Video generation | Sora-2 |
| Vectorizar texto para RAG | Embeddings | text-embedding-3-large |
| Transcribir / sintetizar voz | Speech models | MAI-Transcribe-1, MAI-Voice-1, Azure Speech |
| Personalizar comportamiento con ejemplos | Modelo que admita **fine-tuning** | gpt-4.1, gpt-4o-mini, Phi |
| Datos que no pueden salir de una geografía | Disponibilidad regional / data zone | Ver [[Despliegue y configuración de modelos]] |
| Código abierto / autohospedable | Open weights | Llama, Mistral, Phi, DeepSeek |

## Ejemplo

Un equipo necesita un asistente barato que además entienda fotos de productos. Elige `gpt-5-mini`: multimodal (visión), bajo coste, disponible "direct from Azure". Para las ilustraciones de marketing añade `gpt-image-1-mini`.

## Comparaciones

| Familia | Proveedor | Fuerte en | Notas |
|---|---|---|---|
| GPT-5 / GPT-4.1 / o-series | OpenAI | Chat, razonamiento, multimodal | "Azure OpenAI"; los más usados en los labs |
| Phi | Microsoft | SLM, bajo coste, local | Se ejecuta incluso en navegador |
| MAI | Microsoft | Imagen, voz, transcripción | Modelos propios de Microsoft (2026) |
| Llama | Meta | Open weights | Fine-tuning, autohospedaje |
| Mistral, DeepSeek, Grok, Claude, Gemma | Varios | Alternativas de chat/razonamiento | Disponibilidad variable |
| FLUX | Black Forest Labs | Generación de imagen | FLUX.2-pro en los labs |

## 🧠 Memorizar

> [!important]
> - Catálogo = **Discover → Models**. Se filtra por **tarea de inferencia** y colección.
> - **Direct from Azure / sold by Azure** = alojado y facturado por Azure, sin Marketplace.
> - Elegir modelo = **tarea + modalidad + coste/latencia + contexto + región + fine-tuning**.
> - **dall-e-3 fue retirado (marzo 2026)**: para imágenes se usan **gpt-image** y otros.
> - Multimodal para **ver** imágenes ≠ modelo para **generar** imágenes.

## Tips para AI-901

> [!tip]
> - ⭐ "Necesitan analizar fotos enviadas por clientes en un chat" → modelo **multimodal** (p. ej., gpt-5-mini), no Azure Vision necesariamente.
> - ⭐ "Crear imágenes para una campaña" → modelo **text to image** (gpt-image, FLUX).
> - 🔥 "Reducir costes de un chatbot sencillo" → modelo **mini/nano** o **Phi**.
> - 🔥 "Buscar semánticamente" → modelo de **embeddings**.
> - ⚠️ Un modelo de embeddings **no chatea**; un modelo de imagen **no responde texto**.
> - ⚠️ La disponibilidad depende de **región y cuota**: el lab sugiere otro modelo gpt o crear el proyecto en otra región si no hay cuota.

## ⚠️ Errores comunes

- Elegir siempre el modelo más grande.
- Confundir "Azure OpenAI" (subconjunto del catálogo) con "Foundry Models" (todo el catálogo).

## 💡 Escenario

Una app de accesibilidad describe en voz alta lo que el usuario fotografía con el móvil. ¿Qué modelos combinarías?

<details><summary>Respuesta</summary>

Un modelo **multimodal** que acepte imagen (gpt-5-mini) para describir la foto y **texto a voz** (Azure Speech o un modelo de voz como MAI-Voice-1) para leerla. Alternativa: Azure Vision (captions) + Speech.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué modelo elegirías para generar ilustraciones a partir de descripciones de texto?
- A) text-embedding-3-large · B) gpt-image-1-mini · C) Phi · D) MAI-Transcribe-1

<details><summary>Respuesta</summary>

**B.** A vectoriza texto, C es un SLM de texto, D transcribe audio.
</details>

**2.** Un cliente exige que los modelos estén alojados y soportados directamente por Azure, sin acuerdos con terceros en Marketplace. ¿Qué colección del catálogo?
- A) Partners & community · B) Direct from Azure (sold directly by Azure) · C) Hugging Face · D) Foundry Local

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** ¿Qué característica debe tener un modelo para responder a un prompt que incluye una imagen?
- A) Fine-tuning · B) Capacidad multimodal (entrada de imagen) · C) Ventana de contexto grande · D) Temperature 0

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Large Language Models]] · [[Generative AI]]
- [[Despliegue y configuración de modelos]]
- [[Modelos multimodales (visión en prompts)]] · [[Generación de imágenes y vídeo]]
- [[Microsoft Foundry]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
