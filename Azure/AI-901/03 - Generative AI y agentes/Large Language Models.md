---
tags: [ai-901, azure, ia, generative-ai, llm]
módulo: Generative AI y agentes
peso_examen: Muy alto
objetivo_oficial: "1.2.1 Describe how generative AI models work"
aliases: [LLM, Modelos de lenguaje, Transformer, SLM]
---

# Large Language Models

## ¿Qué es?

Un **Large Language Model (LLM)** es un modelo de lenguaje de gran tamaño, basado en la arquitectura **transformer**, entrenado con enormes cantidades de texto para **predecir el siguiente token**. De esa capacidad emergen la conversación, el resumen, la traducción, el razonamiento y la generación de código.

Un **SLM** (*Small Language Model*, p. ej., la familia **Phi** de Microsoft) es un modelo más pequeño, más barato y rápido, ejecutable incluso en un navegador o en el dispositivo (Foundry Local), con menor capacidad.

## ¿Para qué sirve?

Es el motor de las apps generativas y de los agentes. En AI-901 debes poder **describir cómo funciona** (objetivo 1.2.1) sin entrar en matemáticas.

## Conceptos clave

- **Tokenización**: el texto se divide en **tokens** (trozos de palabras). Ver [[Tokens y Embeddings]].
- **Embeddings**: cada token se representa como un **vector** numérico que captura su significado; palabras relacionadas quedan cerca en el espacio vectorial.
- **Transformer**: arquitectura con dos partes conceptuales, *encoder* (entender la entrada) y *decoder* (generar la salida). Los LLM generativos usan sobre todo el decoder.
- **Atención (self-attention)**: mecanismo que permite al modelo ponderar qué tokens del contexto son relevantes para cada token; así entiende que "her" se refiere a "Ada Lovelace" varias frases atrás.
- **Codificación posicional**: informa del orden de los tokens.
- **Parámetros / pesos**: los números aprendidos durante el entrenamiento (miles de millones).
- **Ventana de contexto**: número máximo de tokens (entrada + salida) que el modelo maneja a la vez.
- **Preentrenamiento** (con datos masivos) → **ajuste** (instrucciones, RLHF) → **fine-tuning** opcional con tus ejemplos.
- **Modelos de razonamiento**: variantes que "piensan" más pasos antes de responder (más lentos, mejores en problemas complejos).

## Cómo funciona (paso a paso)

```
Prompt ──▶ tokenización ──▶ embeddings + posición ──▶ capas transformer (atención)
      ──▶ distribución de probabilidad del siguiente token ──▶ elegir token (temperature/top_p)
      ──▶ añadirlo a la salida ──▶ repetir hasta fin o max tokens
```

1. El prompt (instrucciones + historial + mensaje) se convierte en tokens.
2. Cada token pasa a un vector (embedding).
3. Las capas de atención relacionan cada token con el resto del contexto.
4. El modelo produce probabilidades para el siguiente token; `temperature` y `top_p` deciden cuánto arriesgar.
5. Se repite token a token. Por eso las respuestas pueden "salir en streaming".

## Ejemplo

Prompt: "La capital de Francia es". El modelo asigna alta probabilidad a "París" porque en el entrenamiento ese patrón aparece muchísimas veces. Con `temperature` alta podría elegir algo menos probable.

## Comparaciones

| Tipo de modelo | Tamaño | Coste/latencia | Cuándo usarlo | Ejemplos en Foundry |
|---|---|---|---|---|
| **LLM** | Muy grande | Mayores | Tareas complejas, razonamiento, multimodal | gpt-5, gpt-5-mini, Claude, Llama grande |
| **SLM** | Pequeño | Bajos | Dispositivos, baja latencia, tareas acotadas | Phi, gpt-5-nano |
| **Modelo de razonamiento** | Grande | Alto (piensa más) | Problemas de varios pasos | o-series, gpt-5 con reasoning effort |
| **Modelo de embeddings** | Pequeño | Bajo | Vectorizar texto para búsqueda semántica y RAG | text-embedding-3 |
| **Multimodal** | Grande | Mayores | Imagen/audio + texto | gpt-5-mini (visión), gpt-realtime (audio) |

## 🧠 Memorizar

> [!important]
> - LLM = **transformer** + **atención** + predicción del **siguiente token**.
> - **Tokens** → **embeddings** (vectores de significado).
> - **Ventana de contexto** = límite de tokens de entrada + salida.
> - **SLM (Phi)** = pequeño, barato, rápido, ejecutable localmente.
> - Los LLM no consultan bases de datos: **generan**.

## Tips para AI-901

> [!tip]
> - ⭐ Pregunta típica: "¿Qué mecanismo permite al modelo relacionar palabras distantes en un texto?" → **atención**.
> - 🔥 "¿Qué representación numérica captura el significado de los tokens?" → **embeddings**.
> - 🔥 "Necesitan ejecutar el modelo en dispositivos con poca capacidad / sin conexión" → **SLM** (Phi, Foundry Local).
> - ⚠️ "Encoder vs decoder" no suele preguntarse en detalle; basta saber que los generativos usan el decoder.

## ⚠️ Errores comunes

- Pensar que "más grande siempre es mejor". El examen premia elegir el modelo **adecuado al caso** (coste, latencia, capacidad).
- Confundir embeddings (vectores) con tokens (trozos de texto).

## 💡 Escenario

Una app móvil debe clasificar mensajes cortos en el propio teléfono, sin enviar datos a la nube y con latencia mínima. ¿Qué tipo de modelo?

<details><summary>Respuesta</summary>

Un **SLM** como Phi, ejecutado localmente (por ejemplo con Foundry Local). Un LLM en la nube incumple los requisitos de privacidad local y latencia.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué hace un modelo de lenguaje generativo en cada paso de la generación?
- A) Busca la frase más parecida en su base de datos
- B) Predice el siguiente token más probable dado el contexto
- C) Traduce el prompt a otro idioma
- D) Clasifica el prompt en una categoría

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Qué son los embeddings?
- A) Filtros de contenido · B) Representaciones vectoriales del significado del texto · C) Parámetros de despliegue · D) Herramientas de un agente

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** ¿Qué característica de un modelo limita la cantidad de texto que puede procesar y generar en una sola interacción?
- A) Temperature · B) Ventana de contexto · C) Top_p · D) Embedding

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Generative AI]] · [[Tokens y Embeddings]]
- [[Catálogo de modelos de Foundry]] · [[Despliegue y configuración de modelos]]
- [[Prompts y Prompt Engineering]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
