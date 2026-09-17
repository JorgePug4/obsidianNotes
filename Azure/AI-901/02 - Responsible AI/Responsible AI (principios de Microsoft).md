---
tags: [ai-901, azure, ia, responsible-ai]
módulo: Responsible AI
peso_examen: Muy alto
aliases: [Responsible AI, IA responsable, Principios de IA responsable]
---

# Responsible AI (principios de Microsoft)

## ¿Qué es?

**Responsible AI (IA responsable)** es el conjunto de principios y prácticas de Microsoft para que los sistemas de IA se diseñen, construyan y operen de forma **ética, segura y confiable**. Se concreta en **seis principios** y en el *Microsoft Responsible AI Standard*, que traduce los principios en requisitos.

## ¿Para qué sirve?

Los modelos pueden discriminar, fallar en situaciones no previstas, filtrar datos, excluir a personas, ser opacos o no tener a nadie que responda por ellos. Los principios sirven para **anticipar esos riesgos** durante todo el ciclo de vida de la solución.

## Los seis principios

| Principio | Pregunta clave | Riesgo que evita | Ejemplo de medida |
|---|---|---|---|
| **Fairness** (equidad) | ¿Trata a todos los grupos por igual? | Sesgo y discriminación | Evaluar el modelo por género, edad, origen; datos representativos |
| **Reliability and safety** (fiabilidad y seguridad) | ¿Funciona de forma consistente y sin causar daño? | Errores, comportamientos inesperados, daño físico | Pruebas rigurosas, umbrales de confianza, supervisión humana |
| **Privacy and security** (privacidad y seguridad) | ¿Protege los datos personales y resiste ataques? | Fugas de datos, uso indebido, ataques (prompt injection) | Cifrado, minimización de datos, PII redaction, prompt shields |
| **Inclusiveness** (inclusión) | ¿Sirve a todas las personas? | Excluir a grupos o personas con discapacidad | Accesibilidad, varios idiomas, voz y texto |
| **Transparency** (transparencia) | ¿Se entiende qué hace, con qué datos y qué límites tiene? | Confianza ciega, decisiones inexplicables | Documentar capacidades y limitaciones, informar de que se habla con IA, citar fuentes |
| **Accountability** (responsabilidad) | ¿Quién responde por el sistema? | Nadie responsable, sin gobernanza | Marco de gobierno, revisión humana, auditorías, cumplimiento legal |

Mnemotecnia: **F-R-P-I-T-A** (Fairness, Reliability, Privacy, Inclusiveness, Transparency, Accountability).

## Cómo se aplican en Foundry

| Principio | Herramienta o práctica en Microsoft Foundry |
|---|---|
| Fairness | Evaluaciones de calidad por grupos, datasets de prueba diversos |
| Reliability and safety | **Guardrails / filtros de contenido**, evaluaciones de seguridad, red teaming, groundedness |
| Privacy and security | **PII redaction** (Azure Language), **Prompt Shields**, identidad de Entra ID, control de acceso (RBAC) |
| Inclusiveness | Azure Speech (voz), Translator (idiomas), modelos multilingües |
| Transparency | *Transparency notes* de los modelos, citas en respuestas RAG, tracing, avisar de que es una IA |
| Accountability | Foundry control plane, gobernanza, registros y evaluaciones, humanos en el bucle |

## Ejemplo

Un banco despliega un agente que preaprueba préstamos. Aplica **fairness** comprobando que la tasa de aprobación no difiera injustificadamente por sexo o barrio; **reliability** con umbrales y revisión humana en casos dudosos; **privacy** minimizando datos y redactando PII; **inclusiveness** ofreciendo canal de voz y varios idiomas; **transparency** explicando al cliente los criterios; **accountability** con un comité que responde de las decisiones.

## 🧠 Memorizar

> [!important]
> - Los **seis** principios y su idea en dos palabras: **equidad · fiabilidad/seguridad · privacidad/seguridad · inclusión · transparencia · responsabilidad**.
> - Son de **Microsoft** y se aplican a **todo el ciclo de vida** (diseño, desarrollo, despliegue, operación).
> - **Transparency ≠ Accountability**: explicar cómo funciona vs tener a alguien que responda.
> - **Reliability and safety ≠ Privacy and security**: comportamiento correcto y seguro vs protección de datos y ataques.

## Tips para AI-901

> [!tip]
> - ⭐ Las preguntas dan una **situación** y preguntan **qué principio**. Busca la palabra clave: *sesgo/grupos* → fairness; *falla/daño/inesperado* → reliability & safety; *datos personales/ataque/fuga* → privacy & security; *discapacidad/idiomas/todos* → inclusiveness; *explicar/informar/limitaciones* → transparency; *quién responde/gobernanza/legal* → accountability.
> - 🔥 En IA generativa: alucinaciones → reliability (y transparency si no se avisa); jailbreak/prompt injection → security; contenido dañino → safety; citar fuentes → transparency.
> - ⚠️ "Explicar que las respuestas pueden contener errores" es **transparency**, no accountability.

## ⚠️ Errores comunes

- Mezclar *safety* (no causar daño) con *security* (proteger contra ataques).
- Pensar que Responsible AI es solo un paso final. Es continuo.

## 💡 Escenario

Un asistente médico generativo debe mostrar siempre un aviso de que no sustituye a un profesional y enlazar las fuentes de cada afirmación. ¿Qué principio se aplica?

<details><summary>Respuesta</summary>

**Transparency.** Se informa al usuario de la naturaleza y límites del sistema y se muestran las fuentes. Si además hubiera revisión médica obligatoria, eso apoyaría **accountability**.
</details>

## Preguntas que podrían aparecer

**1.** Un modelo de selección de personal rechaza más candidatas mujeres con el mismo perfil que hombres. ¿Qué principio se está incumpliendo?
- A) Transparency · B) Fairness · C) Inclusiveness · D) Accountability

<details><summary>Respuesta</summary>

**B.** Hay sesgo entre grupos. C (inclusión) trataría de que el sistema sea usable por todos; aquí el problema es discriminación en el resultado.
</details>

**2.** Relaciona: (1) un vehículo autónomo debe comportarse de forma segura ante lluvia intensa; (2) se cifran las transcripciones de las llamadas; (3) la app ofrece subtítulos y lectura en voz alta.

<details><summary>Respuesta</summary>

(1) Reliability and safety · (2) Privacy and security · (3) Inclusiveness.
</details>

## Relacionado

- [[Fairness (Equidad)]] · [[Reliability and Safety (Fiabilidad y seguridad)]] · [[Privacy and Security (Privacidad y seguridad)]]
- [[Inclusiveness (Inclusión)]] · [[Transparency (Transparencia)]] · [[Accountability (Responsabilidad)]]
- [[Azure AI Content Safety y guardrails]]
- [[Generative AI]] (limitaciones y riesgos)

← Volver al índice: [[00 - Índice - Responsible AI]]
