---
tags: [ai-901, azure, ia, laboratorio, sdk, python, foundry]
módulo: Laboratorios
duración: ~30 min
---

# Lab 08 · Cliente Python con el Foundry SDK

## Objetivo

Escribir y entender los **dos clientes** que evalúa el examen: una aplicación ligera de **chat** contra un modelo desplegado y una aplicación ligera contra un **agente**.

## Requisitos

Python 3.12, Azure CLI con `az login` hecho, y un proyecto de Foundry con un modelo desplegado (labs 01 y 02).

```bash
python -m venv .venv && source .venv/bin/activate
pip install openai azure-ai-projects azure-identity python-dotenv
```

Guarda los endpoints en un `.env` (nunca en el código):

```
AZURE_OPENAI_ENDPOINT=https://<recurso>.openai.azure.com/openai/v1/
PROJECT_ENDPOINT=https://<recurso>.services.ai.azure.com/api/projects/<proyecto>
MODEL_DEPLOYMENT=gpt-5-mini
```

## Pasos

### Parte 1: cliente de chat (objetivo 2.1.3)

1. Copia el código de ejemplo desde el playground del modelo (**Call model / View code**, Python).
2. Crea `chat_client.py`:

```python
import os
from openai import OpenAI

client = OpenAI(base_url=os.environ["AZURE_OPENAI_ENDPOINT"], api_key=os.environ["AZURE_OPENAI_KEY"])

while True:
    prompt = input('\nPregunta (o "quit"): ')
    if prompt.lower() == "quit":
        break
    response = client.responses.create(
        model=os.environ["MODEL_DEPLOYMENT"],
        instructions="You are an expert in the history of computing. Be concise.",
        input=prompt,
    )
    print(response.output_text)
```

3. Ejecútalo y comprueba que `instructions` cambia el comportamiento en todas las respuestas.

### Parte 2: cliente de agente (objetivo 2.1.5)

4. Copia el código desde el agente (**Continue in code**).
5. Crea `agent_client.py`:

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

project_client = AIProjectClient(
    endpoint=os.environ["PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project_client.get_openai_client()

response = openai_client.responses.create(
    input=[{"role": "user", "content": "Tell me what you can help with."}],
    extra_body={"agent_reference": {"name": "computing-historian", "version": "1", "type": "agent_reference"}},
)
print(response.output_text)
```

6. Ejecuta `az login` y después el script. Observa que **no envías instrucciones ni herramientas**: ya están en el agente.
7. **Experimenta.** Cambia el `agent_reference` a otra versión del agente, prueba a enviar varios turnos y compara las respuestas del cliente de chat y del cliente de agente ante la misma pregunta.

## Qué está ocurriendo

Son dos puertas distintas al mismo proyecto. El **cliente de chat** habla directamente con un despliegue de modelo por el endpoint compatible con OpenAI, y tú aportas el prompt de sistema en cada llamada. El **cliente de agente** se conecta al **proyecto** con identidad de Microsoft Entra, obtiene un cliente OpenAI ya autenticado y referencia un agente por **nombre y versión**; el servicio se encarga de las instrucciones, las herramientas y la recuperación de conocimiento.

`DefaultAzureCredential` usa tu sesión de `az login` en desarrollo y una identidad administrada en producción: por eso el patrón recomendado es **sin claves**.

## Qué debo aprender para AI-901

- Objetivo **2.1.3**: cliente de chat ligero con el Foundry SDK.
- Objetivo **2.1.5**: cliente ligero para un agente.
- Distinguir `instructions` / `input` / `output_text` de la **Responses API**.
- Que `model=` recibe el **nombre del despliegue**.
- Que `AIProjectClient` exige **Entra ID** y que el agente se referencia con `agent_reference`.
- Que las claves nunca van en el código.

## Relacionado

- [[Foundry SDK y cliente de chat]] · [[Agentes de IA (Foundry Agent Service)]]
- [[Prompts y Prompt Engineering]] · [[Despliegue y configuración de modelos]]
- [[Microsoft Entra ID]] · [[Azure Key Vault]]

← Volver al índice: [[00 - Índice - Laboratorios]]
