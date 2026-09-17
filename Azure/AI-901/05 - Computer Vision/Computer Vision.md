---
tags: [ai-901, azure, ia, vision]
módulo: Computer Vision
peso_examen: Muy alto
objetivo_oficial: "1.3.4 Identify features and capabilities of computer vision and image-generation models"
aliases: [Visión artificial, Visión por computador, Image analysis, Object detection, Image classification]
---

# Computer Vision

## ¿Qué es?

La **visión artificial (computer vision)** es el área de la IA que permite a las máquinas **interpretar imágenes y vídeo**: describir lo que aparece, clasificarlo, localizar objetos, leer texto o analizar caras. Con los modelos generativos aparece la dirección contraria: **crear** imágenes y vídeo.

## ¿Para qué sirve?

Control de calidad industrial, inventario en tienda, seguridad, accesibilidad (describir imágenes), moderación de contenido, digitalización de documentos, asistentes que "ven" fotos que envía el usuario.

## Tareas clásicas de visión (hay que reconocerlas)

| Tarea | Qué hace | Salida | Ejemplo |
|---|---|---|---|
| **Image classification** | Asigna **una etiqueta** a la imagen completa | Clase + confianza | "Esta foto es un teclado" |
| **Object detection** | Localiza **varios objetos** y los clasifica | Clases + **bounding boxes** + confianza | Contar productos en una estantería |
| **Image analysis / captioning** | Genera una **descripción** de la imagen (y *dense captions* por regiones) | Frase descriptiva | "Un ordenador vintage sobre una mesa" |
| **Tagging** | Lista de **etiquetas** de objetos, escenas y conceptos | Etiquetas + confianza | interior, ordenador, retro |
| **OCR / Read** | Extrae **texto** impreso o manuscrito | Texto + coordenadas | Leer la matrícula o un cartel. Ver [[OCR (Reconocimiento óptico de caracteres)]] |
| **Face detection / analysis** | Detecta caras y atributos; opcionalmente identifica o verifica | Cajas, puntos, atributos | Desbloqueo, conteo de asistentes. Ver [[Reconocimiento facial (Face)]] |
| **Segmentación / smart crop** | Separa el objeto del fondo, recorta de forma inteligente | Máscara, recorte | Miniaturas |
| **Análisis de vídeo** | Detección de movimiento, seguimiento, resumen de escenas | Eventos, transcripción | Videovigilancia |
| **Embeddings multimodales** | Vectoriza imágenes para búsqueda por similitud | Vector | "Busca fotos parecidas a esta" |

## Generación visual

| Tarea | Qué hace | Modelos |
|---|---|---|
| **Text to image** | Crea una imagen desde una descripción | gpt-image-1/2, FLUX.2-pro, MAI-Image-2 |
| **Image editing / inpainting** | Modifica una imagen existente | gpt-image-2 y equivalentes |
| **Video generation** | Genera vídeo desde texto | Sora-2 |

Ver [[Generación de imágenes y vídeo]].

## Dos enfoques en Foundry

| Enfoque | Cómo | Cuándo |
|---|---|---|
| **Modelo multimodal** (gpt-5-mini y similares) | Se envía la imagen dentro del **prompt** y se pregunta en lenguaje natural | Interpretación flexible, conversación, razonamiento sobre la imagen |
| **Azure Vision in Foundry Tools** | API especializada: captions, tags, objetos, OCR, caras | Resultados estructurados, etiquetas y coordenadas, volumen |
| **Content Understanding** | Extrae **campos** definidos por ti de imágenes | Cuando quieres datos, no descripciones |

## Ejemplo (laboratorio oficial)

Se despliega `gpt-5-mini`, se fija la instrucción *"You are an AI assistant that helps people identify vintage computer hardware"*, se sube una foto con el botón **Upload image** y se pregunta *"What can you tell me about this?"*. El modelo describe e identifica el hardware. Luego se despliega un modelo **text to image** y se genera *"A vintage PC with a CRT monitor"*.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **Clasificación** vs **detección de objetos** | Una etiqueta para toda la imagen / varias etiquetas **con posición** |
| **Caption** vs **tags** | Frase descriptiva / lista de palabras |
| **OCR** vs **extracción de campos** | Todo el texto / valores asignados a campos ([[Azure Content Understanding]]) |
| **Detección de caras** vs **reconocimiento** | Hay una cara / **quién** es |
| **Modelo multimodal** vs **modelo de imagen** | **Entiende** imágenes / **genera** imágenes |
| **Visión** vs **extracción de información** | Interpretar la escena / obtener datos estructurados |

## 🧠 Memorizar

> [!important]
> - **Clasificación = 1 etiqueta para la imagen. Detección = objetos + bounding boxes.**
> - **Caption** describe; **tags** etiquetan; **OCR** lee texto.
> - Multimodal = imagen **de entrada**; text-to-image = imagen **de salida**.
> - "Contar objetos" o "localizar" siempre implica **detección de objetos**.

## Tips para AI-901

> [!tip]
> - ⭐ "¿Cuántos coches hay y dónde están?" → detección de objetos.
> - ⭐ "¿Qué es esta foto?" → clasificación o caption.
> - ⭐ "Describir imágenes para personas ciegas" → captioning (accesibilidad, [[Inclusiveness (Inclusión)]]).
> - 🔥 "El usuario sube una foto al chat y pregunta por ella" → **modelo multimodal**.
> - 🔥 "Crear una ilustración para la campaña" → **modelo text to image**.
> - ⚠️ Leer el texto de un ticket es OCR; obtener *total* y *fecha* es Content Understanding.

## ⚠️ Errores comunes

- Confundir clasificación con detección (la clave son las **coordenadas**).
- Elegir Azure Vision cuando el escenario describe un **chat con imágenes** (ahí es un modelo multimodal).

## 💡 Escenario

Un supermercado quiere comprobar automáticamente que cada estantería tiene el número correcto de unidades de cada producto a partir de fotos. ¿Qué tarea de visión?

<details><summary>Respuesta</summary>

**Detección de objetos**: hay que localizar y contar varias instancias por imagen, no solo etiquetar la foto.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué tarea de visión devuelve las coordenadas de cada elemento localizado en la imagen?
- A) Clasificación de imágenes · B) Detección de objetos · C) Captioning · D) Tagging

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** Una app de accesibilidad necesita generar una frase que describa cada fotografía. ¿Qué capacidad?
- A) OCR · B) Image captioning · C) Detección de objetos · D) Generación de imágenes

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** ¿Qué tipo de modelo necesitas para que un chatbot analice una foto enviada por el usuario y responda en texto?
- A) Text to image · B) Multimodal con entrada de imagen · C) Embeddings · D) Speech

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Azure AI Vision]] · [[Reconocimiento facial (Face)]] · [[OCR (Reconocimiento óptico de caracteres)]]
- [[Modelos multimodales (visión en prompts)]] · [[Generación de imágenes y vídeo]]
- [[Extracción de información]] · [[Cargas de trabajo de IA]]
- [[Lab 05 - Visión y generación de imágenes]]

← Volver al índice: [[00 - Índice - Computer Vision]]
