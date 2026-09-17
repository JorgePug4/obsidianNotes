---
tags: [ai-901, azure, ia, vision, generative-ai, foundry]
módulo: Computer Vision
peso_examen: Alto
objetivo_oficial: "2.3.2 Create new visual outputs by using generative models"
aliases: [Text to image, Generación de imágenes, Image generation, Sora, gpt-image, DALL-E]
---

# Generación de imágenes y vídeo

## ¿Qué es?

Los **modelos generativos de imagen y vídeo** crean **contenido visual nuevo** a partir de una descripción en texto (*text to image*, *text to video*) o a partir de otra imagen (*image editing*, *inpainting*, variaciones).

## ¿Para qué sirve?

Ilustraciones y creatividades de marketing, prototipos de diseño, imágenes para catálogos y sitios web, vídeos cortos promocionales, edición y retoque automatizados.

## Modelos en Foundry (2026)

| Tarea de inferencia | Modelos habituales | Notas |
|---|---|---|
| **Text to image** | `gpt-image-1`, `gpt-image-1-mini`, `gpt-image-2`, `FLUX.2-pro`, `MAI-Image-2` | `dall-e-3` fue **retirado en marzo de 2026**; se usa la familia **gpt-image** |
| **Edición de imagen** | `gpt-image-2` y equivalentes | Inpainting, variaciones, preservación de rostros |
| **Video generation** | `Sora-2` | Puede requerir solicitar acceso |

## Cómo funciona en el portal (lab oficial)

1. En **Models → Deploy a base model**, filtrar **Collections: Direct from Azure** e **Inference tasks: Text to image**.
2. Desplegar un modelo disponible (por ejemplo `gpt-image-1-mini` o `FLUX.2-pro`).
3. Se abre el **image playground**: escribir el prompt (*"A vintage PC with a CRT monitor"*) y revisar la imagen generada.
4. **View code** para obtener el cliente.

Para vídeo, el flujo es el mismo filtrando por **Video generation** y desplegando `Sora-2`; se abre el **video playground** y el código de ejemplo usa la **API REST**.

## Código de ejemplo (imagen)

```python
import base64
from openai import OpenAI

client = OpenAI(base_url=endpoint, api_key=api_key)

img = client.images.generate(
    model=deployment_name,      # despliegue text-to-image
    prompt="A cute baby polar bear",
    n=1,
    size="1024x1024",
)
image_bytes = base64.b64decode(img.data[0].b64_json)
open("output.png", "wb").write(image_bytes)
```

Clave para reconocerlo: `client.images.generate(...)` con `prompt`, `n` (número de imágenes) y `size`, y la respuesta en **base64**.

## Parámetros típicos

| Parámetro | Qué controla |
|---|---|
| `prompt` | La descripción de la imagen |
| `size` | Resolución (los modelos recientes llegan a 4K) |
| `n` | Número de imágenes por petición |
| `quality` / `style` | Calidad y estilo, según modelo |

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **Text to image** vs **modelo multimodal** | **Genera** imágenes / **entiende** imágenes |
| **Generación** vs **edición** | Crear desde cero / modificar una imagen existente |
| **Imagen** vs **vídeo** | `images.generate` / API de video generations (asíncrona, por trabajos) |
| **Generar una imagen** vs **buscar una imagen** | Modelo generativo / búsqueda por embeddings multimodales |

## 🧠 Memorizar

> [!important]
> - Se filtra el catálogo por **inference task: text to image / video generation**.
> - **dall-e-3 retirado (marzo 2026)** → familia **gpt-image**.
> - La imagen suele devolverse en **base64**; el vídeo, mediante un **trabajo asíncrono**.
> - Disponibilidad sujeta a **región, cuota y, en algunos casos, solicitud de acceso**.

## Tips para AI-901

> [!tip]
> - ⭐ "Crear imágenes para la web a partir de descripciones" → modelo **text to image**.
> - ⭐ Código con `images.generate(prompt=…)` → generación de imágenes.
> - 🔥 "Modificar una foto existente manteniendo el rostro" → edición/inpainting (gpt-image-2).
> - ⚠️ Un modelo de chat multimodal **no genera** imágenes salvo que use la herramienta de generación de imágenes.
> - ⚠️ Responsible AI: la generación de imágenes tiene riesgos de contenido dañino y derechos de autor; se controla con **filtros de contenido** y **protected material detection**.

## 💡 Escenario

Una agencia necesita 200 variaciones de una ilustración de producto para pruebas A/B, con la misma estética. ¿Qué modelo y qué parámetros?

<details><summary>Respuesta</summary>

Un modelo **text to image** (por ejemplo gpt-image-1-mini por coste), con un prompt detallado que fije la estética, generando varias imágenes por petición con `n` y un `size` adecuado. Para mantener coherencia con una imagen base, usar **edición/variaciones** de un modelo como gpt-image-2.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué filtro usarías en el catálogo de modelos para encontrar modelos que crean imágenes?
- A) Chat completion · B) Text to image · C) Embeddings · D) Speech

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Qué modelo de generación de imágenes fue retirado en 2026 y sustituido por la familia gpt-image?
- A) FLUX · B) dall-e-3 · C) Sora · D) Phi

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** Verdadero o falso: para generar vídeo a partir de texto en Foundry se usa un modelo de chat multimodal.

<details><summary>Respuesta</summary>

**Falso.** Se despliega un modelo de **video generation** (por ejemplo Sora-2) y se usa su playground o su API.
</details>

## Relacionado

- [[Computer Vision]] · [[Modelos multimodales (visión en prompts)]]
- [[Catálogo de modelos de Foundry]] · [[Despliegue y configuración de modelos]]
- [[Azure AI Content Safety y guardrails]]
- [[Lab 05 - Visión y generación de imágenes]]

← Volver al índice: [[00 - Índice - Computer Vision]]
