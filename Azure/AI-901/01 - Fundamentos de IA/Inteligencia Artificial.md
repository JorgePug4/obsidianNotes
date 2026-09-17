---
tags: [ai-901, azure, ia, fundamentos]
módulo: Fundamentos de IA
peso_examen: Alto
---

# Inteligencia Artificial

## ¿Qué es?

La **Inteligencia Artificial (IA)** es software que imita capacidades humanas: entender lenguaje, reconocer imágenes, tomar decisiones a partir de datos, generar contenido nuevo o actuar de forma autónoma para completar tareas.

En la práctica, casi toda la IA actual se basa en **modelos**: funciones matemáticas entrenadas con grandes cantidades de datos que, ante una entrada (texto, imagen, audio…), producen una salida (una predicción, una etiqueta, un texto generado).

## ¿Para qué sirve?

Resuelve problemas donde escribir reglas a mano es inviable: leer millones de documentos, detectar objetos en fotos, transcribir llamadas, responder preguntas en lenguaje natural o automatizar flujos de trabajo con **agentes**.

## Conceptos clave

- **Modelo**: el "cerebro" entrenado. Recibe entradas y devuelve salidas.
- **Entrenamiento**: proceso de ajustar el modelo con datos. Ver [[Machine Learning (fundamentos)]].
- **Inferencia**: usar un modelo ya entrenado para obtener una respuesta.
- **IA predictiva (ML clásico)**: predice una etiqueta o un valor (¿es spam?, ¿cuánto venderé?).
- **IA generativa**: crea contenido nuevo (texto, imágenes, código, audio). Ver [[Generative AI]].
- **IA agéntica**: un modelo que, con instrucciones y herramientas, **realiza tareas** de varios pasos. Ver [[Agentes de IA (Foundry Agent Service)]].
- **Modelo multimodal**: acepta y/o produce varios tipos de datos (texto + imagen + audio).

## Cómo funciona (visión de conjunto)

```
Datos ──entrenamiento──▶ Modelo ──inferencia──▶ Salida
                           ▲
                 prompt / imagen / audio (entrada)
```

En AI-901 no se te pide entrenar nada. Se te pide **reconocer qué tipo de IA necesita un escenario** y **usar modelos ya entrenados** desde [[Microsoft Foundry]].

## Ejemplo

Una aseguradora recibe fotos de coches accidentados, partes escritos y llamadas telefónicas. Usa IA de visión para localizar daños en las fotos, extracción de información para leer los partes, voz para transcribir las llamadas y un agente generativo que redacta el resumen del siniestro y consulta la póliza.

## Servicios relacionados

Toda la IA en Azure se construye hoy desde **[[Microsoft Foundry]]**: modelos del catálogo (Foundry Models), agentes (Foundry Agent Service), conocimiento (Foundry IQ) y servicios preconstruidos ([[Foundry Tools (servicios de IA de Azure)]]).

## Comparaciones

| Tipo de IA | Qué hace | Ejemplo | Concepto clave |
|---|---|---|---|
| **ML predictivo** | Predice etiqueta o número a partir de datos históricos | Predecir abandono de clientes | Entrenar con datos etiquetados |
| **IA generativa** | Genera contenido nuevo a partir de un prompt | Redactar un correo, crear una imagen | Prompt → completion |
| **IA agéntica** | Ejecuta tareas de varios pasos usando herramientas | Reservar un viaje consultando APIs | Instrucciones + herramientas + conocimiento |
| **IA preconstruida (Foundry Tools)** | Capacidad lista para usar, resultado determinista | Detectar PII, transcribir audio | No se entrena; se llama a una API |

## 🧠 Memorizar

> [!important]
> - IA = software que imita capacidades humanas mediante **modelos**.
> - **Entrenar** ≠ **inferir**. En el examen casi siempre estás en inferencia.
> - Tres familias que el examen espera que distingas: **predictiva (ML clásico)**, **generativa** y **agéntica**.
> - **Multimodal** = varios tipos de entrada/salida en un mismo modelo.

## Tips para AI-901

> [!tip]
> - ⭐ Si el escenario dice "*crear*", "*redactar*", "*generar*", "*resumir en lenguaje natural*" → IA generativa.
> - ⭐ Si dice "*realizar una tarea de varios pasos*", "*usar herramientas*", "*actuar en nombre del usuario*" → agente.
> - 🔥 Si dice "*predecir un valor*", "*clasificar según datos históricos*" → ML clásico ([[Tipos de Machine Learning]]).
> - ⚠️ "Un modelo generativo también puede clasificar texto" es cierto, pero si la pregunta pide **resultados deterministas y estructurados**, la respuesta es un servicio de Foundry Tools.

## ⚠️ Errores comunes

- Pensar que toda IA "aprende sola" en producción. Los modelos preentrenados no cambian cuando los usas.
- Confundir **modelo** (el artefacto entrenado) con **servicio** (la API que lo expone) y con **agente** (modelo + instrucciones + herramientas).

## 💡 Escenario

Una tienda online quiere un asistente que responda preguntas sobre pedidos consultando su base de datos y que, si hace falta, inicie una devolución. ¿Qué tipo de IA es?

<details><summary>Respuesta</summary>

**IA agéntica.** El asistente no solo genera texto: usa herramientas (consultar pedidos, iniciar devoluciones) para completar tareas. Se construiría como un agente en Foundry con instrucciones, un modelo y herramientas.
</details>

## Preguntas que podrían aparecer

**1.** ¿Cuál de las siguientes afirmaciones describe mejor un modelo de IA generativa?
- A) Predice una categoría a partir de datos etiquetados
- B) Crea contenido nuevo (texto, imágenes, código) a partir de un prompt
- C) Agrupa datos similares sin etiquetas
- D) Ejecuta reglas definidas por un programador

<details><summary>Respuesta</summary>

**B.** A es clasificación (ML predictivo), C es clustering, D no es IA.
</details>

**2.** Verdadero o falso: usar un modelo ya entrenado para obtener una respuesta se llama *entrenamiento*.

<details><summary>Respuesta</summary>

**Falso.** Se llama **inferencia**. El entrenamiento es el proceso previo de ajustar el modelo con datos.
</details>

## Relacionado

- [[Cargas de trabajo de IA]]
- [[Machine Learning (fundamentos)]]
- [[Generative AI]]
- [[Agentes de IA (Foundry Agent Service)]]
- [[Microsoft Foundry]]

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
