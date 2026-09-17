---
tags: [ai-901, azure, ia, vision, foundry-tools, servicio]
módulo: Computer Vision
peso_examen: Alto
aliases: [Azure Vision, Azure Vision in Foundry Tools, Computer Vision service, Image Analysis]
---

# Azure AI Vision

## ¿Qué es?

**Azure Vision in Foundry Tools** (antes *Azure AI Vision* y *Computer Vision*) es el servicio preconstruido de Azure para **analizar imágenes y vídeo** con resultados estructurados: descripciones, etiquetas, objetos, personas, texto (OCR) y recortes inteligentes.

## ¿Para qué sirve?

Obtener de una imagen datos **predecibles y con coordenadas** sin escribir prompts: útil en catálogos, moderación, accesibilidad, inventario y digitalización a gran escala.

## Capacidades (Image Analysis)

| Capacidad | Qué devuelve |
|---|---|
| **Caption** | Una frase que describe la imagen completa |
| **Dense captions** | Descripciones de varias **regiones** de la imagen |
| **Tags** | Etiquetas de objetos, escenas y conceptos con confianza |
| **Object detection** | Objetos con **bounding boxes** |
| **People detection** | Personas con bounding boxes |
| **OCR / Read** | Texto impreso y manuscrito con su posición |
| **Smart crop** | Recorte inteligente centrado en lo relevante |
| **Multimodal embeddings** | Vectores de imagen/texto para búsqueda por similitud |

Servicios relacionados de la familia: **Face** ([[Reconocimiento facial (Face)]]) y **Custom Vision** (clasificación y detección entrenadas con tus propias imágenes).

> [!info] Nota de versiones
> La API *Image Analysis 4.0* está marcada como en camino de retirada (anunciada para septiembre de 2028) y varias capacidades de personalización se han reorganizado (Custom Vision como servicio aparte). Para AI-901 basta con conocer **qué capacidades ofrece el servicio** y **cuándo elegirlo frente a un modelo multimodal**.

## Cómo funciona

```
Imagen (URL o binario) ─▶ Azure Vision ─▶ JSON: { caption, tags[], objects[{box, label, confidence}], text[] }
```

## Ejemplo

Un portal inmobiliario procesa 50 000 fotos al día: genera **captions** para accesibilidad, **tags** para filtrar por "piscina" o "chimenea", **smart crop** para las miniaturas y **OCR** para leer carteles de "se vende".

## 📌 Azure Vision vs modelo multimodal

| | Azure Vision | Modelo multimodal (gpt-5-mini…) |
|---|---|---|
| Interfaz | API con parámetros | **Prompt** en lenguaje natural |
| Salida | JSON fijo: etiquetas, cajas, confianza | Texto libre (o JSON si lo pides) |
| Coordenadas | ✅ Sí | Aproximadas o no disponibles |
| Razonamiento sobre la escena | Limitado | ✅ Alto ("¿por qué esta pieza está defectuosa?") |
| Coste/latencia | Bajos | Mayores |
| Volumen masivo | ✅ | Más caro |
| Conversación | No | ✅ |

## 🧠 Memorizar

> [!important]
> - Nombre actual: **Azure Vision in Foundry Tools**.
> - Capacidades: **caption, dense captions, tags, objetos, personas, OCR/Read, smart crop, embeddings multimodales**.
> - Devuelve **bounding boxes** y **confianza**; un modelo multimodal, no.
> - **Custom Vision** = clasificación/detección con tus imágenes.

## Tips para AI-901

> [!tip]
> - ⭐ "Etiquetas y coordenadas para miles de imágenes" → Azure Vision.
> - ⭐ "Conversar sobre una imagen o razonar sobre ella" → modelo multimodal.
> - 🔥 "Entrenar un detector con nuestras propias piezas" → Custom Vision.
> - ⚠️ Si piden **campos de un documento o ticket**, es [[Azure Content Understanding]], no Vision.

## 💡 Escenario

Una aseguradora recibe fotos de daños y necesita, para cada una, una descripción textual, etiquetas para el buscador y las coordenadas de los elementos dañados, procesando decenas de miles al día. ¿Qué eliges?

<details><summary>Respuesta</summary>

**Azure Vision** (caption + tags + object detection): salida estructurada con coordenadas y confianza, coste bajo y apta para volumen. Un modelo multimodal serviría para razonar sobre casos concretos, no para el procesamiento masivo estructurado.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué capacidad de Azure Vision genera descripciones de distintas regiones de una misma imagen?
- A) Tags · B) Dense captions · C) Smart crop · D) OCR

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** Una empresa quiere entrenar un modelo con sus propias fotos para reconocer tres tipos de pieza. ¿Qué servicio?
- A) Azure Vision – Image Analysis · B) Custom Vision · C) Content Understanding · D) Face

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Computer Vision]] · [[OCR (Reconocimiento óptico de caracteres)]] · [[Reconocimiento facial (Face)]]
- [[Modelos multimodales (visión en prompts)]] · [[Azure Content Understanding]]
- [[Foundry Tools (servicios de IA de Azure)]]

← Volver al índice: [[00 - Índice - Computer Vision]]
