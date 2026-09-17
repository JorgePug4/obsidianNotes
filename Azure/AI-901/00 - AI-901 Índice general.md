---
tags: [ai-901, ai-900, azure, ia, MOC]
tipo: MOC
certificación: Microsoft Certified - Azure AI Fundamentals (examen AI-901)
aliases: [AI-901, AI-900, Azure AI Fundamentals, Índice AI-901]
revisado: 2026-09-17
---

# 🧠 Azure AI Fundamentals · AI-901 (antes AI-900)

> [!warning] Lee esto primero: el examen ya no es AI-900
> Microsoft **retiró el examen AI-900 el 30 de junio de 2026** y lo sustituyó por **AI-901: Microsoft Azure AI Fundamentals** (beta desde el 21 de abril de 2026, disponible de forma general desde junio de 2026). La certificación sigue llamándose **Microsoft Certified: Azure AI Fundamentals**, pero el temario cambió por completo: pasa de "describir servicios sueltos" a **reconocer conceptos de IA e implementar soluciones sencillas con Microsoft Foundry**.
>
> Estas notas están construidas sobre la **guía oficial de habilidades de AI-901 (versión del 15 de abril de 2026)**. El contenido heredado de AI-900 que ya no se evalúa está marcado como 🟡 *contexto*. Detalle del cambio: [[AI-900 vs AI-901 - Qué cambió]].

## Ficha del examen

| Dato | Valor |
|---|---|
| Examen | **AI-901: Microsoft Azure AI Fundamentals** |
| Certificación | Microsoft Certified: Azure AI Fundamentals |
| Nivel | Fundamentals (sin prerrequisitos formales) |
| Puntuación mínima | **700 / 1000** |
| Formato | Opción múltiple, opción múltiple con varias respuestas, arrastrar/soltar, hot area, sí/no. Preguntas de escenario y algunas con capturas del portal de Foundry o fragmentos de Python |
| Perfil oficial del candidato | "Al inicio de tu carrera en desarrollo de soluciones de IA": conocimiento conceptual de IA en Azure, **nociones de sintaxis de Python** y familiaridad con recursos de Azure |
| Funciones evaluadas | Casi todo GA. Puede haber preguntas sobre funciones en *preview* si son de uso común |
| Guía oficial | learn.microsoft.com/credentials/certifications/resources/study-guides/ai-901 |

## Dominios oficiales (habilidades medidas a 15 de abril de 2026)

| # | Dominio | Peso | Carpeta de estas notas |
|---|---|---|---|
| 1 | **Identify AI concepts and responsibilities** (en el detalle: *capabilities*) | **40–45 %** | 01 Fundamentos · 02 Responsible AI · 03 GenAI (conceptos) · 04/05/06 (conceptos) |
| 2 | **Implement AI solutions by using Microsoft Foundry** | **55–60 %** | 03 GenAI y agentes · 04 Texto y voz · 05 Visión · 06 Extracción · 07 Laboratorios |

### Desglose del dominio 1 (40–45 %)

| Sección | Habilidades | Notas que la cubren |
|---|---|---|
| 1.1 Principios de Responsible AI | Fairness, reliability & safety, privacy & security, inclusiveness, transparency, accountability | [[00 - Índice - Responsible AI]] |
| 1.2 Componentes y configuración de modelos | Cómo funcionan los modelos generativos · elegir modelo según capacidades · opciones de despliegue y parámetros | [[Large Language Models]] · [[Catálogo de modelos de Foundry]] · [[Despliegue y configuración de modelos]] |
| 1.3 Cargas de trabajo de IA | Escenarios (GenAI y agentes, texto, voz, visión, extracción) · técnicas de análisis de texto · reconocimiento y síntesis de voz · visión y generación de imágenes · técnicas de extracción de información | [[Cargas de trabajo de IA]] · [[Técnicas de análisis de texto]] · [[Reconocimiento y síntesis de voz]] · [[Computer Vision]] · [[Extracción de información]] |

### Desglose del dominio 2 (55–60 %)

| Sección | Habilidades | Notas que la cubren |
|---|---|---|
| 2.1 Apps generativas y agentes con Foundry | Prompts de sistema y usuario · desplegar y probar un modelo en el portal · cliente de chat ligero con el Foundry SDK · crear y probar un agente en el portal · cliente ligero para un agente | [[Prompts y Prompt Engineering]] · [[Despliegue y configuración de modelos]] · [[Foundry SDK y cliente de chat]] · [[Agentes de IA (Foundry Agent Service)]] |
| 2.2 Texto y voz con Foundry | App ligera con análisis de texto · responder a prompts hablados con un modelo multimodal · app ligera con Azure Speech in Foundry Tools | [[Azure AI Language]] · [[Azure AI Speech]] |
| 2.3 Visión y generación de imágenes con Foundry | Interpretar imágenes en prompts con un modelo multimodal · generar imágenes con modelos generativos · app ligera con visión | [[Modelos multimodales (visión en prompts)]] · [[Generación de imágenes y vídeo]] · [[Azure AI Vision]] |
| 2.4 Extracción de información con Foundry | Documentos y formularios · imágenes · audio y vídeo · app ligera con Azure Content Understanding | [[Azure Content Understanding]] |

## Mapa de la carpeta

```
🧠 AI-901
├── 01 - Fundamentos de IA        → [[00 - Índice - Fundamentos de IA]]
│     IA · cargas de trabajo · ML (contexto) · Microsoft Foundry · Foundry Tools
├── 02 - Responsible AI           → [[00 - Índice - Responsible AI]]
│     6 principios · Content Safety y guardrails
├── 03 - Generative AI y agentes  → [[00 - Índice - Generative AI y agentes]]
│     LLM · tokens/embeddings · prompts · RAG/Foundry IQ · catálogo · despliegue · SDK · agentes · Copilot
├── 04 - Texto y voz              → [[00 - Índice - Texto y voz]]
│     NLP · técnicas de texto · Azure AI Language · voz · Azure AI Speech · Translator
├── 05 - Computer Vision          → [[00 - Índice - Computer Vision]]
│     visión · Azure AI Vision · Face · OCR · multimodales · generación de imágenes
├── 06 - Extracción de información → [[00 - Índice - Extracción de información]]
│     técnicas · Content Understanding · Document Intelligence
├── 07 - Laboratorios             → [[00 - Índice - Laboratorios]]
└── 08 - Repaso y simulacro
      [[AI-901 - Guía de repaso final]] · [[AI-901 - Escenarios rápidos (Problema → Tecnología)]]
      [[AI-901 - Simulacro final (50 preguntas)]] · [[AI-901 - Simulacro final - Respuestas y explicaciones]]
      [[AI-901 - Cobertura del temario oficial]] · [[AI-900 vs AI-901 - Qué cambió]]
```

## Ruta de estudio sugerida (2–3 semanas)

1. **Semana 1 · Conceptos (dominio 1).** Fundamentos → Responsible AI → conceptos de GenAI (LLM, tokens, prompts, RAG) → conceptos de texto, voz, visión y extracción. Cierra cada carpeta con su `99 - Repaso final`.
2. **Semana 2 · Foundry (dominio 2).** Microsoft Foundry → catálogo y despliegue → SDK → agentes → laboratorios 01 a 08 en orden. Aquí está el 55–60 % del examen: no lo dejes para el final.
3. **Semana 3 · Consolidación.** [[AI-901 - Guía de repaso final]] → [[AI-901 - Escenarios rápidos (Problema → Tecnología)]] → simulacro de 50 preguntas → repasa los fallos en la nota de origen.

## Leyenda de las notas

| Marca | Significado |
|---|---|
| ⭐ Imprescindible | Concepto que debo dominar sin dudar |
| 🔥 Alta importancia | Cae con frecuencia; conocerlo especialmente bien |
| ⚠️ Cuidado | Se confunde fácilmente en preguntas |
| 🧠 Memorizar | Definiciones, diferencias o nombres que hay que recordar |
| 📌 Diferencia clave | Comparación entre tecnologías similares |
| 💡 Escenario | Pregunta tipo "qué tecnología usarías" |
| 🟡 Contexto | Heredado de AI-900 o conocimiento adicional; no es objetivo explícito de AI-901 |

## Cambios de nombre que sí debes reconocer

| Nombre antiguo | Nombre actual (2026) |
|---|---|
| Azure AI Studio → Azure AI Foundry | **Microsoft Foundry** (anunciado en Ignite, nov. 2025; efectivo en 2026) |
| Azure Cognitive Services → Azure AI Services | **Foundry Tools** (Azure Language, Azure Speech, Azure Vision, Azure Translator, Azure Content Understanding… "in Foundry Tools") |
| Azure AI Form Recognizer → Azure AI Document Intelligence | **Azure Document Intelligence**, ahora parte de **Azure Content Understanding in Foundry Tools** |
| Azure OpenAI Service | Modelos de OpenAI dentro de **Foundry Models** ("sold directly by Azure") |
| Azure AI Agent Service | **Foundry Agent Service** |

Las preguntas pueden usar el nombre antiguo o el nuevo. Trátalos como sinónimos, pero responde con el actual.

## Enlaces con el resto del vault

- Identidad y autenticación sin claves (Entra ID, `DefaultAzureCredential`): [[Microsoft Entra ID]], [[Azure RBAC]].
- Secretos y claves de API: [[Azure Key Vault]].
- Almacenamiento de documentos para Content Understanding y Foundry IQ: [[Azure Blob Storage]].
- Índice general de ingeniería: [[🗺️ Índice - Ingeniería de Software]].
