---
tags: [ai-901, azure, ia, laboratorios, MOC]
certificación: Azure AI Fundamentals (AI-901)
módulo: Laboratorios
---

# AI-901 · Laboratorios

> [!info] De dónde salen estos laboratorios
> Están basados en los **ejercicios oficiales de Microsoft Learning** para AI-901, resumidos al nivel que necesita un examen *Fundamentals*: qué vas a hacer, los pasos esenciales, qué está ocurriendo por debajo y qué objetivo del examen refuerza cada uno. No son tutoriales largos: el objetivo es **reconocer el flujo del portal y del código**, que es lo que evalúa el dominio 2 (55–60 %).
>
> - Laboratorios con Azure: `microsoftlearning.github.io/mslearn-ai-fundamentals`
> - Laboratorios conceptuales sin suscripción (modelos en el navegador): `microsoftlearning.github.io/mslearn-ai-concepts`

> [!warning] Antes de empezar
> Necesitas una **suscripción de Azure** con permisos para crear un recurso de Microsoft Foundry. Los despliegues de modelos están sujetos a **cuota regional**: si un modelo no está disponible, usa otro modelo *gpt* (gpt-5-nano, gpt-5.4-mini) o crea el proyecto en otra región. **Elimina el grupo de recursos al terminar** para no acumular coste.

| Lab | Qué practicas | Objetivos del examen |
|---|---|---|
| [[Lab 01 - Explorar Microsoft Foundry]] | Crear un proyecto y recorrer el portal | Base del dominio 2 |
| [[Lab 02 - Desplegar un modelo y crear un agente]] | Desplegar, promptear, herramientas, conocimiento, guardar como agente, código | 2.1.1–2.1.5 |
| [[Lab 03 - Análisis de texto en Foundry]] | Resumen con un LLM vs Azure Language (idioma, PII) | 1.3.2, 2.2.1 |
| [[Lab 04 - Agente de voz con Voice Live]] | Voice mode en un agente, sesión de voz, código cliente | 1.3.3, 2.2.2, 2.2.3 |
| [[Lab 05 - Visión y generación de imágenes]] | Imagen en el prompt, text to image, vídeo | 1.3.4, 2.3.1–2.3.3 |
| [[Lab 06 - Content Understanding (extracción de información)]] | Read → Layout → campos, y el SDK | 1.3.5, 2.4.1–2.4.4 |
| [[Lab 07 - Foundry IQ (conocimiento para agentes)]] | Knowledge base, permisos, grounding con citas | 2.1.4, RAG |
| [[Lab 08 - Cliente Python con el Foundry SDK]] | Cliente de chat y cliente de agente en código | 2.1.3, 2.1.5 |

Orden recomendado: 01 → 02 → 07 → 08 → 03 → 04 → 05 → 06.

> [!tip] Si no tienes suscripción de Azure
> Los laboratorios conceptuales de Microsoft usan **apps en el navegador** con un modelo pequeño local (Phi) y cubren los mismos conceptos: *Chat Playground* (`aka.ms/chat-playground`), *Language Playground* (`aka.ms/language-app`), *Information Extractor* (`aka.ms/info-extractor`) y *Model Coder* (`aka.ms/model-coder`). Sirven para entender el flujo, aunque no reproducen el portal de Foundry.

---

Volver al índice general: [[00 - AI-901 Índice general]]
