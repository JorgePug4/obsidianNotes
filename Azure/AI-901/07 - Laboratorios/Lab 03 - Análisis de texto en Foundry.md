---
tags: [ai-901, azure, ia, laboratorio, nlp]
módulo: Laboratorios
duración: ~25 min
---

# Lab 03 · Análisis de texto en Foundry

## Objetivo

Comparar los **dos enfoques** de análisis de texto: un **modelo generativo** guiado por prompts y las **herramientas especializadas de Azure Language**, que devuelven resultados estructurados y deterministas.

## Pasos

### Parte 1: modelo generativo

1. En el proyecto, despliega `gpt-5-mini` desde **Discover → Models** (o usa un despliegue existente).
2. En el playground, fija las **Instructions**: `You are an AI assistant that analyzes and summarizes text.`
3. Pega una reseña larga (el lab usa una reseña de revista del Commodore 64 de 1982) con el prompt `Summarize this review as a single short paragraph:` y revisa el resumen.

### Parte 2: Azure Language in Foundry Tools

4. Ve a **Build → Services** y observa los servicios disponibles (voz, traducción, lenguaje, content understanding).
5. **Detección de idioma.** Abre **Azure Language – Language detection**, elige un documento de muestra y pulsa **Detect**. Después edita el texto y pega una etiqueta en alemán (por ejemplo, la de un Amstrad CPC 464) para ver el idioma detectado.
6. **PII.** Cambia a **Text PII Redaction**, prueba una muestra y luego pega una factura ficticia con nombre, dirección y teléfono. Revisa las entidades detectadas, su categoría y el **texto redactado**.
7. **Código.** Abre la pestaña **Code** y estudia el ejemplo de Python con `TextAnalyticsClient` y `recognize_pii_entities`.

## Qué está ocurriendo

El LLM resuelve la tarea de resumen porque los modelos de lenguaje nacen del procesamiento de lenguaje natural: resumir, extraer entidades y clasificar por sentimiento se le dan bien. Azure Language, en cambio, aplica modelos especializados que devuelven **JSON con categorías y confianza**, siempre igual: eso es lo que necesita un pipeline automatizado o un requisito de cumplimiento.

## Qué debo aprender para AI-901

- Objetivo **1.3.2**: las técnicas de análisis de texto (frases clave, entidades, sentimiento, resumen) y las que ves aquí (idioma, PII).
- Objetivo **2.2.1**: construir una app ligera con análisis de texto.
- Cuándo elegir **servicio determinista** y cuándo **modelo generativo**.
- Que **Build → Services** es el sitio de los Foundry Tools.

## Relacionado

- [[Técnicas de análisis de texto]] · [[Azure AI Language]]
- [[Procesamiento de Lenguaje Natural (NLP)]] · [[Privacy and Security (Privacidad y seguridad)]]

← Volver al índice: [[00 - Índice - Laboratorios]]
