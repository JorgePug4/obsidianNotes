---
tags: [ai-901, azure, ia, laboratorio, extraccion, content-understanding]
módulo: Laboratorios
duración: ~25 min
---

# Lab 06 · Content Understanding (extracción de información)

## Objetivo

Recorrer la **escalera de analizadores** de Azure Content Understanding: leer texto (**Read**), entender la estructura (**Layout**) y extraer **campos** de un tipo de documento (**Receipt**), y ver el código del SDK.

## Pasos

1. **Proyecto.** Crea o abre un proyecto de Foundry en una región compatible (el lab sugiere *West US*, *Sweden Central* o *Australia East*, entre otras).
2. **Abrir el playground.** Ve a **Build → Services** y selecciona **Content Understanding**.
3. **OCR / Read.** Elige **OCR/Read** con modalidad **Document**, selecciona una imagen de muestra y pulsa **Run analysis**. Revisa las pestañas **Markdown**, **Paragraphs** y **Result**. Sube después imágenes propias con texto legible (el lab usa fotos de placas de circuito impreso) y repite.
4. **Layout.** Cambia al analizador **Layout** y ejecuta el análisis sobre una muestra. Ahora aparecen también **Tables** y se aprecian la jerarquía y el orden de lectura.
5. **Campos.** Selecciona la categoría **Procurement** y el analizador **Receipt**. Revisa las pestañas **Fields** (vista legible) y **Result** (el JSON que recibiría una aplicación).
6. **Código.** Abre la pestaña **Code** y estudia el ejemplo de Python: `ContentUnderstandingClient`, `analyzer_id = "prebuilt-receipt"` y `client.begin_analyze(...)` con su *poller*.
7. **Limpieza.** Elimina el grupo de recursos.

## Qué está ocurriendo

Cada nivel añade capacidad sobre el anterior. **Read** solo digitaliza el texto. **Layout** interpreta cómo está organizado (párrafos, tablas, jerarquía), lo que ya permite procesar documentos con estructura conocida. **Receipt** combina OCR con un modelo generativo para decidir **qué valor corresponde a qué campo**, que es lo que necesita una automatización real. El análisis es **asíncrono** porque puede tardar: por eso el SDK devuelve un *poller*.

## Qué debo aprender para AI-901

- Objetivo **1.3.5**: técnicas para extraer información de texto, imágenes, audio y vídeo.
- Objetivos **2.4.1–2.4.4**: extraer de documentos y formularios, imágenes, audio y vídeo, y construir una app ligera de extracción.
- La escalera **Read → Layout → campos** y cuándo usar cada nivel.
- Reconocer `analyzer_id` y `begin_analyze` en el código.
- Que el resultado trae **campos, confianza y grounding**.

## Relacionado

- [[Azure Content Understanding]] · [[Extracción de información]]
- [[OCR (Reconocimiento óptico de caracteres)]] · [[Azure AI Document Intelligence]]

← Volver al índice: [[00 - Índice - Laboratorios]]
