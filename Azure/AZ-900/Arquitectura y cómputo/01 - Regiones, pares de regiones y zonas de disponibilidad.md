---
tags: [az-900, azure, arquitectura, regiones, zonas-disponibilidad]
modulo: Arquitectura y cómputo
peso_examen: Alto
---

# Regiones, pares de regiones y zonas de disponibilidad

## Concepto

La infraestructura física de Azure se organiza en capas: **centros de datos** agrupados en **zonas de disponibilidad**, que a su vez forman **regiones**, que se emparejan en **pares de regiones** dentro de una **geografía**. Cada capa protege contra un tipo de fallo distinto.

**Problema que resuelve:** el hardware falla, los centros de datos sufren incendios o cortes de luz, y las regiones enteras pueden verse afectadas por desastres naturales. Distribuir los recursos en varias capas físicas evita que un único incidente tumbe tu aplicación.

**Para qué se utiliza:** para elegir dónde desplegar cada recurso según la latencia que necesitan los usuarios, las leyes de residencia de datos y el nivel de disponibilidad requerido.

## Características principales

### Centro de datos (datacenter)
- Edificio físico con servidores, alimentación, refrigeración y red propias. Es la unidad más pequeña. Nunca eliges un datacenter concreto.

### Región
- **Conjunto de centros de datos** cercanos entre sí, conectados por una red de baja latencia. Tiene un nombre (por ejemplo, *West Europe*, *East US*).
- Es **la unidad en la que despliegas**: casi todo recurso pide una región.
- Cada región tiene su propio **catálogo de servicios** y **precios**; no todos los servicios están en todas las regiones.
- Algunos servicios son **globales** (no piden región): Microsoft Entra ID, Azure DNS, Traffic Manager, Front Door.

### Geografía
- Área que agrupa regiones y respeta **fronteras de residencia de datos y cumplimiento** (por ejemplo, Europa, Estados Unidos). Los datos replicados entre regiones emparejadas nunca salen de su geografía.

### Regiones soberanas
- Regiones **aisladas** del Azure público para cumplir requisitos legales de gobiernos: **Azure Government** (EE. UU.), **Azure China** (operada por 21Vianet). Requieren contrato aparte y tienen catálogo propio.

### Zonas de disponibilidad (Availability Zones)
- Dentro de una región, **uno o más centros de datos físicamente separados**, cada uno con **alimentación, refrigeración y red independientes**.
- Una región con zonas tiene **un mínimo de tres**. Están lo bastante lejos para que un incendio o un corte no afecte a dos a la vez, y lo bastante cerca para replicar de forma **síncrona**.
- Protegen contra el **fallo de un centro de datos completo**.
- Tres formas de usar las zonas:

| Tipo de servicio | Cómo se despliega | Ejemplo |
|---|---|---|
| **Zonal** | Tú fijas el recurso en una zona concreta (y replicas tú en otra) | VM, disco administrado, IP pública |
| **Con redundancia de zona** | Azure replica automáticamente entre zonas | [[Redundancia de almacenamiento|Almacenamiento ZRS]], Azure SQL, Load Balancer estándar |
| **Siempre disponible (no zonal)** | Servicios globales resistentes a fallos de zona y región | Entra ID, DNS, Traffic Manager |

### Pares de regiones (region pairs)
- Cada región se **empareja con otra de la misma geografía**, separada por **al menos 300 millas (≈480 km)**. Ejemplos: West Europe ↔ North Europe; East US ↔ West US.
- Beneficios:
  - Protegen contra **desastres que afectan a una región entera** (terremoto, huracán, corte eléctrico masivo).
  - Las **actualizaciones planificadas** de Azure se aplican **de forma secuencial**, nunca a las dos a la vez.
  - Si hay una interrupción amplia, **una región del par tiene prioridad de recuperación**.
  - Los datos replicados geográficamente (GRS) **se quedan en la misma geografía** (residencia de datos).
- Algunos servicios usan el par de forma automática (GRS de almacenamiento); otros requieren que tú diseñes la replicación.
- Existen regiones nuevas **sin par**, que ofrecen redundancia solo con zonas de disponibilidad.

### Resumen de qué protege cada capa

| Capa | Protege contra | Latencia entre copias | Replicación |
|---|---|---|---|
| Varias VMs en un mismo datacenter (conjunto de disponibilidad) | Fallo de un rack o un host | Mínima | Síncrona |
| Zonas de disponibilidad | Fallo de un **centro de datos** | Baja (misma región) | Síncrona |
| Pares de regiones | Fallo de una **región** | Alta (cientos de km) | Asíncrona |

## Casos de uso

- Usuarios en España: se despliega en *West Europe* o *Spain Central* para minimizar latencia.
- Una app crítica necesita 99,99 %: VMs repartidas en **tres zonas** tras un balanceador.
- Un banco exige que los datos nunca salgan de la UE: región europea con GRS al par (también europeo).
- Plan de recuperación ante desastres: réplica en la región emparejada con conmutación por error.

## Comparaciones

| Necesidad | Solución | No confundir con |
|---|---|---|
| Sobrevivir al fallo de un datacenter | Zonas de disponibilidad | Conjunto de disponibilidad (solo protege dentro del datacenter) |
| Sobrevivir al fallo de toda una región | Par de regiones / GRS | Zonas (todas en la misma región) |
| Cumplir leyes de residencia de datos | Geografía / región adecuada | Zona (no tiene que ver con fronteras legales) |
| Cumplir requisitos de un gobierno concreto | Región soberana (Azure Government) | Región normal con Policy |
| Servicio que no pide región al crearlo | Servicio global (Entra ID, DNS) | Servicio regional mal configurado |

## Conceptos que debo memorizar

> [!important]
> - **Datacenter ⊂ Zona ⊂ Región ⊂ Geografía.**
> - **Región** = unidad de despliegue; no todos los servicios están en todas las regiones.
> - **Zonas**: mínimo **3** por región; datacenters separados con **alimentación, refrigeración y red propias**; protegen contra fallo de **datacenter**; replicación **síncrona**.
> - **Pares**: misma geografía, **≥300 millas**, actualizaciones **secuenciales**, prioridad de recuperación; protegen contra fallo de **región**.
> - **Soberanas**: Azure Government y Azure China, aisladas del Azure público.
> - Servicios **globales** (sin región): Entra ID, DNS, Traffic Manager, Front Door.

## Tips para AZ-900

> [!tip]
> - Palabras clave zonas: *"centro de datos"*, *"alimentación y refrigeración independientes"*, *"dentro de una región"*, *"síncrono"*.
> - Palabras clave pares: *"desastre regional"*, *"300 millas"*, *"misma geografía"*, *"actualizaciones una a una"*.
> - "¿Cuántas zonas como mínimo?" → **tres**.
> - Trampa: "Las zonas de disponibilidad protegen contra la pérdida de una región". **Falso**, todas están dentro de la región.
> - Trampa: "Puedes elegir en qué centro de datos se crea tu VM". **Falso**, eliges región y opcionalmente zona.
> - Trampa: "Todos los servicios de Azure están disponibles en todas las regiones". **Falso**.

## Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas que una aplicación siga funcionando aunque un centro de datos completo de la región quede fuera de servicio. ¿Qué debes usar?

- A) Un conjunto de disponibilidad
- B) Zonas de disponibilidad
- C) Un par de regiones
- D) Una región soberana

**Respuesta: B.** Las zonas son datacenters independientes dentro de la región; repartir la app entre ellas sobrevive a la caída de uno.
- A) Solo protege contra fallos de rack o host dentro de un mismo datacenter.
- C) Protege contra la caída de la región entera; es más de lo pedido y más costoso y lento (asíncrono).
- D) Es una región aislada para requisitos gubernamentales, no una técnica de disponibilidad.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre los pares de regiones de Azure es verdadera?

- A) Las dos regiones del par pueden estar en geografías distintas.
- B) Las actualizaciones planificadas se aplican a ambas regiones simultáneamente.
- C) Las regiones del par están separadas por al menos 300 millas.
- D) Cada región de Azure tiene exactamente tres regiones emparejadas.

**Respuesta: C.** La separación mínima de 300 millas es lo que protege contra desastres regionales.
- A) Siempre están en la misma geografía para respetar la residencia de datos.
- B) Se aplican de forma secuencial, precisamente para que una siga disponible.
- D) Cada región tiene como máximo un par.

## 🧠 Resumen para el examen

1. Jerarquía física: datacenter → zona de disponibilidad → región → geografía.
2. La región es la unidad de despliegue; los servicios y precios varían por región.
3. Zonas: mínimo tres, independientes en energía/refrigeración/red, protegen contra fallo de datacenter.
4. Pares: misma geografía, ≥300 millas, actualizaciones secuenciales, protegen contra fallo de región.
5. Regiones soberanas (Government, China) están aisladas del Azure público.
6. Servicios globales (Entra ID, DNS) no requieren región.

---
