---
tags: [ai-901, azure, ia, fundamentos, repaso]
módulo: Fundamentos de IA
---

# 🎯 Repaso final · Fundamentos de IA

Nota de consolidación del módulo 01. Úsala el día antes del examen.

## 1. Los conceptos más importantes

1. La IA se basa en **modelos** entrenados; en AI-901 casi siempre los **usas**, no los entrenas.
2. Tres familias: **ML predictivo**, **IA generativa**, **IA agéntica**.
3. Las **cinco cargas de trabajo** oficiales: GenAI/agentes, texto, voz, visión, extracción de información.
4. **Microsoft Foundry** es la plataforma unificada: recurso → proyectos; portal Discover/Build/Operate.
5. **Foundry Tools** = servicios preconstruidos (antes Azure AI Services / Cognitive Services) con resultados deterministas.
6. **Foundry Models** (catálogo), **Foundry Agent Service** (agentes), **Foundry IQ** (conocimiento), **guardrails** (seguridad).
7. **Azure Machine Learning** es para entrenar modelos propios; en AI-901 es un distractor.
8. Regresión = número, clasificación = categoría, clustering = grupos (🟡 contexto).

## 2. Tabla de servicios y propósito

| Servicio | Propósito | Concepto clave |
|---|---|---|
| [[Microsoft Foundry]] | Crear apps y agentes de IA | Proyecto, playground, agente, SDK |
| [[Foundry Tools (servicios de IA de Azure)]] | Capacidades preconstruidas | Determinista, JSON, confianza |
| [[Azure Machine Learning]] | Entrenar modelos propios | AutoML, designer, endpoint |
| Foundry IQ | Conocimiento para agentes | Knowledge base, Azure AI Search |
| Foundry Agent Service | Ejecutar agentes | Instrucciones + modelo + herramientas |

## 3. Diferencias que más se confunden

| Par confuso | Cómo distinguirlo |
|---|---|
| Foundry vs Azure ML | Usar modelos y agentes / entrenar modelos propios |
| Foundry Tools vs modelo generativo | Salida estructurada y predecible / salida libre guiada por prompt |
| Modelo vs agente | Responde / decide y usa herramientas |
| Recurso de Foundry vs proyecto | Padre con servicios / hijo con activos de una solución |
| Visión vs extracción de información | Entender la imagen / obtener campos y datos |
| Clasificación vs clustering | Con etiquetas / sin etiquetas |

## 4. Diez tips de examen

> [!tip]
> 1. Primero identifica la **carga de trabajo**, luego el servicio.
> 2. "Generar / redactar / resumir" → modelo generativo. "Actuar / usar herramientas" → agente.
> 3. "Resultados consistentes en un pipeline" → Foundry Tools.
> 4. "Entrenar con datos históricos" → Azure ML (y solo entonces).
> 5. "Facturas, recibos, formularios, audio, vídeo → datos estructurados" → Content Understanding.
> 6. Nombres nuevos: Microsoft Foundry, Foundry Tools, Foundry Agent Service, Foundry IQ.
> 7. Portal `ai.azure.com`: Discover (catálogo) · Build (desarrollo) · Operate (administración).
> 8. Cliente de proyecto = identidad Entra (`DefaultAzureCredential`), nunca clave.
> 9. Un modelo multimodal cruza cargas de trabajo (imagen + texto + audio).
> 10. No sobreestudies ML clásico: en AI-901 es contexto.

## 5. Preguntas de repaso

**1.** ¿Qué carga de trabajo corresponde a convertir un formulario escaneado en un registro JSON con nombre, fecha e importe?
- A) Visión · B) Extracción de información · C) Análisis de texto · D) Voz

<details><summary>Respuesta</summary>

**B.** Extracción de información (Content Understanding). Visión describiría la imagen, no extraería campos.
</details>

**2.** ¿Qué componente de Microsoft Foundry organiza los modelos, agentes y datos de una solución concreta?
- A) Suscripción · B) Grupo de recursos · C) Proyecto · D) Playground

<details><summary>Respuesta</summary>

**C.** El proyecto es hijo del recurso de Foundry y agrupa los activos de una solución.
</details>

**3.** Un equipo quiere un resultado determinista para detectar PII en documentos. ¿Qué opción es más adecuada?
- A) Un prompt a `gpt-5-mini` · B) Azure Language in Foundry Tools · C) Azure Machine Learning · D) Un agente con búsqueda web

<details><summary>Respuesta</summary>

**B.** Servicio preconstruido con salida estructurada. A es variable, C requiere entrenar, D no aplica.
</details>

**4.** Verdadero o falso: Microsoft Foundry es el nuevo nombre de Azure AI Foundry.

<details><summary>Respuesta</summary>

**Verdadero.** Y Azure AI Foundry fue antes Azure AI Studio.
</details>

**5.** ¿Qué diferencia principal hay entre un modelo desplegado y un agente en Foundry?
- A) El agente no usa modelos
- B) El agente encapsula modelo, instrucciones y herramientas y expone un endpoint propio
- C) El modelo solo funciona con voz
- D) No hay diferencia

<details><summary>Respuesta</summary>

**B.** El agente añade instrucciones, herramientas y conocimiento sobre el modelo, de forma que el cliente no tiene que gestionar el prompt de sistema ni la lógica RAG.
</details>

**6.** Predecir la temperatura de mañana a partir de datos históricos es:
- A) Clasificación · B) Regresión · C) Clustering · D) IA generativa

<details><summary>Respuesta</summary>

**B.** La salida es un número.
</details>

**7.** ¿Qué servicio de Foundry Tools usarías para leer texto de fotos de carteles y devolverlo como texto plano?
- A) Azure Speech · B) Azure Translator · C) Content Understanding (OCR/Read) · D) Azure Machine Learning

<details><summary>Respuesta</summary>

**C.** OCR/Read en Content Understanding (también lo hace Azure Vision). A es voz, B traduce, D entrena modelos.
</details>

**8.** Un agente debe consultar el estado de pedidos en una API y responder por chat. ¿Qué elemento del agente permite llamar a la API?
- A) Instrucciones · B) Herramientas (tools) · C) Modelo · D) Guardrails

<details><summary>Respuesta</summary>

**B.** Las herramientas dan al agente acceso a datos y acciones externas.
</details>

## Checklist

- [ ] Comprendo los conceptos fundamentales (modelo, entrenamiento, inferencia, generativa, agéntica).
- [ ] Sé identificar las cinco cargas de trabajo de IA en un escenario.
- [ ] Sé diferenciar Foundry, Foundry Tools y Azure Machine Learning.
- [ ] Puedo resolver escenarios básicos "necesidad → servicio".
- [ ] Conozco los nombres actuales de la plataforma y los servicios.
- [ ] Puedo responder las preguntas de práctica.

## 🧠 Última pasada antes del examen

> [!important] Lo que no puede fallarte
> - Cargas de trabajo: **GenAI/agentes · texto · voz · visión · extracción**.
> - Foundry: **recurso → proyecto**; **Discover / Build / Operate**; **Models · Agent Service · Tools · IQ · guardrails**.
> - Foundry Tools = determinista. LLM = flexible.
> - Azure ML = entrenar. Foundry = usar.

Volver al índice: [[00 - Índice - Fundamentos de IA]] · [[00 - AI-901 Índice general]]
