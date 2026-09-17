---
tags: [ai-901, azure, ia, generative-ai, agentes, repaso]
módulo: Generative AI y agentes
---

# 🎯 Repaso final · Generative AI y agentes

## 1. Los conceptos más importantes

1. Un LLM es un **transformer** que **predice el siguiente token** usando **atención**; los tokens se representan como **embeddings**.
2. **Tokens** → coste, latencia y **ventana de contexto**. **Embeddings** → búsqueda semántica y RAG.
3. **Prompt de sistema (Instructions)** = rol, reglas, formato. **Prompt de usuario** = tarea. **Few-shot** = ejemplos en el prompt.
4. **Grounding/RAG** reduce alucinaciones; **Foundry IQ** centraliza conocimiento sobre Azure AI Search; **web search** aporta actualidad pública.
5. **Catálogo (Discover → Models)**: elegir por tarea, modalidad, coste, contexto, región. **Direct from Azure** = alojado por Azure.
6. **Despliegue**: Standard (pago por token) · Provisioned · Batch; ámbitos Global · Data Zone · Regional. **Playground** para probar.
7. **Parámetros**: temperature (aleatoriedad), top_p (alternativa), max tokens (longitud de salida).
8. **Cliente de chat**: OpenAI SDK + `responses.create(model, instructions, input)` → `output_text`.
9. **Agente** = modelo + instrucciones + herramientas + conocimiento; runtime **agents / conversations / responses**.
10. **Cliente de agente**: `AIProjectClient` + `DefaultAzureCredential` + `get_openai_client()` + `agent_reference`.

## 2. Tabla de servicios y propósito

| Servicio / componente | Propósito | Concepto clave |
|---|---|---|
| Foundry Models (catálogo) | Elegir y desplegar modelos | Tarea de inferencia, colección |
| Playground | Probar modelos/agentes sin código | Instructions, Tools, Save as agent |
| Foundry Agent Service | Crear y ejecutar agentes | Tools, conversations, versión |
| Foundry IQ | Conocimiento para agentes | Knowledge base, output mode, citas |
| File search | Grounding rápido con archivos | Vector store |
| Web search | Grounding con Internet | Bing |
| Foundry SDK (`azure-ai-projects`) | Conectar apps a proyectos y agentes | `AIProjectClient` |
| OpenAI SDK | Llamar a modelos desplegados | Responses API |
| Content Safety / guardrails | Seguridad de entradas y salidas | 4 categorías, Prompt Shields, groundedness |

## 3. Diferencias que más se confunden

| Par confuso | Cómo distinguirlo |
|---|---|
| Prompt de sistema vs usuario | Desarrollador, una vez, reglas / usuario, cada turno, tarea |
| Few-shot vs fine-tuning | Ejemplos en el prompt / reentrenar el modelo |
| RAG vs fine-tuning | Aportar datos en la consulta / cambiar comportamiento con ejemplos |
| File search vs Foundry IQ | Un agente, archivos subidos / varios agentes, fuentes empresariales centralizadas |
| Web search vs file search | Internet público / tus documentos |
| Temperature vs max tokens | Aleatoriedad / longitud de salida |
| Max tokens vs ventana de contexto | Salida / total entrada + salida |
| Standard vs Provisioned vs Batch | Pago por uso / capacidad reservada / asíncrono barato |
| Global vs Data Zone vs Regional | Cualquier región / zona geográfica / región fija |
| Modelo desplegado vs agente | Endpoint de inferencia / modelo + instrucciones + herramientas versionado |
| OpenAI SDK vs `AIProjectClient` | Endpoint `/openai/v1/` con clave o Entra / project endpoint solo Entra |
| Responses API vs Chat Completions | `instructions`/`input`/`output_text` / `messages`/`choices[0].message.content` |
| Modelo multimodal vs modelo de imagen | Entiende imágenes / genera imágenes |
| Copilot Studio vs Foundry | Low-code M365 / pro-code, control total |

## 4. Diez tips de examen

> [!tip]
> 1. "No debe inventar; usar nuestros documentos" → RAG (Foundry IQ / file search), no fine-tuning.
> 2. "Consistencia" → temperature baja + formato en el prompt de sistema.
> 3. "Se corta la respuesta" → max tokens.
> 4. "Residencia de datos en la UE" → Data Zone EU o regional.
> 5. "Alto volumen no interactivo al menor coste" → Batch.
> 6. "Rendimiento garantizado" → Provisioned.
> 7. "Sin código, probar rápido" → playground; "web básica para el agente" → Preview web app.
> 8. Cliente de agente: project endpoint + Entra + `agent_reference` (nombre y versión).
> 9. "Acciones sobre sistemas externos" → agente con function/OpenAPI; "cálculos" → code interpreter.
> 10. Las claves nunca en el código; preferir `DefaultAzureCredential`.

## 5. Preguntas de repaso

**1.** ¿Qué parámetro ajustarías para que un asistente de brainstorming proponga ideas más variadas?

<details><summary>Respuesta</summary>

**Temperature** (subirla) o top_p.
</details>

**2.** ¿Qué componente de Foundry permite que tres agentes compartan la misma base de conocimiento?

<details><summary>Respuesta</summary>

**Foundry IQ** (knowledge base sobre Azure AI Search).
</details>

**3.** Un fragmento de código usa `client.chat.completions.create(messages=[...])`. ¿Qué API es y cómo se obtiene el texto?

<details><summary>Respuesta</summary>

Chat Completions; `response.choices[0].message.content`.
</details>

**4.** ¿Qué tipo de despliegue elegirías para clasificar 20 millones de correos históricos durante un fin de semana al menor coste?

<details><summary>Respuesta</summary>

**Batch.**
</details>

**5.** ¿Qué diferencia hay entre pulsar "Save as agent" y seguir usando el modelo en el playground?

<details><summary>Respuesta</summary>

"Save as agent" crea un activo versionado con modelo, instrucciones y herramientas y un endpoint propio que los clientes consumen sin gestionar el prompt de sistema ni RAG.
</details>

**6.** Nombra tres herramientas que puede usar un agente en Foundry.

<details><summary>Respuesta</summary>

Web search, file search, code interpreter (también Foundry IQ, function calling/OpenAPI, MCP, image generation).
</details>

**7.** ¿Qué son los embeddings y para qué se usan en RAG?

<details><summary>Respuesta</summary>

Vectores numéricos que representan significado; permiten encontrar los fragmentos más relevantes por similitud semántica.
</details>

**8.** Verdadero o falso: un modelo desplegado como "Global Standard" procesa las peticiones únicamente en la región del recurso.

<details><summary>Respuesta</summary>

**Falso.** Global puede procesar en cualquier región; para restringir se usa Data Zone o regional.
</details>

**9.** ¿Qué campo del código de un cliente de agente identifica al agente concreto?

<details><summary>Respuesta</summary>

`extra_body={"agent_reference": {"name": ..., "version": ..., "type": "agent_reference"}}`.
</details>

**10.** ¿Qué técnica usarías para que el modelo clasifique tickets con las cinco etiquetas exactas de tu equipo sin reentrenarlo?

<details><summary>Respuesta</summary>

Prompt de sistema con las etiquetas y **few-shot** (ejemplos), temperature baja.
</details>

## Checklist

- [ ] Comprendo cómo funciona un LLM (tokens, embeddings, atención, siguiente token).
- [ ] Sé escribir prompts de sistema y usuario eficaces y distinguir few-shot, RAG y fine-tuning.
- [ ] Sé elegir un modelo del catálogo según capacidades.
- [ ] Sé desplegar y probar un modelo en el portal y qué hacen temperature, top_p y max tokens.
- [ ] Reconozco el código de un cliente de chat y de un cliente de agente.
- [ ] Sé crear y probar un agente con herramientas y conocimiento en el portal.
- [ ] Puedo responder las preguntas de práctica.

## 🧠 Última pasada antes del examen

> [!important] Lo que no puede fallarte
> - LLM = transformer + atención + siguiente token. Tokens = coste y contexto. Embeddings = significado.
> - Sistema = reglas; usuario = tarea; few-shot = ejemplos.
> - RAG/Foundry IQ para datos propios; web search para actualidad; fine-tuning para estilo.
> - Deploy → playground → Instructions → Tools → Save as agent → Preview web app → Continue in code.
> - temperature ↓ = determinista; max tokens = longitud; Standard / Provisioned / Batch; Global / Data Zone / Regional.
> - `OpenAI(base_url, api_key).responses.create(model, instructions, input).output_text`.
> - `AIProjectClient(endpoint, DefaultAzureCredential()).get_openai_client().responses.create(input, extra_body=agent_reference)`.

Volver al índice: [[00 - Índice - Generative AI y agentes]] · [[00 - AI-901 Índice general]]
