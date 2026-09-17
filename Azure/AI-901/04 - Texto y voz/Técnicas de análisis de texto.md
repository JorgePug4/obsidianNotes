---
tags: [ai-901, azure, ia, nlp, analisis-de-texto]
módulo: Texto y voz
peso_examen: Muy alto
objetivo_oficial: "1.3.2 Describe common text analysis techniques, including keyword extraction, entity detection, sentiment analysis, and summarization"
aliases: [Análisis de texto, Sentiment analysis, Key phrase extraction, Entity recognition, Summarization, NER]
---

# Técnicas de análisis de texto

## ¿Qué es?

El **análisis de texto** extrae información estructurada de texto no estructurado. La guía oficial nombra **cuatro técnicas** de forma explícita: **extracción de palabras/frases clave**, **detección de entidades**, **análisis de sentimiento** y **resumen**. En los laboratorios aparecen además **detección de idioma** y **detección/redacción de PII**.

## Las técnicas oficiales

| Técnica | Qué hace | Entrada | Salida | Ejemplo |
|---|---|---|---|---|
| **Keyword / key phrase extraction** | Identifica los **conceptos principales** del texto | Texto | Lista de frases clave | De una reseña: "sonido SID", "teclado", "precio" |
| **Entity detection (NER)** | Localiza y **clasifica entidades**: personas, lugares, organizaciones, fechas, cantidades, productos | Texto | Entidades + categoría + confianza + posición | "Margaret Ellis" → Persona; "14 septiembre 1984" → Fecha |
| **Sentiment analysis** | Determina si la opinión es **positiva, negativa, neutra o mixta** | Texto | Etiqueta + puntuaciones (0–1) por frase y documento | Reseña de producto → 85 % positiva |
| **Summarization** | Condensa el texto en sus ideas principales | Texto largo | Resumen | Resumir una reseña de revista en un párrafo |

Dos modos de resumen: **extractivo** (selecciona las frases más relevantes del original) y **abstractivo** (redacta frases nuevas; es lo que hacen los LLM).

## Otras técnicas que aparecen en los labs

| Técnica | Qué hace | Nota |
|---|---|---|
| **Language detection** | Detecta el idioma principal y devuelve nombre, código ISO y confianza | Primer paso habitual de un pipeline, para enrutar el texto |
| **PII detection / redaction** | Detecta y oculta datos personales: nombres, correos, teléfonos, direcciones, tarjetas… | Clave para privacidad y cumplimiento |
| **Entity linking** | Vincula entidades a una base de conocimiento (Wikipedia) | 🟡 En retirada |
| **Clasificación de texto** | Asigna categorías a documentos | Con LLM o modelo personalizado |
| **Opinion mining** | Asocia el sentimiento a aspectos concretos ("la batería es mala, la pantalla excelente") | Extensión del sentimiento |

## Cómo funciona

```
Texto ─▶ detección de idioma ─▶ análisis (entidades / sentimiento / frases clave / PII / resumen)
     ─▶ JSON con resultados y puntuaciones de confianza
```

Con un **LLM** el mismo trabajo se hace con un prompt: *"Resume esta reseña en un párrafo"* o *"Extrae las entidades y devuélvelas en JSON"*.

## Ejemplo (laboratorio oficial)

Se pega una reseña del Commodore 64 de 1982 en el playground de `gpt-5-mini` con la instrucción *"You are an AI assistant that analyzes and summarizes text"* y el prompt *"Summarize this review as a single short paragraph"*. Después se compara con **Azure Language**: detección de idioma sobre una etiqueta en alemán y redacción de PII sobre una factura con nombre, dirección y teléfono.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **Frases clave** vs **entidades** | Frases clave = temas principales (texto libre). Entidades = elementos **clasificados** en categorías (persona, lugar, fecha) |
| **Entidades (NER)** vs **PII** | NER clasifica cualquier entidad; PII se centra en datos **personales** y permite **redactarlos** |
| **Sentimiento** vs **clasificación** | Sentimiento = positivo/negativo/neutro/mixto. Clasificación = categorías que tú defines |
| **Resumen extractivo** vs **abstractivo** | Frases del original / frases nuevas |
| **Detección de idioma** vs **traducción** | Saber en qué idioma está / convertirlo a otro ([[Azure AI Translator]]) |

## 🧠 Memorizar

> [!important]
> - Las **cuatro oficiales**: *keyword extraction · entity detection · sentiment analysis · summarization*.
> - Sentimiento devuelve **positivo, negativo, neutro o mixto** con puntuaciones.
> - **PII redaction** oculta datos personales; es la respuesta a escenarios de privacidad.
> - **Detección de idioma** devuelve idioma + código ISO + confianza.
> - Toda salida trae **confianza** (0–1).

## Tips para AI-901

> [!tip]
> - ⭐ "Saber si las opiniones son positivas o negativas" → **análisis de sentimiento**.
> - ⭐ "Identificar nombres de personas, lugares y fechas" → **detección de entidades (NER)**.
> - ⭐ "Obtener los temas principales de miles de documentos" → **extracción de frases clave**.
> - ⭐ "Condensar un documento largo" → **resumen**.
> - 🔥 "Ocultar datos personales antes de almacenar" → **PII redaction** (Azure Language).
> - 🔥 "Enrutar el texto al modelo correcto según el idioma" → **detección de idioma**.
> - ⚠️ "Mixto" es un resultado válido del análisis de sentimiento (hay partes positivas y negativas).

## ⚠️ Errores comunes

- Responder "clasificación de texto" cuando piden sentimiento.
- Confundir frases clave con entidades.
- Creer que la detección de idioma traduce.

## 💡 Escenario

Una empresa recibe miles de comentarios de clientes y necesita saber si son positivos, negativos o neutros. ¿Qué capacidad de Azure debería utilizar?

<details><summary>Respuesta</summary>

**Análisis de sentimiento**, disponible en **Azure Language in Foundry Tools** (resultado determinista, ideal para volumen) o mediante un **modelo generativo** con un prompt si se necesita más matiz. Para el escenario descrito, con miles de comentarios y necesidad de consistencia, el servicio especializado es la respuesta esperada.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué técnica de análisis de texto identifica y clasifica personas, organizaciones y fechas dentro de un documento?
- A) Extracción de frases clave · B) Detección de entidades · C) Análisis de sentimiento · D) Resumen

<details><summary>Respuesta</summary>

**B.** A devuelve temas sin clasificar; C mide opinión; D condensa.
</details>

**2.** Un hospital debe eliminar nombres y teléfonos de las notas clínicas antes de archivarlas. ¿Qué capacidad usas?
- A) Detección de idioma · B) Extracción de frases clave · C) Detección y redacción de PII · D) Resumen abstractivo

<details><summary>Respuesta</summary>

**C.**
</details>

**3.** ¿Cuáles son los valores posibles de un análisis de sentimiento en Azure Language?
- A) Alto, medio, bajo
- B) Positivo, negativo, neutro, mixto
- C) Verdadero, falso
- D) 1 a 5 estrellas

<details><summary>Respuesta</summary>

**B.**
</details>

**4.** Verdadero o falso: el resumen extractivo genera frases nuevas que no aparecen en el texto original.

<details><summary>Respuesta</summary>

**Falso.** Eso es el resumen **abstractivo**; el extractivo selecciona frases del original.
</details>

## Relacionado

- [[Procesamiento de Lenguaje Natural (NLP)]] · [[Azure AI Language]]
- [[Azure AI Translator]] · [[Extracción de información]]
- [[Privacy and Security (Privacidad y seguridad)]] (PII)
- [[Lab 03 - Análisis de texto en Foundry]]

← Volver al índice: [[00 - Índice - Texto y voz]]
