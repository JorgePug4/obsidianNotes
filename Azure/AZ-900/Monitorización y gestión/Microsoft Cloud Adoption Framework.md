# Microsoft Cloud Adoption Framework

> [!warning] Relevancia para AZ-900: MEDIA-BAJA
> El CAF aparecía en versiones anteriores del outline. En la versión 2026 no está listado como tema propio, pero conviene reconocerlo porque suele aparecer como opción en preguntas sobre "guía para adoptar la nube".

## Concepto

El **Microsoft Cloud Adoption Framework for Azure (CAF)** es una **colección de documentación, guías, mejores prácticas y herramientas** que Microsoft publica para ayudar a las organizaciones a **planificar y ejecutar su adopción de la nube**. No es un servicio de Azure: es una **metodología**.

Problema que resuelve: muchas empresas migran a la nube sin estrategia, sin plan de gobernanza y sin preparación del equipo. El CAF ordena ese proceso.

## Características principales

Fases principales que Microsoft describe:

| Fase | Pregunta que responde |
|---|---|
| **Estrategia** (Strategy) | ¿Por qué vamos a la nube? Motivaciones y resultados de negocio. |
| **Plan** | ¿Qué migramos y en qué orden? Inventario y plan de adopción. |
| **Preparación** (Ready) | ¿Está lista nuestra zona de aterrizaje (landing zone) en Azure? |
| **Adopción** (Adopt) | Migrar e innovar: mover cargas y crear nuevas. |
| **Gobernanza** (Govern) | Controlar coste, seguridad y cumplimiento. |
| **Administración** (Manage) | Operar y monitorizar lo desplegado. |
| **Seguridad** (Secure) | Proteger a lo largo de todo el ciclo. |

- El CAF es **gratuito** y está en Microsoft Learn.
- Incluye la idea de **landing zone**: un entorno Azure preconfigurado con red, identidad y gobernanza listos para recibir cargas.

> [!warning] CAF vs Well-Architected Framework
> El **CAF** trata sobre **cómo adopta la nube una organización** (estrategia, personas, gobernanza). El **Well-Architected Framework** trata sobre **cómo diseñar bien una carga concreta** (fiabilidad, seguridad, coste, rendimiento, excelencia operativa). El WAF no entra en AZ-900, pero si lo ves como distractor, ya sabes distinguirlo.

## Casos de uso

- Una empresa tradicional decide migrar 200 servidores y no sabe por dónde empezar: sigue el CAF.
- Un CIO necesita justificar ante dirección la inversión en Azure: fase de Estrategia del CAF.

## Conceptos que debo memorizar

> [!important]
> - CAF = **guía y mejores prácticas** para adoptar Azure. **No es un servicio**.
> - Fases: Estrategia, Plan, Preparación, Adopción, Gobernanza, Administración (más Seguridad transversal).
> - Introduce el concepto de **landing zone**.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *guía, documentación, mejores prácticas, plan de adopción, metodología, ciclo de vida de adopción*.
> - Si la pregunta pide "un servicio para..." el CAF **nunca** es la respuesta: no es un servicio.
> - Si la pregunta pide "orientación para migrar a la nube de forma ordenada", el CAF sí es la respuesta.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu organización quiere seguir una metodología probada para planificar la migración a Azure, incluyendo estrategia, gobernanza y operación. ¿Qué debes consultar?

- A) Azure Advisor
- B) Microsoft Cloud Adoption Framework
- C) Azure Policy
- D) Azure Service Health

**Respuesta: B.** El CAF es la guía de adopción de Microsoft.
- A) Advisor da recomendaciones sobre recursos ya desplegados.
- C) Policy aplica reglas, no planifica adopción.
- D) Service Health informa de incidencias de Azure.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre el Cloud Adoption Framework es correcta?

- A) Es un servicio de Azure que se paga por uso.
- B) Es una colección de documentación y mejores prácticas para la adopción de la nube.
- C) Sustituye a Azure Policy.
- D) Solo aplica a migraciones desde AWS.

**Respuesta: B.**
- A) No es un servicio ni tiene coste.
- C) Complementa a Policy dentro de la fase de gobernanza.
- D) Aplica a cualquier origen.

## 🧠 Resumen para el examen

1. CAF = metodología y documentación gratuita de Microsoft para adoptar la nube.
2. No es un servicio de Azure.
3. Fases: Estrategia, Plan, Preparación, Adopción, Gobernanza, Administración.
4. Landing zone = entorno preconfigurado listo para cargas.
5. Distinto del Well-Architected Framework (diseño de cargas).

---
