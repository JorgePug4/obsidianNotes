---
tags: [ai-901, azure, ia, machine-learning, servicio]
módulo: Fundamentos de IA
peso_examen: Bajo (distractor)
aliases: [Azure ML]
---

# Azure Machine Learning

> [!info] 🟡 Contexto
> Azure Machine Learning **no aparece en la guía oficial de AI-901**. Lo cubro porque es el distractor habitual en preguntas de "¿qué servicio usarías?" y para que sepas cuándo **no** es la respuesta.

## ¿Qué es?

**Azure Machine Learning** es la plataforma de Azure para el **ciclo de vida completo de modelos de ML propios**: preparar datos, entrenar, evaluar, registrar, desplegar y monitorizar (MLOps).

## ¿Para qué sirve?

Para científicos de datos e ingenieros de ML que necesitan **entrenar sus propios modelos** con sus datos (regresión, clasificación, clustering, deep learning) y publicarlos como endpoints.

## Conceptos clave

- **Workspace (área de trabajo)**: recurso raíz que agrupa datos, experimentos, modelos y endpoints.
- **Compute**: instancias y clústeres para entrenar.
- **Automated ML (AutoML)**: prueba automáticamente algoritmos y elige el mejor modelo.
- **Designer**: interfaz visual de arrastrar y soltar para crear pipelines sin código.
- **Notebooks / SDK de Python**: desarrollo con código.
- **Endpoints**: el modelo entrenado expuesto como API para inferencia.
- **Registro de modelos**, **pipelines**, **MLOps**.

## Ejemplo

Una eléctrica entrena con AutoML un modelo de regresión que predice el consumo por hora usando datos históricos y meteorología, y lo despliega como endpoint que consume su aplicación de planificación.

## 📌 Azure Machine Learning vs Microsoft Foundry

| | Azure Machine Learning | Microsoft Foundry |
|---|---|---|
| Objetivo | **Entrenar** y operar modelos propios (ML clásico y deep learning) | **Usar** modelos preentrenados, apps generativas, agentes y Foundry Tools |
| Usuario | Científico de datos, ingeniero ML | Desarrollador de apps y agentes de IA |
| Datos | Tus datasets etiquetados | Prompts, documentos para grounding, contenido a analizar |
| Palabras clave | AutoML, designer, entrenar, experimento, endpoint de ML | catálogo de modelos, playground, agente, Foundry SDK, Content Understanding |
| AI-901 | 🟡 distractor | ⭐ núcleo del examen |

## 🧠 Memorizar

> [!important]
> - Azure ML = **entrenar modelos propios** (AutoML, designer, notebooks, endpoints).
> - Si el escenario **no** dice "entrenar con nuestros datos", casi nunca es Azure ML.

## Tips para AI-901

> [!tip]
> - ⚠️ "Analizar el sentimiento de reseñas" NO es Azure ML: es Azure Language o un modelo generativo en Foundry.
> - ⚠️ "Extraer campos de facturas" NO es Azure ML: es Content Understanding.
> - 🔥 "Predecir la demanda entrenando con ventas históricas" SÍ es Azure ML (AutoML).

## Preguntas que podrían aparecer

**1.** Un equipo de datos quiere entrenar y comparar automáticamente varios algoritmos para predecir bajas de clientes a partir de su historial, sin escribir código. ¿Qué usarías?
- A) Foundry Agent Service · B) Azure Machine Learning (Automated ML) · C) Azure Content Understanding · D) Azure Language

<details><summary>Respuesta</summary>

**B.** Es ML supervisado con datos propios y sin código: AutoML. A crea agentes, C extrae datos de contenido, D analiza texto.
</details>

## Relacionado

- [[Machine Learning (fundamentos)]]
- [[Tipos de Machine Learning]]
- [[Microsoft Foundry]]

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
