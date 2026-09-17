---
tags: [ai-901, azure, ia, extraccion, documentos, foundry-tools, servicio]
módulo: Extracción de información
peso_examen: Medio-alto
aliases: [Document Intelligence, Form Recognizer, Azure Document Intelligence in Foundry Tools]
---

# Azure AI Document Intelligence

## ¿Qué es?

**Azure Document Intelligence in Foundry Tools** (antes *Azure AI Document Intelligence* y, antes, **Form Recognizer**) es el servicio especializado en **extraer datos de documentos**: texto, estructura, tablas, pares clave-valor y campos de tipos de documento concretos. Desde 2026 forma parte de **[[Azure Content Understanding]]**, que lo engloba y amplía a imágenes, audio y vídeo.

## ¿Para qué sirve?

Procesamiento automatizado de documentos de negocio: facturas, recibos, documentos de identidad, tarjetas de visita, formularios fiscales, contratos y formularios propios.

## Tipos de modelos

| Tipo | Qué hace | Ejemplos |
|---|---|---|
| **Read** | Extrae texto e idioma | Documentos escaneados |
| **Layout** | Texto + estructura: párrafos, **tablas**, casillas, orden de lectura | Informes, formularios |
| **Preconstruidos (prebuilt)** | Campos ya definidos para tipos comunes | Factura, recibo, documento de identidad, tarjeta de visita, W-2, contrato, cheque |
| **Personalizados (custom)** | Entrenados con **tus** formularios (pocos ejemplos etiquetados) | Un formulario interno propio |
| **Composed** | Varios modelos personalizados agrupados | Clasificar y extraer en un paso |

## Cómo funciona

```
Documento ─▶ OCR ─▶ detección de estructura ─▶ modelo (prebuilt o custom)
        ─▶ JSON: campos + valores + confianza + posición (bounding regions)
```

## Ejemplo

Una empresa entrena un modelo **personalizado** con 20 ejemplos de su formulario de pedido interno y, a partir de ahí, extrae automáticamente referencia, cliente, líneas y total de cada pedido nuevo.

## 📌 Document Intelligence vs Content Understanding

| | Document Intelligence | Content Understanding |
|---|---|---|
| Modalidades | **Documentos** (y sus imágenes) | Documentos, **imágenes, audio y vídeo** |
| Definición de campos | Modelos preconstruidos o **entrenamiento** con ejemplos etiquetados | Esquema descrito en **lenguaje natural** |
| Enfoque | Determinista, orientado a formularios estructurados | Multimodal y generativo, esquemas flexibles |
| Estado en 2026 | Integrado dentro de Content Understanding en Foundry Tools | Servicio de referencia en AI-901 |
| Cuándo elegirlo | Formulario estructurado, repetitivo y estable | Contenido variado, multimodal o esquema a medida |

Regla del examen: **formulario determinista y estructurado → prebuilt de Document Intelligence; esquema en lenguaje natural o contenido multimodal → Content Understanding**.

## 🧠 Memorizar

> [!important]
> - Nombre anterior: **Form Recognizer**.
> - Modelos: **Read · Layout · prebuilt · custom · composed**.
> - Los **prebuilt** más citados: **factura, recibo, documento de identidad, tarjeta de visita**.
> - Hoy forma parte de **Content Understanding**.

## Tips para AI-901

> [!tip]
> - ⭐ "Facturas y recibos con modelos ya listos" → prebuilt (vía Content Understanding).
> - 🔥 "Nuestro formulario interno, con ejemplos etiquetados" → modelo **personalizado**.
> - ⚠️ Si el escenario incluye **audio o vídeo**, la respuesta ya no es Document Intelligence: es Content Understanding.

## 💡 Escenario

Una empresa procesa un formulario interno propio, siempre con el mismo diseño, y dispone de decenas de ejemplos etiquetados. ¿Qué opción?

<details><summary>Respuesta</summary>

Un **modelo personalizado** de Document Intelligence (dentro de Content Understanding): el formato es estable y repetitivo, y el entrenamiento con ejemplos da resultados deterministas y precisos.
</details>

## Preguntas que podrían aparecer

**1.** ¿Cómo se llamaba anteriormente Azure AI Document Intelligence?
- A) Form Recognizer · B) Text Analytics · C) Computer Vision · D) LUIS

<details><summary>Respuesta</summary>

**A.**
</details>

**2.** ¿Qué modelo de Document Intelligence extrae tablas y la estructura de un documento sin campos predefinidos?
- A) Read · B) Layout · C) Invoice · D) Custom

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Azure Content Understanding]] · [[Extracción de información]]
- [[OCR (Reconocimiento óptico de caracteres)]]
- [[Foundry Tools (servicios de IA de Azure)]]

← Volver al índice: [[00 - Índice - Extracción de información]]
