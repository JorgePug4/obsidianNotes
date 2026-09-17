---
tags: [ai-901, azure, ia, responsible-ai]
módulo: Responsible AI
peso_examen: Alto
objetivo_oficial: "1.1.1 Describe considerations for fairness in an AI solution"
---

# Fairness (Equidad)

## ¿Qué significa?

Los sistemas de IA deben **tratar a todas las personas de forma justa**, sin que sus resultados favorezcan o perjudiquen a un grupo por características como sexo, edad, etnia, origen, discapacidad o nivel económico.

## Riesgo que intenta evitar

**Sesgo (bias)**: el modelo reproduce o amplifica desigualdades presentes en los datos de entrenamiento o en el diseño. El sesgo suele ser **involuntario** y **difícil de ver** sin medirlo.

## Consideraciones en una solución

- **Datos representativos**: si un grupo está infrarrepresentado, el modelo funcionará peor con él.
- **Evaluar por subgrupos**: medir precisión, tasa de aprobación, falsos positivos… por cada grupo, no solo en global.
- **Evitar variables proxy**: el código postal puede actuar como sustituto del origen.
- **Prompts y salidas generativas**: revisar que el modelo no produzca estereotipos (p. ej., asociar profesiones a un género).
- **Mitigar y volver a medir**: reequilibrar datos, ajustar umbrales, añadir revisión humana.

## Ejemplo

Un modelo que recomienda anuncios de empleo muestra menos ofertas de ingeniería a mujeres. La empresa mide la distribución por sexo, reequilibra los datos y añade una comprobación de paridad antes de publicar.

## Ejemplo con IA generativa

Se pide a un modelo "genera una imagen de un director general" y siempre produce hombres de mediana edad. Aplicar fairness implica evaluar esas salidas y ajustar prompts, guardrails o el modelo.

## 🧠 Memorizar

> [!important]
> - Fairness = **sin sesgo entre grupos**.
> - Herramienta principal: **evaluar por subgrupos** con datos representativos.
> - El sesgo viene sobre todo de los **datos**.

## Tips para AI-901

> [!tip]
> - ⭐ Palabras clave: *sesgo, discriminación, grupos demográficos, mismas oportunidades, resultados diferentes para personas similares*.
> - ⚠️ No lo confundas con **inclusiveness**: fairness es sobre **resultados justos**; inclusiveness es sobre que el sistema sea **accesible y útil para todos**.

## Preguntas que podrían aparecer

**1.** Un modelo de riesgo crediticio ofrece peores condiciones a solicitantes de determinados barrios con perfiles financieros idénticos. ¿Qué consideración de Responsible AI aborda este problema?
- A) Transparency · B) Reliability and safety · C) Fairness · D) Privacy and security

<details><summary>Respuesta</summary>

**C.** Resultados distintos para personas equivalentes según un atributo (barrio) es un problema de equidad.
</details>

**2.** ¿Cuál es la práctica más directa para detectar falta de equidad en un modelo?
- A) Cifrar los datos de entrenamiento
- B) Evaluar el rendimiento del modelo por separado para distintos grupos
- C) Publicar la documentación del modelo
- D) Añadir un aviso de que el sistema usa IA

<details><summary>Respuesta</summary>

**B.** A es privacy; C y D son transparency.
</details>

## Relacionado

- [[Responsible AI (principios de Microsoft)]]
- [[Inclusiveness (Inclusión)]]
- [[Evaluación de modelos de ML]]

← Volver al índice: [[00 - Índice - Responsible AI]]
