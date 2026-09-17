---
tags: [ai-901, azure, ia, vision, repaso]
módulo: Computer Vision
---

# 🎯 Repaso final · Computer Vision

## 1. Los conceptos más importantes

1. **Clasificación** = una etiqueta para la imagen; **detección de objetos** = objetos + **bounding boxes**.
2. **Caption** describe, **tags** etiquetan, **dense captions** describen regiones.
3. **OCR/Read** extrae texto; **Layout** añade estructura y tablas.
4. **Face**: detección (libre) vs verificación 1:1, identificación 1:N y liveness (**Limited Access**); atributos de emoción/edad **retirados**.
5. **Modelo multimodal** = imagen **en el prompt**; **text to image** = imagen **de salida**.
6. En el catálogo se filtra por **inference task**: chat, text to image, video generation.
7. Tres formas de "ver": **modelo multimodal** (razonar), **Azure Vision** (estructurado a escala), **Content Understanding** (campos).

## 2. Tabla necesidad → servicio

| Necesidad | Servicio / capacidad |
|---|---|
| Describir una imagen en una frase | Azure Vision (caption) o modelo multimodal |
| Etiquetar miles de imágenes con coordenadas | Azure Vision (tags + object detection) |
| Conversar sobre una foto enviada por el usuario | Modelo multimodal |
| Leer el texto de una foto | OCR/Read (Content Understanding o Azure Vision) |
| Obtener las tablas de un PDF | Analizador **Layout** |
| Extraer total y fecha de un ticket | Content Understanding (`prebuilt-receipt`) |
| Comparar un selfie con un documento | Face (verificación 1:1) |
| Comprobar que hay una persona real ante la cámara | Face (liveness) |
| Crear una ilustración desde texto | Modelo text to image (gpt-image, FLUX) |
| Crear un vídeo corto desde texto | Sora-2 |
| Entrenar un detector con imágenes propias | Custom Vision |

## 3. Diferencias que más se confunden

| Par confuso | Cómo distinguirlo |
|---|---|
| Clasificación vs detección | Etiqueta global / objetos con posición |
| Caption vs tags | Frase / lista |
| OCR vs Layout vs campos | Texto / estructura / valores por campo |
| Detección facial vs identificación | Hay cara / quién es |
| Verificación 1:1 vs identificación 1:N | Dos caras / grupo |
| Multimodal vs text to image | Entiende / genera |
| Azure Vision vs modelo multimodal | Estructurado y barato a escala / flexible y conversacional |
| Vision vs Content Understanding | Interpretar la imagen / extraer campos |

## 4. Tips de examen

> [!tip]
> 1. Si la pregunta menciona **coordenadas, cajas o conteo**, piensa en detección de objetos.
> 2. Si menciona **chat, conversación o razonar sobre la imagen**, es multimodal.
> 3. Si menciona **campos concretos de un documento**, cambia a Content Understanding.
> 4. Emoción, edad o género a partir de la cara: **ya no está disponible** (Responsible AI).
> 5. Generar imágenes: filtra el catálogo por **text to image**; dall-e-3 está retirado.
> 6. Accesibilidad con descripciones de imagen → también toca [[Inclusiveness (Inclusión)]].

## 5. Preguntas de repaso

**1.** ¿Qué capacidad necesitas para contar cuántas botellas hay en una estantería?

<details><summary>Respuesta</summary>

**Detección de objetos** (devuelve una caja por instancia).
</details>

**2.** ¿Qué diferencia hay entre `input_image` en un prompt y `images.generate`?

<details><summary>Respuesta</summary>

`input_image` envía una imagen **al** modelo para que la interprete (multimodal); `images.generate` pide al modelo que **cree** una imagen.
</details>

**3.** Un banco necesita evitar que alguien use la foto de otra persona para verificarse. ¿Qué capacidad?

<details><summary>Respuesta</summary>

**Liveness detection** (Face, Limited Access).
</details>

**4.** ¿Qué analizador devuelve el texto de un documento junto con sus tablas en Markdown?

<details><summary>Respuesta</summary>

**Layout** (Azure Content Understanding).
</details>

**5.** Verdadero o falso: Azure Vision puede identificar a una persona concreta por su cara.

<details><summary>Respuesta</summary>

**Falso.** Azure Vision detecta personas y caras; la **identificación** corresponde a Azure AI Face, con acceso limitado.
</details>

**6.** ¿Qué modelo elegirías para que un asistente describa a un usuario ciego lo que muestra la cámara de su móvil?

<details><summary>Respuesta</summary>

Un **modelo multimodal** con entrada de imagen (o Azure Vision captions), combinado con **text to speech** para leer la descripción.
</details>

## Checklist

- [ ] Comprendo las tareas de visión y sé reconocerlas en escenarios.
- [ ] Sé diferenciar clasificación, detección, captioning, tagging y OCR.
- [ ] Conozco las capacidades y restricciones de Face.
- [ ] Sé enviar una imagen en un prompt a un modelo multimodal y reconocer ese código.
- [ ] Sé desplegar y usar un modelo de generación de imágenes o vídeo.
- [ ] Sé cuándo usar Azure Vision, un modelo multimodal o Content Understanding.
- [ ] Puedo responder las preguntas de práctica.

Volver al índice: [[00 - Índice - Computer Vision]] · [[00 - AI-901 Índice general]]
