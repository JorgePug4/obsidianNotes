---
tags: [ai-901, azure, ia, nlp, foundry-tools, servicio]
módulo: Texto y voz
peso_examen: Muy alto
objetivo_oficial: "2.2.1 Build a lightweight application that includes text analysis"
aliases: [Azure Language, Azure Language in Foundry Tools, Text Analytics, Language service]
---

# Azure AI Language

## ¿Qué es?

**Azure Language in Foundry Tools** (antes *Azure AI Language* y, antes, *Text Analytics*) es el servicio preconstruido de Azure para **procesamiento de lenguaje natural** con resultados **estructurados y deterministas**. En el portal de Foundry se prueba en **Build → Services**.

## ¿Para qué sirve?

Añadir a una aplicación o a un agente capacidades de NLP concretas sin escribir prompts ni entrenar modelos, con salida JSON predecible y puntuaciones de confianza. El objetivo 2.2.1 pide saber **construir una aplicación ligera con análisis de texto**.

## Capacidades

| Capacidad | Qué devuelve | Estado |
|---|---|---|
| **Language detection** | Idioma principal, código ISO y confianza | ✅ Vigente en Foundry |
| **PII detection / redaction** (texto y conversaciones) | Entidades personales detectadas, categoría, confianza y **texto redactado** | ✅ Vigente en Foundry |
| **Named Entity Recognition (NER)** | Entidades clasificadas (persona, lugar, organización, fecha, cantidad…) | ✅ Vigente |
| **Sentiment analysis y opinion mining** | Positivo/negativo/neutro/mixto con puntuaciones | ⚠️ En deprecación progresiva |
| **Key phrase extraction** | Frases clave | ⚠️ En deprecación progresiva |
| **Summarization** (documento y conversación) | Resumen extractivo/abstractivo | ⚠️ En deprecación progresiva |
| **Conversational Language Understanding (CLU)** | Intenciones y entidades de una frase | ⚠️ En deprecación progresiva |
| **Custom Question Answering (CQA)** | Respuestas desde una base de preguntas | ⚠️ En deprecación progresiva |
| **Custom text classification** | Categorías propias | ⚠️ En deprecación progresiva |

> [!warning] Cambio importante (2026)
> Microsoft anunció la **deprecación progresiva** de varias funciones clásicas de Azure Language (extracción de frases clave, entity linking, sentimiento y opinion mining, resumen, CLU, custom question answering, orchestration workflow y clasificación personalizada), con retirada final prevista hacia **marzo de 2029**. La recomendación oficial es implementar esas capacidades con **modelos de Microsoft Foundry**. En los laboratorios actuales de AI-901, las funciones que se muestran en el portal son **detección de idioma** y **redacción de PII**. Para el examen: conoce las técnicas (siguen siendo objetivos conceptuales en 1.3.2) y sabe que el camino recomendado para sentimiento, frases clave y resumen es hoy un **modelo generativo**.

## Cómo funciona

1. El recurso de Foundry expone el servicio; también puede crearse un recurso de Language independiente.
2. Se prueba en el **playground** del portal (Build → Services → *Azure Language – Language detection* / *Text PII Redaction*): eliges texto de ejemplo, lo editas o subes un archivo y pulsas **Detect**.
3. La pestaña **Code** genera el código cliente de ejemplo.

Código de ejemplo del laboratorio oficial (PII):

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

client = TextAnalyticsClient(endpoint=endpoint, credential=AzureKeyCredential(key))
response = client.recognize_pii_entities(documents, language="en")
for doc in response:
    print(doc.redacted_text)
    for entity in doc.entities:
        print(entity.text, entity.category, entity.confidence_score)
```

## Ejemplo (laboratorio oficial)

Se detecta que la etiqueta de un Amstrad CPC 464 está en **alemán** y, en una factura de 1984, se identifican como PII el nombre "Margaret Ellis", la dirección "128 High Street, Reading" y el teléfono.

## 📌 Diferencias clave

| Necesidad | Azure Language | Modelo generativo |
|---|---|---|
| Redactar PII de forma auditable | ✅ Recomendado | Menos fiable |
| Detectar idioma para enrutar | ✅ Rápido y barato | Posible, más caro |
| Resumir con un estilo concreto | Limitado | ✅ Recomendado hoy |
| Analizar sentimiento con matices | Etiquetas fijas | ✅ Más flexible |
| Coste y latencia | Bajos | Mayores |

| Azure Language | Azure Translator | Azure Speech | Content Understanding |
|---|---|---|---|
| Analizar **texto** | **Traducir** texto | **Audio** ↔ texto | Extraer **campos** de archivos |

## 🧠 Memorizar

> [!important]
> - Nombre actual: **Azure Language in Foundry Tools**.
> - En el portal: **Build → Services**.
> - Lo vigente y demostrado en los labs: **detección de idioma** y **PII redaction**.
> - Devuelve **JSON + confianza**; es **determinista**.
> - Sentimiento, frases clave y resumen: concepto vigente para el examen, pero implementación recomendada con **modelos de Foundry**.

## Tips para AI-901

> [!tip]
> - ⭐ "Detectar y ocultar información personal" → Azure Language (PII).
> - ⭐ "Determinar el idioma antes de procesar" → Azure Language (language detection).
> - 🔥 "Resultados idénticos en cada ejecución para un pipeline" → servicio, no LLM.
> - ⚠️ No confundas con **Translator** (traducir) ni con **Content Understanding** (extraer campos de documentos).

## 💡 Escenario

Una plataforma recibe comentarios en varios idiomas y debe archivarlos sin datos personales y etiquetados por idioma. ¿Qué diseño?

<details><summary>Respuesta</summary>

**Azure Language**: primero *language detection* para etiquetar el idioma, después *PII redaction* para eliminar datos personales antes de archivar. Si además se quiere un resumen o el sentimiento, se añade un modelo generativo de Foundry.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué servicio usarías para detectar automáticamente en qué idioma está escrito un documento?
- A) Azure Translator · B) Azure Language · C) Azure Speech · D) Content Understanding

<details><summary>Respuesta</summary>

**B.** Translator traduce, pero la capacidad de *detección de idioma* como tal es de Azure Language (Translator también detecta el idioma de origen para traducir).
</details>

**2.** ¿Qué ventaja tiene Azure Language frente a un modelo generativo para detectar PII en un pipeline de cumplimiento?
- A) Es más creativo
- B) Devuelve resultados estructurados y deterministas con puntuaciones de confianza
- C) Genera texto nuevo
- D) No necesita endpoint

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Técnicas de análisis de texto]] · [[Procesamiento de Lenguaje Natural (NLP)]]
- [[Foundry Tools (servicios de IA de Azure)]] · [[Azure AI Translator]]
- [[Privacy and Security (Privacidad y seguridad)]]
- [[Lab 03 - Análisis de texto en Foundry]]

← Volver al índice: [[00 - Índice - Texto y voz]]
