---
tags: [az-900, azure, conceptos-nube, beneficios, escalabilidad, alta-disponibilidad]
modulo: Conceptos de la nube
peso_examen: Alto
---

# Beneficios de la nube

## Concepto

El examen agrupa los **beneficios de usar servicios en la nube** en cuatro parejas: **alta disponibilidad y escalabilidad**, **confiabilidad y previsibilidad**, **seguridad y gobernanza**, y **capacidad de administración**. Cada uno tiene una definición precisa y el AZ-900 pregunta por ellas casi literalmente.

**Problema que resuelve:** un CPD propio tiene límites físicos (no puedes duplicar servidores en minutos), puntos únicos de fallo y costes fijos. La nube ofrece capacidad casi ilimitada, redundancia global y herramientas de control integradas.

**Para qué se utiliza:** para justificar la adopción de la nube y para elegir el diseño correcto (por ejemplo, zonas de disponibilidad para alta disponibilidad, autoescalado para elasticidad).

## Características principales

### 1. Alta disponibilidad (High Availability)

- Que el servicio esté **operativo el máximo tiempo posible**, incluso cuando algo falla.
- Se expresa como **SLA** (Service Level Agreement): porcentaje de tiempo garantizado. Los "nueves":

| SLA | Tiempo de inactividad al año | Al mes |
|---|---|---|
| 99 % | ~3,65 días | ~7,3 horas |
| 99,9 % ("tres nueves") | ~8,76 horas | ~43,8 minutos |
| 99,95 % | ~4,38 horas | ~21,9 minutos |
| 99,99 % ("cuatro nueves") | ~52,6 minutos | ~4,4 minutos |
| 99,999 % ("cinco nueves") | ~5,26 minutos | ~26 segundos |

- Si combinas varios servicios, el SLA compuesto es el **producto** de los SLA individuales (baja).
- En Azure se consigue con [[01 - Regiones, pares de regiones y zonas de disponibilidad]], conjuntos de disponibilidad y balanceadores.
- Si Microsoft incumple el SLA, se aplican **créditos de servicio** en la factura.

### 2. Escalabilidad (Scalability)

- Capacidad de **ajustar los recursos a la demanda**. Dos direcciones:

| Tipo | Qué hace | Ejemplo | Otro nombre |
|---|---|---|---|
| **Vertical** (scale up / down) | Más o menos potencia **al mismo recurso** | Pasar una VM de 2 a 8 vCPU | "Escalar hacia arriba" |
| **Horizontal** (scale out / in) | **Más o menos instancias** del recurso | Pasar de 2 a 10 VMs | "Escalar hacia fuera" |

- Horizontal es la preferida en la nube: no tiene techo y no requiere reiniciar.
- Herramienta clave: [[03 - Máquinas virtuales, Scale Sets y conjuntos de disponibilidad|Virtual Machine Scale Sets]].

### 3. Elasticidad (Elasticity)

- Escalado **automático** en ambas direcciones según la demanda real. La escalabilidad es la *capacidad* de crecer; la elasticidad es hacerlo **solo y en ambos sentidos** para no pagar de más.

### 4. Agilidad (Agility)

- **Rapidez** para desplegar y cambiar recursos: minutos en lugar de semanas o meses.

### 5. Confiabilidad (Reliability)

- Capacidad de **recuperarse de fallos** y seguir funcionando. Se apoya en la **redundancia** global de Azure: si una región cae, otra puede tomar el relevo.
- Se relaciona con la **recuperación ante desastres** (DR) y con [[Redundancia de almacenamiento]].

### 6. Previsibilidad (Predictability)

- **De rendimiento**: sabes cómo se comportará el sistema gracias al autoescalado, el balanceo de carga y la alta disponibilidad.
- **De coste**: puedes pronosticar el gasto con [[02 - Calculadora de precios y calculadora de TCO]] y seguirlo con [[03 - Microsoft Cost Management y etiquetas]].

### 7. Seguridad (Security)

- El proveedor mantiene la **seguridad física** y ofrece herramientas (MFA, cifrado, [[Microsoft Defender for Cloud]]) que serían caras de montar en local.
- Puedes elegir el nivel de control: IaaS para controlar el SO, PaaS/SaaS para delegar más.

### 8. Gobernanza (Governance)

- Aplicar **estándares corporativos** y **cumplimiento** normativo de forma automática con [[Azure Policy]], [[Azure RBAC]] y [[Bloqueos de recursos (Locks)]]. Ver [[Gobernanza en Azure]].

### 9. Capacidad de administración (Manageability)

- **Administración *de* la nube**: escalado automático, despliegue por plantillas, monitorización y alertas ([[Azure Monitor]]), reemplazo automático de recursos con fallos.
- **Administración *en* la nube**: las herramientas para hacerlo: portal, CLI, PowerShell, API, app móvil. Ver [[04 - Portal, Cloud Shell, CLI, PowerShell y app móvil]].

## Casos de uso

- Una app de reservas necesita **99,99 %** de disponibilidad: se despliega en varias zonas de disponibilidad tras un balanceador.
- Una plataforma de vídeo tiene picos por la noche: **elasticidad** con autoescalado que añade instancias a las 20:00 y las quita a las 02:00.
- Un equipo necesita un entorno de pruebas para mañana: **agilidad**, se crea en minutos con una plantilla.
- Un fabricante quiere que ningún recurso se cree fuera de Europa: **gobernanza** con Azure Policy.

## Comparaciones

| Par confuso | Cómo distinguirlos |
|---|---|
| **Escalabilidad vs Elasticidad** | Escalabilidad: *poder* crecer (manual o automático). Elasticidad: crecer y **encoger automáticamente** según demanda. |
| **Escalado vertical vs horizontal** | Vertical: la misma máquina más potente. Horizontal: más máquinas. |
| **Alta disponibilidad vs Confiabilidad** | HA: minimizar el tiempo caído (SLA). Confiabilidad: recuperarse de fallos (redundancia, DR). |
| **Agilidad vs Elasticidad** | Agilidad: rapidez para **desplegar/cambiar**. Elasticidad: ajuste automático de **capacidad**. |
| **Previsibilidad de rendimiento vs de coste** | Rendimiento: el sistema responde igual bajo carga. Coste: sabes cuánto pagarás. |
| **Seguridad vs Gobernanza** | Seguridad: proteger contra amenazas. Gobernanza: cumplir reglas y estándares de la organización. |

## Conceptos que debo memorizar

> [!important]
> - **HA** = tiempo activo, medido por **SLA** (99,9 %, 99,99 %...). Más nueves = menos caída.
> - **Vertical** = más potencia a lo mismo. **Horizontal** = más instancias.
> - **Elasticidad** = escalado **automático en ambos sentidos**.
> - **Agilidad** = rapidez de despliegue.
> - **Confiabilidad** = recuperarse de fallos gracias a redundancia.
> - **Previsibilidad** = rendimiento y coste anticipables.
> - **Gobernanza** = estándares y cumplimiento aplicados automáticamente.
> - **Manageability** = gestionar *la* nube (automatización) y *en* la nube (herramientas).

## Tips para AZ-900

> [!tip]
> - "Añadir más servidores" → **horizontal**. "Más CPU/RAM a un servidor" → **vertical**.
> - "Automáticamente según la demanda" → **elasticidad** (aunque también es escalabilidad, la palabra *automática* apunta a elasticidad).
> - "Desplegar en minutos" → **agilidad**.
> - "Seguir funcionando aunque falle un componente" → **HA**. "Recuperarse tras un desastre" → **confiabilidad**.
> - "Pronosticar la factura" → **previsibilidad de coste**.
> - Trampa: SLA de 99,9 % permite casi **9 horas** de caída al año, no 9 minutos.
> - Trampa: combinar dos servicios con 99,9 % no da 99,9 % total, da **menos** (99,8 %).

## Ejemplo de pregunta de examen

**Pregunta 1.** Una aplicación añade automáticamente máquinas virtuales cuando la CPU supera el 70 % y las elimina cuando baja del 30 %. ¿Qué beneficio de la nube describe este comportamiento?

- A) Agilidad
- B) Alta disponibilidad
- C) Elasticidad
- D) Previsibilidad

**Respuesta: C.** Ajustar la capacidad automáticamente en ambas direcciones es elasticidad.
- A) La agilidad es la rapidez para desplegar, no el ajuste automático.
- B) HA trata del tiempo activo, no de la capacidad.
- D) La previsibilidad es anticipar rendimiento y coste.

**Pregunta 2.** Un administrador cambia el tamaño de una máquina virtual de 4 vCPU a 16 vCPU para atender más carga. ¿Qué tipo de escalado ha realizado?

- A) Escalado horizontal
- B) Escalado vertical
- C) Escalado elástico
- D) Escalado geográfico

**Respuesta: B.** Aumentar la potencia de la misma máquina es escalar verticalmente (scale up).
- A) Horizontal sería añadir más VMs.
- C) No es un tipo de escalado; la elasticidad es el ajuste automático.
- D) No existe como categoría del examen.

## 🧠 Resumen para el examen

1. HA = tiempo activo garantizado por SLA; más nueves, menos caída.
2. Escalabilidad vertical (más potencia) vs horizontal (más instancias).
3. Elasticidad = escalado automático en ambos sentidos. Agilidad = desplegar rápido.
4. Confiabilidad = recuperarse de fallos por redundancia. Previsibilidad = rendimiento y coste anticipables.
5. Seguridad = proteger; gobernanza = cumplir estándares; manageability = gestionar la nube y en la nube.

---
