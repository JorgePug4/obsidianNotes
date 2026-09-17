---
tags: [bases-de-datos, sistemas-distribuidos, nosql, consistencia]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [CAP, Teorema CAP, BASE, Consistencia eventual]
---

# Teorema CAP y BASE

> [!info] Propuesto por **Eric Brewer** (2000) y demostrado formalmente por Gilbert y Lynch (2002). Es el marco mental para entender por qué los sistemas distribuidos "sacrifican" consistencia.

## ¿Qué es?

El **Teorema CAP** dice que un sistema de datos **distribuido** (varios nodos que replican datos) solo puede garantizar **dos de estas tres propiedades a la vez**:

| Letra | Propiedad | Significado |
|---|---|---|
| **C** | **Consistency** (Consistencia) | Toda lectura devuelve la escritura más reciente, en **cualquier nodo**. Todos ven el mismo dato al mismo tiempo. |
| **A** | **Availability** (Disponibilidad) | Toda petición recibe una respuesta (no error), aunque sea con datos no del todo actualizados. |
| **P** | **Partition tolerance** (Tolerancia a particiones) | El sistema sigue funcionando aunque la red **separe** a los nodos y no puedan comunicarse entre sí. |

> [!important] La lectura correcta del teorema
> En una red real, las particiones **van a ocurrir** (cables, switches, zonas de disponibilidad caídas). Por tanto, **P no es opcional**. La decisión real es: **cuando hay una partición, ¿prefiero Consistencia (CP) o Disponibilidad (AP)?**
> - **CP**: ante una partición, algunos nodos **rechazan** peticiones (error o timeout) para no devolver datos posiblemente desactualizados.
> - **AP**: ante una partición, todos los nodos **responden** con lo que tienen, aunque pueda estar desactualizado; se reconcilia después.
> Un sistema "CA" solo existe si no hay red distribuida (una sola máquina).

## ¿Para qué sirve?

- **Elegir base de datos**: entender por qué Cassandra o DynamoDB responden siempre pero pueden devolver datos viejos, y por qué un clúster de PostgreSQL o etcd puede rechazar escrituras si pierde el quórum.
- **Diseñar microservicios**: sin base de datos compartida, la consistencia entre servicios es **eventual** por naturaleza ([[Saga Pattern]], [[Transactional Outbox]]).
- **Gestionar expectativas del negocio**: "¿por qué el pedido tarda 2 segundos en aparecer en el historial?" tiene respuesta en CAP.

## Conceptos relacionados

- [[ACID en Bases de Datos|ACID]] → modelo de consistencia fuerte de las bases relacionales. **La "C" de ACID (integridad de reglas) no es la "C" de CAP (todos los nodos ven lo mismo).**
- [[Bases de Datos SQL vs NoSQL]] → la mayoría de NoSQL distribuidas son AP por defecto, configurables.
- [[Saga Pattern]], [[CQRS (Command Query Responsibility Segregation)|CQRS]], [[Event Sourcing]] → patrones que asumen y gestionan la consistencia eventual.
- [[Azure Cosmos DB]] → ejemplo real de base que deja **elegir el nivel de consistencia** por operación.
- [[Redundancia de almacenamiento]] → la replicación geográfica de Azure Storage (GRS) es asíncrona: un ejemplo de consistencia eventual entre regiones.
- [[🏗️ Diseño de Microservicios]] → sección de gestión de datos.

## ¿Cómo funciona?

### Escenario: dos nodos y una partición

```
        Cliente A                          Cliente B
           │ escribe saldo = 50               │ lee saldo
           ▼                                  ▼
     ┌──────────┐    ✂ partición ✂     ┌──────────┐
     │  Nodo 1  │ ─ ─ ─ ─ ✕ ─ ─ ─ ─ ─ │  Nodo 2  │
     │ saldo=50 │   no pueden hablar   │ saldo=80 │  (valor antiguo)
     └──────────┘                      └──────────┘
```

¿Qué hace el Nodo 2 cuando el Cliente B pregunta?

| Opción | Comportamiento | Sistema |
|---|---|---|
| **CP** | "No puedo confirmar que tengo el dato más reciente" → **error / espera** | Zookeeper, etcd, HBase, MongoDB (por defecto, escrituras al primario), clúster SQL con quórum |
| **AP** | "Toma 80" (dato **posiblemente viejo**) → **responde** | Cassandra, DynamoDB, CouchDB, Riak, DNS, Cosmos DB en consistencia eventual |

Cuando la partición se cura, el sistema AP **reconcilia**: última escritura gana (*last-writer-wins*), relojes vectoriales, CRDTs o resolución en la aplicación.

### Consistencia fuerte vs eventual

- **Consistencia fuerte** (*strong / linearizable*): después de confirmar una escritura, **cualquier** lectura posterior la ve. Es lo que da una base relacional de un solo nodo.
- **Consistencia eventual**: después de una escritura, **si dejan de llegar escrituras**, todos los nodos **acabarán** convergiendo al mismo valor. Mientras tanto, distintas lecturas pueden ver distintos valores.

Entre ambos hay niveles intermedios. [[Azure Cosmos DB]] expone cinco: *Strong, Bounded Staleness, Session, Consistent Prefix, Eventual*. **Session** (un cliente siempre ve sus propias escrituras) es el más usado porque resuelve el caso más molesto: "guardé y no lo veo".

### PACELC: la extensión práctica

Daniel Abadi (2012) señaló que CAP solo habla de qué pasa **durante una partición**. Pero el resto del tiempo también hay un *trade-off*: **latencia vs consistencia**. Replicar sincrónicamente a 3 regiones da consistencia fuerte pero suma cientos de milisegundos.

> **PACELC**: si hay **P**artición, elige **A** o **C**; **E**n caso contrario (*else*), elige **L**atencia o **C**onsistencia.

Cassandra y DynamoDB son PA/EL (favorecen disponibilidad y latencia). Un clúster SQL sincrónico es PC/EC.

## BASE: el modelo alternativo a ACID

Los sistemas AP siguen el modelo **BASE**, acrónimo deliberadamente opuesto a [[ACID en Bases de Datos|ACID]]:

| Letra | Significado | Qué implica |
|---|---|---|
| **BA** | **Basically Available** | El sistema responde siempre, aunque parte de los datos esté desactualizada o parte de los nodos caída |
| **S** | **Soft state** | El estado puede cambiar **sin nuevas escrituras**, por la propagación de réplicas en segundo plano |
| **E** | **Eventually consistent** | Con el tiempo, todas las réplicas convergen |

| | ACID | BASE |
|---|---|---|
| Objetivo | Corrección inmediata | Disponibilidad y escala |
| Consistencia | Fuerte | Eventual |
| Coste | Bloqueos, coordinación, menor escala horizontal | La aplicación debe tolerar datos desactualizados y resolver conflictos |
| Dónde | Relacionales, transacciones financieras | NoSQL distribuidas, redes sociales, catálogos, carritos, métricas |

## Ejemplo aplicado a microservicios

1. El servicio **Pedidos** confirma un pedido y publica `PedidoConfirmado` ([[Transactional Outbox]]).
2. El servicio **Historial** consume el evento 300 ms después y actualiza su vista.
3. Si el usuario abre el historial en esos 300 ms, **no ve el pedido**.

Esto es consistencia eventual. Soluciones en la interfaz: mostrar el pedido de forma optimista desde el cliente, indicar "procesando", o leer del servicio de Pedidos para esa pantalla concreta (consistencia de sesión).

## Ventajas de aceptar consistencia eventual

- **Disponibilidad**: el sistema no se cae porque una región no responda.
- **Latencia baja**: no hay que esperar a que todas las réplicas confirmen.
- **Escala horizontal** sin coordinación global.

## Desventajas / Limitaciones

- La aplicación y la interfaz deben **diseñarse para datos desactualizados**.
- **Resolución de conflictos** cuando dos nodos aceptan escrituras contradictorias.
- Más difícil de razonar y de probar.
- Inadecuada donde un dato viejo es inaceptable (saldo antes de autorizar un pago, control de stock estricto).

## Comparación rápida de sistemas

| Sistema | Tendencia | Comentario |
|---|---|---|
| PostgreSQL / SQL Server (un nodo) | CA (no distribuido) | Consistencia fuerte; la disponibilidad depende del único nodo |
| PostgreSQL con réplica síncrona / etcd / Zookeeper | **CP** | Rechazan escrituras sin quórum |
| MongoDB | **CP** por defecto | Lecturas del primario; configurable |
| Cassandra / DynamoDB | **AP** | Consistencia ajustable por operación (quórum) |
| [[Azure Cosmos DB]] | Configurable | Cinco niveles de consistencia |
| Redis Cluster | AP (aprox.) | Replicación asíncrona |

## Puntos clave

- CAP: con una **partición** solo puedes elegir **C o A**; **P** es obligatorio en la práctica.
- **CP** = errores antes que datos viejos. **AP** = datos viejos antes que errores.
- La "C" de **CAP** ≠ la "C" de **ACID**.
- **BASE** = Basically Available, Soft state, Eventually consistent: el modelo de los sistemas AP.
- **PACELC** añade el *trade-off* latencia vs consistencia cuando no hay partición.
- Los microservicios con *database-per-service* son **eventualmente consistentes** entre sí por diseño.

## Errores comunes

- Decir que "una base es CA" (solo tiene sentido sin red distribuida).
- Elegir AP para saldos bancarios o stock estricto.
- Elegir CP y sorprenderse de que el sistema devuelva errores cuando cae una zona.
- No diseñar la interfaz para la consistencia eventual y generar tickets de "mis datos desaparecen".
- Confundir consistencia de CAP con integridad referencial de ACID.

## 🎯 Para entrevistas y exámenes

- *"Explica CAP"* → Con una partición, eliges consistencia o disponibilidad; P no es negociable.
- *"¿Es MongoDB CP o AP?"* → CP por defecto (escrituras y lecturas al primario), ajustable.
- *"¿Qué es BASE?"* → El modelo de consistencia eventual, opuesto a ACID.
- *"¿Cómo manejas consistencia eventual en la UI?"* → Optimistic UI, estados "procesando", consistencia de sesión.
- En **AZ-900**: Cosmos DB ofrece **cinco niveles de consistencia** configurables; GRS replica de forma **asíncrona** a la región secundaria.

## Referencias

- Eric Brewer, *CAP Twelve Years Later: How the "Rules" Have Changed* (2012): https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/
- Gilbert & Lynch, *Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services* (2002)
- Daniel Abadi, *Consistency Tradeoffs in Modern Distributed Database System Design* (PACELC, 2012)
- Martin Kleppmann, *Designing Data-Intensive Applications*, capítulo 9 *Consistency and Consensus*
- Microsoft Learn, *Consistency levels in Azure Cosmos DB*: https://learn.microsoft.com/azure/cosmos-db/consistency-levels

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
