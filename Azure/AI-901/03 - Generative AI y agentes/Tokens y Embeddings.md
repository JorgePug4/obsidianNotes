---
tags: [ai-901, azure, ia, generative-ai, llm]
módulo: Generative AI y agentes
peso_examen: Alto
aliases: [Tokens, Embeddings, Ventana de contexto]
---

# Tokens y Embeddings

## ¿Qué es?

- **Token**: la unidad mínima en que un modelo de lenguaje divide el texto. No es exactamente una palabra: puede ser una palabra corta, un fragmento de palabra, un signo o un espacio. En inglés, 1 token ≈ 4 caracteres ≈ ¾ de palabra. En otros idiomas suele haber más tokens por palabra.
- **Embedding**: un **vector** de números que representa el significado de un token, frase o documento. Textos con significado parecido tienen vectores cercanos.

## ¿Para qué sirve?

- Los **tokens** determinan **coste** (se factura por tokens de entrada y salida), **latencia** y el **límite de contexto** del modelo.
- Los **embeddings** permiten **búsqueda semántica**: encontrar documentos por significado, no por palabras exactas. Son la base de la **búsqueda vectorial** que usa RAG y Foundry IQ.

## Conceptos clave

- **Ventana de contexto** (*context window*): máximo de tokens que caben en una interacción (prompt + historial + respuesta). Si se supera, hay que resumir o recortar el historial.
- **Max tokens / max output tokens**: parámetro que limita la longitud de la respuesta.
- **Tokens de entrada vs de salida**: se facturan de forma distinta; la salida suele ser más cara.
- **Modelo de embeddings**: modelo específico (p. ej., `text-embedding-3-large`) que convierte texto en vectores; no genera texto.
- **Índice vectorial / vector store**: almacén de embeddings para buscar por similitud (Azure AI Search, el *vector store* de file search).
- **Similitud (coseno)**: medida de cercanía entre vectores.

## Cómo funciona

```
"Reserva un vuelo a Madrid" ─▶ tokens: ["Res","erva"," un"," vuelo"," a"," Madrid"]
                              ─▶ embedding: [0.12, -0.87, 0.33, … ] (cientos/miles de dimensiones)
Consulta ─▶ embedding ─▶ buscar vectores cercanos en el índice ─▶ documentos relevantes ─▶ prompt aumentado
```

## Ejemplo

Un buscador de ayuda interna: cada artículo se convierte en un embedding y se guarda en un índice. Cuando el usuario escribe "no puedo entrar en el correo", su consulta se vectoriza y el índice devuelve el artículo "Restablecer contraseña de Outlook", aunque no comparta palabras.

## 📌 Diferencias clave

| Concepto | Es… | No es… |
|---|---|---|
| Token | Fragmento de texto que el modelo procesa | Una palabra exacta |
| Embedding | Vector numérico de significado | El texto en sí ni un token |
| Ventana de contexto | Límite de tokens por interacción | Memoria permanente del modelo |
| Max tokens | Límite de la **respuesta** | Límite del prompt |

## 🧠 Memorizar

> [!important]
> - **Tokens** = unidades de texto → coste, latencia y límite de contexto.
> - **Embeddings** = vectores de significado → búsqueda semántica y RAG.
> - **Ventana de contexto** incluye entrada **y** salida.
> - Para embeddings se usa un **modelo de embeddings**, no un modelo de chat.

## Tips para AI-901

> [!tip]
> - ⭐ "¿Qué determina el coste de una llamada al modelo?" → número de tokens de entrada y salida.
> - 🔥 "El modelo pierde el hilo en conversaciones largas" → se excede la ventana de contexto → resumir/recortar historial.
> - 🔥 "Buscar documentos por significado y no por palabra clave" → embeddings + búsqueda vectorial.
> - ⚠️ Al crear una knowledge base en Foundry IQ se elige un **modelo de embeddings** además del modelo de chat.

## ⚠️ Errores comunes

- Contar palabras como tokens (subestima el coste).
- Pensar que subir `max tokens` mejora la calidad: solo permite respuestas más largas.

## 💡 Escenario

Una empresa quiere que su agente encuentre la política correcta aunque el empleado use sinónimos o describa el problema con otras palabras. ¿Qué tecnología lo permite?

<details><summary>Respuesta</summary>

**Embeddings y búsqueda vectorial** (Azure AI Search / Foundry IQ): se comparan significados, no palabras exactas.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué afirmación sobre los tokens es correcta?
- A) Un token siempre equivale a una palabra
- B) Los tokens determinan el coste y el límite de contexto de una interacción
- C) Los tokens son vectores numéricos
- D) Los tokens solo existen en modelos de imagen

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Para qué se usa un modelo de embeddings en una solución RAG?
- A) Para generar la respuesta final
- B) Para convertir documentos y consultas en vectores y encontrar los fragmentos más relevantes
- C) Para filtrar contenido dañino
- D) Para transcribir audio

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Large Language Models]]
- [[Grounding, RAG y Foundry IQ]]
- [[Despliegue y configuración de modelos]] (max tokens)

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
