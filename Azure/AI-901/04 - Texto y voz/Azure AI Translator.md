---
tags: [ai-901, azure, ia, nlp, traduccion, foundry-tools, servicio]
módulo: Texto y voz
peso_examen: Alto
aliases: [Azure Translator, Translator, Text Translator, Azure Translator in Foundry Tools]
---

# Azure AI Translator

## ¿Qué es?

**Azure Translator in Foundry Tools** (antes *Azure AI Translator*) es el servicio de **traducción automática neuronal** de Azure: traduce **texto** y **documentos** entre más de cien idiomas conservando el formato.

## ¿Para qué sirve?

Localizar contenido, atender a clientes en su idioma, traducir documentación y normalizar textos multilingües antes de analizarlos.

## Capacidades

| Capacidad | Detalle |
|---|---|
| **Traducción de texto** | Uno o varios idiomas de destino en la misma llamada |
| **Detección del idioma de origen** | Automática si no se especifica |
| **Traducción de documentos** | Archivos completos (Word, PDF, PowerPoint…) **conservando el formato**, de forma asíncrona |
| **Transliteración** | Convertir entre alfabetos (por ejemplo, japonés a caracteres latinos) |
| **Diccionario y ejemplos** | Traducciones alternativas y ejemplos de uso |
| **Custom Translator** | Modelos de traducción personalizados con terminología propia |
| **Filtro de improperios** | Marcar o eliminar lenguaje ofensivo |

## Cómo funciona

```
Texto ("Hola") + idioma destino (en, fr) ─▶ Translator ─▶ {"en": "Hello", "fr": "Bonjour"}
```

En el portal de Foundry se accede desde **Build → Services** (el laboratorio de análisis de texto lo sugiere para traducir la etiqueta en alemán que se detecta con Azure Language).

## Ejemplo

Un fabricante publica sus manuales en 12 idiomas: usa **traducción de documentos** para convertir los PDF conservando maquetación, y **Custom Translator** para que los términos técnicos de su catálogo se traduzcan siempre igual.

## 📌 Diferencias clave

| Necesidad | Servicio |
|---|---|
| Traducir **texto** | Azure Translator |
| Traducir **documentos** conservando formato | Azure Translator (document translation) |
| Traducir **voz** | [[Azure AI Speech]] (speech translation) |
| Saber en qué idioma está un texto | [[Azure AI Language]] (language detection) o el propio Translator |
| Traducir con un estilo libre o resumir a la vez | Modelo generativo |

## 🧠 Memorizar

> [!important]
> - Translator = **texto y documentos**. Voz = **Speech**.
> - Traduce a **varios idiomas destino** en una sola petición y **detecta** el idioma de origen.
> - **Document translation** conserva el formato.
> - **Custom Translator** para terminología propia.

## Tips para AI-901

> [!tip]
> - ⭐ "Traducir el contenido de un sitio web o un catálogo" → Translator.
> - 🔥 "Traducir una conversación hablada" → Speech (speech translation).
> - ⚠️ Un LLM también traduce, pero si la pregunta exige un servicio dedicado, gran volumen o formato conservado, la respuesta es Translator.

## 💡 Escenario

Una ONG necesita publicar sus informes en PDF en cinco idiomas manteniendo tablas y maquetación. ¿Qué capacidad?

<details><summary>Respuesta</summary>

**Document translation** de Azure Translator: traduce archivos completos conservando el formato original.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué servicio traduce documentos Word y PDF conservando su formato?
- A) Azure Language · B) Azure Translator · C) Azure Speech · D) Content Understanding

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** Una empresa quiere traducir en tiempo real lo que dicen los participantes de una videollamada. ¿Qué servicio es el principal?
- A) Azure Translator · B) Azure Speech (speech translation) · C) Azure Language · D) Azure Vision

<details><summary>Respuesta</summary>

**B.** La entrada es audio.
</details>

## Relacionado

- [[Azure AI Language]] · [[Azure AI Speech]]
- [[Foundry Tools (servicios de IA de Azure)]]
- [[Inclusiveness (Inclusión)]]

← Volver al índice: [[00 - Índice - Texto y voz]]
