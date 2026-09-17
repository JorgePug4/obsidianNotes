---
tags: [ai-901, azure, ia, foundry, plataforma]
módulo: Fundamentos de IA
peso_examen: Muy alto
aliases: [Foundry, Azure AI Foundry, Azure AI Studio, Foundry portal]
---

# Microsoft Foundry

## ¿Qué es?

**Microsoft Foundry** (antes *Azure AI Foundry*, y antes *Azure AI Studio*) es la **plataforma unificada de Azure para crear, desplegar y operar aplicaciones y agentes de IA**. Reúne en un solo lugar el catálogo de modelos, los agentes, el conocimiento, las evaluaciones, la seguridad y los servicios preconstruidos.

El renombrado a *Microsoft Foundry* se anunció en Ignite (noviembre de 2025) y es efectivo desde 2026. En documentación y preguntas verás los tres nombres; son la misma cosa.

## ¿Para qué sirve?

Es el sitio donde ocurre **todo el dominio 2 del examen (55–60 %)**: desplegar un modelo, chatear con él en el *playground*, escribir prompts de sistema, crear un agente, conectarle conocimiento y herramientas, probar servicios de Foundry Tools y obtener código de ejemplo para tu aplicación.

## Conceptos clave

- **Recurso de Microsoft Foundry** (*Foundry resource*): el recurso de Azure "padre" que se crea en una suscripción/grupo de recursos/región. Proporciona los servicios en la nube.
- **Proyecto** (*project*): contenedor **hijo** del recurso que organiza modelos desplegados, agentes, datos, conexiones y evaluaciones de una solución. Un recurso puede tener varios proyectos.
- **Endpoints y claves del proyecto**: en la página Home ves el *API key*, el *Project endpoint* (`https://<recurso>.services.ai.azure.com/api/projects/<proyecto>`) y el *Azure OpenAI endpoint* (`https://<recurso>.openai.azure.com/openai/v1/`).
- **Portal de Foundry**: `https://ai.azure.com`.
- **Foundry Models**: el catálogo (modelos de OpenAI, Microsoft, Meta, Mistral, DeepSeek, xAI, Anthropic, Black Forest Labs…). Ver [[Catálogo de modelos de Foundry]].
- **Foundry Agent Service**: crear y ejecutar agentes. Ver [[Agentes de IA (Foundry Agent Service)]].
- **Foundry Tools**: servicios de IA preconstruidos (Language, Speech, Vision, Translator, Content Understanding…). Ver [[Foundry Tools (servicios de IA de Azure)]].
- **Foundry IQ**: capa de conocimiento (basada en Azure AI Search) para conectar agentes a datos. Ver [[Grounding, RAG y Foundry IQ]].
- **Guardrails / Content Safety**: filtros de contenido y controles de seguridad. Ver [[Azure AI Content Safety y guardrails]].
- **Foundry SDK**: bibliotecas (`azure-ai-projects`, OpenAI SDK) para consumir modelos y agentes desde código. Ver [[Foundry SDK y cliente de chat]].
- **Foundry Local**: ejecutar modelos en el dispositivo (🟡 contexto).

## Cómo funciona: el portal

| Página | Qué encuentras |
|---|---|
| **Home** | Endpoints y clave del proyecto, accesos rápidos ("Build an agent") |
| **Discover** | Catálogo de **modelos** y servicios, puntos de partida |
| **Build** | Donde desarrollas: **Agents**, **Workflows**, **Models** (deployments), **Fine-tuning**, **Tools**, **Knowledge** (Foundry IQ), **Guardrails**, **Memory**, **Data**, **Evaluations**, **Services** (playgrounds de Foundry Tools) |
| **Operate** | Assets, cumplimiento, **quota**, administración |
| **Docs** | Documentación |

Flujo típico que reproducen los laboratorios oficiales:

```
Crear proyecto ─▶ Discover: elegir modelo ─▶ Deploy ─▶ Playground (chat)
      │                                                     │
      │                                    Instructions (system prompt) + Tools + Knowledge
      │                                                     ▼
      └──────────────────────────────────────────▶ Save as agent ─▶ Preview web app / Continue in code
```

## Ejemplo

Una empresa quiere un asistente interno de políticas de gastos: crea un proyecto en Foundry, despliega `gpt-5-mini`, escribe las instrucciones, crea una *knowledge base* en Foundry IQ con el PDF de la política, la conecta al agente y consume el agente desde una app Python con `AIProjectClient`. Ver [[Lab 07 - Foundry IQ (conocimiento para agentes)]].

## Comparaciones

| Servicio | Qué es | Cuándo se usa | Concepto clave |
|---|---|---|---|
| **Microsoft Foundry** | Plataforma de apps y agentes de IA | Casi siempre en AI-901 | Proyecto + modelo + agente |
| **Azure Machine Learning** | Entrenar modelos propios | Datos históricos, ML clásico | AutoML, designer |
| **Microsoft Copilot Studio** | Crear copilots/agentes low-code para M365 y Teams | Creadores de negocio, sin código | Temas, conectores |
| **Azure OpenAI (nombre antiguo)** | Los modelos de OpenAI dentro de Foundry Models | Es parte de Foundry | "sold directly by Azure" |

## 🧠 Memorizar

> [!important]
> - Foundry = **recurso** (padre) + **proyectos** (hijos).
> - Portal: `ai.azure.com` → **Discover / Build / Operate**.
> - Los cinco pilares: **Foundry Models · Foundry Agent Service · Foundry Tools · Foundry IQ · control plane (gobierno, guardrails, quota)**.
> - Para conectarse a un **proyecto** desde código se usa **identidad de Entra ID** (`DefaultAzureCredential`); la autenticación con clave **no** está soportada para el cliente de proyecto.
> - Nombres antiguos: Azure AI Studio → Azure AI Foundry → **Microsoft Foundry**.

## Tips para AI-901

> [!tip]
> - ⭐ Si la pregunta muestra una captura con "Discover / Build / Operate", es el portal de Foundry.
> - ⭐ "¿Dónde despliego un modelo y lo pruebo sin código?" → catálogo de modelos + **playground** de Foundry.
> - 🔥 "Necesito que varios agentes compartan las mismas fuentes de conocimiento" → **Foundry IQ**.
> - 🔥 "Quiero un resultado determinista de detección de PII / OCR / transcripción" → **Foundry Tools** (Build → Services).
> - ⚠️ Un **proyecto** no es una suscripción ni un grupo de recursos: es una unidad lógica dentro del recurso de Foundry.

## ⚠️ Errores comunes

- Confundir el *Azure OpenAI endpoint* (para llamar a modelos con el OpenAI SDK) con el *Project endpoint* (para `AIProjectClient` y agentes).
- Pensar que Foundry sirve para entrenar modelos desde cero (eso es Azure ML). Foundry permite *fine-tuning* de modelos base, que es otra cosa.

## 💡 Escenario

Un desarrollador debe probar varios modelos de chat, elegir uno, definir un prompt de sistema, añadir búsqueda web y exponer el resultado como agente que consumirá una app. ¿Qué plataforma y qué flujo?

<details><summary>Respuesta</summary>

**Microsoft Foundry**: Discover → desplegar modelo → playground con *Instructions* → añadir herramienta *web_search* → *Save as agent* → obtener código con *Continue in code* (Foundry SDK).
</details>

## Preguntas que podrían aparecer

**1.** ¿Cuál es la relación entre un recurso de Microsoft Foundry y un proyecto?
- A) Son sinónimos
- B) El proyecto es el recurso padre y contiene varios recursos de Foundry
- C) El recurso de Foundry es el padre y puede contener varios proyectos
- D) Un proyecto es un grupo de recursos de Azure

<details><summary>Respuesta</summary>

**C.** El recurso aporta los servicios; los proyectos organizan los activos de cada solución.
</details>

**2.** ¿En qué página del portal de Foundry gestionas los agentes, los despliegues de modelos, el conocimiento y los guardrails?
- A) Home · B) Discover · C) Build · D) Operate

<details><summary>Respuesta</summary>

**C.** Build es la página de desarrollo. Discover es el catálogo; Operate es administración y cuota.
</details>

**3.** Verdadero o falso: para conectar una aplicación a un proyecto de Foundry con `AIProjectClient` puedes usar la clave de API del proyecto.

<details><summary>Respuesta</summary>

**Falso.** El cliente de proyecto requiere autenticación con Microsoft Entra ID (por ejemplo `DefaultAzureCredential`). La clave sí sirve para llamar directamente al endpoint de OpenAI de un modelo desplegado.
</details>

## Relacionado

- [[Foundry Tools (servicios de IA de Azure)]]
- [[Catálogo de modelos de Foundry]] · [[Despliegue y configuración de modelos]]
- [[Agentes de IA (Foundry Agent Service)]] · [[Grounding, RAG y Foundry IQ]]
- [[Foundry SDK y cliente de chat]]
- [[Lab 01 - Explorar Microsoft Foundry]]
- [[Microsoft Entra ID]] (identidad para `DefaultAzureCredential`)

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
