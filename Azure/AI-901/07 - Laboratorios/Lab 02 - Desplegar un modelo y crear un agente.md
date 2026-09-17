---
tags: [ai-901, azure, ia, laboratorio, foundry, agentes]
módulo: Laboratorios
duración: ~35 min
---

# Lab 02 · Desplegar un modelo y crear un agente

## Objetivo

Recorrer el flujo completo del dominio 2: **desplegar** un modelo, **chatear** con él, escribir **instrucciones**, añadir **herramientas** y **conocimiento**, **guardarlo como agente** y obtener el **código cliente**. Es el laboratorio más importante del examen.

## Pasos

1. **Desplegar el modelo.** En **Discover → Models**, busca `gpt-5-mini`, revisa su tarjeta y pulsa **Deploy** con la configuración por defecto. Al terminar se abre el **playground**.
2. **Chatear.** Pregunta `Who was Ada Lovelace?` y luego `Tell me more about her work with Charles Babbage.` Observa que el modelo entiende "her" porque el **historial** se envía con cada prompt. Pulsa **New chat** y prueba `List three facts about Ada Lovelace.`: el prompt cambia el estilo de la respuesta.
3. **Instrucciones (prompt de sistema).** En el panel izquierdo escribe:
   ```
   You are an expert in the history of computing and AI. You only answer questions about significant people and events in the development of computing, and about notable vintage computers. Do not engage in conversations on any topic that is unrelated to computing history.
   ```
   Pregunta `Tell me about ELIZA.` y después algo fuera de alcance, como `What's the capital of Spain?`: el modelo debe rehusar.
4. **Herramienta de búsqueda web.** En **Tools → Add**, activa **Web search**. Reinicia el chat y pide `Find a vintage computer store near Seattle`.
5. **Conocimiento (file search).** Descarga el documento de ejemplo `vintage_computer_identifiers.docx`, súbelo en **Tools** creando un índice y pregunta `I have a printed circuit board with the "ASSY 250425" on it. What can you tell me about it?` La respuesta ahora se basa en el archivo.
6. **Guardar como agente.** Pulsa **Save as agent** y llámalo `computing-historian`. Revisa la pestaña **YAML**: verás `kind: prompt`, el `model`, las `instructions` y las `tools` (`web_search` y `file_search` con su `vector_store_ids`).
7. **Probar y publicar.** Pregunta `Who are you?` para comprobar que el agente conoce su rol. En **Publish → Preview web app** ábrelo en una web básica.
8. **Ver el código.** Pulsa **</> Continue in code** y estudia el ejemplo con `AIProjectClient`, `DefaultAzureCredential`, `get_openai_client()` y `agent_reference`.
9. **Limpieza.** Elimina el grupo de recursos.

## Qué está ocurriendo

Al principio el modelo responde solo con lo que "recuerda" del entrenamiento; puede inventar. Las **instrucciones** acotan su comportamiento, las **herramientas** le dan acceso a información externa y el **file search** implementa RAG sin que tú escribas la lógica. Al guardarlo como **agente**, todo eso queda encapsulado y versionado tras un endpoint: el cliente ya no necesita enviar el prompt de sistema ni implementar la recuperación.

## Qué debo aprender para AI-901

- Objetivo **2.1.2**: desplegar un modelo e interactuar con él en el portal.
- Objetivo **2.1.1**: diferencia entre prompt de sistema (Instructions) y prompt de usuario.
- Objetivo **2.1.4**: crear y probar un agente con herramientas y conocimiento.
- Objetivo **2.1.5**: reconocer el código del cliente de agente.
- Que el historial de conversación se reenvía en cada turno.

## Relacionado

- [[Despliegue y configuración de modelos]] · [[Prompts y Prompt Engineering]]
- [[Agentes de IA (Foundry Agent Service)]] · [[Grounding, RAG y Foundry IQ]]
- [[Lab 07 - Foundry IQ (conocimiento para agentes)]] · [[Lab 08 - Cliente Python con el Foundry SDK]]

← Volver al índice: [[00 - Índice - Laboratorios]]
