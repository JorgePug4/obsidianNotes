---
tags: [az-900, azure, conceptos-nube, responsabilidad-compartida]
modulo: Conceptos de la nube
peso_examen: Alto
---

# Qué es la computación en la nube

## Concepto

La **computación en la nube** (cloud computing) es la **entrega de servicios de computación a través de Internet**: potencia de cálculo (servidores, máquinas virtuales), almacenamiento, redes, bases de datos, análisis e inteligencia artificial, entre otros. Tú consumes esos servicios **bajo demanda** y pagas solo por lo que usas, mientras el proveedor (Microsoft, en el caso de Azure) mantiene el hardware y los centros de datos.

**Problema que resuelve:** montar infraestructura propia exige comprar servidores, alquilar espacio, pagar electricidad y refrigeración, contratar técnicos y adivinar cuánta capacidad vas a necesitar dentro de tres años. La nube elimina esa inversión inicial y ese riesgo: pides lo que necesitas hoy y lo devuelves mañana.

**Para qué se utiliza:**
- Alojar aplicaciones y sitios web sin comprar servidores.
- Almacenar y respaldar datos de forma duradera.
- Escalar rápidamente cuando la demanda crece (rebajas, campañas, picos estacionales).
- Acceder a tecnologías avanzadas (IA, big data, IoT) sin construir la plataforma desde cero.

## Características principales

- **Autoservicio bajo demanda**: creas recursos desde un portal o una API, sin pedirlos a nadie.
- **Acceso amplio por red**: todo se consume por Internet, desde cualquier dispositivo.
- **Agrupación de recursos**: el proveedor comparte infraestructura entre muchos clientes (multitenencia), lo que abarata el coste.
- **Elasticidad rápida**: la capacidad crece y decrece automáticamente según la demanda.
- **Servicio medido**: pagas por consumo real (segundos de CPU, GB almacenados, GB transferidos).

### El modelo de responsabilidad compartida

En la nube la seguridad y la administración **se reparten** entre el proveedor y el cliente. **Cuánto asume cada uno depende del tipo de servicio** (ver [[05 - IaaS, PaaS, SaaS y serverless]]).

| Responsabilidad | Local (on-premises) | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Datos e información | Cliente | Cliente | Cliente | Cliente |
| Dispositivos (PCs, móviles) | Cliente | Cliente | Cliente | Cliente |
| Cuentas e identidades | Cliente | Cliente | Cliente | Cliente |
| Infraestructura de identidad y directorio | Cliente | Cliente | **Compartida** | **Compartida** |
| Aplicaciones | Cliente | Cliente | **Compartida** | Microsoft |
| Controles de red | Cliente | Cliente | **Compartida** | Microsoft |
| Sistema operativo | Cliente | Cliente | Microsoft | Microsoft |
| Hosts físicos | Cliente | Microsoft | Microsoft | Microsoft |
| Red física | Cliente | Microsoft | Microsoft | Microsoft |
| Centro de datos físico | Cliente | Microsoft | Microsoft | Microsoft |

Regla mental: **cuanto más "as a Service", más asume Microsoft**. Pero **los datos, los dispositivos y las identidades son siempre del cliente**, en cualquier modelo.

## Casos de uso

- Una startup lanza su producto sin comprar ni un servidor: todo en Azure desde el primer día.
- Una tienda online triplica sus servidores durante el Black Friday y los apaga el lunes siguiente.
- Un hospital guarda copias de seguridad en la nube para no depender de cintas físicas.
- Una empresa migra su correo a Microsoft 365 (SaaS) y deja de administrar servidores de Exchange.

## Comparaciones

| Concepto | Qué es | No confundir con |
|---|---|---|
| Computación en la nube | Consumir servicios de cómputo por Internet, bajo demanda | Simplemente "tener un servidor en otro edificio" (eso es hosting tradicional) |
| Responsabilidad compartida | Reparto de tareas de seguridad y gestión entre proveedor y cliente | "Microsoft se encarga de toda la seguridad" (falso: los datos y las identidades siempre son tuyos) |
| Servicio medido | Facturar por uso real | Tarifa plana mensual |

## Conceptos que debo memorizar

> [!important]
> - Cloud computing = **servicios de computación entregados por Internet, bajo demanda y con pago por uso**.
> - Responsabilidad compartida: **Microsoft cubre siempre lo físico** (centro de datos, red, hosts); **el cliente cubre siempre datos, dispositivos e identidades**.
> - Lo que queda en medio (SO, red virtual, aplicaciones) **depende del modelo**: en IaaS lo lleva el cliente; en SaaS lo lleva Microsoft; en PaaS se comparte.

## Tips para AZ-900

> [!tip]
> - Palabras clave: *bajo demanda, por Internet, pago por uso, sin comprar hardware, el proveedor mantiene*.
> - Si la pregunta dice "¿quién es responsable de X en IaaS?" y X es el **sistema operativo** o los **parches**: el **cliente**. Si X es el **hardware**: **Microsoft**.
> - Trampa: "En SaaS Microsoft es responsable de los datos". **Falso**. Los datos son siempre responsabilidad del cliente.
> - Trampa: "La responsabilidad compartida solo aplica a IaaS". **Falso**. Aplica a los tres modelos con distinto reparto.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tu empresa despliega una máquina virtual en Azure (IaaS). ¿Quién es responsable de aplicar las actualizaciones de seguridad del sistema operativo?

- A) Microsoft
- B) El cliente
- C) Es una responsabilidad compartida
- D) El proveedor del sistema operativo

**Respuesta: B.** En IaaS el cliente gestiona todo desde el sistema operativo hacia arriba: parches, antivirus, aplicaciones y datos.
- A) Microsoft gestiona el host físico y el hipervisor, no el SO del invitado.
- C) El SO en IaaS es responsabilidad íntegra del cliente, no compartida.
- D) El proveedor publica los parches, pero aplicarlos es tarea del cliente.

**Pregunta 2.** ¿Cuál de las siguientes responsabilidades corresponde **siempre** al cliente, independientemente del modelo de servicio?

- A) La red física
- B) El sistema operativo
- C) Los datos y las cuentas de usuario
- D) Los hosts físicos

**Respuesta: C.** Datos, dispositivos e identidades nunca dejan de ser responsabilidad del cliente.
- A y D) Son siempre de Microsoft en cualquier modelo de nube.
- B) Depende del modelo: cliente en IaaS, Microsoft en PaaS y SaaS.

## 🧠 Resumen para el examen

1. Cloud computing = servicios de cómputo entregados por Internet, bajo demanda y con pago por uso.
2. Cinco características: autoservicio, acceso por red, recursos compartidos, elasticidad, servicio medido.
3. Responsabilidad compartida: lo físico es de Microsoft; datos, dispositivos e identidades son del cliente.
4. El reparto de lo intermedio (SO, red, apps) depende de si es IaaS, PaaS o SaaS.
5. Cuanto más "as a Service", menos responsabilidad para el cliente.

---
