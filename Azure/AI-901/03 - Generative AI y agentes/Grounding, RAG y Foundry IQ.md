---
tags: [ai-901, azure, ia, generative-ai, rag, foundry]
módulo: Generative AI y agentes
peso_examen: Muy alto
aliases: [RAG, Grounding, Foundry IQ, Retrieval Augmented Generation, Azure AI Search, Knowledge base]
---

# Grounding, RAG y Foundry IQ

## ¿Qué es?

- **Grounding** (fundamentar): dar al modelo **información fiable y actual** para que sus respuestas se basen en ella y no solo en lo que "recuerda" del entrenamiento.
- **RAG (Retrieval Augmented Generation)**: el patrón más común de grounding. Antes de generar, se **recupera** (retrieve) contenido relevante de una fuente, se **añade** (augment) al prompt y el modelo **genera** (generate) la respuesta con ese contexto.
- **Foundry IQ**: la capa de **conocimiento** de Microsoft Foundry, construida sobre **Azure AI Search**, que centraliza las fuentes de datos en **knowledge bases** reutilizables por varios agentes sin programar la lógica de recuperación en cada uno.

## ¿Para qué sirve?

Reducir **alucinaciones**, responder con **datos propios** (políticas, catálogos, manuales), incluir **información posterior al entrenamiento** y ofrecer **citas** (transparencia).

## Conceptos clave

- **Fuente de conocimiento** (*knowledge source*): archivos subidos, Azure Blob Storage, SharePoint, OneLake, sitios web, índices existentes.
- **Índice / vector store**: los documentos se trocean (*chunking*), se vectorizan con un **modelo de embeddings** y se guardan para búsqueda por similitud.
- **Búsqueda híbrida**: combina búsqueda por palabras clave y vectorial (Azure AI Search).
- **Knowledge base** (Foundry IQ): agrupa fuentes y define cómo recuperar y devolver conocimiento.
- **Output mode** en Foundry IQ: **extractive data** (devuelve texto literal de las fuentes) o **answer synthesis** (un modelo compone la respuesta a partir de las fuentes).
- **Retrieval reasoning effort**, **answer instructions** (formato de la respuesta) y **retrieval instructions** (cuándo/cómo buscar en cada base).
- **Citas**: referencias a los fragmentos usados.
- **File search** (herramienta de agente): versión sencilla: subes archivos, se crea un vector store y el agente los consulta.
- **Web search**: herramienta que recupera información **pública y actual** de Internet (no es RAG sobre tus datos, pero también es grounding).

## Cómo funciona (RAG)

```
Pregunta del usuario
   │ 1. embedding de la pregunta
   ▼
Índice (Azure AI Search / vector store) ──▶ 2. fragmentos más relevantes
   │
   ▼ 3. prompt = instrucciones + fragmentos + pregunta
Modelo generativo ──▶ 4. respuesta fundamentada + citas
```

Con Foundry IQ, los pasos 1 a 3 los hace la knowledge base; el agente solo la tiene "conectada".

## Ejemplo (laboratorio oficial)

Un agente de gastos responde "¿Cuánto puedo reclamar por un taxi?" con una cifra plausible pero inventada. Se crea en Foundry IQ una knowledge base `expenses-documentation` con el archivo `expenses_policy.docx` (output mode *answer synthesis*), se concede a la identidad administrada del proyecto el rol **Search Data Index Reader** sobre el recurso de búsqueda, y se conecta al agente. Ahora responde con la cifra real y una **cita** al documento. Ver [[Lab 07 - Foundry IQ (conocimiento para agentes)]].

## Comparaciones

| Opción | Qué es | Cuándo usarla | Concepto clave |
|---|---|---|---|
| **Contexto en el prompt** | Pegar el texto en el prompt | Pocos datos, puntual | Limitado por la ventana de contexto |
| **File search (agente)** | Subir archivos → vector store → herramienta del agente | Un agente, pocos documentos, rápido | `file_search` + `vector_store_ids` |
| **Foundry IQ** | Knowledge bases sobre Azure AI Search, reutilizables | Varios agentes, muchas fuentes, gobierno centralizado | Knowledge source, output mode, citas |
| **Azure AI Search directo** | Servicio de búsqueda/índices (knowledge mining) | Soluciones a medida, búsqueda empresarial | Índice, búsqueda híbrida |
| **Web search** | Buscar en Internet | Información pública y actual | No usa tus datos |
| **Fine-tuning** | Reentrenar con ejemplos | Cambiar estilo/comportamiento | No sirve para datos cambiantes |

## 🧠 Memorizar

> [!important]
> - **RAG = Recuperar → Aumentar el prompt → Generar.**
> - Grounding **reduce alucinaciones** y permite **citas**.
> - **Foundry IQ** = conocimiento centralizado para agentes, basado en **Azure AI Search**.
> - Output modes: **extractive data** vs **answer synthesis**.
> - Datos propios → **RAG**; comportamiento/estilo → **fine-tuning**; actualidad pública → **web search**.
> - El proyecto necesita permiso (**Search Data Index Reader**) sobre el recurso de búsqueda.

## Tips para AI-901

> [!tip]
> - ⭐ "El agente debe responder según nuestros documentos internos" → knowledge base (Foundry IQ) o file search.
> - ⭐ "Varios agentes deben compartir las mismas fuentes sin duplicar lógica" → Foundry IQ.
> - 🔥 "Queremos que la respuesta sea texto literal de la fuente, no reelaborado" → extractive data.
> - 🔥 "Incluir noticias de hoy" → web search, no RAG sobre documentos.
> - ⚠️ RAG no cambia el modelo. Fine-tuning sí.
> - ⚠️ Al crear la knowledge base se eligen **dos modelos**: uno de chat (para sintetizar) y uno de **embeddings** (para indexar).

## ⚠️ Errores comunes

- Responder "fine-tuning" a un escenario de "el modelo no conoce nuestros datos".
- Olvidar la parte de **permisos** (identidad del proyecto sobre Azure AI Search) cuando el agente no encuentra la base.

## 💡 Escenario

Un despacho de abogados quiere un asistente que responda sobre sus 5 000 contratos, con referencias exactas al contrato y cláusula, y que el mismo conocimiento lo usen tres agentes distintos. ¿Qué diseño?

<details><summary>Respuesta</summary>

**Foundry IQ**: una knowledge base con los contratos como fuente (Blob/SharePoint), conectada a los tres agentes. Las citas dan las referencias. File search valdría para un solo agente y pocos archivos; fine-tuning no aporta conocimiento verificable.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué describe mejor el patrón RAG?
- A) Reentrenar el modelo con documentos de la empresa
- B) Recuperar información relevante de una fuente y añadirla al prompt antes de generar la respuesta
- C) Filtrar las respuestas con Content Safety
- D) Reducir la temperature para evitar alucinaciones

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Qué componente de Microsoft Foundry permite crear bases de conocimiento reutilizables por varios agentes?
- A) Foundry Models · B) Foundry IQ · C) Foundry Tools · D) Guardrails

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** En una knowledge base de Foundry IQ, ¿qué output mode devuelve fragmentos literales de las fuentes sin reelaborarlos?
- A) Answer synthesis · B) Extractive data · C) Reasoning · D) Streaming

<details><summary>Respuesta</summary>

**B.**
</details>

**4.** Verdadero o falso: la herramienta de búsqueda web de un agente permite fundamentar respuestas en los documentos internos de la empresa.

<details><summary>Respuesta</summary>

**Falso.** Busca en Internet; para documentos internos se usa file search o Foundry IQ.
</details>

## Relacionado

- [[Generative AI]] · [[Tokens y Embeddings]]
- [[Agentes de IA (Foundry Agent Service)]]
- [[Transparency (Transparencia)]] · [[Azure AI Content Safety y guardrails]] (groundedness)
- [[Lab 02 - Desplegar un modelo y crear un agente]] · [[Lab 07 - Foundry IQ (conocimiento para agentes)]]
- [[Azure Blob Storage]] (fuente de documentos)

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
