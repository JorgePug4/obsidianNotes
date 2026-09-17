---
tags: [ai-901, azure, ia, machine-learning]
módulo: Fundamentos de IA
peso_examen: Bajo (contexto)
---

# Evaluación de modelos de ML

> [!info] 🟡 Contexto heredado de AI-900. En AI-901 la evaluación que sí importa es la de **modelos y agentes generativos en Foundry** (evaluaciones, groundedness, seguridad). Ver [[Despliegue y configuración de modelos]] y [[Azure AI Content Safety y guardrails]].

## ¿Qué es?

Medir cómo de bien predice un modelo usando datos que **no** se usaron para entrenarlo (validación/prueba).

## Métricas de clasificación

Se calculan a partir de la **matriz de confusión** (verdaderos/falsos positivos y negativos):

| Métrica | Qué mide | Cuándo importa |
|---|---|---|
| **Accuracy (exactitud)** | % de predicciones correctas | Clases equilibradas |
| **Precision** | De los que predije positivos, ¿cuántos lo eran? | Cuando un falso positivo es caro (marcar spam un correo importante) |
| **Recall (sensibilidad)** | De los positivos reales, ¿cuántos detecté? | Cuando un falso negativo es caro (no detectar una enfermedad) |
| **F1 score** | Media armónica de precision y recall | Equilibrio entre ambas, clases desbalanceadas |

## Métricas de regresión

| Métrica | Qué mide |
|---|---|
| **MAE** (error absoluto medio) | Error medio en las unidades del valor |
| **RMSE** (raíz del error cuadrático medio) | Igual, pero penaliza más los errores grandes |
| **R²** (coeficiente de determinación) | Qué parte de la variación explica el modelo (1 = perfecto) |

## Overfitting y underfitting

- **Overfitting**: excelente en entrenamiento, malo en prueba. El modelo memorizó. Solución: más datos, regularización, modelo más simple.
- **Underfitting**: malo en ambos. El modelo es demasiado simple.

## Ejemplo

Un modelo de detección de cáncer con **accuracy** del 99 % puede ser inútil si solo el 1 % de los casos son positivos y el modelo siempre dice "negativo". Ahí importa el **recall**.

## 🧠 Memorizar

> [!important]
> - **Precision** → evita falsos positivos. **Recall** → evita falsos negativos. **F1** → equilibrio.
> - **Accuracy engaña** con clases desbalanceadas.
> - **Overfitting** = memorizar el entrenamiento.
> - Regresión se evalúa con **MAE, RMSE, R²**.

## Tips para AI-901

> [!tip]
> - 🟡 Estas métricas ya no son un objetivo explícito. Si aparecen, será en una pregunta de contexto: "¿qué métrica mide la proporción de positivos reales detectados?" → recall.
> - 🔥 Lo que sí evalúa AI-901 es la **evaluación de salidas generativas**: groundedness (¿está apoyado en las fuentes?), relevancia, coherencia, seguridad del contenido.

## Preguntas que podrían aparecer

**1.** Un modelo clasifica transacciones como fraude. Es crítico no dejar pasar fraudes reales aunque haya falsas alarmas. ¿Qué métrica priorizas?
- A) Accuracy · B) Precision · C) Recall · D) R²

<details><summary>Respuesta</summary>

**C.** Recall mide cuántos fraudes reales se detectan. B minimizaría falsas alarmas, que aquí se toleran. D es de regresión.
</details>

## Relacionado

- [[Machine Learning (fundamentos)]]
- [[Tipos de Machine Learning]]
- [[Azure Machine Learning]]

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
