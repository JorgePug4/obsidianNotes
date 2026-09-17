---
tags: [ai-901, azure, ia, generative-ai, agentes, foundry]
módulo: Generative AI y agentes
peso_examen: Muy alto
objetivo_oficial: "2.1.4 Create and test a single-agent solution in the Foundry portal · 2.1.5 Create a lightweight client application for an agent"
aliases: [Agente, Agentes, Foundry Agent Service, AI agent, Agentic AI]
---

# Agentes de IA (Foundry Agent Service)

## ¿Qué es?

Un **agente de IA** es una aplicación que combina un **modelo generativo** con **instrucciones**, **herramientas** (*tools*) y **conocimiento** para **realizar tareas** de forma autónoma o semiautónoma, no solo responder. **Foundry Agent Service** es el servicio de Microsoft Foundry que permite **definir, ejecutar, probar y publicar** agentes (antes *Azure AI Agent Service*).

Lo que dice el lab oficial: *"para crear una experiencia realmente agéntica hay que encapsular el modelo, sus instrucciones y su configuración de herramientas en un agente; así los clientes se conectan a su endpoint sin tener que especificar el prompt de sistema ni implementar su propia lógica RAG"*.

## ¿Para qué sirve?

Asistentes que consultan sistemas (pedidos, CRM), buscan en la web o en documentos, ejecutan código, llaman APIs y encadenan pasos. En AI-901 debes saber **crear y probar un agente en el portal** y **reconocer el código de un cliente** que lo consume.

## Componentes de un agente

| Componente | Qué es | En el portal |
|---|---|---|
| **Modelo** | El LLM desplegado que razona | Lista desplegable de modelos |
| **Instrucciones** | Prompt de sistema: rol, reglas, tono | Cuadro *Instructions* |
| **Herramientas (tools)** | Capacidades externas que el agente decide usar | Sección *Tools* → Add |
| **Conocimiento (knowledge)** | Fuentes para grounding: file search, Foundry IQ | *Tools* (file search) o *Knowledge* |
| **Parámetros** | temperature, top_p, etc. | Configuración |
| **Guardrails** | Filtros y controles de seguridad | Build → Guardrails |
| **Voice mode** | Voice Live para hablar con el agente | Interruptor *Voice mode* |
| **Versión** | Cada guardado crea una versión (`nombre:1`, `:2`) | YAML |

## Herramientas que debes reconocer

| Herramienta | Qué permite | Ejemplo |
|---|---|---|
| **Web search** (grounding con Bing) | Información pública actual de Internet | "Busca una tienda de informática vintage cerca de Seattle" |
| **File search** | Buscar en archivos subidos (vector store) | Identificar placas por número de serie |
| **Foundry IQ / knowledge base** | Bases de conocimiento centralizadas (Azure AI Search) | Política de gastos |
| **Code interpreter** | Ejecutar código Python en sandbox para cálculos, gráficos, archivos | Analizar un CSV |
| **Function calling / OpenAPI** | Llamar a funciones o APIs propias | Consultar el estado de un pedido |
| **MCP tools / conectores** (SharePoint, Fabric IQ, Azure Logic Apps…) | Integrar sistemas empresariales | Leer documentos de SharePoint |
| **Image generation** | Generar imágenes desde el agente | Crear una ilustración |

## Cómo funciona (tiempo de ejecución)

```
Usuario ─▶ mensaje ─▶ Agente (modelo + instrucciones)
                        │ decide: ¿necesito una herramienta?
                        ├─ sí ─▶ llama a la herramienta ─▶ resultado ─▶ vuelve a razonar
                        └─ no ─▶ responde
Conversación (thread) guarda el estado multi-turno; cada respuesta es un "response".
```

Componentes de runtime del servicio: **agents**, **conversations** (hilos con estado) y **responses**.

## Crear y probar un agente en el portal (lab oficial)

1. **Home → Build an agent** (o **Build → Agents**), nombre p. ej. `computing-historian`.
2. Elegir el **modelo** desplegado, escribir **Instructions**, **Save**.
3. Añadir **Tools**: *Web search*, subir un archivo para *File search* (crea un índice/vector store), o conectar una **knowledge base** de Foundry IQ.
4. **Probar** en el playground del agente: chatear, comprobar que rechaza temas fuera de alcance, ver **citas** y la **traza** (qué herramientas llamó).
5. Alternativa: chatear con el modelo en su playground y pulsar **Save as agent**.
6. Ver la definición en la pestaña **YAML** (`kind: prompt`, `model`, `instructions`, `tools: web_search / file_search + vector_store_ids`).
7. **Publish → Preview web app** para probarlo en una web básica.
8. **Continue in code** para obtener el cliente.

## Cliente ligero para un agente (lab oficial)

```python
# pip install azure-ai-projects>=2.1.0
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

project_client = AIProjectClient(
    endpoint="https://ai-resrce.services.ai.azure.com/api/projects/ai-project",
    credential=DefaultAzureCredential(),
)
openai_client = project_client.get_openai_client()

response = openai_client.responses.create(
    input=[{"role": "user", "content": "Tell me what you can help with."}],
    extra_body={"agent_reference": {"name": "computing-historian", "version": "1", "type": "agent_reference"}},
)
print(response.output_text)
```

Claves para leerlo: se conecta al **proyecto** con **Entra ID**, obtiene un cliente OpenAI y usa la **Responses API** indicando el **agente y su versión** en `extra_body["agent_reference"]`. No hace falta pasar instrucciones ni herramientas: ya están en el agente. Para mantener una conversación multi-turno se usan **conversations** (hilos) o `previous_response_id`.

## Ejemplo

Un agente de gastos: instrucciones "asesoras sobre políticas de gastos", modelo `gpt-5-mini`, knowledge base de Foundry IQ con la política, herramienta OpenAPI para consultar el saldo del empleado. El cliente web envía "¿Cuánto puedo reclamar por un taxi?" y recibe la cifra con cita.

## 📌 Diferencias clave

| | Modelo desplegado | Agente |
|---|---|---|
| Qué expone | Un endpoint de inferencia | Un endpoint con **modelo + instrucciones + herramientas + conocimiento** |
| Quién gestiona el prompt de sistema | El cliente en cada llamada | El agente (una vez) |
| Herramientas | No (salvo que el cliente las implemente) | Sí, orquestadas por el servicio |
| Estado de conversación | El cliente reenvía historial | **Conversations** gestionadas por el servicio |
| Código cliente | `responses.create(model=…, instructions=…)` | `responses.create(extra_body={"agent_reference": …})` |

| Agente único (single-agent) | Multiagente / workflows |
|---|---|
| Un agente con sus herramientas: **lo que evalúa AI-901** | Varios agentes que colaboran, orquestación (🟡 contexto) |

## 🧠 Memorizar

> [!important]
> - Agente = **modelo + instrucciones + herramientas + conocimiento** (+ guardrails, versión).
> - Herramientas: **web search, file search, Foundry IQ, code interpreter, function/OpenAPI, MCP**.
> - Portal: **Build → Agents** o **Save as agent**; probar en el playground; **Preview web app**; **Continue in code**.
> - Cliente: `AIProjectClient` + `DefaultAzureCredential` + `get_openai_client()` + `responses.create(... agent_reference)`.
> - Runtime: **agents · conversations · responses**.

## Tips para AI-901

> [!tip]
> - ⭐ "Los clientes no deben gestionar el prompt de sistema ni la lógica RAG" → encapsular en un **agente**.
> - ⭐ "Información actual de Internet" → **web search**; "documentos subidos" → **file search**; "conocimiento compartido entre agentes" → **Foundry IQ**.
> - 🔥 "Ejecutar cálculos o generar gráficos a partir de datos" → **code interpreter**.
> - 🔥 "Consultar nuestra API de pedidos" → **function calling / OpenAPI tool**.
> - 🔥 "Hablar con el agente por voz" → **Voice mode (Voice Live)**.
> - ⚠️ El cliente de agente usa el **project endpoint** y **Entra ID**, no la clave.
> - ⚠️ Cambiar las instrucciones o herramientas crea una **nueva versión**; el cliente referencia una versión concreta.

## ⚠️ Errores comunes

- Confundir "chat con un modelo con instrucciones" con "agente": el agente es el activo guardado y versionado con endpoint propio.
- Pensar que el agente ejecuta herramientas "siempre": el modelo **decide** cuándo usarlas según el prompt.

## 💡 Escenario

Una empresa de logística quiere un asistente en Teams que, ante "¿dónde está mi pedido 123?", consulte su API de seguimiento y responda; además debe conocer las condiciones de envío publicadas en su intranet. ¿Cómo lo diseñas en Foundry?

<details><summary>Respuesta</summary>

Un **agente** en Foundry Agent Service: modelo de chat, instrucciones con el rol y el alcance, una herramienta **OpenAPI/function** para la API de seguimiento y una **knowledge base** (Foundry IQ o file search) con las condiciones de envío. Se consume desde el cliente de Teams mediante el SDK con `agent_reference`.
</details>

## Preguntas que podrían aparecer

**1.** ¿Cuál de los siguientes NO es un componente típico de un agente en Foundry?
- A) Modelo · B) Instrucciones · C) Herramientas · D) Dataset de entrenamiento etiquetado

<details><summary>Respuesta</summary>

**D.** Los agentes usan modelos preentrenados; no se entrenan con datasets etiquetados.
</details>

**2.** Un agente debe responder con información de un catálogo en PDF que el equipo sube al portal. ¿Qué herramienta añades?
- A) Web search · B) File search · C) Code interpreter · D) Image generation

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** En el código de un cliente de agente aparece `extra_body={"agent_reference": {"name": "expenses-agent", "version": "2", ...}}`. ¿Qué indica?
- A) El nombre del modelo a desplegar
- B) El agente y la versión concreta a la que se envía la petición
- C) El prompt de sistema
- D) El índice de búsqueda

<details><summary>Respuesta</summary>

**B.**
</details>

**4.** ¿Cómo puedes probar rápidamente un agente en una interfaz web sin escribir código?
- A) Azure portal → Monitor · B) Publish → Preview web app · C) Content Understanding Studio · D) Azure Machine Learning designer

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Generative AI]] · [[Prompts y Prompt Engineering]]
- [[Grounding, RAG y Foundry IQ]] · [[Foundry SDK y cliente de chat]]
- [[Azure AI Speech]] (Voice Live) · [[Azure AI Content Safety y guardrails]]
- [[Lab 02 - Desplegar un modelo y crear un agente]] · [[Lab 04 - Agente de voz con Voice Live]] · [[Lab 07 - Foundry IQ (conocimiento para agentes)]]
- [[Copilot y asistentes de Microsoft]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
