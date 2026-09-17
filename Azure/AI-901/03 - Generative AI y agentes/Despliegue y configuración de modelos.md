---
tags: [ai-901, azure, ia, generative-ai, foundry, despliegue]
módulo: Generative AI y agentes
peso_examen: Muy alto
objetivo_oficial: "1.2.3 Identify appropriate model deployment options and configuration parameters · 2.1.2 Deploy a model and interact with it in the Foundry portal"
aliases: [Deployment, Despliegue de modelos, Temperature, Top_p, Max tokens, Playground]
---

# Despliegue y configuración de modelos

## ¿Qué es?

Un modelo del catálogo no se puede usar hasta que se **despliega** (*deploy*) en un proyecto de Foundry. El **despliegue** crea un endpoint con un nombre (*deployment name*), un tipo de despliegue, una cuota y un filtro de contenido. Después se **interactúa** con él en el **playground** o desde código, ajustando **parámetros** como `temperature`, `top_p` y `max tokens`.

## ¿Para qué sirve?

Es literalmente lo que piden los objetivos 1.2.3 y 2.1.2: saber **qué opción de despliegue elegir** (coste, latencia, residencia de datos, capacidad) y **qué hace cada parámetro**.

## Cómo se despliega en el portal (laboratorio oficial)

1. **Discover → Models**, buscar el modelo (p. ej., `gpt-5-mini`) y abrir su tarjeta.
2. **Deploy** con la configuración por defecto (o personalizar nombre, tipo de despliegue, versión, cuota en tokens por minuto, filtro de contenido).
3. Esperar un minuto; se abre el **playground** del modelo.
4. En el playground: escribir **Instructions** (prompt de sistema), chatear, **New chat** para borrar el historial, ajustar parámetros, añadir **Tools** y **Knowledge**, y finalmente **Save as agent** o ver el código (**Call model / View code**).

> Si no hay cuota regional para un modelo, el lab sugiere usar otro modelo *gpt* (gpt-5-nano, gpt-5.4-mini) o crear el proyecto en otra región.

## Opciones de despliegue

| Opción | Qué es | Cuándo |
|---|---|---|
| **Serverless API (estándar, pago por token)** | Azure aloja el modelo; pagas por tokens consumidos | La opción por defecto y la de todos los labs |
| **Provisioned (PTU, capacidad reservada)** | Reservas capacidad de proceso con rendimiento garantizado | Cargas altas y predecibles, latencia estable |
| **Batch** | Procesamiento **asíncrono** de grandes lotes con descuento | Trabajos no interactivos (clasificar millones de textos de noche) |
| **Managed compute** (preview) | Modelos abiertos/partner/propios en GPU dedicada gestionada por Foundry | Modelos que no están como serverless, control total |
| **Foundry Local** | En el dispositivo | Sin conexión, privacidad local (🟡) |

Dentro de serverless, el **ámbito de procesamiento** define dónde se ejecutan las peticiones:

| Ámbito | Datos se procesan en… | Uso |
|---|---|---|
| **Global** (Global Standard / Global Provisioned / Global Batch) | Cualquier región de Azure del mundo | Máxima disponibilidad y menor coste |
| **Data Zone** (EU o US) | Solo dentro de la zona geográfica (UE o EE. UU.) | Residencia de datos regional |
| **Regional / Standard** | Solo en la región del recurso | Requisitos estrictos de residencia |
| **Developer** | Bajo coste para pruebas, sin SLA | Desarrollo (🟡) |

## Parámetros de configuración

| Parámetro | Qué controla | Valores | Efecto práctico |
|---|---|---|---|
| **Temperature** | Aleatoriedad al elegir el siguiente token | 0 → 2 (habitual 0–1) | Baja = respuestas más deterministas y repetibles; alta = más creativas y variadas |
| **Top_p** (nucleus sampling) | Restringe la elección a los tokens cuya probabilidad acumulada llega a *p* | 0 → 1 | Alternativa a temperature; se recomienda ajustar solo uno de los dos |
| **Max tokens / Max output tokens** | Longitud máxima de la **respuesta** | Entero | Limita coste y corta respuestas largas |
| **Stop sequences** | Cadenas que detienen la generación | Texto | Formatos controlados |
| **Frequency / presence penalty** | Penalizan repetir tokens / temas | -2 → 2 | Menos repeticiones |
| **Past messages / historial** | Cuántos turnos previos se envían | Entero | Memoria conversacional vs coste |
| **Reasoning effort** (modelos de razonamiento) | Cuánto "piensa" el modelo | low/medium/high | Calidad vs latencia y coste |
| **Filtro de contenido** | Umbrales de Content Safety por categoría | Configurable | Seguridad |

## Ejemplo

Un generador de descripciones de producto para un e-commerce: `temperature 0.2` y `max tokens 150` para textos consistentes y cortos; despliegue **Global Standard** para coste mínimo. Un servicio de atención al cliente 24×7 con 2 000 usuarios concurrentes: **Provisioned** para latencia estable. Clasificar 5 millones de reseñas cada noche: **Batch**.

## 📌 Diferencias clave

| Par | Distinción |
|---|---|
| Temperature vs top_p | Ambos controlan aleatoriedad; temperature reescala probabilidades, top_p recorta la cola. Ajusta uno |
| Max tokens vs ventana de contexto | Límite de la respuesta vs límite total (entrada + salida) del modelo |
| Standard vs Provisioned | Pago por uso, rendimiento compartido vs capacidad reservada garantizada |
| Global vs Data Zone vs Regional | Cualquier región / zona geográfica / región concreta (residencia de datos) |
| Batch vs Realtime | Asíncrono y barato vs interactivo |
| Deployment name vs model name | El nombre que tú das al despliegue (lo que usa el código) vs el nombre del modelo del catálogo |

## 🧠 Memorizar

> [!important]
> - Flujo: **catálogo → Deploy → playground → Instructions → Tools/Knowledge → Save as agent / View code**.
> - **Temperature baja = determinista; alta = creativa.** Ajusta temperature **o** top_p, no ambos.
> - **Max tokens** limita la **salida**.
> - Tipos: **Standard (pago por token) · Provisioned (reservado) · Batch (asíncrono)**; ámbitos **Global · Data Zone · Regional**.
> - Los despliegues están sujetos a **cuota regional**.
> - En código se usa el **deployment name**, no el nombre del modelo.

## Tips para AI-901

> [!tip]
> - ⭐ "Respuestas consistentes y repetibles para un pipeline" → temperature baja (0–0.2).
> - ⭐ "Brainstorming creativo" → temperature alta.
> - 🔥 "Los datos no deben salir de la UE" → **Data Zone (EU)** o despliegue **regional** en una región europea.
> - 🔥 "Rendimiento garantizado para una app crítica" → **Provisioned**.
> - 🔥 "Procesar millones de documentos sin prisa al menor coste" → **Batch**.
> - ⚠️ "Las respuestas se cortan a mitad" → sube **max tokens**, no temperature.
> - ⚠️ El **playground** sirve para probar sin código; para producción se usa el SDK.

## ⚠️ Errores comunes

- Subir temperature para "mejorar" respuestas factuales (empeora).
- Confundir el despliegue del modelo con el proyecto o con el agente.

## 💡 Escenario

Un banco europeo despliega un modelo para redactar respuestas a clientes. Requisitos: los datos deben procesarse dentro de la UE, las respuestas deben ser muy consistentes y no superar 200 palabras. ¿Cómo lo configuras?

<details><summary>Respuesta</summary>

Despliegue **Data Zone Standard (EU)** (o regional en una región de la UE), **temperature ≈ 0–0.2**, **max tokens** acotado (unos 300 tokens) y un prompt de sistema que fije formato y longitud.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué parámetro reduces para que un modelo dé respuestas más deterministas?
- A) Max tokens · B) Temperature · C) Past messages · D) Reasoning effort

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** Una empresa necesita procesar de forma asíncrona 10 millones de textos al mes con el menor coste posible y sin requisitos de latencia. ¿Qué tipo de despliegue?
- A) Provisioned · B) Global Standard · C) Batch · D) Managed compute

<details><summary>Respuesta</summary>

**C.**
</details>

**3.** ¿Qué opción de despliegue garantiza que las peticiones se procesen solo dentro de una zona geográfica como la Unión Europea?
- A) Global Standard · B) Data Zone Standard · C) Batch · D) Foundry Local

<details><summary>Respuesta</summary>

**B.**
</details>

**4.** Tras desplegar un modelo en el portal de Foundry, ¿dónde puedes probarlo de forma interactiva sin escribir código?
- A) Azure portal · B) Playground · C) Azure Machine Learning studio · D) Content Understanding Studio

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Catálogo de modelos de Foundry]] · [[Large Language Models]]
- [[Prompts y Prompt Engineering]] · [[Tokens y Embeddings]]
- [[Foundry SDK y cliente de chat]] · [[Azure AI Content Safety y guardrails]]
- [[Lab 02 - Desplegar un modelo y crear un agente]]

← Volver al índice: [[00 - Índice - Generative AI y agentes]]
