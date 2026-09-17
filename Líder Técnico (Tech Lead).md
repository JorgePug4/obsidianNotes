---
tags: [liderazgo, carrera, arquitectura, equipo]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [Tech Lead, Líder Técnico]
---

# Líder Técnico (Tech Lead)

> [!info] El rol varía mucho entre empresas. Esta nota describe el consenso de la industria (Fournier, Larson, Team Topologies); adapta los detalles a tu organización.

## ¿Qué es?

Un **líder técnico** es la persona responsable de **guiar técnicamente a un equipo de desarrollo**: toma o facilita las decisiones de arquitectura, asegura la calidad técnica y ayuda al equipo a entregar soluciones **mantenibles y escalables**.

Sigue siendo un rol **técnico y cercano al código**, pero su medida de éxito ya no es lo que produce individualmente sino **lo que el equipo produce gracias a su influencia**.

> «Un líder técnico no es quien más código escribe, sino quien ayuda al equipo a tomar mejores decisiones técnicas.»

## ¿Para qué sirve el rol?

- Dar al equipo una **dirección técnica coherente**: sin Tech Lead, cada desarrollador toma decisiones locales que no encajan entre sí.
- Ser el **puente** entre negocio/producto y el detalle técnico: traducir requisitos a diseño y riesgos técnicos a lenguaje de negocio.
- **Multiplicar** al equipo: desbloquear, enseñar, revisar y elevar el nivel de todos.
- Proteger la **salud a largo plazo** del sistema (deuda técnica, seguridad, rendimiento) frente a la presión de entregar rápido.

## Conceptos relacionados

Un Tech Lead necesita dominar, al menos a nivel de criterio, el resto de esta base de conocimiento:

- [[Clean Architecture]] y [[Principios SOLID]] → criterios para revisar código y estructurar soluciones.
- [[🧠 Domain-Driven Design (DDD) - Curso Completo|DDD]] → para modelar el negocio y trazar límites entre módulos y equipos.
- [[🏗️ Diseño de Microservicios]] → decidir cuándo (y cuándo no) distribuir un sistema.
- [[Bases de Datos SQL vs NoSQL]] y [[ACID en Bases de Datos|ACID]] → elegir persistencia con criterio.
- [[Diseño Api Rest|Diseño de APIs REST]] → definir contratos entre equipos.
- Cloud y gobernanza: [[Gobernanza en Azure]], [[Azure RBAC]], [[Microsoft Cloud Adoption Framework]] → el Tech Lead suele ser quien lleva estas conversaciones con infraestructura.

## Responsabilidades principales

### 1. Arquitectura y diseño

- Definir la arquitectura de las soluciones del equipo, alineada con la arquitectura global de la empresa.
- Elegir tecnologías y patrones adecuados **al problema**, no a la moda.
- Diseñar sistemas escalables y mantenibles; anticipar cuellos de botella.
- Validar las decisiones técnicas importantes y **documentarlas** (ver ADRs más abajo).

### 2. Liderazgo técnico

- Guiar al equipo en buenas prácticas y estándares.
- Promover [[Principios SOLID|SOLID]], *Clean Code* y patrones de diseño **con criterio**, evitando la sobreingeniería.
- Resolver (o ayudar a resolver) los problemas técnicos más complejos.
- **Revisar código** y dar retroalimentación constructiva y accionable.
- Ser el referente al que el equipo acude cuando está bloqueado.

### 3. Calidad y mantenimiento

- Asegurar la calidad del código: revisiones, estándares, análisis estático.
- **Gestionar la deuda técnica**: hacerla visible, priorizarla y negociar tiempo para pagarla.
- Impulsar pruebas automatizadas (unitarias, integración, contrato) y CI/CD.
- Velar por rendimiento, seguridad y observabilidad desde el diseño.

### 4. Coordinación

- Trabajar con Product Owners, QA, DevOps, Arquitectos y otros equipos.
- Ayudar en las **estimaciones técnicas** y en el desglose de épicas en tareas.
- Identificar y comunicar **riesgos técnicos** a tiempo.
- Priorizar el trabajo técnico frente al de producto (negociación constante).
- Definir e integrar **contratos** con otros equipos (APIs, eventos).

### 5. Mentoría

- Apoyar el crecimiento técnico del equipo.
- Capacitar a desarrolladores junior y semi-senior mediante *pairing*, revisiones y sesiones técnicas.
- Compartir conocimiento y **delegar** decisiones para que otros crezcan (no acaparar).
- Crear un entorno donde sea seguro preguntar y equivocarse.

## ¿Cómo se ejerce en el día a día?

### Reparto del tiempo (orientativo)

| Actividad | Proporción típica |
|---|---|
| Diseño, revisiones de código, desbloqueos y *pairing* | 40 – 50 % |
| Código propio (tareas no críticas para la ruta del sprint) | 20 – 30 % |
| Reuniones con producto, otros equipos, planificación | 20 – 30 % |

> [!warning] La trampa más común
> Seguir siendo el desarrollador más productivo del equipo **y además** liderar. Si el Tech Lead toma siempre las tareas críticas, se convierte en cuello de botella y el equipo no crece. Toma tareas que no bloqueen a nadie si te interrumpen.

### Herramientas del rol

- **ADR (Architecture Decision Record)**: documento corto (contexto, decisión, alternativas, consecuencias) por cada decisión técnica relevante. Evita rediscutir lo mismo y explica el "por qué" a quien llegue después.
- **Definición de Hecho (DoD)** y estándares del equipo escritos.
- **Registro de deuda técnica** priorizado y visible para producto.
- **Diagramas C4** para comunicar la arquitectura a distintos niveles.
- **Revisiones de código** con criterios explícitos: corrección, legibilidad, tests, seguridad, rendimiento.

### Cómo dar feedback en una revisión de código

1. Distinguir **bloqueante** (bug, seguridad, rompe el diseño) de **sugerencia** (estilo, alternativa).
2. Explicar el **por qué**, no solo el qué.
3. Preguntar antes de asumir ("¿qué pasa si el token expira aquí?").
4. Reconocer lo bueno, no solo señalar lo malo.

## Habilidades necesarias

### Técnicas

- Arquitectura de software (monolito modular, microservicios, orientación a eventos).
- Bases de datos SQL y NoSQL; modelado de datos.
- Cloud y DevOps: CI/CD, contenedores, infraestructura como código, observabilidad.
- Diseño de APIs y contratos.
- Testing en todos los niveles.
- Seguridad básica (OWASP Top 10, gestión de secretos, autenticación/autorización).

### Blandas (las que marcan la diferencia)

- **Comunicación**: explicar lo técnico a no técnicos y viceversa.
- **Liderazgo sin autoridad formal**: influir por criterio, no por jerarquía (en muchas empresas el Tech Lead no es jefe de nadie).
- **Toma de decisiones** con información incompleta, y saber cuándo revertirlas.
- **Resolución de conflictos** técnicos: facilitar que el equipo decida, no imponer.
- **Pensamiento crítico** y pragmatismo: la solución perfecta que no se entrega no vale nada.
- **Delegación** y confianza.

## Comparación con roles cercanos

| Rol | Enfoque principal | Horizonte | ¿Gestiona personas? | ¿Escribe código? |
|---|---|---|---|---|
| **Tech Lead** | Ejecución técnica y liderazgo de **un equipo** | Sprints a trimestres | No formalmente (influye, mentoriza) | Sí, menos que antes |
| **Arquitecto de software** | Estrategia técnica, visión global, estándares **entre equipos** | Trimestres a años | No | Poco; prototipos y revisiones |
| **Engineering Manager** | Personas, procesos, carrera, contratación, presupuesto | Trimestres a años | **Sí** | Casi nada |
| **Staff / Principal Engineer** | Problemas técnicos transversales de alto impacto, sin equipo fijo | Trimestres a años | No | Sí, en lo más difícil |
| **Senior Developer** | Entrega técnica de alta calidad dentro del equipo | Sprints | No | Sí, la mayoría del tiempo |

> [!tip] Tech Lead vs Engineering Manager
> En algunas empresas un mismo rol combina ambos (*Tech Lead Manager*). Es difícil hacer bien las dos cosas: la gestión de personas consume el tiempo que la dirección técnica necesita. Cuando existan por separado, el Tech Lead **decide el cómo técnico**; el EM **cuida a las personas y el proceso**.

## Indicadores de un buen líder técnico

- El equipo **entrega con calidad** de forma sostenida, no a base de héroes.
- Hay **menos bloqueos** técnicos y se resuelven más rápido.
- El código es **mantenible**: alguien nuevo puede entenderlo y cambiarlo.
- Las decisiones técnicas **tienen justificación** escrita (ADRs) y el equipo las conoce.
- El equipo **crece técnicamente**: los juniors de hace un año hoy revisan código.
- El Tech Lead **puede irse de vacaciones** sin que el equipo se detenga.
- Producto **confía** en las estimaciones y en los avisos de riesgo.

## Anti-patrones del rol

| Anti-patrón | Síntoma | Corrección |
|---|---|---|
| **El héroe** | Hace todo lo difícil él mismo; el equipo espera | Delegar lo difícil con acompañamiento |
| **Torre de marfil** | Diseña sin tocar código ni escuchar al equipo | Seguir en el código; diseñar en conjunto |
| **Dictador técnico** | Impone decisiones sin explicar | Facilitar, argumentar, documentar en ADRs |
| **El que dice sí a todo** | Acepta cada fecha y cada cambio; el equipo se quema | Negociar alcance y hacer visible el coste |
| **Perfeccionista** | Rechaza PRs por estilo; bloquea entregas | Separar bloqueante de sugerencia |
| **Coleccionista de tecnologías** | Introduce un framework nuevo cada trimestre | Evaluar con criterio y pilotos acotados |

## Puntos clave

- El Tech Lead **multiplica** al equipo; su éxito se mide en el equipo, no en sus commits.
- Cinco áreas: **arquitectura, liderazgo técnico, calidad, coordinación, mentoría**.
- Sigue siendo técnico y cercano al código, pero **no** el cuello de botella.
- Las decisiones se **documentan** (ADRs) y se **explican**.
- Distinto de **Arquitecto** (visión global) y **Engineering Manager** (personas y procesos).

## Preguntas de repaso

1. ¿Qué diferencia a un Tech Lead de un Senior Developer muy bueno?
2. ¿Por qué el Tech Lead no debería tomar siempre las tareas más críticas del sprint?
3. ¿Qué es un ADR y qué problema resuelve?
4. ¿Cómo harías visible la deuda técnica a un Product Owner?
5. ¿Qué harías si el equipo no está de acuerdo con una decisión de arquitectura que consideras correcta?

## 🎯 Para entrevistas

- Tener **ejemplos concretos**: una decisión técnica que tomaste y documentaste, un conflicto técnico que facilitaste, un junior al que ayudaste a crecer, una deuda técnica que negociaste.
- Saber explicar **cómo decides** entre opciones técnicas (criterios, prototipos, reversibilidad).
- Mostrar que entiendes el **negocio**, no solo la tecnología.
- Preguntas típicas: *"¿Cómo manejas a un desarrollador senior que no sigue los estándares?"*, *"¿Cómo priorizas deuda técnica frente a funcionalidades?"*, *"Cuéntame una decisión técnica de la que te arrepientes."*

## Referencias

- Camille Fournier, *The Manager's Path* (O'Reilly, 2017), capítulo *Tech Lead*
- Will Larson, *Staff Engineer: Leadership Beyond the Management Track* (2021)
- Pat Kua, *Talking with Tech Leads* (2014) y su blog sobre el rol: https://www.patkua.com/blog/the-definition-of-a-tech-lead/
- Matthew Skelton & Manuel Pais, *Team Topologies* (IT Revolution, 2019)
- Michael Nygard, *Documenting Architecture Decisions* (ADRs): https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- Simon Brown, *C4 model*: https://c4model.com/

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
