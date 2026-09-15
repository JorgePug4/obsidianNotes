---
tags: [AZ-900, Azure, Identidad, Entra]
módulo: Autenticación y autorización
---

# Confianza cero (Zero Trust)

## Concepto

**Confianza cero** es un **modelo de seguridad** (no un producto) que parte de la idea de que **ninguna solicitud es confiable por defecto**, aunque venga de dentro de la red corporativa. Cada acceso se verifica de forma explícita.

Problema que resuelve: el modelo antiguo de "castillo y foso" asumía que todo lo que estaba dentro del firewall era seguro. Con trabajo remoto, dispositivos personales y nube, ese perímetro desapareció. Un atacante que entra a la red ya no debería tener acceso a todo.

Para qué se usa: diseñar la estrategia de seguridad de identidades, dispositivos, aplicaciones, datos, infraestructura y red.

## Características principales

Los **tres principios** de Confianza cero (memorizarlos):

1. **Verificar explícitamente**: autenticar y autorizar siempre, usando todos los datos disponibles (identidad, ubicación, dispositivo, servicio, clasificación de datos, anomalías).
2. **Usar acceso con privilegios mínimos**: dar solo el acceso necesario, el tiempo necesario (JIT/JEA), con políticas adaptativas.
3. **Asumir la brecha (assume breach)**: actuar como si el atacante ya estuviera dentro. Segmentar, cifrar de extremo a extremo, monitorizar y detectar.

Otras claves:
- Lema: **"Nunca confíes, siempre verifica"** (*never trust, always verify*).
- Se aplica a **seis pilares**: identidades, dispositivos, aplicaciones, datos, infraestructura, redes.
- Microsoft lo implementa con [[Autenticación multifactor (MFA)|MFA]], [[Acceso condicional]], RBAC, Defender, segmentación de red, etc.

> [!warning] Confusión frecuente
> Confianza cero **no** significa "no confiar en nadie" ni "bloquear todo". Significa **verificar cada solicitud** antes de conceder el acceso mínimo necesario. Tampoco es una herramienta que se instala; es un enfoque que guía cómo configurar las herramientas.

## Casos de uso

- Empleado que trabaja desde casa con su laptop personal: no se le da acceso por estar "en la VPN", sino que se verifica identidad (MFA), estado del dispositivo y sensibilidad del recurso.
- Administrador que solo recibe privilegios elevados durante 2 horas para una tarea concreta (privilegio mínimo).
- Red segmentada para que un servidor comprometido no pueda alcanzar la base de datos de nómina (assume breach).

## Comparaciones

| | Modelo perimetral tradicional | Confianza cero |
|---|---|---|
| Supuesto | Dentro de la red = confiable | Nada es confiable por defecto |
| Verificación | Una vez, al entrar | En cada solicitud |
| Acceso | Amplio una vez dentro | Mínimo necesario |
| Control principal | Firewall, VPN | Identidad, dispositivo, contexto |
| Ante una brecha | El atacante se mueve libremente | Segmentación limita el daño |

## Conceptos que debo memorizar

> [!important]
> - Tres principios: **Verificar explícitamente · Privilegio mínimo · Asumir la brecha**.
> - Lema: **Nunca confíes, siempre verifica**.
> - Es un **modelo / estrategia**, no un servicio.
> - Reemplaza al modelo de **perímetro** (castillo y foso).
> - La **identidad** es el nuevo perímetro.

## Tips para AZ-900

> [!tip]
> - Si una opción dice "confiar en los dispositivos dentro de la red corporativa", es lo **contrario** de Confianza cero.
> - Pregunta típica: "¿Cuál es un principio de Confianza cero?" Las respuestas correctas siempre son una de las tres: verificar explícitamente, privilegio mínimo, asumir brecha. Distractores habituales: "confiar en la red interna", "usar solo contraseñas fuertes", "aplicar un único firewall".
> - Palabras clave: *never trust, always verify*, *assume breach*, *least privilege*, *verify explicitly*.
> - Relación con otros temas: MFA y Acceso condicional son las herramientas que hacen realidad "verificar explícitamente".

## Ejemplo de pregunta de examen

**Pregunta 1.** ¿Cuál de los siguientes es un principio del modelo de Confianza cero?
- A) Confiar en todos los dispositivos conectados a la red corporativa
- B) Asumir que ya existe una brecha de seguridad
- C) Otorgar acceso de administrador a todos los usuarios para facilitar el trabajo
- D) Verificar la identidad únicamente en el primer inicio de sesión del día

**Respuesta: B.** "Asumir la brecha" es uno de los tres principios. A contradice "nunca confíes". C contradice el privilegio mínimo. D contradice la verificación explícita en cada solicitud.

**Pregunta 2.** Una empresa quiere adoptar un modelo de seguridad en el que cada solicitud de acceso se autentique y autorice sin importar si proviene de la red interna. ¿Qué modelo describe esto?
- A) Defensa en profundidad
- B) Modelo de responsabilidad compartida
- C) Confianza cero
- D) Seguridad perimetral

**Respuesta: C.** Verificar cada solicitud sin confiar en la ubicación de red es la definición de Confianza cero. A es un modelo de capas (complementario, pero no es esta definición). B describe qué asegura Microsoft y qué asegura el cliente. D es el modelo que Confianza cero reemplaza.

## 🧠 Resumen para el examen

1. Confianza cero = nunca confíes, siempre verifica.
2. Tres principios: verificar explícitamente, privilegio mínimo, asumir la brecha.
3. No es un producto; es un modelo de seguridad.
4. Sustituye al modelo perimetral (castillo y foso).
5. La identidad se convierte en el nuevo perímetro.
6. MFA y Acceso condicional lo ponen en práctica.
7. Se aplica a identidades, dispositivos, apps, datos, infraestructura y red.

---

← Volver al índice: [[AZ-900 - Autenticación y Autorización (índice)]]
