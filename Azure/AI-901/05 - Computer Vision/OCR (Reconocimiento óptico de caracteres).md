---
tags: [ai-901, azure, ia, vision, ocr]
módulo: Computer Vision
peso_examen: Alto
aliases: [OCR, Read, Optical Character Recognition, Reconocimiento óptico de caracteres]
---

# OCR (Reconocimiento óptico de caracteres)

## ¿Qué es?

El **OCR (Optical Character Recognition)** es la técnica que **extrae texto de imágenes y documentos escaneados** y lo convierte en texto digital manipulable. En Azure la capacidad se llama **OCR / Read**.

## ¿Para qué sirve?

Digitalizar documentos en papel, leer carteles, matrículas, etiquetas y números de serie, hacer buscable un archivo escaneado y alimentar procesos posteriores (traducción, análisis, extracción de campos).

## Qué devuelve

| Elemento | Detalle |
|---|---|
| **Texto** | Líneas y palabras reconocidas, impresas o manuscritas |
| **Posición** | Coordenadas (bounding box) de cada línea y palabra |
| **Confianza** | Puntuación por palabra o línea |
| **Idioma** | Detección del idioma del texto |
| **Estructura** (con *Layout*) | Párrafos, **tablas**, títulos, orden de lectura, salida **Markdown** |

## Dónde está en Azure

| Opción | Qué ofrece |
|---|---|
| **Azure Content Understanding – analizador OCR/Read** | Extrae el texto de documentos e imágenes. Es lo que muestran los labs oficiales |
| **Azure Content Understanding – analizador Layout** | Añade estructura: párrafos, tablas, jerarquía, Markdown |
| **Azure Vision – Read/OCR** | OCR dentro del análisis de imagen general |
| **Modelo multimodal** | Puede leer texto de una imagen dentro de una conversación, sin coordenadas garantizadas |

## Cómo funciona

```
Imagen/PDF ─▶ OCR ─▶ texto + coordenadas + confianza
                 └─▶ (Layout) párrafos, tablas, Markdown
                        └─▶ (analizador de campos) total, fecha, proveedor…
```

El laboratorio oficial recorre exactamente esa escalera: **Read** (texto en bruto) → **Layout** (estructura) → **Receipt** (campos). Ver [[Lab 06 - Content Understanding (extracción de información)]].

## Ejemplo (laboratorio oficial)

Se suben fotos de **placas de circuito impreso** y el analizador **OCR/Read** extrae las referencias serigrafiadas ("ASSY 250425"), que luego sirven para identificar el equipo.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **OCR/Read** vs **Layout** | Texto plano / texto **con estructura**, tablas y Markdown |
| **OCR** vs **extracción de campos** | Todo el texto / valores asignados a **campos concretos** |
| **OCR** vs **detección de objetos** | Leer texto / localizar objetos |
| **OCR** vs **transcripción** | Texto en **imágenes** / texto a partir de **audio** ([[Azure AI Speech]]) |

## 🧠 Memorizar

> [!important]
> - OCR = **texto de imágenes**, impreso o manuscrito, con **posición y confianza**.
> - **Read** = texto; **Layout** = estructura y tablas; **analizador de campos** = datos concretos.
> - OCR es el **primer paso** de la mayoría de soluciones de extracción documental.

## Tips para AI-901

> [!tip]
> - ⭐ "Digitalizar formularios en papel para poder buscarlos" → OCR.
> - ⭐ "Necesito las tablas de un PDF respetando filas y columnas" → **Layout**.
> - 🔥 "Necesito el importe total y la fecha del ticket" → analizador de campos de [[Azure Content Understanding]], no OCR a secas.
> - ⚠️ OCR no interpreta el significado: solo lee.

## 💡 Escenario

Un museo quiere que las fichas mecanografiadas de su archivo sean buscables por texto, sin extraer campos concretos. ¿Qué capacidad?

<details><summary>Respuesta</summary>

**OCR / Read**: basta con convertir el texto de las imágenes en texto digital indexable. Si más adelante quisieran campos como "autor" o "año", pasarían a un analizador de campos de Content Understanding.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué analizador usarías para obtener el texto de un documento **y** su estructura en tablas y párrafos?
- A) OCR/Read · B) Layout · C) Receipt · D) Caption

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** Verdadero o falso: el OCR asigna automáticamente los valores leídos a campos como "proveedor" o "total".

<details><summary>Respuesta</summary>

**Falso.** Eso lo hace un analizador de campos (por ejemplo `prebuilt-receipt` o `prebuilt-invoice`); el OCR solo extrae el texto y su posición.
</details>

## Relacionado

- [[Computer Vision]] · [[Azure AI Vision]]
- [[Azure Content Understanding]] · [[Azure AI Document Intelligence]] · [[Extracción de información]]
- [[Lab 06 - Content Understanding (extracción de información)]]

← Volver al índice: [[00 - Índice - Computer Vision]]
