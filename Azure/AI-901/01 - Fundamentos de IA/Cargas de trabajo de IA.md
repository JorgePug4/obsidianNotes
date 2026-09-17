---
tags: [ai-901, azure, ia, fundamentos, cargas-de-trabajo]
módulo: Fundamentos de IA
peso_examen: Muy alto
objetivo_oficial: "1.3.1 Identify scenarios for common AI workloads"
---

# Cargas de trabajo de IA

## ¿Qué es?

Una **carga de trabajo de IA** (*AI workload*) es un tipo de problema que la IA resuelve. La guía oficial de AI-901 nombra **cinco** que debes reconocer en escenarios: **IA generativa y agéntica**, **análisis de texto**, **voz**, **visión artificial** y **extracción de información**.

## ¿Para qué sirve?

Es la habilidad más transversal del examen: casi todas las preguntas de escenario empiezan por identificar la carga de trabajo, y de ahí sale el servicio o el modelo correcto.

## Las cinco cargas de trabajo oficiales

| Carga de trabajo | Qué hace | Entrada típica | Salida típica | Dónde se implementa en Foundry |
|---|---|---|---|---|
| **IA generativa y agentes** | Genera texto, código, imágenes; los agentes además usan herramientas para completar tareas | Prompt (texto, imagen, audio) | Contenido nuevo, acciones | Modelos del catálogo, Foundry Agent Service. Ver [[Generative AI]] |
| **Análisis de texto** | Extrae significado de texto: palabras clave, entidades, sentimiento, resumen, idioma, PII | Texto | Datos estructurados (JSON) o resumen | Azure Language in Foundry Tools o un modelo generativo. Ver [[Técnicas de análisis de texto]] |
| **Voz (Speech)** | Reconocimiento (voz → texto) y síntesis (texto → voz), traducción hablada, agentes de voz | Audio | Texto / audio | Azure Speech in Foundry Tools (Voice Live), modelos multimodales. Ver [[Reconocimiento y síntesis de voz]] |
| **Visión artificial** | Interpreta imágenes y vídeo: descripción, etiquetas, objetos, caras, OCR; y genera imágenes | Imagen / vídeo / prompt | Etiquetas, descripciones, coordenadas, imágenes nuevas | Modelos multimodales, modelos de imagen, Azure Vision in Foundry Tools. Ver [[Computer Vision]] |
| **Extracción de información** | Convierte contenido no estructurado (documentos, formularios, imágenes, audio, vídeo) en datos estructurados | Archivo multimodal | Campos + JSON con confianza | Azure Content Understanding in Foundry Tools. Ver [[Extracción de información]] |

## Cómo se decide en una pregunta

1. Localiza el **tipo de entrada** (texto, audio, imagen, documento, prompt).
2. Localiza el **resultado deseado** (crear contenido, clasificar, transcribir, extraer campos, actuar).
3. Fíjate en si pide **resultado determinista/estructurado** (servicio preconstruido) o **flexible en lenguaje natural** (modelo generativo).
4. Si el sistema debe **realizar acciones** o **usar herramientas** → agente.

## Ejemplo

Un ayuntamiento recibe solicitudes ciudadanas por correo, formularios escaneados y mensajes de voz.

| Necesidad | Carga de trabajo |
|---|---|
| Detectar el idioma y el sentimiento de los correos | Análisis de texto |
| Leer los campos de los formularios escaneados | Extracción de información |
| Transcribir los mensajes de voz | Voz |
| Redactar automáticamente una respuesta borrador | IA generativa |
| Un asistente que consulte el estado del expediente y agende una cita | Agente |

## 📌 Diferencias clave

| Par que se confunde | Cómo distinguirlo |
|---|---|
| **Análisis de texto** vs **IA generativa** | Análisis = extraer datos del texto que ya existe. Generativa = crear texto nuevo. Resumir se considera técnica de análisis de texto, aunque lo haga un LLM |
| **Visión** vs **Extracción de información** | Visión = entender qué hay en la imagen (objetos, personas, descripción). Extracción = sacar **datos** (campos de una factura, texto de un cartel, transcripción de un vídeo) |
| **Voz** vs **Análisis de texto** | Voz = la entrada o salida es **audio**. Una vez transcrito, lo que hagas con el texto es análisis de texto |
| **Modelo generativo** vs **Agente** | Un modelo responde. Un agente además **decide y usa herramientas** (búsqueda web, archivos, APIs) para completar una tarea |
| **Servicio preconstruido (Foundry Tools)** vs **Modelo generativo** | Preconstruido = salida estructurada, predecible, sin prompt. Generativo = flexible, guiado por prompt, puede variar |

## 🧠 Memorizar

> [!important]
> - Las 5 cargas de trabajo: **GenAI/agentes · texto · voz · visión · extracción de información**.
> - **Multimodal** cruza cargas: un mismo modelo puede ver una imagen y responder en texto.
> - Extracción de información = **Azure Content Understanding** (documentos, imágenes, audio y vídeo).

## Tips para AI-901

> [!tip]
> - ⭐ Palabras clave → carga: "resumir/redactar/generar" → GenAI · "sentimiento/entidades/frases clave/PII" → texto · "transcribir/locución/dictar" → voz · "identificar objetos/describir foto/caras" → visión · "facturas/formularios/campos/OCR/transcripción de vídeo con campos" → extracción.
> - 🔥 "El asistente debe **consultar** sistemas y **realizar** acciones" → agente, no un simple chat.
> - ⚠️ OCR aparece tanto en visión como en extracción. Si solo quieren el texto de una imagen → visión/OCR; si quieren **campos** (total, fecha, proveedor) → Content Understanding.

## ⚠️ Errores comunes

- Elegir un modelo generativo cuando el enunciado exige **resultados consistentes en un pipeline automatizado** (ahí gana Foundry Tools).
- Confundir "generar una imagen" (GenAI/visión generativa) con "analizar una imagen" (visión).

## 💡 Escenario

Una cadena de restaurantes quiere procesar automáticamente miles de tickets de compra fotografiados por los empleados y obtener proveedor, fecha e importe total en JSON. ¿Qué carga de trabajo es y qué herramienta usarías?

<details><summary>Respuesta</summary>

**Extracción de información** con **Azure Content Understanding** (analizador *prebuilt-receipt*). No es "visión" porque el objetivo no es describir la foto, sino extraer campos estructurados.
</details>

## Preguntas que podrían aparecer

**1.** Una empresa quiere que un sistema escuche llamadas de soporte en tiempo real, responda con voz y consulte el CRM para dar el estado de un pedido. ¿Qué combinación describe mejor la solución?
- A) Análisis de texto + extracción de información
- B) Voz + agente con herramientas
- C) Visión + IA generativa
- D) Solo reconocimiento de voz

<details><summary>Respuesta</summary>

**B.** Hay audio de entrada y salida (voz) y acciones sobre sistemas externos (agente con herramientas). A no cubre voz ni acciones; C no aplica; D solo transcribe.
</details>

**2.** ¿Cuál de estos escenarios corresponde a análisis de texto y NO a IA generativa?
- A) Escribir la descripción de un producto a partir de sus características
- B) Determinar si las reseñas de un producto son positivas o negativas
- C) Crear una imagen promocional del producto
- D) Redactar un correo de disculpa a un cliente

<details><summary>Respuesta</summary>

**B.** Es análisis de sentimiento. A y D generan texto; C genera una imagen.
</details>

**3.** Relaciona cada necesidad con su carga de trabajo: (1) transcribir reuniones, (2) extraer el número de póliza de un PDF, (3) describir el contenido de fotos subidas por usuarios.

<details><summary>Respuesta</summary>

(1) Voz · (2) Extracción de información · (3) Visión artificial.
</details>

## Relacionado

- [[Inteligencia Artificial]]
- [[Generative AI]] · [[Agentes de IA (Foundry Agent Service)]]
- [[Técnicas de análisis de texto]] · [[Reconocimiento y síntesis de voz]]
- [[Computer Vision]] · [[Extracción de información]]
- [[Foundry Tools (servicios de IA de Azure)]]
- [[AI-901 - Escenarios rápidos (Problema → Tecnología)]]

← Volver al índice: [[00 - Índice - Fundamentos de IA]]
