---
tags: [ai-901, azure, ia, machine-learning]
módulo: Fundamentos de IA
peso_examen: Bajo (contexto)
---

# Tipos de Machine Learning

> [!info] 🟡 Contexto heredado de AI-900. Útil para razonar escenarios y descartar distractores; no es objetivo explícito de AI-901.

## ¿Qué es?

Clasificación de los problemas de ML según **si hay etiquetas** en los datos y **qué tipo de salida** se busca.

## Los tipos que debes reconocer

| Tipo | ¿Datos etiquetados? | Salida | Pregunta que responde | Ejemplo |
|---|---|---|---|---|
| **Regresión** (supervisado) | Sí | Un **número** | ¿Cuánto? | Predecir ventas, temperatura, precio |
| **Clasificación** (supervisado) | Sí | Una **categoría** | ¿Cuál / es o no es? | Spam o no, diagnóstico, tipo de fruta |
| **Clustering** (no supervisado) | No | **Grupos** | ¿Qué se parece a qué? | Segmentar clientes sin categorías previas |
| **Detección de anomalías** | Normalmente no | Alerta de valor raro | ¿Es normal? | Fraude, fallo de sensor |

- **Aprendizaje supervisado**: el dataset incluye la respuesta correcta (*label*). Regresión y clasificación.
- **Aprendizaje no supervisado**: no hay etiquetas; el algoritmo descubre estructura. Clustering.
- **Clasificación binaria** (dos clases) vs **multiclase** (varias clases, una sola correcta).

## Cómo funciona (idea)

- Regresión: ajusta una función que aproxime los valores numéricos (p. ej., una recta).
- Clasificación: aprende fronteras que separan las clases.
- Clustering: agrupa por similitud de features (p. ej., *k-means* con *k* grupos).

## Ejemplo

Un supermercado:
- Predecir cuántas unidades venderá mañana → **regresión**.
- Decidir si una reseña es positiva o negativa → **clasificación** (en AI-901 lo harías con análisis de sentimiento, ver [[Técnicas de análisis de texto]]).
- Agrupar clientes por hábitos de compra sin categorías previas → **clustering**.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| Regresión vs clasificación | Número → regresión. Categoría → clasificación |
| Clasificación vs clustering | Con etiquetas conocidas → clasificación. Sin etiquetas, "descubrir grupos" → clustering |
| Clustering vs detección de anomalías | Agrupar lo parecido vs señalar lo raro |

## 🧠 Memorizar

> [!important]
> - **Regresión = número · Clasificación = categoría · Clustering = grupos sin etiquetas.**
> - Supervisado = con etiquetas (regresión, clasificación). No supervisado = sin etiquetas (clustering).

## Tips para AI-901

> [!tip]
> - 🔥 "¿Cuánto…?" → regresión. "¿Es A o B?" → clasificación. "Segmentar sin categorías previas" → clustering.
> - ⚠️ Un escenario de **análisis de sentimiento** o **clasificación de imágenes** es, técnicamente, clasificación, pero en AI-901 la respuesta esperada es la **carga de trabajo/servicio** (texto o visión), no "clasificación".

## 💡 Escenario

Una empresa de streaming quiere agrupar usuarios con patrones de visualización parecidos para crear campañas, sin tener definidas categorías. ¿Qué tipo de ML?

<details><summary>Respuesta</summary>

**Clustering** (no supervisado): no hay etiquetas previas y se buscan grupos por similitud.
</details>

## Preguntas que podrían aparecer

**1.** Predecir el número de bicicletas alquiladas según el clima es un ejemplo de:
- A) Clasificación · B) Regresión · C) Clustering · D) IA generativa

<details><summary>Respuesta</summary>

**B.** La salida es un número.
</details>

**2.** Verdadero o falso: el clustering necesita datos etiquetados.

<details><summary>Respuesta</summary>

**Falso.** Es aprendizaje no supervisado.
</details>

## Relacionado

- [[Machine Learning (fundamentos)]]
- [[Evaluación de modelos de ML]]
- [[Azure Machine Learning]]

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
