---
tags: [ai-901, ai-900, azure, ia, meta]
módulo: Repaso
---

# AI-900 vs AI-901 · Qué cambió

## ¿Qué pasó?

Microsoft anunció en 2026 la "evolución" de la certificación **Azure AI Fundamentals**. El examen **AI-900** se retiró el **30 de junio de 2026** y desde entonces la única forma de obtener la certificación es aprobar **AI-901**. Quien ya tenía la certificación no pierde nada ni tiene que hacer nada.

## Cronología

| Fecha | Hito |
|---|---|
| 15 abr 2026 | Publicación de la guía de habilidades de AI-901 |
| 21 abr 2026 | AI-901 en beta |
| jun 2026 | AI-901 disponible de forma general |
| 30 jun 2026 | Retiro definitivo de AI-900 |

## Comparación de temarios

| Aspecto | AI-900 (retirado) | AI-901 (vigente) |
|---|---|---|
| Dominios | 5: cargas de trabajo (15–20 %), ML (15–20 %), visión (15–20 %), NLP (15–20 %), GenAI (20–25 %) | 2: **conceptos y responsabilidades (40–45 %)** e **implementación con Microsoft Foundry (55–60 %)** |
| Enfoque | Describir qué hace cada servicio | Reconocer conceptos **y saber implementar** soluciones ligeras en el portal y con el SDK |
| Plataforma | Servicios individuales (Azure AI Vision, Language, Speech, Azure OpenAI, Azure ML) | **Microsoft Foundry** como plataforma unificada + **Foundry Tools** |
| Machine Learning clásico | Dominio propio (regresión, clasificación, clustering, Azure ML, AutoML, designer) | Ya no es dominio propio. Solo aparece como contexto para distinguir ML clásico de IA generativa en escenarios |
| Agentes | No | **Sí**: crear y probar un agente en el portal y consumirlo desde un cliente |
| Código | No | **Python básico**: leer/completar fragmentos con el OpenAI SDK y el Foundry SDK |
| Extracción de información | Document Intelligence, OCR | **Azure Content Understanding** (documentos, imágenes, audio, vídeo) |
| Responsible AI | Los 6 principios | Los 6 principios, con más peso en aplicarlos a IA generativa |

## Cómo se refleja en estas notas

- Todo lo que evalúa AI-901 está cubierto en las carpetas 01 a 06 y practicado en la 07.
- Los conceptos de **Machine Learning clásico** se conservan en la carpeta 01 marcados como 🟡 *contexto*, porque ayudan a razonar escenarios y porque aparecen como distractores, pero no los estudies con la profundidad de AI-900.
- Los servicios que han cambiado de nombre aparecen con su nombre actual y una nota con el anterior.

## Relacionado

- [[00 - AI-901 Índice general]]
- [[Microsoft Foundry]]
- [[Cargas de trabajo de IA]]
