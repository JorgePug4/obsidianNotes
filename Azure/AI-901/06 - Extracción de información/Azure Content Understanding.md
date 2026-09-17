---
tags: [ai-901, azure, ia, extraccion, content-understanding, foundry-tools, servicio]
módulo: Extracción de información
peso_examen: Muy alto
objetivo_oficial: "2.4.1 documentos y formularios · 2.4.2 imágenes · 2.4.3 audio y vídeo · 2.4.4 aplicación ligera de extracción"
aliases: [Content Understanding, Azure Content Understanding in Foundry Tools, Analizador, Analyzer]
---

# Azure Content Understanding

## ¿Qué es?

**Azure Content Understanding in Foundry Tools** es el servicio **multimodal de extracción de información** de Microsoft Foundry. Convierte contenido no estructurado (**documentos, formularios, imágenes, audio y vídeo**) en **datos estructurados en JSON**, mediante **analizadores** que extraen, clasifican y generan campos con **puntuaciones de confianza** y **grounding** (la referencia al lugar exacto del que sale cada valor).

Es el servicio con **cuatro objetivos propios** en la guía oficial: no hay otro con tanto peso en el dominio 2.

## ¿Para qué sirve?

Automatizar el procesamiento de documentos (facturas, recibos, identificaciones, contratos), leer texto de imágenes, analizar llamadas y vídeos, y alimentar con datos estructurados a aplicaciones, bases de datos y **agentes** (puede usarse como herramienta de un agente).

## Conceptos clave

- **Analizador (analyzer)**: la configuración que define **qué se extrae y cómo**. Se identifica por un `analyzer_id`.
- **Modalidad**: documento, imagen, audio o vídeo.
- **Esquema de campos**: describes **en lenguaje natural** los campos que quieres y el servicio los extrae.
- **Confianza y grounding**: cada campo trae su puntuación y la ubicación del dato en el origen.
- **Content Understanding Studio / playground**: interfaz del portal (**Build → Services → Content Understanding**) para probar analizadores con muestras o archivos propios.

## Tipos de analizadores

| Categoría | Ejemplos | Qué hacen |
|---|---|---|
| **Base (por modalidad)** | `prebuilt-document`, `prebuilt-image`, `prebuilt-audio`, `prebuilt-video` | Procesamiento básico de cada tipo de contenido |
| **Contenido y estructura** | **OCR/Read**, **Layout** | Texto plano / texto con párrafos, **tablas**, jerarquía y salida **Markdown** |
| **Específicos de dominio** | `prebuilt-invoice`, `prebuilt-receipt`, `prebuilt-idDocument`, contratos | Campos ya definidos para tipos de documento habituales |
| **Para RAG / búsqueda** | `prebuilt-documentSearch`, `prebuilt-videoSearch` | Extraen contenido con comprensión semántica para indexar |
| **Personalizados** | Los que defines tú | Esquema propio de campos, en cualquier modalidad |

## La escalera del laboratorio oficial

El lab recorre tres niveles de capacidad sobre el mismo contenido:

1. **OCR / Read** → extrae el **texto** en bruto (pestañas *Markdown*, *Paragraphs*, *Result*). Se prueba con imágenes de placas de circuito impreso.
2. **Layout** → añade **estructura**: párrafos, **tablas**, jerarquía y orden de lectura.
3. **Receipt** (analizador de dominio) → combina OCR con un modelo generativo para asignar los valores a **campos** (empresa, teléfono, fecha, importes). La pestaña **Fields** muestra el resultado legible y **Result** el JSON que recibiría una aplicación.

Resumen del propio laboratorio: *"Read extrae el texto sin interpretar estructura ni significado; Layout captura estructura y jerarquía; Receipt usa una combinación de capacidades para extraer valores y asignarlos a campos"*.

## Código (aplicación ligera, objetivo 2.4.4)

```python
from azure.ai.contentunderstanding import ContentUnderstandingClient
from azure.ai.contentunderstanding.models import AnalysisInput, AnalysisResult
from azure.core.credentials import AzureKeyCredential
from azure.identity import DefaultAzureCredential

endpoint    = "https://<recurso>.services.ai.azure.com/"
analyzer_id = "prebuilt-receipt"

credential = AzureKeyCredential(key) if key else DefaultAzureCredential()
client = ContentUnderstandingClient(endpoint=endpoint, credential=credential, api_version="2025-11-01")

poller = client.begin_analyze(analyzer_id=analyzer_id, inputs=[AnalysisInput(url=file_url)])
result: AnalysisResult = poller.result()
print(result.as_dict())
```

Claves para reconocerlo en el examen: **`ContentUnderstandingClient`**, **`analyzer_id`** (por ejemplo `prebuilt-receipt`), **`begin_analyze`** (operación **asíncrona**, de ahí el *poller*) y resultado **JSON**.

## Las cuatro modalidades (objetivos 2.4.1–2.4.3)

| Modalidad | Qué extrae | Ejemplo |
|---|---|---|
| **Documentos y formularios** | Texto, tablas, campos de factura/recibo/identificación, firmas, casillas | Automatizar cuentas a pagar |
| **Imágenes** | Texto (OCR), descripciones, campos definidos por esquema | Leer serigrafías de placas, carteles, etiquetas |
| **Audio** | Transcripción, hablantes, resumen, campos (motivo de la llamada, producto, sentimiento) | Analizar llamadas de un call center |
| **Vídeo** | Transcripción, escenas/capítulos, descripciones de fotogramas, campos | Indexar formación, moderar contenido |

Un mismo **esquema de analizador** puede aplicarse a varias modalidades, que es justo lo que pide el objetivo 2.4.4.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **Content Understanding** vs **Document Intelligence** | Multimodal (documento, imagen, audio, vídeo) con esquemas en lenguaje natural / especializado en **documentos**, ahora integrado en Content Understanding |
| **Content Understanding** vs **Azure Vision** | Extraer **campos y datos** / describir y etiquetar imágenes |
| **Content Understanding** vs **Azure Speech** | Extraer **campos** de audio/vídeo / **transcribir** y sintetizar |
| **Content Understanding** vs **modelo multimodal en un chat** | Analizador reutilizable, salida estructurada y con grounding, apto para volumen / conversación flexible |
| **OCR/Read** vs **Layout** vs **analizador de campos** | Texto / estructura / valores por campo |
| **Content Understanding** vs **RAG** | Convertir contenido en datos / recuperar contexto para responder |

## 🧠 Memorizar

> [!important]
> - **Cuatro modalidades**: documento, imagen, audio, vídeo.
> - **Analizadores**: `prebuilt-document/image/audio/video`, **OCR/Read**, **Layout**, `prebuilt-invoice`, `prebuilt-receipt`, `prebuilt-idDocument`, `prebuilt-documentSearch`, `prebuilt-videoSearch`, y **personalizados**.
> - Escalera: **Read → Layout → campos**.
> - Salida: **JSON con campos, confianza y grounding**; análisis **asíncrono** (`begin_analyze`).
> - En el portal: **Build → Services → Content Understanding**.

## Tips para AI-901

> [!tip]
> - ⭐ Si el escenario nombra **facturas, recibos, formularios, documentos de identidad** → Content Understanding.
> - ⭐ Si dice "**de documentos, imágenes, audio y vídeo**" en la misma frase → Content Understanding (es el único que cubre las cuatro).
> - 🔥 "Describimos los campos en lenguaje natural" → analizador personalizado.
> - 🔥 "Necesitamos saber de dónde salió cada dato para revisarlo" → **grounding** y confianza.
> - ⚠️ Solo el texto → **OCR/Read**. Tablas → **Layout**. Campos → analizador de dominio o propio.
> - ⚠️ En el playground, la extracción de campos puede requerir desplegar modelos.

## ⚠️ Errores comunes

- Elegir Azure Vision para facturas (describe, no extrae campos).
- Elegir Azure Speech para "extraer el importe reclamado de una llamada" (transcribe, no extrae campos).

## 💡 Escenario

Una empresa quiere automatizar el reembolso de gastos a partir de fotos de tickets que envían los empleados, obteniendo comercio, fecha, impuestos y total en JSON. ¿Qué configuras?

<details><summary>Respuesta</summary>

**Azure Content Understanding** con el analizador **`prebuilt-receipt`**: OCR + asignación de campos, salida JSON con confianza para revisar manualmente los casos dudosos. La app se construye con `ContentUnderstandingClient.begin_analyze`.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué servicio extrae información estructurada de documentos, imágenes, audio y vídeo dentro de Microsoft Foundry?
- A) Azure Vision · B) Azure Content Understanding · C) Azure Speech · D) Azure Language

<details><summary>Respuesta</summary>

**B.** Es el único que cubre las cuatro modalidades con analizadores y campos.
</details>

**2.** ¿Qué analizador usarías para obtener las **tablas** de un informe escaneado respetando filas y columnas?
- A) OCR/Read · B) Layout · C) prebuilt-invoice · D) prebuilt-audio

<details><summary>Respuesta</summary>

**B.**
</details>

**3.** En el código de una app de extracción aparece `analyzer_id = "prebuilt-receipt"` y `client.begin_analyze(...)`. ¿Qué indica `begin_analyze`?
- A) Que el análisis es síncrono e inmediato
- B) Que es una operación de larga duración (asíncrona) cuyo resultado se recoge con un poller
- C) Que se está entrenando un modelo
- D) Que se genera una imagen

<details><summary>Respuesta</summary>

**B.**
</details>

**4.** Una empresa quiere extraer el motivo de la llamada y el producto mencionado de 20 000 grabaciones. ¿Qué opción es la más adecuada?
- A) Azure Speech solo · B) Content Understanding con un analizador de audio y esquema de campos · C) Azure Translator · D) Custom Vision

<details><summary>Respuesta</summary>

**B.** Speech transcribiría, pero no asignaría los valores a campos estructurados.
</details>

## Relacionado

- [[Extracción de información]] · [[Azure AI Document Intelligence]]
- [[OCR (Reconocimiento óptico de caracteres)]] · [[Azure AI Speech]] · [[Azure AI Vision]]
- [[Foundry Tools (servicios de IA de Azure)]] · [[Agentes de IA (Foundry Agent Service)]]
- [[Lab 06 - Content Understanding (extracción de información)]]

← Volver al índice: [[00 - Índice - Extracción de información]]
