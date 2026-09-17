---
tags: [ai-901, azure, ia, repaso, escenarios]
módulo: Repaso final
---

# ⚡ AI-901 · Escenarios rápidos (Problema → Tecnología)

> [!tip] Cómo usar esta nota
> Tapa la columna derecha y responde en voz alta. El examen es, en gran parte, esta tabla disfrazada de escenarios. Si dudas en alguna, abre la nota enlazada.

## Generative AI y agentes

| Necesidad | Respuesta |
|---|---|
| Redactar, resumir o traducir con flexibilidad | Modelo generativo del catálogo ([[Generative AI]]) |
| Que el asistente use información de Internet actual | Herramienta **web search** del agente |
| Que responda con documentos internos, con citas | **RAG**: file search o **Foundry IQ** ([[Grounding, RAG y Foundry IQ]]) |
| Que varios agentes compartan las mismas fuentes | **Foundry IQ** |
| Que ejecute cálculos o genere gráficos de un CSV | Herramienta **code interpreter** |
| Que consulte nuestra API interna | Herramienta **function calling / OpenAPI** |
| Que realice tareas de varios pasos por su cuenta | **Agente** ([[Agentes de IA (Foundry Agent Service)]]) |
| Cambiar el estilo o formato de todas las respuestas | **Prompt de sistema** (Instructions) |
| Que use nuestras cinco etiquetas exactas al clasificar | **Few-shot** + temperature baja |
| Especializar el modelo en un estilo con muchos ejemplos | **Fine-tuning** |
| Respuestas consistentes y repetibles | **Temperature baja** |
| Respuestas más creativas y variadas | **Temperature alta** |
| Las respuestas se cortan | Subir **max tokens** |
| Bajar el coste de un chatbot sencillo | Modelo **mini/nano** o SLM (Phi) |
| Ejecutar el modelo en el dispositivo, sin conexión | **SLM / Foundry Local** |
| Rendimiento garantizado para una app crítica | Despliegue **Provisioned** |
| Procesar millones de textos de noche al menor coste | Despliegue **Batch** |
| Los datos no pueden salir de la UE | **Data Zone (EU)** o despliegue regional |
| Probar un modelo sin escribir código | **Playground** del portal |
| Probar un agente en una web sencilla | **Publish → Preview web app** |
| Cliente de chat en Python | OpenAI SDK, `responses.create(model, instructions, input)` |
| Cliente de un agente en Python | `AIProjectClient` + `DefaultAzureCredential` + `agent_reference` |
| Evitar poner claves en el código | **`DefaultAzureCredential`** (identidad de Entra) |

## Responsible AI

| Necesidad | Respuesta |
|---|---|
| El modelo trata peor a un grupo demográfico | **Fairness**: evaluar por subgrupos |
| El modelo inventa datos | **Reliability**: grounding + groundedness detection |
| Bloquear contenido violento o de autolesión | **Filtros de contenido / guardrails** (safety) |
| "Ignora tus instrucciones anteriores" | **Prompt Shields** (security) |
| Instrucciones ocultas en un documento que lee el agente | **Prompt Shields** (inyección indirecta) |
| Eliminar nombres y teléfonos antes de archivar | **PII redaction** (Azure Language) |
| Evitar reproducir contenido con copyright | **Protected material detection** |
| La app debe servir a personas con discapacidad | **Inclusiveness**: Speech, subtítulos, descripciones |
| Publicar límites y capacidades del sistema | **Transparency** |
| Nombrar responsables y poder auditar decisiones | **Accountability** |

## Texto

| Necesidad | Respuesta |
|---|---|
| Saber si los comentarios son positivos o negativos | **Análisis de sentimiento** |
| Identificar personas, lugares, fechas y organizaciones | **Detección de entidades (NER)** |
| Obtener los temas principales | **Extracción de frases clave** |
| Condensar un documento largo | **Resumen** |
| Saber en qué idioma está un texto | **Detección de idioma** ([[Azure AI Language]]) |
| Ocultar datos personales | **PII redaction** |
| Traducir texto o documentos conservando formato | [[Azure AI Translator]] |

## Voz

| Necesidad | Respuesta |
|---|---|
| Convertir voz en texto | **Speech to text** ([[Azure AI Speech]]) |
| Convertir texto en voz | **Text to speech** |
| Transcribir miles de grabaciones sin prisa | **Batch transcription** |
| Saber qué participante dice cada frase | **Diarización** |
| Ajustar tono, pausas y pronunciación | **SSML** |
| Conversación de voz natural con interrupciones | **Voice Live** (Voice mode del agente) |
| Traducir una conversación hablada | **Speech translation** |

## Visión

| Necesidad | Respuesta |
|---|---|
| ¿Qué aparece en la foto? (una etiqueta) | **Clasificación de imágenes** |
| Localizar y contar objetos | **Detección de objetos** (bounding boxes) |
| Describir la imagen en una frase | **Caption** ([[Azure AI Vision]]) |
| Etiquetar millones de imágenes con coordenadas | **Azure Vision** |
| Chatear sobre una foto que sube el usuario | **Modelo multimodal** ([[Modelos multimodales (visión en prompts)]]) |
| Leer el texto de una imagen | **OCR / Read** |
| Comparar un selfie con el documento | **Face – verificación 1:1** |
| Identificar quién es entre empleados registrados | **Face – identificación 1:N** (Limited Access) |
| Evitar suplantación con una foto impresa | **Face – liveness** |
| Entrenar un detector con nuestras imágenes | **Custom Vision** |
| Crear ilustraciones desde texto | Modelo **text to image** (gpt-image, FLUX) |
| Crear un vídeo corto desde texto | **Sora-2** |

## Extracción de información

| Necesidad | Respuesta |
|---|---|
| Solo el texto de un PDF escaneado | **OCR / Read** |
| Texto con párrafos y tablas | **Layout** |
| Proveedor, fecha e importe de una factura | **`prebuilt-invoice`** ([[Azure Content Understanding]]) |
| Campos de un ticket de compra | **`prebuilt-receipt`** |
| Campos de un DNI o pasaporte | **`prebuilt-idDocument`** |
| Campos de nuestro formulario propio, con ejemplos etiquetados | Modelo **custom** de Document Intelligence |
| Campos descritos en lenguaje natural sobre contenido variado | Analizador **personalizado** de Content Understanding |
| Motivo y producto mencionados en una llamada | **Content Understanding (audio)** |
| Capítulos y datos de un vídeo | **Content Understanding (vídeo)** |
| Indexar documentos para que un agente los consulte | **Foundry IQ** / `prebuilt-documentSearch` |

## Plataforma

| Necesidad | Respuesta |
|---|---|
| Entrenar un modelo con datos históricos etiquetados | [[Azure Machine Learning]] (AutoML) |
| Crear un asistente sin código para Teams | **Copilot Studio** |
| Crear un agente a medida con SDK y evaluaciones | **Microsoft Foundry** |
| Organizar modelos, agentes y datos de una solución | **Proyecto** de Foundry |
| Revisar cuota y administración | Página **Operate** |
| Probar los servicios preconstruidos | **Build → Services** (Foundry Tools) |

---

Volver a: [[AI-901 - Guía de repaso final]] · [[00 - AI-901 Índice general]]
