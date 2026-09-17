---
tags: [ai-901, azure, ia, simulacro, examen, respuestas]
certificación: Azure AI Fundamentals (AI-901)
módulo: Repaso final
---

# ✅ AI-901 · Simulacro final · Respuestas y explicaciones

> [!warning] No abras esta nota antes de completar el simulacro
> Las preguntas están en [[AI-901 - Simulacro final (50 preguntas)]]. Cada respuesta incluye por qué las demás opciones son incorrectas y la nota donde repasar el concepto.

## Plantilla rápida

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|
| B | C | B | C | B | B | C | B | B | C |

| 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 |
|---|---|---|---|---|---|---|---|---|---|
| A | B | C | B | B | B | A | C | B | B |

| 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 |
|---|---|---|---|---|---|---|---|---|---|
| B | C | B | B | B | B | C | B | B | B |

| 31 | 32 | 33 | 34 | 35 | 36 | 37 | 38 | 39 | 40 |
|---|---|---|---|---|---|---|---|---|---|
| B | B | C | C | A | B | B | C | B | B |

| 41 | 42 | 43 | 44 | 45 | 46 | 47 | 48 | 49 | 50 |
|---|---|---|---|---|---|---|---|---|---|
| B | B | B | B | B | B | C | C | B | B |

---

## Dominio 1 · Conceptos y responsabilidades

**1. Respuesta correcta: B (IA generativa).**
Explicación: crear texto nuevo a partir de datos de entrada es generación de contenido.
Por qué las otras son incorrectas: A extraería información del texto existente, no lo crearía; C convierte archivos en datos estructurados; D trabaja con imágenes.
Concepto relacionado: [[Cargas de trabajo de IA]] · [[Generative AI]]

**2. Respuesta correcta: C (Fairness).**
Explicación: resultados distintos para personas con méritos equivalentes según un atributo (procedencia geográfica) es un problema de equidad.
Por qué las otras son incorrectas: A se refiere a explicar el sistema; B a que funcione sin causar daño; D a que existan responsables y gobernanza.
Concepto relacionado: [[Fairness (Equidad)]]

**3. Respuesta correcta: B (predice el siguiente token).**
Explicación: los modelos generativos producen la salida token a token según las probabilidades condicionadas por el contexto.
Por qué las otras son incorrectas: A describe un buscador; C describe un clasificador; D describe software basado en reglas.
Concepto relacionado: [[Large Language Models]]

**4. Respuesta correcta: C (Transparency).**
Explicación: documentar capacidades, datos y limitaciones e informar de que se interactúa con una IA es exactamente el principio de transparencia.
Por qué las otras son incorrectas: A busca que el sistema sea accesible para todos; B protege datos y frente a ataques; D trata la equidad de los resultados.
Concepto relacionado: [[Transparency (Transparencia)]]

**5. Respuesta correcta: B (representaciones vectoriales del significado).**
Explicación: los embeddings sitúan textos con significado parecido cerca en un espacio vectorial, lo que habilita la búsqueda semántica y RAG.
Por qué las otras son incorrectas: A son los guardrails; C son parámetros como temperature; D son las tools del agente.
Concepto relacionado: [[Tokens y Embeddings]]

**6. Respuesta correcta: B (Reliability and safety).**
Explicación: los umbrales de confianza con revisión humana evitan que un error del modelo cause daño.
Por qué las otras son incorrectas: A mediría sesgos entre grupos; C se refiere a accesibilidad; D a informar y explicar.
Concepto relacionado: [[Reliability and Safety (Fiabilidad y seguridad)]]

**7. Respuesta correcta: C (detección de entidades).**
Explicación: el reconocimiento de entidades nombradas localiza y **clasifica** elementos en categorías como persona, organización, ubicación o fecha.
Por qué las otras son incorrectas: A devuelve temas sin clasificar; B mide la polaridad de la opinión; D solo identifica el idioma.
Concepto relacionado: [[Técnicas de análisis de texto]]

**8. Respuesta correcta: B (un SLM).**
Explicación: los modelos de lenguaje pequeños, como la familia Phi, están pensados para bajo coste, baja latencia y ejecución local.
Por qué las otras son incorrectas: A exige mucha capacidad de cómputo; C solo vectoriza texto; D genera vídeo.
Concepto relacionado: [[Large Language Models]] · [[Catálogo de modelos de Foundry]]

**9. Respuesta correcta: B (odio, sexual, violencia y autolesión).**
Explicación: son las cuatro categorías de la moderación de contenido, con severidades safe, low, medium y high.
Por qué las otras son incorrectas: A pertenece a la seguridad del correo; C mezcla otras capacidades (PII, protected material); D son los valores del análisis de sentimiento.
Concepto relacionado: [[Azure AI Content Safety y guardrails]]

**10. Respuesta correcta: C (Inclusiveness).**
Explicación: ofrecer varios canales, varios idiomas y compatibilidad con tecnologías de asistencia amplía el acceso a todas las personas.
Por qué las otras son incorrectas: A trata de la gobernanza; B de la equidad de resultados; D de la fiabilidad del sistema.
Concepto relacionado: [[Inclusiveness (Inclusión)]]

**11. Respuesta correcta: A (el número de tokens de entrada y salida).**
Explicación: la facturación y la ventana de contexto se miden en tokens.
Por qué las otras son incorrectas: B afecta a la aleatoriedad, no al coste; C no determina el precio por llamada; D influye en disponibilidad y residencia, no en el consumo.
Concepto relacionado: [[Tokens y Embeddings]]

**12. Respuesta correcta: B (regresión con datos históricos).**
Explicación: hay histórico etiquetado y se busca predecir un valor numérico: es machine learning supervisado de regresión.
Por qué las otras son incorrectas: A no aprovecha los datos históricos de forma fiable para una predicción numérica; C extrae datos de archivos; D busca información pública.
Concepto relacionado: [[Tipos de Machine Learning]] · [[Azure Machine Learning]]

**13. Respuesta correcta: C (atención).**
Explicación: el mecanismo de self-attention pondera la relevancia de cada token del contexto respecto a los demás, aunque estén lejos.
Por qué las otras son incorrectas: A divide el texto en tokens; B aporta el orden; D es una operación interna de estabilización.
Concepto relacionado: [[Large Language Models]]

**14. Respuesta correcta: B (jailbreak; Prompt Shields).**
Explicación: es un intento de anular las instrucciones del sistema, es decir, un ataque de prompt directo, que detectan los Prompt Shields.
Por qué las otras son incorrectas: A corresponde a información inventada; C a sesgos; D a contenido con derechos de autor.
Concepto relacionado: [[Azure AI Content Safety y guardrails]] · [[Privacy and Security (Privacidad y seguridad)]]

**15. Respuesta correcta: B (diarización).**
Explicación: la diarización separa e identifica los distintos hablantes dentro de una misma grabación.
Por qué las otras son incorrectas: A genera voz; C convierte entre alfabetos; D traduce el contenido hablado.
Concepto relacionado: [[Reconocimiento y síntesis de voz]]

**16. Respuesta correcta: B (detección de objetos).**
Explicación: localizar y contar varias instancias exige bounding boxes por objeto.
Por qué las otras son incorrectas: A asigna una sola etiqueta a la imagen completa; C genera una frase descriptiva; D lee texto.
Concepto relacionado: [[Computer Vision]]

**17. Respuesta correcta: A (1:1 frente a 1:N).**
Explicación: la verificación compara dos rostros; la identificación busca una coincidencia dentro de un grupo registrado.
Por qué las otras son incorrectas: B describe detección y calidad; C inventa una limitación de formato; D es falsa.
Concepto relacionado: [[Reconocimiento facial (Face)]]

**18. Respuesta correcta: C (extracción de información).**
Explicación: se convierten documentos, imágenes y audio en campos estructurados; es el caso de uso de Azure Content Understanding.
Por qué las otras son incorrectas: A opera sobre texto ya disponible; B se limita a interpretar imágenes; D describe un sistema que ejecuta tareas, no el objetivo aquí.
Concepto relacionado: [[Extracción de información]] · [[Azure Content Understanding]]

**19. Respuesta correcta: B (ajusta un modelo base con ejemplos propios).**
Explicación: el fine-tuning adapta comportamiento, formato o estilo partiendo de un modelo ya entrenado.
Por qué las otras son incorrectas: A describe el caso de uso de RAG; C los filtros de contenido son independientes; D entrenar desde cero es otra cosa completamente distinta.
Concepto relacionado: [[Generative AI]] · [[Grounding, RAG y Foundry IQ]]

**20. Respuesta correcta: B (Accountability).**
Explicación: comités de revisión, propietarios designados y vías de apelación son mecanismos de responsabilidad y gobernanza.
Por qué las otras son incorrectas: A sería explicar el sistema; C proteger datos; D garantizar que funciona sin causar daño.
Concepto relacionado: [[Accountability (Responsabilidad)]]

**21. Respuesta correcta: B (capacidad multimodal con entrada de imagen).**
Explicación: no todos los modelos del catálogo aceptan imágenes; hay que elegir uno con visión.
Por qué las otras son incorrectas: A es un modo de despliegue; C solo genera vectores; D es un parámetro de aleatoriedad.
Concepto relacionado: [[Modelos multimodales (visión en prompts)]]

**22. Respuesta correcta: C (resumen).**
Explicación: condensar un texto largo conservando las ideas principales es, por definición, summarization.
Por qué las otras son incorrectas: A clasifica elementos concretos; B devuelve temas sueltos; D solo identifica el idioma.
Concepto relacionado: [[Técnicas de análisis de texto]]

**23. Respuesta correcta: B (Falso).**
Explicación: un modelo desplegado no se modifica con el uso. Para cambiar su conocimiento se usa grounding; para cambiar su comportamiento, fine-tuning.
Concepto relacionado: [[Generative AI]]

---

## Dominio 2 · Implementación con Microsoft Foundry

**24. Respuesta correcta: B (el recurso es el padre y contiene proyectos).**
Explicación: el recurso de Foundry aporta los servicios en la nube; los proyectos organizan los activos de cada solución.
Por qué las otras son incorrectas: A y C invierten la jerarquía; D confunde un concepto lógico con un límite de facturación.
Concepto relacionado: [[Microsoft Foundry]]

**25. Respuesta correcta: B (el prompt de sistema).**
Explicación: en el portal de Foundry, *Instructions* es donde se define el rol, las reglas y el formato para toda la conversación.
Por qué las otras son incorrectas: A es el cuadro de chat; C se define al desplegar; D se gestiona aparte.
Concepto relacionado: [[Prompts y Prompt Engineering]]

**26. Respuesta correcta: B (Foundry IQ).**
Explicación: fundamentar las respuestas en los documentos propios y devolver citas es exactamente lo que aporta una knowledge base.
Por qué las otras son incorrectas: A aumentaría la variabilidad; C no es la vía para inyectar datos concretos y verificables; D solo alarga la respuesta.
Concepto relacionado: [[Grounding, RAG y Foundry IQ]]

**27. Respuesta correcta: C (Batch).**
Explicación: el despliegue por lotes procesa de forma asíncrona grandes volúmenes con descuento.
Por qué las otras son incorrectas: A es el modo interactivo de pago por token; B reserva capacidad para latencia estable, más caro; D es para modelos en GPU dedicada.
Concepto relacionado: [[Despliegue y configuración de modelos]]

**28. Respuesta correcta: B (envía prompt de sistema y de usuario y muestra la respuesta).**
Explicación: en la Responses API, `instructions` es el prompt de sistema, `input` el del usuario y `output_text` la respuesta.
Por qué las otras son incorrectas: A usaría `images.generate`; C necesitaría un modelo de embeddings y un índice; D sería un flujo de entrenamiento.
Concepto relacionado: [[Foundry SDK y cliente de chat]]

**29. Respuesta correcta: B (modelo, instrucciones, herramientas y conocimiento).**
Explicación: es la definición de agente que muestra el YAML del portal.
Por qué las otras son incorrectas: A pertenece al machine learning supervisado; C son opciones de despliegue; D son datos de conexión.
Concepto relacionado: [[Agentes de IA (Foundry Agent Service)]]

**30. Respuesta correcta: B (temperature cercana a 0).**
Explicación: una temperatura baja reduce la aleatoriedad en la elección de tokens y hace las respuestas más deterministas.
Por qué las otras son incorrectas: A aumenta la variabilidad; C truncaría las respuestas; D no afecta a la consistencia del texto.
Concepto relacionado: [[Despliegue y configuración de modelos]]

**31. Respuesta correcta: B (identidad de Microsoft Entra).**
Explicación: el cliente de proyecto no admite autenticación por clave porque da acceso a recursos privilegiados del proyecto.
Por qué las otras son incorrectas: A sirve para el endpoint de modelos, no para el proyecto; C y D no aplican.
Concepto relacionado: [[Foundry SDK y cliente de chat]] · [[Microsoft Entra ID]]

**32. Respuesta correcta: B (Data Zone Standard EU).**
Explicación: los despliegues de zona de datos restringen el procesamiento a la geografía indicada.
Por qué las otras son incorrectas: A y C pueden procesar en cualquier región del mundo; D ejecuta en el dispositivo, lo que no es lo que plantea el escenario.
Concepto relacionado: [[Despliegue y configuración de modelos]]

**33. Respuesta correcta: C (web search).**
Explicación: la herramienta de búsqueda web permite al agente recuperar información pública actual.
Por qué las otras son incorrectas: A consulta archivos propios; B ejecuta código; D genera imágenes.
Concepto relacionado: [[Agentes de IA (Foundry Agent Service)]]

**34. Respuesta correcta: C (Build).**
Explicación: Build es la página de desarrollo: agentes, despliegues, herramientas, conocimiento, guardrails y evaluaciones.
Por qué las otras son incorrectas: A muestra endpoints y accesos rápidos; B es el catálogo; D es administración y cuota.
Concepto relacionado: [[Microsoft Foundry]]

**35. Respuesta correcta: A (Preview web app).**
Explicación: publica el agente en una interfaz de chat básica para probarlo sin desarrollar nada.
Por qué las otras son incorrectas: B muestra límites de uso; C es el catálogo de modelos; D sirve para evaluar calidad y seguridad.
Concepto relacionado: [[Agentes de IA (Foundry Agent Service)]] · [[Lab 02 - Desplegar un modelo y crear un agente]]

**36. Respuesta correcta: B (el agente y su versión).**
Explicación: `agent_reference` indica a qué agente y a qué versión concreta se dirige la petición.
Por qué las otras son incorrectas: A se indicaría en `model`; C se configura en el agente; D se define en el despliegue.
Concepto relacionado: [[Agentes de IA (Foundry Agent Service)]]

**37. Respuesta correcta: B (un servicio especializado de Foundry Tools).**
Explicación: gran volumen, coste bajo y resultados idénticos ante la misma entrada apuntan a un servicio determinista, no a un modelo generativo.
Por qué las otras son incorrectas: A es más caro y menos predecible; C exigiría entrenar un modelo propio sin necesidad; D no aporta nada al análisis de sentimiento.
Concepto relacionado: [[Azure AI Language]] · [[Foundry Tools (servicios de IA de Azure)]]

**38. Respuesta correcta: C (Voice Live).**
Explicación: Voice Live integra reconocimiento, síntesis, detección de turnos e interrupciones en una API bidireccional en tiempo real.
Por qué las otras son incorrectas: A transcribe archivos de forma asíncrona; B crea voces personalizadas; D identifica hablantes.
Concepto relacionado: [[Azure AI Speech]]

**39. Respuesta correcta: B (Azure Language, PII).**
Explicación: la detección y redacción de información personal identificable está pensada para cumplimiento, con salida estructurada y confianza.
Por qué las otras son incorrectas: A detecta contenido dañino, no PII; C traduce; D analiza imágenes.
Concepto relacionado: [[Azure AI Language]] · [[Privacy and Security (Privacidad y seguridad)]]

**40. Respuesta correcta: B (Azure Translator).**
Explicación: la traducción de documentos procesa archivos completos conservando la maquetación.
Por qué las otras son incorrectas: A analiza texto pero no traduce documentos; C trabaja con audio; D extrae campos.
Concepto relacionado: [[Azure AI Translator]]

**41. Respuesta correcta: B (envía imagen y pregunta a un modelo multimodal).**
Explicación: el contenido del mensaje combina `input_text` e `input_image`, que es el patrón de visión en el prompt.
Por qué las otras son incorrectas: A usaría `images.generate`; C requeriría un modelo de embeddings; D sería Content Understanding.
Concepto relacionado: [[Modelos multimodales (visión en prompts)]]

**42. Respuesta correcta: B (Text to image).**
Explicación: el catálogo se filtra por tarea de inferencia; text to image agrupa los modelos generadores de imágenes.
Por qué las otras son incorrectas: A son modelos de chat; C vectorizan texto; D genera vídeo, no imágenes.
Concepto relacionado: [[Generación de imágenes y vídeo]] · [[Catálogo de modelos de Foundry]]

**43. Respuesta correcta: B (Content Understanding con analizador de facturas).**
Explicación: el analizador de dominio asigna los valores leídos a campos concretos y devuelve JSON con confianza.
Por qué las otras son incorrectas: A solo etiquetaría la imagen; C trabaja con audio; D implicaría entrenar un modelo sin necesidad.
Concepto relacionado: [[Azure Content Understanding]]

**44. Respuesta correcta: B (Layout).**
Explicación: Layout añade al texto la estructura del documento: párrafos, jerarquía, orden de lectura y tablas.
Por qué las otras son incorrectas: A devuelve solo el texto; C extrae campos de documentos de identidad; D prepara vídeo para búsqueda.
Concepto relacionado: [[OCR (Reconocimiento óptico de caracteres)]] · [[Azure Content Understanding]]

**45. Respuesta correcta: B (Content Understanding con analizador de audio).**
Explicación: se necesitan campos estructurados a partir de audio, no solo la transcripción.
Por qué las otras son incorrectas: A transcribiría pero no asignaría campos; C traduce; D analiza imágenes.
Concepto relacionado: [[Azure Content Understanding]] · [[Extracción de información]]

**46. Respuesta correcta: B (una interpreta, la otra genera).**
Explicación: `input_image` entrega una imagen al modelo para que razone sobre ella; `images.generate` pide la creación de una imagen nueva.
Por qué las otras son incorrectas: A invierte los papeles; C ignora que son tareas distintas; D confunde con la generación de vídeo.
Concepto relacionado: [[Modelos multimodales (visión en prompts)]] · [[Generación de imágenes y vídeo]]

**47. Respuesta correcta: C (code interpreter).**
Explicación: permite ejecutar código en un entorno aislado para calcular, analizar archivos y generar gráficos.
Por qué las otras son incorrectas: A busca en Internet; B recupera texto de documentos; D crea imágenes a partir de descripciones.
Concepto relacionado: [[Agentes de IA (Foundry Agent Service)]]

**48. Respuesta correcta: C (`begin_analyze` con un poller).**
Explicación: el prefijo `begin_` indica una operación de larga duración cuyo resultado se recoge con `poller.result()`.
Por qué las otras son incorrectas: A es la credencial; B identifica el analizador; D fija la versión de la API.
Concepto relacionado: [[Azure Content Understanding]]

**49. Respuesta correcta: B (`chat-prod`).**
Explicación: el parámetro `model` recibe el **nombre del despliegue**, que puede diferir del nombre del modelo base.
Por qué las otras son incorrectas: A es el modelo del catálogo, no el despliegue; C y D no se indican en ese parámetro.
Concepto relacionado: [[Foundry SDK y cliente de chat]] · [[Despliegue y configuración de modelos]]

**50. Respuesta correcta: B (Foundry IQ con una knowledge base compartida).**
Explicación: centraliza las fuentes y la lógica de recuperación para que varios agentes las reutilicen.
Por qué las otras son incorrectas: A duplicaría índices y mantenimiento en cada agente; C no aporta conocimiento verificable ni actualizable; D es para entrenar modelos propios.
Concepto relacionado: [[Grounding, RAG y Foundry IQ]]

---

## Qué hacer con los fallos

1. Anota el **número** de cada fallo y abre la nota enlazada en "Concepto relacionado".
2. Si fallaste **dos o más** preguntas del mismo módulo, repasa su nota `99 - Repaso final` completa.
3. Vuelve a hacer el simulacro dos días después; si repites el mismo fallo, el problema es el concepto, no el despiste.

| Preguntas | Módulo a repasar |
|---|---|
| 2, 4, 6, 9, 10, 14, 20 | [[99 - Repaso final - Responsible AI]] |
| 1, 12, 18, 23 | [[99 - Repaso final - Fundamentos de IA]] |
| 3, 5, 8, 11, 13, 19, 24–36, 49, 50 | [[99 - Repaso final - Generative AI y agentes]] |
| 7, 15, 22, 37, 38, 39, 40 | [[99 - Repaso final - Texto y voz]] |
| 16, 17, 21, 41, 42, 46 | [[99 - Repaso final - Computer Vision]] |
| 43, 44, 45, 48 | [[99 - Repaso final - Extracción de información]] |

---

Volver a: [[AI-901 - Simulacro final (50 preguntas)]] · [[AI-901 - Guía de repaso final]] · [[00 - AI-901 Índice general]]
