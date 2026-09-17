---
tags: [ai-901, azure, ia, generative-ai, copilot]
módulo: Generative AI y agentes
peso_examen: Bajo (contexto)
aliases: [Copilot, Microsoft Copilot, Copilot Studio]
---

# Copilot y asistentes de Microsoft

> [!info] 🟡 Contexto
> AI-900 preguntaba por "copilots". La guía de AI-901 no los nombra explícitamente, pero el término aparece en escenarios y conviene situarlo respecto a Foundry.

## ¿Qué es?

Un **copilot** es un asistente de IA generativa integrado en una aplicación que ayuda al usuario a realizar tareas en lenguaje natural (redactar, resumir, analizar, automatizar), normalmente con acceso al contexto del usuario (documentos, correo, datos).

## La familia Copilot de Microsoft

| Producto | Qué es | Quién lo usa |
|---|---|---|
| **Microsoft Copilot** | Asistente de consumo/empresa basado en modelos de OpenAI con búsqueda web | Cualquier usuario |
| **Microsoft 365 Copilot** | Copilot dentro de Word, Excel, Outlook, Teams, con los datos de la organización (Microsoft Graph) | Empleados |
| **GitHub Copilot** | Asistente de programación en el editor | Desarrolladores |
| **Copilot Studio** | Herramienta **low-code** para crear copilots/agentes personalizados para M365, Teams y web | Creadores de negocio ("makers") |
| **Copilot in Azure / Security Copilot / Dynamics 365 Copilot** | Copilots especializados | Administradores, analistas |

## Relación con Microsoft Foundry

| | Copilot Studio | Microsoft Foundry |
|---|---|---|
| Enfoque | **Low-code**, plantillas, conectores | **Pro-code**, control total, SDK |
| Usuario | Makers de negocio | Desarrolladores |
| Modelos | Gestionados por Microsoft | Elección libre del catálogo |
| Integración | Nativa con M365 y Teams | Cualquier app; también puede publicar en Teams/M365 |
| Cuándo | Asistente empresarial rápido sin código | Agentes a medida, multimodalidad, evaluaciones, guardrails avanzados |

Ambos pueden combinarse: un agente de Foundry puede exponerse en Copilot Studio o en Microsoft 365.

## 🧠 Memorizar

> [!important]
> - Copilot = asistente generativo integrado en una app.
> - **Copilot Studio** = crear copilots **sin código**. **Foundry** = crear agentes **con código y control total**.

## Tips para AI-901

> [!tip]
> - ⚠️ "Sin escribir código, para usuarios de negocio, en Teams" → Copilot Studio. "Desarrolladores, SDK, elegir modelo, evaluaciones" → Foundry.

## Preguntas que podrían aparecer

**1.** Un departamento de RR. HH. sin desarrolladores quiere un asistente en Teams que responda preguntas sobre las políticas internas. ¿Qué herramienta es la más adecuada?
- A) Microsoft Foundry SDK · B) Copilot Studio · C) Azure Machine Learning · D) Content Understanding

<details><summary>Respuesta</summary>

**B.** Low-code y orientado a M365/Teams.
</details>

## Relacionado

- [[Agentes de IA (Foundry Agent Service)]]
- [[Generative AI]]
- [[Microsoft Foundry]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
