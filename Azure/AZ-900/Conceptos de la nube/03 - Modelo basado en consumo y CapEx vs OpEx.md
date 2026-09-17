---
tags: [az-900, azure, conceptos-nube, costes, capex-opex]
modulo: Conceptos de la nube
peso_examen: Alto
---

# Modelo basado en consumo y CapEx vs OpEx

## Concepto

El **modelo basado en consumo** (consumption-based model) significa que **pagas solo por los recursos que usas, mientras los usas**. No hay compra anticipada de capacidad, no pagas por lo que está apagado y puedes dejar de pagar en cuanto eliminas el recurso.

**Problema que resuelve:** en un centro de datos propio tienes que comprar servidores para el pico máximo previsto, y la mayor parte del tiempo están infrautilizados. Si te quedas corto, el proyecto se retrasa meses. El consumo elimina ambas ineficiencias.

**Para qué se utiliza:** para convertir un gasto de inversión grande e incierto en un gasto operativo pequeño, predecible y ajustable.

## Características principales

### Dos formas de gastar

| | **CapEx** (Capital Expenditure, gasto de capital) | **OpEx** (Operational Expenditure, gasto operativo) |
|---|---|---|
| Qué es | Inversión **inicial** en activos físicos | Gasto **recurrente** por servicios o consumo |
| Ejemplo | Comprar servidores, construir un CPD | Factura mensual de Azure, alquiler, electricidad |
| Cuándo se paga | Por adelantado, de golpe | A medida que se usa |
| Contabilidad | Se **amortiza** durante años | Se deduce en el mismo periodo |
| Flexibilidad | Baja: el hardware ya está comprado | Alta: subes o bajas cada mes |
| Riesgo | Alto: puedes comprar de más o de menos | Bajo: ajustas a la demanda real |
| Modelo | Típico de **on-premises** | Típico de **la nube** |

La nube pública transforma CapEx en OpEx: **no hay coste inicial**, solo la factura por uso.

### Cómo se mide el consumo en Azure

- **Cómputo**: por segundo o por hora de máquina virtual encendida; por ejecución y memoria en Functions.
- **Almacenamiento**: por GB al mes, más operaciones de lectura/escritura.
- **Red**: los datos que **entran** (ingress) suelen ser gratis; los que **salen** a Internet (egress) se cobran.
- **Servicios PaaS**: por nivel (tier) contratado o por unidades de rendimiento.

### Ventajas del modelo de consumo

- **Sin coste inicial** ni necesidad de adivinar la capacidad.
- **Sin pagar por infraestructura infrautilizada**: se apaga y deja de costar.
- **Escalado a demanda**: pagas más solo cuando usas más.
- **Presupuesto más fino**: se puede seguir y limitar con [[03 - Microsoft Cost Management y etiquetas]].

### Comparación de modelos de precios de la nube

| Modelo de precio | Cómo funciona | Cuándo compensa |
|---|---|---|
| **Pago por uso** (pay-as-you-go) | Precio de lista, sin compromiso | Cargas variables o de prueba |
| **Instancias reservadas** (1 o 3 años) | Compromiso a cambio de hasta ~72 % de descuento | Cargas estables y previsibles |
| **Planes de ahorro** (savings plans) | Compromiso de gasto por hora, aplicable a varios servicios | Cómputo estable pero cambiante en tipo |
| **Spot** | Capacidad sobrante muy barata que Azure puede retirar | Trabajos interrumpibles (batch, pruebas) |
| **Azure Hybrid Benefit** | Reutilizar licencias Windows/SQL propias | Migraciones desde local con licencias vigentes |

## Casos de uso

- Un equipo de desarrollo crea entornos de prueba por la mañana y los elimina por la tarde: solo paga las horas de uso.
- Una web de resultados electorales necesita cien servidores una noche al año: con consumo paga una noche, no un año.
- Una empresa con un servidor de producción estable durante tres años compra una **instancia reservada** para ahorrar.

## Comparaciones

| Concepto | Descripción | No confundir con |
|---|---|---|
| Modelo de consumo | Pagas por lo que usas | Suscripción de tarifa plana |
| CapEx | Compra inicial de activos | OpEx (gasto recurrente) |
| OpEx | Gasto recurrente por uso | CapEx (inversión) |
| Instancia reservada | Descuento por compromiso de tiempo | Spot (descuento por capacidad interrumpible) |

## Conceptos que debo memorizar

> [!important]
> - **Consumo = pagas por lo que usas, cuando lo usas, sin coste inicial.**
> - **CapEx** = inversión inicial en hardware (on-premises). **OpEx** = gasto operativo recurrente (nube).
> - La nube convierte **CapEx en OpEx**.
> - Ingress (entrada) gratis; egress (salida a Internet) se paga.
> - Reservadas = compromiso 1/3 años con descuento; Spot = barato pero interrumpible.

## Tips para AZ-900

> [!tip]
> - Palabras clave para consumo: *"pagar solo por lo que se usa"*, *"sin coste inicial"*, *"apagar y dejar de pagar"*, *"reducir infraestructura infrautilizada"*.
> - Palabras clave para CapEx: *"comprar"*, *"inversión inicial"*, *"amortizar"*, *"hardware propio"*.
> - Palabras clave para OpEx: *"factura mensual"*, *"alquiler"*, *"suscripción"*, *"gasto operativo"*.
> - Trampa: "En la nube no hay OpEx". **Falso**. La nube es casi todo OpEx.
> - Trampa: "El modelo de consumo garantiza el coste más bajo siempre". **Falso**. Para cargas estables, las reservadas son más baratas que el pago por uso.

## Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa quiere evitar la compra de servidores y pagar únicamente por la capacidad que consuma cada mes. ¿Qué ventaja de la nube describe?

- A) Alta disponibilidad
- B) Modelo basado en consumo
- C) Escalabilidad vertical
- D) Gobernanza

**Respuesta: B.** Pagar solo por lo consumido, sin inversión inicial, es la definición del modelo de consumo.
- A) Se refiere a que el servicio esté disponible, no al coste.
- C) Es aumentar la capacidad de un recurso, no una forma de pagar.
- D) Son las reglas de control sobre los recursos.

**Pregunta 2.** ¿Cuál de las siguientes afirmaciones sobre CapEx y OpEx es correcta?

- A) Migrar a la nube aumenta el CapEx.
- B) Los pagos mensuales por máquinas virtuales de Azure son CapEx.
- C) Comprar un servidor para el centro de datos propio es CapEx.
- D) OpEx implica una gran inversión inicial.

**Respuesta: C.** Comprar un activo físico es gasto de capital.
- A) La nube reduce el CapEx y lo convierte en OpEx.
- B) Los pagos por uso son OpEx.
- D) Es justo lo contrario: OpEx no requiere inversión inicial.

## 🧠 Resumen para el examen

1. Consumo = pagas lo que usas, sin coste inicial, dejas de pagar al apagar.
2. CapEx = comprar hardware (local). OpEx = pagar por uso (nube). La nube convierte CapEx en OpEx.
3. Egress (salida a Internet) se cobra; ingress no.
4. Opciones de precio: pago por uso, reservadas (1/3 años), planes de ahorro, Spot, Hybrid Benefit.
5. Reservadas para cargas estables; Spot para trabajos interrumpibles.

---
