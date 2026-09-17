---
tags: [ai-901, azure, ia, vision, face, servicio]
módulo: Computer Vision
peso_examen: Medio-alto
aliases: [Face, Azure AI Face, Reconocimiento facial, Face detection]
---

# Reconocimiento facial (Face)

## ¿Qué es?

**Azure AI Face** es el servicio especializado en **caras**: detectarlas en una imagen, analizar sus atributos, comparar si dos caras pertenecen a la misma persona, buscar caras similares y, con acceso aprobado, **identificar** personas contra un grupo registrado.

## ¿Para qué sirve?

Control de acceso, verificación de identidad en onboarding digital, desbloqueo, conteo de personas, organización de fotos.

## Capacidades

| Capacidad | Qué hace | Ejemplo |
|---|---|---|
| **Detection** | Localiza caras: bounding box, puntos faciales (landmarks), pose, gafas, oclusión, desenfoque | Contar personas en una sala |
| **Verification (1:1)** | ¿Estas dos caras son la misma persona? | Comparar selfie con el DNI |
| **Identification (1:N)** | ¿Quién es esta cara dentro de un grupo registrado? | Acceso a un edificio |
| **Find similar / grouping** | Caras parecidas o agrupadas por persona | Organizar un álbum |
| **Liveness detection** | Comprobar que hay una **persona real** delante de la cámara y no una foto o un vídeo | Antifraude en onboarding |

> [!warning] Acceso limitado y Responsible AI
> Las capacidades de **identificación**, **verificación** y **liveness** de Face están sujetas a **Limited Access**: hay que solicitar acceso y aceptar un caso de uso admitido. Microsoft **retiró** los atributos inferidos sensibles (emoción, género, edad, sonrisa, vello facial y similares) por motivos de IA responsable. La **detección** de caras y sus atributos técnicos (pose, oclusión, calidad) sigue disponible sin restricción especial.

## Ejemplo

Un banco valida la identidad al abrir una cuenta: **liveness** confirma que hay una persona real; **verification (1:1)** compara ese rostro con la foto del documento. No hace falta identificación 1:N.

## 📌 Diferencias clave

| Par confuso | Cómo distinguirlo |
|---|---|
| **Detección** vs **reconocimiento** | Hay una cara y dónde está / **quién** es |
| **Verificación (1:1)** vs **identificación (1:N)** | Comparar dos caras / buscar en un grupo |
| **Face** vs **Azure Vision** | Especializado en caras / análisis general de imágenes (también detecta personas, pero no las identifica) |
| **Liveness** vs **verificación** | ¿Es una persona real? / ¿es la misma persona? |

## 🧠 Memorizar

> [!important]
> - **Detección ≠ identificación.**
> - **1:1 = verificación**; **1:N = identificación**.
> - **Liveness** evita suplantación con fotos o vídeos.
> - Identificación, verificación y liveness requieren **Limited Access**; los atributos de emoción/género/edad fueron **retirados**.

## Tips para AI-901

> [!tip]
> - ⭐ "Comparar un selfie con la foto del documento" → **verificación 1:1**.
> - ⭐ "Saber quién entra por la puerta entre los empleados registrados" → **identificación 1:N** (acceso limitado).
> - 🔥 "Evitar que usen una foto impresa para hacerse pasar por alguien" → **liveness**.
> - ⚠️ Preguntas sobre "detectar la emoción de los clientes" suelen tener respuesta de Responsible AI: **ya no está disponible**.
> - ⚠️ Face es el servicio con más peso de **privacidad y consentimiento**: relaciónalo con [[Privacy and Security (Privacidad y seguridad)]] y [[Fairness (Equidad)]].

## 💡 Escenario

Un estadio quiere contar cuántas personas hay en cada zona a partir de cámaras, sin identificar a nadie. ¿Qué capacidad y qué consideraciones?

<details><summary>Respuesta</summary>

**Detección de personas/caras** (Azure Vision people detection o Face detection). No hace falta identificación, lo que evita Limited Access y reduce el riesgo de privacidad. Aun así, deben aplicarse consideraciones de privacidad, información a los asistentes y minimización de datos.
</details>

## Preguntas que podrían aparecer

**1.** ¿Qué operación de Face compara dos imágenes para decidir si corresponden a la misma persona?
- A) Detection · B) Verification · C) Identification · D) Find similar

<details><summary>Respuesta</summary>

**B.**
</details>

**2.** ¿Cuál de estos atributos ya NO ofrece Azure AI Face por motivos de IA responsable?
- A) Bounding box de la cara · B) Estimación de emociones · C) Pose de la cabeza · D) Oclusión

<details><summary>Respuesta</summary>

**B.**
</details>

## Relacionado

- [[Computer Vision]] · [[Azure AI Vision]]
- [[Privacy and Security (Privacidad y seguridad)]] · [[Fairness (Equidad)]] · [[Responsible AI (principios de Microsoft)]]

← Volver al índice: [[00 - Índice - Computer Vision]]
