---
tags: [ai-901, azure, ia, repaso, MOC]
certificación: Azure AI Fundamentals (AI-901)
módulo: Repaso final
---

# 🎓 AI-901 · Guía de repaso final

> [!info] Cómo usar esta nota
> Es la última pasada, para el día antes del examen. Cada sección resume lo esencial y enlaza a la nota donde está el detalle. Si algo de aquí no te suena, vuelve a esa nota antes de hacer el [[AI-901 - Simulacro final (50 preguntas)|simulacro]].

## 1. Conceptos que debo memorizar

1. **Dos dominios**: conceptos y responsabilidades (40–45 %) e implementación con Microsoft Foundry (55–60 %). Aprobado: **700/1000**.
2. **Cinco cargas de trabajo**: IA generativa y agentes, análisis de texto, voz, visión, extracción de información.
3. **Seis principios de Responsible AI**: fairness, reliability and safety, privacy and security, inclusiveness, transparency, accountability.
4. Un **LLM** es un transformer que predice el **siguiente token** usando **atención**; los tokens se representan como **embeddings**.
5. **Tokens** = coste, latencia y ventana de contexto. **Embeddings** = significado y búsqueda semántica.
6. **Prompt de sistema** (Instructions) = rol, reglas y formato. **Prompt de usuario** = la tarea. **Few-shot** = ejemplos en el prompt.
7. **Grounding/RAG** reduce alucinaciones y aporta citas. **Fine-tuning** cambia comportamiento, no aporta datos frescos.
8. **Foundry**: recurso → proyectos; portal **Discover / Build / Operate**; pilares **Models · Agent Service · Tools · IQ · control plane**.
9. **Agente** = modelo + instrucciones + herramientas + conocimiento, versionado y con endpoint propio.
10. **Parámetros**: temperature (aleatoriedad), top_p (alternativa), max tokens (longitud de la salida).
11. **Despliegues**: Standard (pago por token), Provisioned (reservado), Batch (asíncrono); ámbitos Global, Data Zone, Regional.
12. **Content Understanding** cubre **cuatro modalidades**: documentos, imágenes, audio y vídeo.
13. **Content Safety**: 4 categorías (odio, sexual, violencia, autolesión), severidades safe/low/medium/high, **Prompt Shields**, **groundedness**, **protected material**.
14. **Foundry Tools** = antes Azure AI Services = antes Cognitive Services. Resultados **deterministas**.
15. El cliente de **proyecto** (`AIProjectClient`) exige **identidad de Entra**, nunca clave.

## 2. Servicios que debo conocer

| Servicio | Propósito | Concepto clave |
|---|---|---|
| [[Microsoft Foundry]] | Plataforma para apps y agentes de IA | Recurso → proyecto; Discover/Build/Operate |
| [[Catálogo de modelos de Foundry]] | Elegir y desplegar modelos | Inference task, Direct from Azure |
| [[Agentes de IA (Foundry Agent Service)]] | Crear y ejecutar agentes | Instrucciones + herramientas + conocimiento |
| [[Grounding, RAG y Foundry IQ]] | Conocimiento para agentes | Knowledge base, citas, answer synthesis |
| [[Foundry SDK y cliente de chat]] | Consumir modelos y agentes desde código | Responses API, `AIProjectClient` |
| [[Azure AI Language]] | Análisis de texto determinista | Idioma, PII, entidades |
| [[Azure AI Speech]] | Voz a texto, texto a voz, Voice Live | Agentes de voz en tiempo real |
| [[Azure AI Translator]] | Traducción de texto y documentos | Conserva formato |
| [[Azure AI Vision]] | Análisis de imágenes | Caption, tags, objetos, OCR |
| [[Reconocimiento facial (Face)]] | Caras | 1:1 verificación, 1:N identificación, liveness |
| [[Azure Content Understanding]] | Extracción multimodal | Analizadores, campos, grounding |
| [[Azure AI Document Intelligence]] | Documentos y formularios | Read, Layout, prebuilt, custom |
| [[Azure AI Content Safety y guardrails]] | Seguridad de entradas y salidas | 4 categorías, Prompt Shields |
| [[Azure Machine Learning]] | Entrenar modelos propios | 🟡 distractor en AI-901 |

## 3. Comparaciones que más se confunden

| Par | Distinción en una línea |
|---|---|
| Foundry vs Azure Machine Learning | Usar modelos y agentes / entrenar modelos propios |
| Foundry Tools vs modelo generativo | Determinista y estructurado / flexible y guiado por prompt |
| Modelo desplegado vs agente | Responde / decide y usa herramientas |
| Prompt de sistema vs de usuario | Reglas permanentes / tarea del turno |
| Few-shot vs fine-tuning | Ejemplos en el prompt / reentrenar |
| RAG vs fine-tuning | Datos en la consulta / comportamiento del modelo |
| File search vs Foundry IQ | Un agente y pocos archivos / varios agentes y fuentes centralizadas |
| Web search vs file search | Internet / tus documentos |
| Temperature vs max tokens | Aleatoriedad / longitud |
| Max tokens vs ventana de contexto | Salida / entrada + salida |
| Standard vs Provisioned vs Batch | Pago por uso / reservado / asíncrono |
| Global vs Data Zone vs Regional | Cualquier región / zona geográfica / región fija |
| OpenAI SDK vs `AIProjectClient` | Modelo con clave o Entra / proyecto solo con Entra |
| Fairness vs inclusiveness | Resultados justos / acceso para todos |
| Safety vs security | Sin daño / sin ataques |
| Transparency vs accountability | Entender / responder |
| Frases clave vs entidades | Temas / elementos clasificados |
| Entidades vs PII | Cualquier entidad / datos personales redactables |
| STT vs TTS | Voz→texto / texto→voz |
| Voice Live vs STT+TTS | Tiempo real con interrupciones / pasos encadenados |
| Speech vs Translator | Audio / texto |
| Clasificación vs detección de objetos | Etiqueta global / objetos con coordenadas |
| Multimodal vs text to image | Entiende imágenes / genera imágenes |
| Detección facial vs identificación | Hay cara / quién es |
| OCR vs Layout vs campos | Texto / estructura y tablas / valores por campo |
| Content Understanding vs Vision | Campos / descripción y etiquetas |
| Content Understanding vs Speech | Campos de audio / transcripción |
| Content Understanding vs Document Intelligence | Multimodal con esquema en lenguaje natural / documentos con modelos entrenados |

## 4. Escenarios rápidos

Están en su propia nota: [[AI-901 - Escenarios rápidos (Problema → Tecnología)]]. Repásala en voz alta hasta responder sin pensar.

## 5. Responsible AI en una tabla

| Principio | Palabra clave del enunciado | Herramienta típica |
|---|---|---|
| Fairness | sesgo, grupos, discriminación | Evaluaciones por subgrupo |
| Reliability and safety | fallo, daño, alucinación, contenido dañino | Guardrails, groundedness, humano en el bucle |
| Privacy and security | PII, cifrado, jailbreak, acceso | PII redaction, Prompt Shields, Entra ID |
| Inclusiveness | discapacidad, idiomas, accesibilidad | Speech, Translator, descripciones de imagen |
| Transparency | explicar, informar, citar, limitaciones | Transparency notes, citas, tracing |
| Accountability | responsable, gobernanza, auditoría, apelación | Marco de gobierno, evaluaciones, revisión humana |

## 6. Mapa conceptual · Generative AI y agentes

```
Generative AI
 ├─ Large Language Models ── transformer, atención, siguiente token
 │    └─ Tokens y Embeddings ── ventana de contexto, búsqueda vectorial
 ├─ Prompts ── sistema vs usuario, few-shot, formato
 ├─ Grounding / RAG ── file search, Foundry IQ, citas, web search
 ├─ Catálogo de modelos ── tarea, modalidad, coste, región
 │    └─ Despliegue ── Standard / Provisioned / Batch · Global / Data Zone / Regional
 │         └─ Parámetros ── temperature, top_p, max tokens
 ├─ Foundry SDK ── Responses API, AIProjectClient, DefaultAzureCredential
 └─ Agentes ── modelo + instrucciones + herramientas + conocimiento → agent_reference
```

## 7. Mapa conceptual · Texto y voz

```
NLP
 ├─ Análisis de texto ── frases clave · entidades · sentimiento · resumen (+ idioma, PII)
 │    └─ Azure AI Language (determinista) vs modelo generativo (flexible)
 ├─ Voz ── STT (tiempo real, batch, diarización) · TTS (voces neuronales, SSML)
 │    └─ Azure AI Speech ── Voice Live (agentes de voz, interrupciones)
 └─ Traducción ── Azure AI Translator (texto y documentos) · Speech (voz)
```

## 8. Mapa conceptual · Computer Vision

```
Computer Vision
 ├─ Analizar ── clasificación · detección de objetos · caption · tags · OCR · caras
 │    ├─ Azure AI Vision (estructurado, coordenadas, escala)
 │    ├─ Face (1:1 verificación, 1:N identificación, liveness · Limited Access)
 │    └─ Modelos multimodales (imagen en el prompt, razonamiento)
 └─ Generar ── text to image (gpt-image, FLUX) · edición · vídeo (Sora)
```

## 9. Mapa conceptual · Extracción de información

```
Extracción de información
 └─ Azure Content Understanding ── documentos · imágenes · audio · vídeo
      ├─ OCR/Read ── texto
      ├─ Layout ── párrafos, tablas, Markdown
      ├─ Dominio ── prebuilt-invoice · prebuilt-receipt · prebuilt-idDocument
      ├─ RAG ── prebuilt-documentSearch · prebuilt-videoSearch
      └─ Personalizados ── campos descritos en lenguaje natural
   (Document Intelligence: Read · Layout · prebuilt · custom, integrado aquí)
```

## 10. Errores frecuentes

1. Responder "Azure Machine Learning" a escenarios que solo **usan** modelos.
2. Confundir **fairness** con **inclusiveness**, y **safety** con **security**.
3. Elegir **fine-tuning** cuando el problema es falta de **datos** (es RAG).
4. Usar **Azure Vision** para facturas (eso es Content Understanding).
5. Usar **Azure Speech** para "extraer campos de una llamada" (transcribe, no extrae campos).
6. Usar **Translator** para audio (es Speech).
7. Confundir **clasificación** con **detección de objetos** (la clave son las coordenadas).
8. Creer que la **clave de API** vale para `AIProjectClient`.
9. Subir la **temperature** para mejorar respuestas factuales.
10. Confundir **max tokens** con la **ventana de contexto**.
11. Olvidar que **PII** no es una categoría de Content Safety.
12. Pensar que un modelo **aprende** de las conversaciones en producción.
13. Confundir el **Azure OpenAI endpoint** con el **Project endpoint**.
14. Olvidar que **dall-e-3 está retirado** y que las imágenes se generan con **gpt-image** y similares.
15. Dar por hecho que sigue vigente el temario de **AI-900** ([[AI-900 vs AI-901 - Qué cambió]]).

## 11. Checklist final

- [ ] **Dominio 1.1** · Responsible AI: los seis principios y sus escenarios.
- [ ] **Dominio 1.2** · Modelos: cómo funcionan, cómo elegirlos, despliegue y parámetros.
- [ ] **Dominio 1.3** · Cargas de trabajo: texto, voz, visión, extracción, GenAI y agentes.
- [ ] **Dominio 2.1** · Apps generativas y agentes en Foundry.
- [ ] **Dominio 2.2** · Texto y voz en Foundry.
- [ ] **Dominio 2.3** · Visión y generación de imágenes en Foundry.
- [ ] **Dominio 2.4** · Extracción de información con Content Understanding.
- [ ] Repasos finales de los seis módulos completados.
- [ ] Laboratorios 01 a 08 realizados (o al menos leídos con el portal abierto).
- [ ] [[AI-901 - Escenarios rápidos (Problema → Tecnología)]] respondido sin mirar.
- [ ] [[AI-901 - Simulacro final (50 preguntas)|Simulacro]] aprobado por encima del 85 %.
- [ ] Fallos del simulacro repasados en su nota de origen.

---

Volver al índice general: [[00 - AI-901 Índice general]]
