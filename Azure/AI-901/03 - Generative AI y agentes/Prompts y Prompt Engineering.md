---
tags: [ai-901, azure, ia, generative-ai, prompts, foundry]
módulo: Generative AI y agentes
peso_examen: Muy alto
objetivo_oficial: "2.1.1 Create effective system and user prompts for generative AI models"
aliases: [Prompt, Prompt engineering, System prompt, Instrucciones]
---

# Prompts y Prompt Engineering

## ¿Qué es?

Un **prompt** es la entrada que se envía a un modelo generativo. En una aplicación de chat hay dos tipos que el examen distingue:

- **Prompt de sistema / instrucciones** (*system prompt*, en el portal de Foundry se llama **Instructions**): mensaje que fija el **rol, el tono, el formato y los límites** del modelo durante toda la conversación. Lo escribe el desarrollador, no el usuario.
- **Prompt de usuario** (*user prompt*): el mensaje concreto de cada turno.

**Prompt engineering** es la práctica de diseñar esos mensajes para obtener respuestas más precisas, seguras y consistentes.

## ¿Para qué sirve?

Es la forma más barata y rápida de controlar un modelo: antes de pensar en fine-tuning, se ajusta el prompt. El objetivo 2.1.1 pide saber **crear prompts de sistema y usuario eficaces**.

## Conceptos clave

- **Rol / persona**: "Eres un experto en historia de la computación".
- **Restricciones**: "Solo respondes sobre historia de la computación. No participes en otros temas".
- **Formato de salida**: "Responde en una lista de tres puntos", "Devuelve JSON con los campos…".
- **Tono y longitud**: "Respuestas concisas y precisas".
- **Contexto / grounding en el prompt**: incluir datos en el propio prompt.
- **Few-shot**: dar uno o varios **ejemplos** de entrada→salida dentro del prompt. *Zero-shot* = sin ejemplos.
- **Cadena de pensamiento**: pedir que razone paso a paso (útil en problemas lógicos).
- **Historial de conversación**: en apps de chat el historial se reenvía con cada prompt; así "her" se entiende como "Ada Lovelace".
- **Plantillas de prompt**: prompts con variables que se rellenan en tiempo de ejecución.

## Cómo funciona

```
[system]  Eres un asistente que analiza y resume texto. Responde en español, en un párrafo.
[user]    Resume esta reseña: …texto…
[assistant] (respuesta)
[user]    ¿Y cuáles eran sus puntos débiles?   ← el historial permite entender "sus"
```

En el **playground** de Foundry: el cuadro **Instructions** es el prompt de sistema; el cuadro de chat es el prompt de usuario. En código (Responses API): `instructions="…"` y `input="…"`.

## Buenas prácticas (lo que el examen considera "eficaz")

| Práctica | Ejemplo |
|---|---|
| Ser específico | "Lista tres hechos sobre Ada Lovelace" mejor que "Háblame de Ada Lovelace" |
| Definir rol y alcance en el sistema | "Solo respondes sobre políticas de gastos" |
| Indicar el formato | "Devuelve un JSON con `proveedor`, `fecha`, `total`" |
| Dar ejemplos (few-shot) | Dos ejemplos de clasificación antes de la entrada real |
| Aportar contexto | Pegar el texto a resumir o conectar una fuente (RAG) |
| Pedir que reconozca lo que no sabe | "Si no está en el contexto, di que no lo sabes" |
| Separar claramente instrucciones y datos | Usar delimitadores para el texto del usuario |
| Iterar | Probar en el playground y refinar |

## Ejemplo (laboratorio oficial)

Instrucciones: *"You are an expert in the history of computing and AI. You only answer questions about significant people and events in the development of computing, and about notable vintage computers. Do not engage in conversations on any topic that is unrelated to computing history."* Al preguntar "¿Cuál es la capital de España?", el modelo rehúsa. Así se demuestra que el prompt de sistema **limita el alcance**.

## 📌 Diferencias clave

| | Prompt de sistema | Prompt de usuario |
|---|---|---|
| Quién lo escribe | Desarrollador | Usuario final |
| Cuándo | Una vez, para toda la conversación | En cada turno |
| Contenido | Rol, reglas, formato, tono | Pregunta o tarea concreta |
| En Foundry | **Instructions** | Cuadro de chat |
| En código | `instructions=` (Responses API) o mensaje con `role: "system"` | `input=` o mensaje con `role: "user"` |

| Prompt engineering | Grounding (RAG) | Fine-tuning |
|---|---|---|
| Cambiar cómo se pide | Dar información en tiempo de consulta | Reentrenar parcialmente con ejemplos |
| Gratis, inmediato | Coste de índice; datos siempre actualizados | Coste y tiempo; cambia comportamiento/estilo |
| Primero | Segundo (cuando faltan datos) | Último recurso |

## 🧠 Memorizar

> [!important]
> - **Sistema** = rol, reglas, formato; **usuario** = tarea concreta.
> - **Few-shot** = ejemplos en el prompt. **Zero-shot** = sin ejemplos.
> - En Foundry el prompt de sistema se llama **Instructions**.
> - Un prompt eficaz es **específico**, define **formato** y **alcance**, y aporta **contexto**.
> - El historial se **reenvía** en cada turno; el modelo no "recuerda" por sí mismo.

## Tips para AI-901

> [!tip]
> - ⭐ "¿Dónde defines que el asistente solo hable de X y responda en formato Y?" → prompt de sistema / Instructions.
> - 🔥 "Las respuestas varían de formato entre llamadas" → especifica el formato en el prompt de sistema (y baja la temperature).
> - 🔥 "Quieres que clasifique con las mismas etiquetas que usa tu equipo" → few-shot con ejemplos.
> - ⚠️ Poner en el prompt de sistema información que cambia a diario (precios, stock) es mala práctica: usa RAG/herramientas.
> - ⚠️ "El modelo no recuerda lo que dije hace tres mensajes" → el cliente no está enviando el historial, o se superó la ventana de contexto.

## ⚠️ Errores comunes

- Confundir *instructions* con *knowledge*: las instrucciones dicen **cómo** comportarse; el conocimiento aporta **qué** saber.
- Pensar que el usuario puede "sobrescribir" el prompt de sistema legítimamente; si lo intenta es un jailbreak (Prompt Shields).

## 💡 Escenario

Un equipo quiere que el asistente devuelva siempre las respuestas como JSON con los campos `titulo`, `resumen` y `sentimiento`, y que rechace preguntas ajenas al catálogo de productos. ¿Dónde lo configuras y cómo?

<details><summary>Respuesta</summary>

En el **prompt de sistema (Instructions)**: definir el rol ("asistente del catálogo de productos"), la restricción de alcance y el formato JSON exacto; opcionalmente añadir un ejemplo (few-shot) y bajar la temperature para consistencia.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué tipo de prompt se usa para establecer el rol y las reglas de comportamiento de un modelo durante toda la conversación?
- A) Prompt de usuario · B) Prompt de sistema · C) Prompt few-shot · D) Completion

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** Un desarrollador incluye en el prompt tres ejemplos de reseñas con su clasificación antes de pedir que clasifique una nueva. ¿Qué técnica usa?
- A) Zero-shot · B) Few-shot · C) Fine-tuning · D) Grounding

<details><summary>Respuesta</summary>

**B.** Los ejemplos van en el prompt; no se reentrena nada.
</details>

**3.** ¿Cuál de estas mejoras hace un prompt más eficaz?
- A) Hacerlo lo más corto y genérico posible
- B) Especificar el formato de salida y el alcance de la respuesta
- C) Subir la temperature al máximo
- D) Eliminar el prompt de sistema

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Generative AI]] · [[Large Language Models]]
- [[Grounding, RAG y Foundry IQ]]
- [[Despliegue y configuración de modelos]] (temperature)
- [[Foundry SDK y cliente de chat]] · [[Agentes de IA (Foundry Agent Service)]]
- [[Lab 02 - Desplegar un modelo y crear un agente]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
