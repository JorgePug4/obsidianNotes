---
tags: [ai-901, azure, ia, extraccion, repaso]
módulo: Extracción de información
---

# 🎯 Repaso final · Extracción de información

## 1. Los conceptos más importantes

1. Extracción = convertir contenido **no estructurado** en **datos estructurados (JSON)**.
2. **Azure Content Understanding** cubre **cuatro modalidades**: documentos, imágenes, audio y vídeo. Es el servicio con cuatro objetivos propios en el examen.
3. Escalera de analizadores: **OCR/Read → Layout → campos**.
4. Analizadores preconstruidos: `prebuilt-document/image/audio/video`, `prebuilt-invoice`, `prebuilt-receipt`, `prebuilt-idDocument`, `prebuilt-documentSearch`, `prebuilt-videoSearch`.
5. Los analizadores **personalizados** se definen describiendo los campos en **lenguaje natural**.
6. La salida trae **confianza** y **grounding**; el análisis es **asíncrono** (`begin_analyze`).
7. **Document Intelligence** (antes Form Recognizer) es el especialista en documentos, hoy integrado en Content Understanding.

## 2. Tabla necesidad → servicio

| Necesidad | Servicio / analizador |
|---|---|
| Solo el texto de una imagen o PDF | OCR / Read |
| Texto con párrafos y tablas en Markdown | Layout |
| Campos de una factura | `prebuilt-invoice` |
| Campos de un ticket | `prebuilt-receipt` |
| Campos de un DNI o pasaporte | `prebuilt-idDocument` |
| Campos de nuestro formulario propio con ejemplos etiquetados | Modelo **custom** de Document Intelligence |
| Campos descritos en lenguaje natural sobre contenido variado | Analizador **personalizado** de Content Understanding |
| Datos de una grabación de audio | Content Understanding (audio) |
| Capítulos y datos de un vídeo | Content Understanding (vídeo) |
| Indexar documentos para RAG | `prebuilt-documentSearch` / Foundry IQ |
| Solo transcribir audio | Azure Speech |
| Solo describir una imagen | Azure Vision o modelo multimodal |

## 3. Diferencias que más se confunden

| Par confuso | Cómo distinguirlo |
|---|---|
| OCR vs Layout vs campos | Texto / estructura y tablas / valores por campo |
| Content Understanding vs Document Intelligence | Multimodal y esquemas en lenguaje natural / documentos y modelos entrenados |
| Content Understanding vs Azure Vision | Campos / descripción y etiquetas |
| Content Understanding vs Azure Speech | Campos de audio / transcripción |
| Extracción vs RAG | Datos estructurados / recuperar contexto para responder |
| Prebuilt vs custom | Tipo estándar / formulario propio |

## 4. Tips de examen

> [!tip]
> 1. "Documentos, imágenes, audio y vídeo" en la misma frase → Content Understanding.
> 2. "Total, fecha, proveedor" → campos, no OCR.
> 3. "Tablas respetando filas y columnas" → Layout.
> 4. "Describimos los campos que queremos" → analizador personalizado.
> 5. "Revisión humana de los casos dudosos" → usa la **confianza** de cada campo.
> 6. `begin_analyze` + `analyzer_id` en el código → Content Understanding.

## 5. Preguntas de repaso

**1.** ¿Qué analizador devuelve el texto en bruto de una imagen, sin estructura ni campos?

<details><summary>Respuesta</summary>

**OCR / Read.**
</details>

**2.** Una empresa quiere extraer de cada vídeo de formación los capítulos y los temas tratados. ¿Qué servicio?

<details><summary>Respuesta</summary>

**Azure Content Understanding** con un analizador de **vídeo**.
</details>

**3.** ¿Qué significa que un campo extraído tenga *grounding*?

<details><summary>Respuesta</summary>

Que el resultado indica **de dónde** procede el valor en el contenido original (página, región, marca de tiempo), lo que permite verificarlo.
</details>

**4.** Verdadero o falso: Azure Vision es la mejor opción para extraer el importe total de miles de facturas.

<details><summary>Respuesta</summary>

**Falso.** Vision describe y etiqueta imágenes; para campos de factura se usa Content Understanding (`prebuilt-invoice`).
</details>

**5.** ¿Qué diferencia principal hay entre un analizador preconstruido y uno personalizado de Content Understanding?

<details><summary>Respuesta</summary>

El preconstruido ya trae los campos definidos para un tipo de contenido habitual; el personalizado extrae los campos que tú describes en lenguaje natural, en la modalidad que necesites.
</details>

## Checklist

- [ ] Comprendo qué es la extracción de información y sus cuatro modalidades.
- [ ] Sé identificar los analizadores de Content Understanding y cuándo usar cada uno.
- [ ] Sé diferenciar OCR, Layout y extracción de campos.
- [ ] Sé diferenciar Content Understanding, Document Intelligence, Vision y Speech.
- [ ] Reconozco el código de una aplicación ligera de extracción.
- [ ] Puedo responder las preguntas de práctica.

Volver al índice: [[00 - Índice - Extracción de información]] · [[00 - AI-901 Índice general]]
