---
tags: [ai-901, azure, ia, extraccion]
módulo: Extracción de información
peso_examen: Muy alto
objetivo_oficial: "1.3.5 Identify techniques to extract information from text, images, audio, and videos"
aliases: [Information extraction, Extracción de datos, Knowledge mining]
---

# Extracción de información

## ¿Qué es?

La **extracción de información** convierte contenido **no estructurado** (documentos, formularios, imágenes, audio, vídeo) en **datos estructurados** que un sistema puede procesar: campos, tablas, JSON.

## ¿Para qué sirve?

Automatizar procesos que hoy dependen de que una persona lea: facturas, recibos, contratos, partes de siniestro, expedientes, llamadas de soporte, vídeos de formación. Es también el paso previo para alimentar búsquedas, bases de datos y agentes.

## Técnicas por modalidad (objetivo 1.3.5)

| Modalidad | Técnicas | Qué se obtiene |
|---|---|---|
| **Texto** | Detección de **entidades (NER)**, frases clave, clasificación, PII, expresiones regulares/reglas | Entidades, categorías, valores |
| **Imágenes** | **OCR/Read** (texto), **Layout** (estructura y tablas), **extracción de campos**, análisis de imagen | Texto, tablas, campos |
| **Documentos y formularios** | OCR + **modelos preconstruidos** (factura, recibo, documento de identidad, contrato) o **esquemas propios** | Campos con valor y confianza |
| **Audio** | **Transcripción** (STT), diarización, y después extracción de campos sobre el transcript | Transcripción, temas, campos (motivo, importe, sentimiento) |
| **Vídeo** | Transcripción + análisis de fotogramas + segmentación en escenas | Resúmenes, capítulos, campos |

## Tres enfoques (y cuándo usar cada uno)

| Enfoque | Cómo funciona | Cuándo | Ejemplo |
|---|---|---|---|
| **Reglas / expresiones regulares** | Patrones fijos sobre el texto | Formatos totalmente predecibles y estables | Extraer un código con formato fijo |
| **Modelos especializados (ML)** | Modelos entrenados para un tipo de documento | Documentos estructurados y frecuentes | `prebuilt-invoice`, `prebuilt-receipt` |
| **Multimodal / generativo con esquema** | Describes en lenguaje natural los campos que quieres y el modelo los extrae de cualquier modalidad | Contenido variado o poco estructurado, esquemas a medida | Analizador propio de **Content Understanding** |

La regla práctica del examen: **formato determinista y conocido → analizador preconstruido; esquema descrito en lenguaje natural o contenido multimodal → Content Understanding**.

## Cómo funciona

```
Archivo (PDF, imagen, audio, vídeo)
   ─▶ OCR / transcripción            (obtener el contenido)
   ─▶ estructura (layout, escenas)   (entender la organización)
   ─▶ campos definidos por esquema   (asignar valores)
   ─▶ JSON con valores, confianza y grounding (dónde se encontró cada dato)
```

## Ejemplo

Una gestoría procesa 3 000 facturas al mes: Content Understanding con el analizador de facturas devuelve proveedor, NIF, fecha, base imponible, IVA y total en JSON, con la posición de cada valor en la página para revisión humana de los casos con baja confianza.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **OCR** vs **extracción de campos** | Todo el texto / valores asignados a campos |
| **Extracción** vs **visión** | Obtener datos / interpretar la escena |
| **Extracción** vs **análisis de texto** | Convertir un archivo en datos / analizar texto que ya tienes |
| **Extracción** vs **RAG** | Convertir contenido en datos estructurados / recuperar fragmentos para responder |
| **Modelo preconstruido** vs **esquema propio** | Tipo de documento estándar / campos que tú defines |

## 🧠 Memorizar

> [!important]
> - Cuatro modalidades: **texto · imágenes · audio · vídeo**.
> - Cadena habitual: **OCR/transcripción → estructura → campos → JSON**.
> - Servicio de referencia en AI-901: **[[Azure Content Understanding]]**.
> - La salida trae **confianza** y **grounding** (dónde está el dato).

## Tips para AI-901

> [!tip]
> - ⭐ "Facturas, recibos, formularios, contratos" → Content Understanding / Document Intelligence.
> - ⭐ "Extraer datos de grabaciones o vídeos" → Content Understanding (audio/vídeo), no solo Speech.
> - 🔥 "Describimos los campos que queremos en lenguaje natural" → analizador personalizado de Content Understanding.
> - ⚠️ Si solo piden el **texto**, es OCR. Si piden **campos**, es extracción.
> - ⚠️ Si piden **responder preguntas** sobre documentos, es **RAG**, no extracción.

## 💡 Escenario

Una aseguradora recibe partes en PDF, fotos de daños y llamadas grabadas, y quiere de todo ello: número de póliza, fecha del siniestro e importe reclamado. ¿Qué servicio y por qué?

<details><summary>Respuesta</summary>

**Azure Content Understanding**: un mismo analizador con el esquema de campos (póliza, fecha, importe) puede aplicarse a las **cuatro modalidades** (documento, imagen, audio y vídeo), devolviendo JSON con confianza. Speech solo transcribiría; Vision solo describiría.
</details>

## Preguntas que podrían aparecer

**1.** ¿Cuál de estas necesidades corresponde a extracción de información y no a análisis de texto?
- A) Medir el sentimiento de una reseña
- B) Obtener proveedor, fecha e importe de una factura escaneada
- C) Detectar el idioma de un correo
- D) Resumir un artículo

<details><summary>Respuesta</summary>

**B.** Las demás operan sobre texto que ya está disponible; B convierte un archivo en campos estructurados.
</details>

**2.** ¿Qué enfoque es preferible cuando el formato del documento es totalmente predecible y estable?
- A) Un analizador preconstruido o basado en reglas
- B) Un prompt abierto a un LLM
- C) Un modelo de embeddings
- D) Un agente con búsqueda web

<details><summary>Respuesta</summary>

**A.**
</details>

## Relacionado

- [[Azure Content Understanding]] · [[Azure AI Document Intelligence]]
- [[OCR (Reconocimiento óptico de caracteres)]] · [[Técnicas de análisis de texto]]
- [[Cargas de trabajo de IA]] · [[Grounding, RAG y Foundry IQ]]
- [[Lab 06 - Content Understanding (extracción de información)]]

← Volver al índice: [[00 - Índice - Extracción de información]]
