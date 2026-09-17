---
tags: [ai-901, azure, ia, generative-ai, foundry, sdk, python]
módulo: Generative AI y agentes
peso_examen: Muy alto
objetivo_oficial: "2.1.3 Create a lightweight chat client application by using the Foundry SDK"
aliases: [Foundry SDK, OpenAI SDK, Responses API, AIProjectClient, Cliente de chat]
---

# Foundry SDK y cliente de chat

## ¿Qué es?

El **Foundry SDK** es el conjunto de bibliotecas para consumir desde código lo que has creado en Microsoft Foundry. En Python, las dos piezas que el examen espera que reconozcas son:

- **OpenAI SDK** (`openai`): para llamar a un **modelo desplegado** usando su endpoint compatible con OpenAI. Usa las APIs **Responses** (la actual y recomendada) o **Chat Completions** (más antigua, todavía común).
- **Azure AI Projects** (`azure-ai-projects`): la clase **`AIProjectClient`** se conecta a un **proyecto** de Foundry (endpoint del proyecto + identidad de Entra) y desde ahí obtiene un cliente OpenAI (`get_openai_client()`), lista despliegues, accede a agentes, conexiones y evaluaciones.

AI-901 no pide programar de memoria, pero sí **leer un fragmento de Python y saber qué hace o qué falta**.

## Autenticación: dos caminos

| Camino | Endpoint | Credencial | Cuándo |
|---|---|---|---|
| **Modelo directo** con OpenAI SDK | *Azure OpenAI endpoint*: `https://<recurso>.openai.azure.com/openai/v1/` | **Clave de API** (`api_key`) o token de Entra | Cliente de chat sencillo |
| **Proyecto** con `AIProjectClient` | *Project endpoint*: `https://<recurso>.services.ai.azure.com/api/projects/<proyecto>` | **Solo Entra ID** (`DefaultAzureCredential`); la clave **no** está soportada | Agentes, activos del proyecto |

`DefaultAzureCredential` (paquete `azure-identity`) usa automáticamente la identidad disponible: `az login` en desarrollo, identidad administrada en producción. Es la práctica recomendada ("keyless").

## Cliente de chat mínimo (Responses API)

```python
from openai import OpenAI

endpoint = "https://your-project-resource.openai.azure.com/openai/v1/"
deployment_name = "gpt-5-mini"          # nombre del DESPLIEGUE
api_key = "<your-api-key>"

client = OpenAI(base_url=endpoint, api_key=api_key)

response = client.responses.create(
    model=deployment_name,
    instructions="You are an expert in the history of computing. Be concise.",  # prompt de sistema
    input="Tell me about the Commodore 64",                                     # prompt de usuario
)
print(response.output_text)
```

Con historial (multi-turno), `input` es una lista de mensajes con `role` (`user`, `assistant`) o se encadena con `previous_response_id`.

## Cliente de chat vía proyecto (Foundry SDK)

```python
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

project_client = AIProjectClient(
    endpoint="https://ai-resrce.services.ai.azure.com/api/projects/ai-project",
    credential=DefaultAzureCredential(),
)
openai_client = project_client.get_openai_client()

response = openai_client.responses.create(
    model="gpt-5-mini",
    input=[{"role": "user", "content": "Who was Ada Lovelace?"}],
)
print(response.output_text)
```

## Chat Completions (API antigua, aún frecuente)

```python
response = client.chat.completions.create(
    model=deployment_name,
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello"},
    ],
)
print(response.choices[0].message.content)
```

## Conceptos clave para leer código

- **`base_url` / `endpoint`**: a dónde se conecta.
- **`model=`**: el **nombre del despliegue**.
- **`instructions=`** (Responses) o mensaje `role: "system"` (Chat Completions): prompt de sistema.
- **`input=`** (Responses) o `messages=` (Chat Completions): prompt de usuario / historial.
- **`response.output_text`** (Responses) vs `response.choices[0].message.content` (Chat Completions).
- **Streaming**: recibir la respuesta token a token para mejorar la percepción de velocidad.
- **Asincronía**: clientes `Async*` para no bloquear la app.
- Entrada **multimodal**: `input` con `{"type": "input_text"}` y `{"type": "input_image", "image_url": …}`. Ver [[Modelos multimodales (visión en prompts)]].
- Variables de entorno / `.env` para no incrustar claves en el código.

## Ejemplo

El lab oficial *Model Coder* y el playground de Foundry ("**Call model / View code**") generan exactamente este código con el endpoint y la clave del proyecto para que lo pegues en VS Code.

## 📌 Diferencias clave

| | OpenAI SDK directo | `AIProjectClient` |
|---|---|---|
| Paquete | `openai` | `azure-ai-projects` (+ `azure-identity`) |
| Endpoint | Azure OpenAI endpoint (`/openai/v1/`) | Project endpoint (`/api/projects/…`) |
| Autenticación | Clave o Entra | **Solo Entra** |
| Alcance | Un modelo desplegado | Todo el proyecto: modelos, **agentes**, conexiones |
| Uso típico | Cliente de chat ligero | Cliente de agente, apps que usan varios activos |

| Responses API | Chat Completions API |
|---|---|
| Actual, recomendada; sirve para modelos **y agentes** | Anterior, muy extendida |
| `instructions` + `input` → `output_text` | `messages` (system/user) → `choices[0].message.content` |

## 🧠 Memorizar

> [!important]
> - Cliente de chat = **OpenAI SDK** + endpoint `/openai/v1/` + **deployment name**.
> - **Responses API**: `instructions`, `input`, `output_text`.
> - **`AIProjectClient`** + **`DefaultAzureCredential`** + `get_openai_client()` para trabajar vía proyecto y con agentes.
> - Claves fuera del código (`.env`, Key Vault); mejor **sin claves** (Entra ID).

## Tips para AI-901

> [!tip]
> - ⭐ Si un fragmento muestra `client.responses.create(model=..., instructions=..., input=...)`, es un cliente de chat con la Responses API; `instructions` es el prompt de sistema.
> - ⭐ Si muestra `AIProjectClient(endpoint=..., credential=DefaultAzureCredential())`, se conecta a un **proyecto** con identidad de Entra.
> - 🔥 Pregunta típica de "qué falta": el **deployment name** en `model=` o la credencial.
> - ⚠️ `model=` recibe el nombre del **despliegue**, que puede no coincidir con el nombre del modelo.
> - ⚠️ El historial no es automático: el cliente reenvía los mensajes anteriores (o usa `previous_response_id`).

## ⚠️ Errores comunes

- Usar el endpoint del proyecto con el OpenAI SDK directo, o la clave con `AIProjectClient`.
- Incrustar claves en el código fuente.

## 💡 Escenario

Un desarrollador necesita una app de consola en Python que envíe preguntas a `gpt-5-mini` desplegado en Foundry, con un rol fijo de "asistente de historia de la informática". ¿Qué usa?

<details><summary>Respuesta</summary>

**OpenAI SDK** con `OpenAI(base_url=<Azure OpenAI endpoint>, api_key=…)` y `client.responses.create(model="gpt-5-mini", instructions="…", input=pregunta)`. Para producción, sustituir la clave por `DefaultAzureCredential` o usar `AIProjectClient.get_openai_client()`.
</details>

## Preguntas que podrían aparecer

**1.** En el siguiente código, ¿qué representa el parámetro `instructions`?
```python
response = client.responses.create(model="gpt-5-mini", instructions="You only answer about Azure.", input=question)
```
- A) El prompt de usuario · B) El prompt de sistema · C) El nombre del despliegue · D) El historial

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Qué clase del Foundry SDK se usa para conectarse a un proyecto y obtener un cliente OpenAI autenticado con Entra ID?
- A) `TextAnalyticsClient` · B) `AIProjectClient` · C) `ContentUnderstandingClient` · D) `SpeechConfig`

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** Verdadero o falso: con `AIProjectClient` puedes autenticarte con la clave de API del proyecto.

<details><summary>Respuesta</summary>

**Falso.** Requiere identidad de Microsoft Entra (`DefaultAzureCredential`).
</details>

## Relacionado

- [[Prompts y Prompt Engineering]] · [[Despliegue y configuración de modelos]]
- [[Agentes de IA (Foundry Agent Service)]] (cliente de agente)
- [[Modelos multimodales (visión en prompts)]] · [[Generación de imágenes y vídeo]] (código de imagen)
- [[Lab 08 - Cliente Python con el Foundry SDK]]
- [[Microsoft Entra ID]] · [[Azure Key Vault]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
