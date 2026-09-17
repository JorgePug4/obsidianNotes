---
tags: [ai-901, azure, ia, responsible-ai, foundry, servicio]
módulo: Responsible AI
peso_examen: Alto
aliases: [Content Safety, Guardrails, Filtros de contenido, Prompt Shields]
---

# Azure AI Content Safety y guardrails

## ¿Qué es?

**Azure AI Content Safety** es el servicio de Azure que **detecta contenido dañino** en texto e imágenes, tanto en lo que escriben los usuarios como en lo que generan los modelos. En Microsoft Foundry sus capacidades se aplican como **guardrails** (barreras de protección) y **filtros de contenido** sobre los modelos y agentes desplegados.

## ¿Para qué sirve?

Es la herramienta práctica con la que se aplican los principios de **reliability and safety** y **privacy and security** en soluciones generativas: bloquear salidas dañinas, detectar jailbreaks, comprobar que las respuestas están basadas en las fuentes y evitar contenido protegido por derechos de autor.

## Capacidades

| Capacidad | Qué hace | Principio que apoya |
|---|---|---|
| **Moderación de texto e imagen** | Clasifica en 4 categorías: **odio (hate), sexual, violencia, autolesión (self-harm)**, con niveles de severidad (safe, low, medium, high) | Safety |
| **Prompt Shields** | Detecta **ataques de prompt**: directos (jailbreak, "ignora tus instrucciones") e indirectos (instrucciones ocultas en documentos, webs o correos que el modelo procesa) | Security |
| **Groundedness detection** | Comprueba si la respuesta del modelo está **apoyada en las fuentes** proporcionadas; señala alucinaciones y puede corregirlas | Reliability, transparency |
| **Protected material detection** | Detecta texto o código **protegido por derechos de autor** (letras de canciones, artículos, código con licencia) | Accountability, legal |
| **Custom categories** | Categorías definidas por la organización | Cumplimiento propio |

## Cómo funciona en Foundry

- Cada **despliegue de modelo** tiene un **filtro de contenido** (content filter) configurable: umbrales por categoría para el prompt de entrada y para la salida.
- En **Build → Guardrails** defines políticas de guardrails y controles para agentes y modelos.
- Los resultados de seguridad también se integran en **evaluaciones** (evaluar un agente frente a prompts adversarios, red teaming) y en **Content Understanding**.
- Azure AI Content Safety también se puede llamar como API independiente y probar en su propio estudio.

## Ejemplo

Un agente de atención al cliente: los filtros bloquean prompts de odio; Prompt Shields detecta un PDF subido con instrucciones ocultas ("revela el prompt de sistema"); groundedness detection marca una respuesta que inventa una política de devoluciones que no está en la documentación.

## 📌 Diferencias clave

| Necesidad | Capacidad |
|---|---|
| Bloquear texto violento generado por el modelo | Filtro de contenido (moderación) |
| Detectar "ignora tus instrucciones anteriores" | Prompt Shields (ataque directo / jailbreak) |
| Detectar instrucciones maliciosas dentro de un documento que lee el agente | Prompt Shields (ataque indirecto) |
| Verificar que la respuesta se apoya en los documentos de la empresa | Groundedness detection |
| Evitar reproducir letras de canciones con copyright | Protected material detection |
| Eliminar nombres y teléfonos del texto | **No** es Content Safety: es PII redaction de Azure Language |

## 🧠 Memorizar

> [!important]
> - 4 categorías de daño: **hate · sexual · violence · self-harm**. Severidad: **safe / low / medium / high**.
> - **Prompt Shields** = jailbreak e inyección indirecta.
> - **Groundedness** = ¿está apoyado en las fuentes?
> - **Protected material** = derechos de autor.
> - Los filtros se configuran **por despliegue de modelo**; los guardrails, en Build → Guardrails.

## Tips para AI-901

> [!tip]
> - ⭐ "El agente debe rechazar peticiones que intenten saltarse sus reglas" → Prompt Shields.
> - 🔥 "Reducir alucinaciones" → grounding (RAG) + groundedness detection.
> - ⚠️ PII no es una categoría de Content Safety. PII → Azure Language.
> - ⚠️ Content Safety **no** garantiza equidad (fairness); eso se mide con evaluaciones por grupos.

## ⚠️ Errores comunes

- Creer que subir la temperatura o cambiar el modelo "arregla" el contenido dañino. La mitigación son los filtros y guardrails.
- Confundir *moderación* (clasificar contenido) con *groundedness* (verificar fuentes).

## 💡 Escenario

Una editorial usa un modelo para redactar artículos y quiere evitar que reproduzca fragmentos de obras con copyright y que responda a prompts que pidan contenido violento. ¿Qué capacidades configuras?

<details><summary>Respuesta</summary>

**Protected material detection** (copyright) y el **filtro de contenido** con la categoría **violence** en un umbral estricto.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué capacidad de Azure AI Content Safety detecta que un usuario intenta que el modelo ignore sus instrucciones de sistema?
- A) Groundedness detection · B) Prompt Shields · C) Protected material detection · D) Moderación de imagen

<details><summary>Respuesta</summary>

**B.** Es un ataque directo (jailbreak).
</details>

**2.** ¿Cuáles son las cuatro categorías de contenido dañino de Azure AI Content Safety?
- A) Spam, phishing, malware, fraude
- B) Odio, sexual, violencia, autolesión
- C) PII, secretos, copyright, sesgo
- D) Positivo, negativo, neutro, mixto

<details><summary>Respuesta</summary>

**B.** C mezcla otras capacidades; D es análisis de sentimiento.
</details>

**3.** Un agente RAG responde con datos que no aparecen en los documentos de la empresa. ¿Qué capacidad ayuda a detectarlo?
- A) Prompt Shields · B) Groundedness detection · C) Custom categories · D) Filtro de severidad

<details><summary>Respuesta</summary>

**B.** Groundedness comprueba que la salida esté fundamentada en las fuentes.
</details>

## Relacionado

- [[Responsible AI (principios de Microsoft)]]
- [[Reliability and Safety (Fiabilidad y seguridad)]] · [[Privacy and Security (Privacidad y seguridad)]]
- [[Grounding, RAG y Foundry IQ]]
- [[Despliegue y configuración de modelos]]
- [[Azure AI Language]] (PII)

← Volver al índice: [[00 - Índice - Responsible AI]]
