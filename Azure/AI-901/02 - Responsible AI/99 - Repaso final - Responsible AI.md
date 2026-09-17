---
tags: [ai-901, azure, ia, responsible-ai, repaso]
módulo: Responsible AI
---

# 🎯 Repaso final · Responsible AI

## 1. Los seis principios en una línea

| Principio | En una frase | Palabras clave del enunciado |
|---|---|---|
| **Fairness** | Resultados justos para todos los grupos | sesgo, discriminación, grupos, paridad |
| **Reliability and safety** | Funciona bien siempre y no causa daño | inesperado, fallo, daño, alucinación, contenido dañino, pruebas |
| **Privacy and security** | Protege datos y resiste ataques | PII, cifrado, consentimiento, jailbreak, inyección, acceso |
| **Inclusiveness** | Útil y accesible para todas las personas | discapacidad, accesibilidad, idiomas, subtítulos |
| **Transparency** | Se entiende qué hace y cuáles son sus límites | explicar, informar, documentar, citas, "es una IA" |
| **Accountability** | Personas responsables y gobernanza | responsable, comité, auditoría, cumplimiento, apelación |

## 2. Herramientas de Foundry por principio

| Herramienta | Principios |
|---|---|
| Filtros de contenido / guardrails (hate, sexual, violence, self-harm) | Safety |
| Prompt Shields | Security |
| Groundedness detection, RAG con citas | Reliability, transparency |
| Protected material detection | Accountability (legal) |
| PII redaction (Azure Language) | Privacy |
| Entra ID, RBAC, Key Vault | Security |
| Speech, Translator, descripciones de imagen | Inclusiveness |
| Transparency notes, tracing, evaluaciones | Transparency, accountability |

## 3. Pares que se confunden

| Par | Distinción |
|---|---|
| Fairness vs Inclusiveness | Resultados justos vs acceso para todos |
| Safety vs Security | Sin daño vs sin ataques |
| Transparency vs Accountability | Entender vs responder |
| Content Safety vs PII redaction | Contenido dañino vs datos personales |

## 4. Tips de examen

> [!tip]
> 1. Busca la **palabra clave** del enunciado y mapea al principio.
> 2. Una alucinación es un problema de **reliability**; ocultar que se habla con IA, de **transparency**.
> 3. Jailbreak e inyección de prompt → **security** → Prompt Shields.
> 4. Si el enunciado pide "el principio que garantiza que haya alguien que responda" → accountability.
> 5. Los principios se aplican en **todo el ciclo de vida**, no al final.

## 5. Preguntas de repaso

**1.** Un modelo de reconocimiento facial funciona peor con personas de piel oscura. ¿Principio?

<details><summary>Respuesta</summary>

**Fairness** (sesgo entre grupos). Si el sistema directamente no fuera usable por ese grupo, podría argumentarse inclusiveness, pero la diferencia de rendimiento es equidad.
</details>

**2.** Antes de lanzar un agente, se ejecutan pruebas con prompts adversarios y se define un umbral para derivar a humanos. ¿Principio?

<details><summary>Respuesta</summary>

**Reliability and safety.**
</details>

**3.** Se elimina la PII de las transcripciones antes de almacenarlas. ¿Principio?

<details><summary>Respuesta</summary>

**Privacy and security** (privacidad).
</details>

**4.** La app añade lectura en voz alta y traducción a lengua de signos por avatar. ¿Principio?

<details><summary>Respuesta</summary>

**Inclusiveness.**
</details>

**5.** Cada respuesta del agente incluye las fuentes y un aviso de que puede contener errores. ¿Principio?

<details><summary>Respuesta</summary>

**Transparency.**
</details>

**6.** La empresa crea un registro de auditoría y un proceso para recurrir decisiones automatizadas ante un responsable. ¿Principio?

<details><summary>Respuesta</summary>

**Accountability.**
</details>

**7.** ¿Qué capacidad detecta instrucciones ocultas en un documento que lee el agente?

<details><summary>Respuesta</summary>

**Prompt Shields** (ataque indirecto).
</details>

**8.** ¿Qué categorías moderan los filtros de contenido de Foundry?

<details><summary>Respuesta</summary>

Odio, sexual, violencia y autolesión, con severidad safe/low/medium/high.
</details>

## Checklist

- [ ] Comprendo los seis principios y su idea central.
- [ ] Sé identificar el principio a partir de un escenario.
- [ ] Sé diferenciar los pares confusos (fairness/inclusiveness, safety/security, transparency/accountability).
- [ ] Conozco las capacidades de Content Safety y guardrails y qué principio apoya cada una.
- [ ] Puedo responder las preguntas de práctica.

Volver al índice: [[00 - Índice - Responsible AI]] · [[00 - AI-901 Índice general]]
