---
tags: [ai-901, azure, ia, laboratorio, rag, agentes, foundry]
módulo: Laboratorios
duración: ~20 min
---

# Lab 07 · Foundry IQ (conocimiento para agentes)

## Objetivo

Comprobar el efecto del **grounding**: un agente que inventa una respuesta pasa a responder con datos reales y **citas** después de conectarle una **knowledge base** de Foundry IQ.

## Pasos

1. **Crear el agente.** En **Home → Build an agent → Start building** (o **Build → Agents**), crea `expenses-agent`, elige un modelo desplegado y escribe las **Instructions**:
   ```
   You are an AI agent that advises employees on expenses policies and expense claim processes.
   ```
   Guarda y prueba con `What can you help me with?`.
2. **Ver el problema.** Pregunta `How much can I claim for a taxi?`. La respuesta parecerá correcta, pero **no está fundamentada**: el agente no conoce la política de la empresa.
3. **Descargar el documento.** Descarga el archivo de ejemplo `expenses_policy.docx` del laboratorio.
4. **Crear el recurso de Foundry IQ.** En el panel izquierdo, abre **Knowledge** y usa **Create a new resource** para crear un recurso de Foundry IQ (Azure AI Search) en el mismo grupo de recursos; nivel de precios **Basic**.
5. **Crear la knowledge base.** Pulsa **Create a knowledge base** y configura:
   - **Name**: `expenses-documentation`
   - **Chat completions model**: el modelo desplegado
   - **Retrieval reasoning effort**: Low
   - **Output mode**: *Answer synthesis*
   - **Answer instructions**: `Answer concisely, based on the available context`
   - **Retrieval instructions**: indica cuándo usar esta fuente
   En **Add knowledge sources**, sube `expenses_policy.docx` con el modelo de embeddings por defecto. Guarda.
6. **Permisos.** En el portal de Azure, abre el recurso de búsqueda de Foundry IQ → **Access control (IAM)** → **Add role assignment** → rol **Search Data Index Reader** → miembro **Managed identity** → la identidad de tu **proyecto** de Foundry. Completa la asignación.
7. **Conectar y probar.** Vuelve a la knowledge base y, en **Use in an agent**, selecciona `expenses-agent`. Repite la pregunta `How much can I claim for a taxi?`: ahora la respuesta incluye la cifra real y una **cita** al documento.
8. **Limpieza.** Elimina el grupo de recursos.

## Qué está ocurriendo

Foundry IQ centraliza el acceso al conocimiento: trocea e indexa el documento con un modelo de **embeddings** y, en cada consulta, recupera los fragmentos relevantes y los pasa al modelo. Con *answer synthesis*, un modelo compone la respuesta a partir de esas fuentes; con *extractive data* se devolvería el texto literal. Como el conocimiento vive fuera del agente, **varios agentes pueden reutilizar la misma base** sin duplicar lógica ni datos, que es la ventaja frente a implementar RAG a mano.

El paso de **permisos** es parte del aprendizaje: sin el rol *Search Data Index Reader*, la identidad administrada del proyecto no puede leer el índice y el agente no encontrará el conocimiento.

## Qué debo aprender para AI-901

- Qué es el **grounding** y cómo reduce alucinaciones (objetivo 1.1.2, fiabilidad).
- Qué es **RAG** y qué aporta **Foundry IQ** frente a implementarlo a mano.
- La diferencia entre **answer synthesis** y **extractive data**.
- Que las respuestas fundamentadas incluyen **citas** (transparencia).
- Que el proyecto necesita **identidad y permisos** sobre el recurso de búsqueda.

## Relacionado

- [[Grounding, RAG y Foundry IQ]] · [[Agentes de IA (Foundry Agent Service)]]
- [[Reliability and Safety (Fiabilidad y seguridad)]] · [[Transparency (Transparencia)]]
- [[Microsoft Entra ID]] · [[Azure RBAC]]

← Volver al índice: [[00 - Índice - Laboratorios]]
