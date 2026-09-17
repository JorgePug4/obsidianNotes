---
tags: [az-900, azure, conceptos-nube, repaso]
modulo: Conceptos de la nube
---

# 🎯 Repaso final de Conceptos de la nube

Nota de consolidación del dominio 1 (25–30 %). Úsala el día antes del examen.

## Los conceptos más importantes

1. **Cloud computing** = servicios de computación entregados por Internet, bajo demanda, con pago por uso.
2. **Responsabilidad compartida**: Microsoft siempre cubre lo físico; el cliente siempre cubre datos, dispositivos e identidades; el resto depende del modelo.
3. **Pública** (proveedor), **privada** (propia, control total), **híbrida** (ambas conectadas), **multinube** (varios proveedores).
4. **Consumo** = pagar por lo que usas. La nube convierte **CapEx** (inversión) en **OpEx** (gasto operativo).
5. **HA** se mide con SLA; más nueves, menos caída.
6. **Vertical** = más potencia al mismo recurso; **horizontal** = más instancias.
7. **Elasticidad** = escalado automático en ambos sentidos. **Agilidad** = desplegar rápido.
8. **Confiabilidad** = recuperarse de fallos. **Previsibilidad** = rendimiento y coste anticipables.
9. **IaaS** (VMs, tú gestionas el SO) → **PaaS** (código y datos) → **SaaS** (solo usas la app).
10. **Serverless** = PaaS sin servidores visibles, escala a cero, pago por ejecución.

## Tabla de conceptos y su propósito

| Concepto | En una frase | Palabra clave de examen |
|---|---|---|
| Computación en la nube | Servicios de cómputo por Internet bajo demanda | *on-demand, pay-as-you-go* |
| Responsabilidad compartida | Reparto de tareas entre Microsoft y cliente según el modelo | *who is responsible for* |
| Nube pública | Infraestructura del proveedor compartida entre clientes | *no upfront cost, no hardware* |
| Nube privada | Infraestructura de uso exclusivo | *full control, own datacenter* |
| Nube híbrida | Local + pública conectadas | *keep on-premises and extend* |
| Multinube | Varios proveedores públicos | *Azure and AWS* |
| Modelo de consumo | Pagas por uso, sin coste inicial | *only pay for what you use* |
| CapEx | Inversión inicial en activos | *purchase, upfront* |
| OpEx | Gasto recurrente por servicios | *monthly, subscription* |
| Alta disponibilidad | Máximo tiempo operativo, garantizado por SLA | *uptime, 99.9 %* |
| Escalabilidad | Ajustar recursos a la demanda | *scale up / scale out* |
| Elasticidad | Escalado automático arriba y abajo | *automatically adds and removes* |
| Agilidad | Desplegar y cambiar rápido | *quickly deploy* |
| Confiabilidad | Recuperarse de fallos | *recover from failure* |
| Previsibilidad | Anticipar rendimiento y coste | *forecast, predictable* |
| Gobernanza | Estándares y cumplimiento aplicados | *compliance, standards* |
| Capacidad de administración | Gestionar la nube y en la nube | *manage via portal/CLI, auto-scale* |
| IaaS | Infraestructura alquilada; tú gestionas el SO | *virtual machine, operating system* |
| PaaS | Plataforma para tu código y datos | *developers, managed database* |
| SaaS | Aplicación lista para usar | *Microsoft 365, per-user* |
| Serverless | Código por eventos sin administrar servidores | *Functions, event-driven, scale to zero* |

## Las diferencias que más fácilmente puedo confundir

> [!warning] Pares de confusión clásicos
> | Confusión | Cómo distinguirlos |
> |---|---|
> | **Escalabilidad vs Elasticidad** | Escalabilidad = poder crecer. Elasticidad = crecer y encoger **automáticamente**. |
> | **Vertical vs Horizontal** | Vertical = misma máquina más grande. Horizontal = más máquinas. |
> | **HA vs Confiabilidad** | HA = tiempo activo (SLA). Confiabilidad = recuperarse de fallos (redundancia). |
> | **Agilidad vs Elasticidad** | Agilidad = velocidad de despliegue. Elasticidad = ajuste de capacidad. |
> | **Híbrida vs Multinube** | Híbrida = local + pública. Multinube = varias públicas. |
> | **Privada vs Pública con Private Endpoint** | Privada = infraestructura exclusiva. Private Endpoint sigue siendo nube pública. |
> | **CapEx vs OpEx** | CapEx = comprar. OpEx = pagar por uso. |
> | **IaaS vs PaaS** | IaaS = tú instalas el SO. PaaS = solo despliegas código. |
> | **PaaS vs SaaS** | PaaS = aún desarrollas. SaaS = solo usas. |
> | **PaaS vs Serverless** | PaaS clásico cobra por plan aunque esté inactivo. Serverless cobra por ejecución y escala a cero. |
> | **Reservadas vs Spot** | Reservadas = compromiso 1/3 años con descuento. Spot = barato pero Azure puede retirarlo. |
> | **Seguridad vs Gobernanza** | Seguridad = proteger de amenazas. Gobernanza = cumplir reglas de la organización. |

## 10 tips de examen

> [!tip]
> 1. "Solo pago por lo que uso" → **modelo de consumo**. "Sin inversión inicial" → **OpEx / nube pública**.
> 2. Cualquier pregunta de "¿quién es responsable de…?" se responde con la tabla de **responsabilidad compartida**: físico = Microsoft; datos e identidades = cliente; SO = cliente solo en IaaS.
> 3. "Automáticamente" + "según la demanda" → **elasticidad**.
> 4. "Más servidores" → **horizontal**. "Servidor más grande" → **vertical**.
> 5. "Mantener el datacenter y extender a Azure" → **híbrida**. "Azure y AWS" → **multinube**.
> 6. "Control total del SO" o "instalar software" → **IaaS**. "Sin gestionar parches" → **PaaS**. "Suscripción por usuario" → **SaaS**.
> 7. "Se ejecuta cuando ocurre un evento y pago por ejecución" → **serverless (Functions)**.
> 8. SLA: 99,9 % ≈ 8,8 h/año; 99,99 % ≈ 53 min/año. Combinar servicios **baja** el SLA total.
> 9. "Recuperarse de un fallo de región" → **confiabilidad**. "Estar disponible el 99,99 %" → **alta disponibilidad**.
> 10. Los datos son **siempre** responsabilidad del cliente, incluso en SaaS. Nunca elijas una opción que se lo atribuya a Microsoft.

## 10 preguntas de repaso tipo AZ-900

**1.** ¿Qué característica de la nube permite que una empresa pague únicamente por los recursos que consume?
- A) Alta disponibilidad · B) Modelo basado en consumo ✅ · C) Escalabilidad · D) Agilidad

**2.** Una organización quiere reducir su gasto de capital (CapEx) migrando a Azure. ¿Qué tipo de gasto aumentará?
- A) Gasto de capital · B) Gasto operativo (OpEx) ✅ · C) Ninguno · D) Gasto de amortización

**3.** ¿Qué modelo de nube describe una empresa que ejecuta su ERP en su propio centro de datos y su sitio web en Azure, conectados por VPN?
- A) Pública · B) Privada · C) Híbrida ✅ · D) Multinube

**4.** Una aplicación aumenta automáticamente el número de instancias en horas punta y las reduce por la noche. ¿Qué beneficio describe?
- A) Agilidad · B) Elasticidad ✅ · C) Confiabilidad · D) Previsibilidad de coste

**5.** ¿Cuál de las siguientes afirmaciones sobre la responsabilidad compartida es correcta?
- A) En SaaS, Microsoft es responsable de los datos del cliente
- B) En IaaS, Microsoft es responsable del sistema operativo
- C) El cliente es siempre responsable de sus datos e identidades ✅
- D) La responsabilidad compartida no aplica a PaaS

**6.** ¿Qué tipo de servicio es Azure SQL Database?
- A) IaaS · B) PaaS ✅ · C) SaaS · D) Nube privada

**7.** Una empresa necesita ejecutar código solo cuando llega un archivo nuevo, sin administrar servidores y pagando por ejecución. ¿Qué modelo describe esta necesidad?
- A) IaaS · B) SaaS · C) Serverless ✅ · D) Nube privada

**8.** ¿Qué significa un SLA del 99,9 %?
- A) Aproximadamente 9 minutos de inactividad al año
- B) Aproximadamente 8,8 horas de inactividad al año ✅
- C) Aproximadamente 3,6 días de inactividad al año
- D) Ninguna inactividad garantizada

**9.** ¿Cuál es una ventaja de la nube privada frente a la pública?
- A) Menor coste inicial · B) Mayor control y exclusividad de la infraestructura ✅ · C) Escalado más rápido · D) Modelo de consumo

**10.** Un administrador duplica la memoria RAM de una máquina virtual. ¿Qué tipo de escalado ha aplicado?
- A) Horizontal · B) Vertical ✅ · C) Elástico · D) Geográfico

## 🧠 Última pasada antes del examen

> [!important] Lo que no puede fallarte
> - Físico = Microsoft · Datos e identidades = cliente · SO = cliente solo en IaaS.
> - Pública / privada / híbrida / multinube. Híbrida = local + pública.
> - Consumo = pago por uso · CapEx → OpEx.
> - Vertical = más grande · Horizontal = más instancias · Elasticidad = automático.
> - IaaS = VM · PaaS = App Service, SQL DB · SaaS = Microsoft 365 · Serverless = Functions.

Volver al índice: [[00 - Índice - Conceptos de la nube]]
