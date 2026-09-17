---
tags: [ai-901, azure, ia, generative-ai]
módulo: Generative AI y agentes
peso_examen: Muy alto
aliases: [IA generativa, GenAI]
---

# Generative AI

## ¿Qué es?

La **IA generativa** es la rama de la IA cuyos modelos **crean contenido nuevo** (texto, código, imágenes, audio, vídeo) a partir de una entrada en lenguaje natural llamada **prompt**. Los modelos más conocidos son los **[[Large Language Models]]** (LLM), pero también hay modelos de imagen (gpt-image, FLUX, MAI-Image), de vídeo (Sora) y de voz.

## ¿Para qué sirve?

Redactar, resumir, traducir, responder preguntas, generar código, crear imágenes, mantener conversaciones y, combinada con herramientas, **actuar** como agente. Es la base de los copilots y de la mayoría de aplicaciones de IA modernas.

## Conceptos clave

- **Prompt**: la entrada. Puede incluir texto, imágenes y audio (multimodal). Ver [[Prompts y Prompt Engineering]].
- **Completion / respuesta**: la salida generada.
- **Instrucciones / prompt de sistema**: mensaje que define rol, tono y límites del modelo.
- **Tokens**: unidades en que el modelo divide el texto; determinan coste y límite de contexto. Ver [[Tokens y Embeddings]].
- **Foundation model**: modelo grande preentrenado que sirve de base para muchas tareas.
- **Modelo multimodal**: acepta texto + imagen (+ audio) y responde en texto u otra modalidad.
- **Grounding / RAG**: dar al modelo datos propios para que responda con ellos. Ver [[Grounding, RAG y Foundry IQ]].
- **Fine-tuning**: ajustar un modelo base con ejemplos propios para cambiar estilo o especializarlo.
- **Agente**: modelo + instrucciones + herramientas que realiza tareas. Ver [[Agentes de IA (Foundry Agent Service)]].
- **Alucinación**: respuesta plausible pero falsa.

## Cómo funciona

Un modelo generativo de texto **predice el siguiente token** una y otra vez, condicionado por el prompt y por lo que ya ha generado. No busca en una base de datos: **genera dinámicamente**. Por eso:

- Es flexible: el mismo modelo resume, traduce o programa según el prompt.
- No está "anclado" a hechos: puede inventar. De ahí el grounding y los guardrails.
- Las respuestas pueden variar entre ejecuciones (controlable con `temperature`).

Los laboratorios oficiales contrastan esto con la "búsqueda" clásica: un buscador devuelve texto que ya existe; un LLM lo genera.

## Ejemplo

Una tienda de informática vintage crea un asistente en Foundry: despliega `gpt-5-mini`, le da la instrucción "Eres un experto en historia de la computación", añade búsqueda web y un documento con números de serie. El asistente resume reseñas antiguas, identifica placas por su número de serie y busca tiendas cercanas.

## Tipos de generación

| Modalidad de salida | Ejemplo de modelo en Foundry | Uso |
|---|---|---|
| Texto / código | gpt-5, gpt-5-mini, gpt-5-nano, Phi, Llama, Mistral, DeepSeek, Claude | Chat, resumen, redacción, código |
| Imagen | gpt-image-1, gpt-image-1-mini, gpt-image-2, FLUX.2-pro, MAI-Image-2 | Ilustraciones, marketing, edición |
| Vídeo | Sora-2 | Clips cortos a partir de texto |
| Audio / voz | gpt-realtime, MAI-Voice-1, voces neuronales de Azure Speech | Agentes de voz, locución |
| Embeddings (no generan contenido, sí vectores) | text-embedding-3 | Búsqueda semántica, RAG |

## Limitaciones y riesgos

| Limitación | Mitigación en Foundry |
|---|---|
| Alucinaciones | Grounding (RAG, Foundry IQ), groundedness detection, citas |
| Conocimiento limitado a la fecha de entrenamiento | Herramienta de búsqueda web, RAG |
| Contenido dañino o sesgado | Filtros de contenido, guardrails, evaluaciones |
| Ataques de prompt (jailbreak) | Prompt Shields |
| Coste y latencia por tokens | Elegir modelo pequeño (mini/nano), limitar `max tokens` |
| No determinista | `temperature` baja; o usar Foundry Tools para tareas estructuradas |

## 📌 Diferencia clave: generativa vs predictiva vs preconstruida

| | Generativa | ML predictivo | Foundry Tools |
|---|---|---|---|
| Salida | Contenido nuevo | Número/categoría | JSON estructurado |
| Entrada | Prompt | Features | Texto/imagen/audio |
| Control | Prompt + parámetros | Entrenamiento | Parámetros de API |

## 🧠 Memorizar

> [!important]
> - GenAI **genera** contenido; no lo busca.
> - **Prompt → completion**; el **prompt de sistema** fija el comportamiento.
> - Riesgo principal: **alucinaciones** → **grounding/RAG**.
> - **Multimodal** = texto + imagen (+ audio) en el mismo prompt.
> - **Fine-tuning** ≠ grounding: ajustar el modelo vs darle datos en tiempo de consulta.

## Tips para AI-901

> [!tip]
> - ⭐ "Responder con información actualizada de la web" → herramienta de búsqueda web. "Responder con documentos internos" → RAG / Foundry IQ / file search.
> - 🔥 "Reducir el coste manteniendo calidad razonable" → modelo más pequeño (mini/nano) y menos tokens.
> - ⚠️ "Cambiar el estilo o formato de forma consistente" se resuelve primero con **prompt de sistema**, no con fine-tuning.
> - ⚠️ Un modelo generativo **no aprende** de las conversaciones en producción.

## ⚠️ Errores comunes

- Creer que la respuesta de un LLM es una consulta a una base de datos.
- Usar fine-tuning cuando lo que falta es **información** (eso es grounding).

## 💡 Escenario

Una empresa quiere un asistente que responda sobre su política de gastos interna sin inventar cifras. ¿Qué enfoque?

<details><summary>Respuesta</summary>

**Grounding con RAG**: crear una knowledge base en Foundry IQ (o un índice de file search) con el documento de la política y conectarla al agente, de modo que responda citando el documento. Añadir groundedness detection si se quiere verificar.
</details>

## Preguntas que podrían aparecer

**1.** ¿Cuál es la principal diferencia entre un modelo generativo y un buscador tradicional?
- A) El buscador genera contenido; el modelo lo busca
- B) El modelo genera dinámicamente una respuesta; el buscador devuelve contenido existente
- C) No hay diferencia
- D) El modelo solo funciona con imágenes

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** Un modelo responde con datos inventados sobre un producto. ¿Cómo se llama este problema y qué lo mitiga?
- A) Overfitting; más datos de entrenamiento
- B) Alucinación; grounding con fuentes propias
- C) Sesgo; evaluación por grupos
- D) Latencia; modelo más grande

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** Verdadero o falso: para que un modelo conozca el catálogo de productos de tu empresa es imprescindible hacer fine-tuning.

<details><summary>Respuesta</summary>

**Falso.** Lo habitual y recomendado es grounding (RAG); el fine-tuning sirve para ajustar estilo o comportamiento con ejemplos, no para inyectar conocimiento cambiante.
</details>

## Relacionado

- [[Large Language Models]] · [[Tokens y Embeddings]]
- [[Prompts y Prompt Engineering]] · [[Grounding, RAG y Foundry IQ]]
- [[Catálogo de modelos de Foundry]] · [[Despliegue y configuración de modelos]]
- [[Agentes de IA (Foundry Agent Service)]]
- [[Generación de imágenes y vídeo]] · [[Modelos multimodales (visión en prompts)]]
- [[Azure AI Content Safety y guardrails]] · [[Responsible AI (principios de Microsoft)]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
