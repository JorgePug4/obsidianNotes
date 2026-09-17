---
tags: [az-900, azure, conceptos-nube, iaas, paas, saas, serverless]
modulo: Conceptos de la nube
peso_examen: Alto
---

# IaaS, PaaS, SaaS y serverless

## Concepto

Los **tipos de servicio en la nube** describen **cuánto administra el proveedor y cuánto administras tú**. Se ordenan de menor a mayor abstracción: **IaaS** (infraestructura), **PaaS** (plataforma) y **SaaS** (software). **Serverless** es un caso extremo de PaaS en el que ni siquiera ves los servidores.

**Problema que resuelve:** no todas las cargas necesitan el mismo control. Un ERP heredado necesita un servidor completo; una web moderna solo necesita dónde ejecutar código; el correo solo necesita usarse. Cada tipo ofrece el equilibrio adecuado entre control y comodidad.

**Para qué se utiliza:** para elegir el servicio de Azure correcto según cuánto quieras (o puedas) administrar.

## Características principales

La analogía clásica: **la pizza**.

| Tipo | Analogía | Tú gestionas | El proveedor gestiona | Ejemplos de Azure |
|---|---|---|---|---|
| **On-premises** | Hacerla en casa | Todo | Nada | Tu CPD |
| **IaaS** | Comprar la masa y hornearla tú | SO, middleware, runtime, apps, datos | Hardware, red, virtualización | [[03 - Máquinas virtuales, Scale Sets y conjuntos de disponibilidad|Máquinas virtuales]], [[Azure Disk Storage]], [[01 - Azure Virtual Network]] |
| **PaaS** | Pedir a domicilio | Apps y datos | Todo lo demás, incluido SO y runtime | [[05 - Azure Functions y App Service|App Service]], Azure SQL Database, [[Azure Cosmos DB]], [[Microsoft Entra Domain Services]] |
| **SaaS** | Ir al restaurante | Solo usarla (y tus datos) | Todo, incluida la aplicación | Microsoft 365, Dynamics 365, Teams |

### IaaS (Infrastructure as a Service)
- Alquilas **infraestructura**: VMs, discos, redes.
- **Máximo control** y máxima responsabilidad: instalas y parcheas el SO, configuras el firewall, gestionas backups.
- Modelo *lift-and-shift*: mover una carga tal cual desde local.
- Casos: migraciones rápidas, software que necesita un SO concreto, entornos de prueba y desarrollo, almacenamiento y backup.

### PaaS (Platform as a Service)
- Alquilas una **plataforma lista** para ejecutar tu código o tus datos.
- No gestionas SO ni parches; te centras en **desarrollar**.
- Incluye herramientas de desarrollo, bases de datos administradas, análisis.
- Casos: desarrollo ágil, aplicaciones web, APIs, bases de datos sin DBA, análisis.

### SaaS (Software as a Service)
- Alquilas la **aplicación terminada**, normalmente por suscripción por usuario.
- Cero administración técnica; solo configuras usuarios y datos.
- Casos: correo, ofimática, CRM, mensajería.

### Serverless
- Subconjunto de PaaS: **ejecutas código sin aprovisionar ni administrar servidores**. El proveedor escala automáticamente y **pagas solo por ejecución** (a cero si no se usa).
- Ejemplos: **Azure Functions** (código por eventos), **Logic Apps** (flujos sin código), **Azure Container Apps** con escalado a cero.
- Sigue habiendo servidores; simplemente **no los ves ni los pagas cuando no trabajan**.

## Casos de uso

- Migrar un servidor de archivos Windows antiguo a Azure sin cambiar nada: **IaaS** (VM).
- Un equipo de desarrollo quiere desplegar una API sin preocuparse por el SO: **PaaS** (App Service).
- Una pyme quiere correo y Office sin servidores: **SaaS** (Microsoft 365).
- Procesar cada imagen subida a un blob con una función que se dispara al subirla: **serverless** (Functions).

## Comparaciones

| Escenario | Tipo | No confundir con |
|---|---|---|
| "Necesitamos control total del SO" | IaaS | PaaS (no da acceso al SO) |
| "No queremos administrar parches del servidor" | PaaS o SaaS | IaaS (los parches son tuyos) |
| "Solo queremos usar la aplicación" | SaaS | PaaS (aún desarrollas la app) |
| "Pagar solo cuando se ejecuta el código" | Serverless | PaaS clásico (App Service cobra por plan aunque esté inactivo) |
| "Base de datos sin administrar el servidor" | PaaS (Azure SQL Database) | IaaS (SQL Server en una VM) |

Relación con la [[01 - Qué es la computación en la nube|responsabilidad compartida]]: **IaaS** = el cliente asume más; **SaaS** = Microsoft asume más; **PaaS** = compartido.

## Conceptos que debo memorizar

> [!important]
> - **IaaS** = VMs, discos, redes. Tú administras el **SO**. Máximo control. *Lift-and-shift*.
> - **PaaS** = plataforma para tu código y datos. Microsoft administra el SO y el runtime. Para **desarrolladores**.
> - **SaaS** = aplicación lista. Solo la usas. Suscripción por usuario.
> - **Serverless** = PaaS sin servidores visibles, escalado automático, **pago por ejecución**. Functions y Logic Apps.
> - Escala de control: on-premises > IaaS > PaaS > SaaS. Escala de comodidad: al revés.

## Tips para AZ-900

> [!tip]
> - Palabras clave IaaS: *máquina virtual, sistema operativo, control total, migrar tal cual, instalar software*.
> - Palabras clave PaaS: *desarrolladores, desplegar código, base de datos administrada, sin gestionar parches*.
> - Palabras clave SaaS: *Microsoft 365, correo, suscripción por usuario, aplicación lista*.
> - Palabras clave serverless: *evento, sin servidores, escala a cero, pago por ejecución*.
> - Trampa: "Azure Virtual Machines es PaaS". **Falso**, es IaaS.
> - Trampa: "Microsoft Entra ID es IaaS". **Falso**, es un servicio de identidad (se considera PaaS/SaaS).
> - Trampa: "Serverless significa que no hay servidores". **Falso**, significa que tú no los administras.

## Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa quiere migrar una aplicación heredada que requiere una versión concreta de Windows Server y software de terceros instalado en el sistema operativo. ¿Qué tipo de servicio debe elegir?

- A) SaaS
- B) PaaS
- C) IaaS
- D) Serverless

**Respuesta: C.** Solo IaaS da acceso al sistema operativo para instalar software y elegir versión.
- A) SaaS no permite instalar nada.
- B) PaaS abstrae el SO; no puedes elegir versión ni instalar software de terceros.
- D) Serverless ejecuta funciones, no servidores completos.

**Pregunta 2.** ¿Cuál de los siguientes es un ejemplo de SaaS?

- A) Azure Virtual Machines
- B) Azure App Service
- C) Microsoft 365
- D) Azure SQL Database

**Respuesta: C.** Microsoft 365 es software terminado que se consume por suscripción.
- A) Es IaaS.
- B y D) Son PaaS.

**Pregunta 3.** Un desarrollador quiere ejecutar un fragmento de código cada vez que llega un mensaje a una cola, pagando solo por las ejecuciones y sin administrar ningún servidor. ¿Qué servicio se ajusta mejor?

- A) Azure Virtual Machines
- B) Azure Functions
- C) Azure Virtual Desktop
- D) Azure Files

**Respuesta: B.** Functions es serverless: se dispara por eventos y factura por ejecución.
- A) Requiere administrar la VM y se paga mientras está encendida.
- C) Es escritorio virtual, no ejecución de código.
- D) Es almacenamiento de archivos.

## 🧠 Resumen para el examen

1. IaaS = infraestructura (VMs); tú gestionas el SO. PaaS = plataforma; tú gestionas código y datos. SaaS = aplicación lista.
2. Más "aaS" = menos control y menos responsabilidad para ti.
3. Serverless = PaaS sin servidores visibles, escala a cero y pago por ejecución (Functions, Logic Apps).
4. Ejemplos: VM = IaaS; App Service y SQL Database = PaaS; Microsoft 365 = SaaS; Functions = serverless.
5. Cada tipo cambia el reparto de la responsabilidad compartida.

---
