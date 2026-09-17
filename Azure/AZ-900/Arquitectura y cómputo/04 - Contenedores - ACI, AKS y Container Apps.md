---
tags: [az-900, azure, computo, contenedores, aci, aks, container-apps]
modulo: Arquitectura y cómputo
peso_examen: Medio
---

# Contenedores: ACI, AKS y Container Apps

## Concepto

Un **contenedor** es un paquete ligero que incluye una aplicación y **todo lo que necesita para ejecutarse** (runtime, librerías, configuración), pero **no un sistema operativo completo**: comparte el kernel del host. Arranca en segundos, ocupa poco y se ejecuta igual en cualquier sitio. La tecnología de referencia es **Docker**.

Azure ofrece tres formas de ejecutar contenedores, de menos a más sofisticación: **Azure Container Instances (ACI)**, **Azure Container Apps** y **Azure Kubernetes Service (AKS)**.

**Problema que resuelve:** las máquinas virtuales son pesadas (cada una lleva su SO), lentas de arrancar y difíciles de replicar exactamente. Los contenedores permiten empaquetar la app una vez y ejecutar muchas copias idénticas de forma rápida y densa.

**Para qué se utiliza:**
- Microservicios: cada servicio en su contenedor, escalado por separado.
- Aplicaciones que deben ejecutarse igual en desarrollo, pruebas y producción.
- Tareas puntuales o por lotes que no justifican una VM.

## Características principales

### Contenedor vs máquina virtual

| | Máquina virtual | Contenedor |
|---|---|---|
| Qué virtualiza | **Hardware** (cada VM tiene su SO) | **Sistema operativo** (comparten kernel) |
| Tamaño | GB | MB |
| Arranque | Minutos | Segundos |
| Aislamiento | Fuerte | Más ligero (a nivel de proceso) |
| Densidad | Pocas por host | Muchas por host |
| Gestión del SO | Tuya | No hay SO propio que gestionar |

### Azure Container Instances (ACI)
- La forma **más rápida y sencilla** de ejecutar un contenedor en Azure: **sin gestionar VMs ni orquestador**.
- Se considera **PaaS/serverless**: pagas **por segundo** de CPU y memoria mientras el contenedor se ejecuta.
- Ideal para **tareas simples, aisladas o puntuales**: procesos por lotes, pruebas, trabajos de compilación.
- No tiene escalado automático ni orquestación avanzada. Permite **grupos de contenedores** (varios contenedores que comparten host y red).

### Azure Container Apps
- Servicio **serverless para microservicios en contenedores**, construido sobre Kubernetes **pero sin exponer Kubernetes**.
- Aporta lo que ACI no tiene: **autoescalado** (incluido **escalado a cero**), balanceo de carga, división de tráfico entre versiones, integración con eventos (colas, HTTP).
- Punto intermedio: más capacidad que ACI, mucha menos complejidad que AKS.

### Azure Kubernetes Service (AKS)
- **Kubernetes administrado**: Azure gestiona el plano de control (gratis) y tú pagas los nodos (VMs) donde corren los contenedores.
- Kubernetes es el **orquestador** estándar: despliega, escala, reinicia y balancea cientos o miles de contenedores automáticamente.
- Máximo control y portabilidad (el mismo Kubernetes funciona en otras nubes y en local, gestionable con [[Azure Arc]]).
- Curva de aprendizaje alta: solo compensa para aplicaciones grandes con muchos contenedores.

### Azure Container Registry (ACR)
- **Registro privado** de imágenes de contenedor en Azure (equivalente privado de Docker Hub). ACI, Container Apps y AKS obtienen sus imágenes de aquí.

## Casos de uso

- Ejecutar un script de procesamiento nocturno empaquetado en Docker: **ACI**.
- Una API con picos irregulares que debe costar cero cuando nadie la llama: **Container Apps**.
- Una plataforma con 40 microservicios, despliegues continuos y equipos de operaciones expertos: **AKS**.
- Guardar las imágenes de la empresa en privado: **ACR**.

## Comparaciones

| Necesidad | Servicio | No confundir con |
|---|---|---|
| Ejecutar un contenedor rápido sin orquestación | ACI | AKS (demasiado complejo) |
| Microservicios con escalado a cero sin gestionar Kubernetes | Container Apps | AKS (exige gestionar clúster) |
| Orquestación completa y control total | AKS | Container Apps (abstrae Kubernetes) |
| Almacenar imágenes privadas | ACR | Blob Storage |
| Ejecutar código sin contenedor, por eventos | [[05 - Azure Functions y App Service|Azure Functions]] | ACI |
| Control del SO | [[03 - Máquinas virtuales, Scale Sets y conjuntos de disponibilidad|VM]] | Contenedor (no hay SO propio) |

## Conceptos que debo memorizar

> [!important]
> - **Contenedor** = app + dependencias, sin SO completo; comparte kernel; arranca en segundos.
> - **ACI** = ejecutar un contenedor **sin gestionar nada**, pago por segundo; para tareas simples.
> - **Container Apps** = microservicios serverless con **escalado a cero**, Kubernetes oculto.
> - **AKS** = **Kubernetes administrado**; orquestación a gran escala; plano de control gratis, pagas los nodos.
> - **ACR** = registro privado de imágenes.
> - Escala de complejidad: ACI < Container Apps < AKS.

## Tips para AZ-900

> [!tip]
> - "La forma **más rápida/simple** de ejecutar un contenedor" → **ACI**.
> - "**Orquestar**", "Kubernetes", "cientos de contenedores" → **AKS**.
> - "Microservicios", "escalar a cero", "sin gestionar Kubernetes" → **Container Apps**.
> - "Contenedor vs VM": el contenedor **no incluye SO** y es más ligero; la VM da más aislamiento.
> - Trampa: "AKS es gratuito". Parcialmente: el plano de control no se cobra, pero los **nodos** sí.
> - Trampa: "Los contenedores requieren gestionar el sistema operativo invitado". **Falso**.

## Ejemplo de pregunta de examen

**Pregunta 1.** Necesitas ejecutar un único contenedor Docker para una tarea de procesamiento de una hora, con la menor sobrecarga administrativa posible. ¿Qué servicio debes usar?

- A) Azure Kubernetes Service
- B) Azure Container Instances
- C) Azure Virtual Machines
- D) Azure Virtual Desktop

**Respuesta: B.** ACI ejecuta contenedores sin gestionar clústeres ni VMs, y cobra por segundo.
- A) AKS requiere crear y administrar un clúster; excesivo para un contenedor.
- C) Habría que instalar Docker y gestionar el SO.
- D) Es un servicio de escritorios, no de contenedores.

**Pregunta 2.** ¿Cuál de las siguientes es una ventaja de los contenedores frente a las máquinas virtuales?

- A) Cada contenedor incluye su propio sistema operativo completo.
- B) Los contenedores arrancan más rápido y consumen menos recursos.
- C) Los contenedores ofrecen mayor aislamiento de hardware.
- D) Los contenedores solo se ejecutan en Azure.

**Respuesta: B.** Al compartir el kernel del host, son ligeros y arrancan en segundos.
- A) Es justo lo contrario: no incluyen SO completo.
- C) El aislamiento de hardware es mayor en las VMs.
- D) Son portables a cualquier entorno que ejecute contenedores.

## 🧠 Resumen para el examen

1. Contenedor = app + dependencias sin SO completo; más ligero y rápido que una VM.
2. ACI = contenedor simple sin infraestructura, pago por segundo.
3. Container Apps = microservicios serverless, escalado a cero, Kubernetes oculto.
4. AKS = Kubernetes administrado para orquestación a gran escala.
5. ACR = registro privado de imágenes.

---
