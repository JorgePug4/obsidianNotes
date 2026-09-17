---
tags: [ai-901, azure, ia, machine-learning]
módulo: Fundamentos de IA
peso_examen: Bajo (contexto)
---

# Machine Learning (fundamentos)

> [!info] 🟡 Contexto
> AI-901 ya no tiene un dominio de Machine Learning. Esta nota existe para que entiendas de dónde salen los modelos y para que distingas **ML clásico** de **IA generativa** en escenarios (objetivo 1.3.1). No memorices más de lo que hay en el bloque 🧠.

## ¿Qué es?

**Machine Learning (ML)** es la rama de la IA en la que un algoritmo **aprende patrones a partir de datos** en lugar de seguir reglas programadas. El resultado del aprendizaje es un **modelo**.

## ¿Para qué sirve?

Para predecir (¿cuánto costará?, ¿es fraude?, ¿qué grupo de clientes es este?) cuando existe historial de datos. Los modelos generativos actuales (LLM) también son modelos de ML, entrenados con cantidades enormes de texto.

## Conceptos clave

- **Dataset**: conjunto de datos de entrenamiento.
- **Features (características)**: las columnas de entrada (edad, tamaño, píxeles…).
- **Label (etiqueta)**: el valor que queremos predecir (precio, "spam/no spam").
- **Training (entrenamiento)**: ajustar el modelo con datos.
- **Validation / Testing**: comprobar el modelo con datos que no vio en el entrenamiento.
- **Model (modelo)**: función f(features) → predicción.
- **Inferencia**: usar el modelo con datos nuevos.
- **Overfitting (sobreajuste)**: el modelo memoriza el entrenamiento y falla con datos nuevos.

## Cómo funciona

```
Datos históricos ─▶ dividir ─▶ entrenamiento (70-80 %) ─▶ modelo
                              └─▶ validación / prueba  ─▶ métricas
Datos nuevos ─▶ modelo ─▶ predicción (inferencia)
```

1. Se preparan los datos (features + label).
2. Un algoritmo ajusta el modelo para minimizar el error.
3. Se evalúa con datos reservados ([[Evaluación de modelos de ML]]).
4. Se despliega como servicio para hacer inferencia.

## Ejemplo

Una inmobiliaria tiene 10 000 ventas (metros, barrio, año, precio). Entrena un modelo de **regresión** con metros/barrio/año como *features* y el precio como *label*. Después, ante un piso nuevo, el modelo predice su precio.

## Servicios relacionados

- [[Azure Machine Learning]]: la plataforma de Azure para entrenar y desplegar modelos propios (🟡 contexto).
- [[Microsoft Foundry]]: donde usas modelos **ya entrenados** (foco de AI-901).

## Comparaciones

| | ML clásico | IA generativa (LLM) |
|---|---|---|
| Entrenamiento | Con tus datos y etiquetas | Preentrenado por el proveedor con datos masivos |
| Salida | Número o categoría | Contenido nuevo (texto, imagen…) |
| Cómo lo guías | Con datos de entrenamiento | Con **prompts** (y opcionalmente grounding o fine-tuning) |
| Ejemplo | Predecir demanda | Redactar un informe |
| Servicio de Azure | Azure Machine Learning | Microsoft Foundry (catálogo de modelos) |

## 🧠 Memorizar

> [!important]
> - **Features** = entradas · **Label** = lo que se predice.
> - **Entrenar** con datos históricos, **inferir** con datos nuevos.
> - **Overfitting** = funciona con los datos de entrenamiento, falla con los nuevos.
> - Los LLM son ML **preentrenado**: tú no los entrenas, los usas con prompts.

## Tips para AI-901

> [!tip]
> - 🔥 Si el escenario habla de "*entrenar un modelo con nuestros datos históricos para predecir…*" → ML clásico → Azure Machine Learning. Si habla de "*generar*" o "*responder en lenguaje natural*" → modelo generativo en Foundry.
> - ⚠️ No confundas **fine-tuning** (ajustar un LLM preentrenado con ejemplos) con **entrenar desde cero**.

## ⚠️ Errores comunes

- Creer que un LLM "aprende" de tus conversaciones en producción. No: solo cambia si lo reentrenas o lo ajustas (fine-tuning).
- Confundir *label* con *feature*.

## 💡 Escenario

Un banco tiene años de transacciones marcadas como "fraude/no fraude" y quiere detectar fraude en nuevas transacciones. ¿ML clásico o IA generativa?

<details><summary>Respuesta</summary>

**ML clásico (clasificación supervisada).** Hay datos históricos etiquetados y se busca una predicción de categoría, no contenido nuevo.
</details>

## Preguntas que podrían aparecer

**1.** En un dataset para predecir el precio de un coche, ¿qué es "precio"?
- A) Feature · B) Label · C) Inferencia · D) Dataset

<details><summary>Respuesta</summary>

**B.** El precio es la etiqueta (lo que se quiere predecir). Marca, año y kilómetros serían features.
</details>

## Relacionado

- [[Tipos de Machine Learning]]
- [[Evaluación de modelos de ML]]
- [[Azure Machine Learning]]
- [[Large Language Models]]

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
