---
tags: [az-900, azure, costes, facturacion]
modulo: Costes y herramientas
peso_examen: Alto
---

# Factores que afectan al coste en Azure

## Concepto

La factura de Azure no depende solo de "cuántos recursos tienes". Depende del **tipo** de recurso, de **cuánto** lo usas, de **dónde** está, de **cuánto tráfico** genera y de **cómo lo compras**. Conocer esos factores es lo que permite optimizar el gasto sin sacrificar rendimiento.

**Problema que resuelve:** sin entender qué se cobra, es fácil dejar VMs encendidas sin uso, pagar tráfico de salida innecesario o elegir la región más cara sin motivo.

**Para qué se utiliza:** para diseñar soluciones económicas, pronosticar el gasto y explicar las variaciones de la factura.

## Características principales

### Los seis factores del temario

| Factor | Cómo afecta | Ejemplo |
|---|---|---|
| **Tipo de recurso** | Cada servicio tiene su propio modelo de precio y sus medidores | Una VM cobra por tiempo; Blob por GB y operaciones; Functions por ejecución |
| **Consumo** | Pagas lo que usas: horas, GB, transacciones | Una VM 24×7 cuesta el doble que una 12 h al día |
| **Mantenimiento** | Recursos huérfanos siguen costando | Discos y IPs públicas de VMs ya borradas; entornos de prueba olvidados |
| **Geografía** | El precio del mismo recurso varía según la región | Un tamaño de VM puede ser más caro en Brasil que en EE. UU. |
| **Tráfico de red** | La **entrada** (ingress) es gratis; la **salida** (egress) a Internet y entre regiones se paga | Descargas masivas de un sitio; replicación entre regiones |
| **Tipo de suscripción y opciones de compra** | Descuentos por compromiso o licencias propias | Enterprise Agreement, reservas, Spot, Hybrid Benefit |

Otros elementos con impacto:
- **Tamaño/nivel** (SKU): Premium cuesta más que Estándar; un tamaño de VM mayor cuesta más.
- **Azure Marketplace**: productos de terceros con licencia propia sumada al coste de infraestructura.
- **Servicios asociados**: una VM también paga discos, IPs, ancho de banda y copias de seguridad.

### Formas de reducir el coste

| Mecanismo | Qué es | Ahorro típico |
|---|---|---|
| **Apagar y desasignar** | No pagar cómputo de VMs sin uso (los discos sí se pagan) | 100 % del cómputo |
| **Redimensionar** (right-sizing) | Elegir el tamaño justo; [[Azure Advisor]] lo recomienda | Variable |
| **Instancias reservadas** | Compromiso de 1 o 3 años para VMs, SQL, Cosmos… | Hasta ~72 % |
| **Planes de ahorro de cómputo** | Compromiso de gasto por hora, flexible entre servicios | Hasta ~65 % |
| **VMs Spot** | Capacidad sobrante que Azure puede retirar con aviso | Hasta ~90 %; solo para cargas interrumpibles |
| **Azure Hybrid Benefit** | Usar licencias Windows Server / SQL Server propias con Software Assurance | Hasta ~40 % en VMs Windows |
| **Niveles de acceso** | Cool/Cold/Archive para datos poco usados ([[Azure Archive Storage y niveles de acceso]]) | Gran ahorro en almacenamiento |
| **Autoescalado** | Tener solo las instancias necesarias en cada momento | Variable |
| **Región más barata** | Si la latencia y la residencia de datos lo permiten | Variable |

### Cuotas y límites
- Cada suscripción tiene **límites** (por ejemplo, vCPU por región). No son un coste, pero afectan a cuánto puedes desplegar. Se pueden solicitar aumentos.

## Casos de uso

- Un entorno de desarrollo se usa solo en horario laboral: **apagado automático** por la noche ahorra dos tercios del cómputo.
- Servidor de producción que estará tres años igual: **instancia reservada** de 3 años.
- Procesado de vídeo por lotes que puede reintentarse: **VMs Spot**.
- Empresa con licencias de Windows Server vigentes: **Azure Hybrid Benefit**.
- Sitio que sirve descargas a todo el mundo: revisar el **egress** y usar [[07 - Azure Content Delivery Network|CDN]].

## Comparaciones

| Concepto | Qué es | No confundir con |
|---|---|---|
| Reserva | Compromiso de tiempo con descuento; capacidad garantizada | Spot (barato pero sin garantía) |
| Spot | Capacidad sobrante que puede desaparecer | Reserva |
| Hybrid Benefit | Reutilizar licencias propias | Reserva (descuento por tiempo, no por licencia) |
| Egress | Tráfico de **salida**, se cobra | Ingress (entrada, gratis) |
| Desasignar VM | Deja de cobrar cómputo | Apagar desde el SO (sigue cobrando) |
| Cuota | Límite técnico de la suscripción | Presupuesto (límite de gasto que defines tú) |

## Conceptos que debo memorizar

> [!important]
> - Factores: **tipo de recurso, consumo, mantenimiento, geografía, tráfico de red, tipo de suscripción**.
> - **Ingress gratis, egress se paga.**
> - El precio **varía por región**.
> - **Reservas** = 1/3 años, hasta 72 %. **Spot** = interrumpible, hasta 90 %. **Hybrid Benefit** = licencias propias.
> - Una VM desasignada no paga cómputo, pero **sus discos sí**.

## Tips para AZ-900

> [!tip]
> - "Reducir coste de una VM que estará años encendida" → **instancia reservada**.
> - "Carga que puede interrumpirse" → **Spot**.
> - "Ya tenemos licencias de Windows Server" → **Azure Hybrid Benefit**.
> - "Datos que salen a Internet" → coste de **egress**.
> - "¿Cuesta lo mismo en todas las regiones?" → **No**.
> - Trampa: "Los datos que entran en Azure se cobran". **Falso**.
> - Trampa: "Una VM apagada no genera coste". Parcial: el **disco** sigue cobrando.

## Ejemplo de pregunta de examen

**Pregunta 1.** ¿Cuál de los siguientes factores afecta al coste de una máquina virtual en Azure?

- A) El número de usuarios que inician sesión en el portal
- B) La región en la que se implementa
- C) El nombre del grupo de recursos
- D) El número de etiquetas asignadas

**Respuesta: B.** El precio de los recursos varía según la región.
- A, C y D) No tienen impacto en la facturación.

**Pregunta 2.** Una empresa ejecutará una carga de trabajo estable en Azure durante los próximos tres años y quiere el mayor descuento posible con capacidad garantizada. ¿Qué debe adquirir?

- A) Máquinas virtuales Spot
- B) Instancias reservadas de tres años
- C) Una suscripción gratuita
- D) Azure Hybrid Benefit

**Respuesta: B.** Las reservas dan el mayor descuento con capacidad garantizada para cargas previsibles.
- A) Spot es más barato pero Azure puede retirarlo; no sirve para cargas estables.
- C) No existe para producción prolongada.
- D) Reduce el coste de licencias, pero no es un descuento por compromiso de tiempo; puede combinarse con la reserva.

## 🧠 Resumen para el examen

1. Seis factores: tipo de recurso, consumo, mantenimiento, geografía, tráfico de red y tipo de suscripción.
2. Ingress gratis; egress se paga. Precio distinto por región.
3. Ahorro: apagar/desasignar, redimensionar, reservas, planes de ahorro, Spot, Hybrid Benefit, niveles de acceso.
4. Reservas = estable y garantizado; Spot = barato e interrumpible; Hybrid Benefit = licencias propias.
5. Los recursos huérfanos (discos, IPs) siguen costando.

---
