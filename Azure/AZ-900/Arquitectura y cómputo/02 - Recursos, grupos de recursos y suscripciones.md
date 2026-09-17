---
tags: [az-900, azure, arquitectura, recursos, grupos-de-recursos, suscripciones]
modulo: Arquitectura y cómputo
peso_examen: Alto
---

# Recursos, grupos de recursos y suscripciones

## Concepto

Azure organiza todo lo que creas en una **jerarquía lógica** de cuatro niveles: **grupos de administración → suscripciones → grupos de recursos → recursos**. Esta nota cubre los tres niveles inferiores; los grupos de administración se explican en [[Gobernanza en Azure]].

**Problema que resuelve:** una organización puede tener miles de recursos. Sin una estructura sería imposible saber quién paga qué, quién puede tocar qué ni cómo borrar un proyecto entero sin dejar restos.

**Para qué se utiliza:** para agrupar recursos por ciclo de vida, separar facturación y límites, y aplicar permisos y políticas por herencia.

## Características principales

### Recurso (resource)
- Cualquier **elemento administrable** que creas en Azure: una VM, un disco, una IP pública, una base de datos, una cuenta de almacenamiento, una VNet.
- Tiene un tipo, una región, un identificador único y, normalmente, etiquetas.
- Se administra a través de **Azure Resource Manager (ARM)**, la capa de gestión que recibe todas las peticiones (portal, CLI, PowerShell, plantillas) y las ejecuta de forma consistente. Ver [[05 - Infraestructura como código - ARM y Bicep]].

### Grupo de recursos (resource group)
- **Contenedor lógico** de recursos que comparten ciclo de vida, permisos o proyecto.
- Reglas que el examen pregunta:
  - Todo recurso **debe** estar en un grupo de recursos.
  - Un recurso pertenece a **un solo** grupo a la vez.
  - Un grupo **no puede contener otros grupos** (no se anidan).
  - Los recursos de un grupo **pueden estar en regiones distintas**. El grupo tiene su propia región, que solo indica dónde se guardan sus metadatos.
  - Un recurso **se puede mover** a otro grupo o suscripción (con excepciones).
  - Al **eliminar el grupo se eliminan todos sus recursos**. Es la forma rápida de limpiar un entorno.
  - Los recursos de distintos grupos **pueden comunicarse** entre sí (el grupo no es una frontera de red).
- El RBAC, las políticas y los bloqueos aplicados al grupo **se heredan** por sus recursos.

### Suscripción (subscription)
- **Unidad de facturación y de límites**. Agrupa grupos de recursos y produce una factura.
- Está asociada a **una única cuenta de Microsoft Entra ID (tenant)**; un tenant puede tener **muchas** suscripciones.
- Define **límites y cuotas** (por ejemplo, número máximo de vCPU por región). Si necesitas más, pides ampliación o creas otra suscripción.
- Sirve para separar entornos (producción / desarrollo), departamentos o proyectos con facturación propia.
- Tipos habituales:

| Tipo | Para quién | Característica |
|---|---|---|
| **Gratuita** (Free) | Nuevos usuarios | Crédito inicial + servicios gratuitos 12 meses |
| **Pago por uso** (Pay-As-You-Go) | Individuos y empresas | Factura mensual por consumo |
| **Enterprise Agreement** | Grandes organizaciones | Compromiso anual con descuentos |
| **Student** | Estudiantes | Crédito sin tarjeta |
| **CSP** (Cloud Solution Provider) | Empresas que compran a través de un partner | Facturación a través del partner |

### La jerarquía completa

```
Tenant de Microsoft Entra ID
└── Grupo de administración raíz
    └── Grupos de administración (hasta 6 niveles)
        └── Suscripciones (facturación, límites)
            └── Grupos de recursos (ciclo de vida)
                └── Recursos (VM, disco, VNet…)
```

- Lo que aplicas arriba (**RBAC, Policy, locks**) se **hereda** hacia abajo.
- Cada nivel es un **ámbito** (scope) posible para asignar roles y políticas. Ver [[Azure RBAC]] y [[Azure Policy]].

## Casos de uso

- Proyecto de tres meses con web, base de datos y almacenamiento: todo en un grupo de recursos; al terminar, se borra el grupo.
- Departamentos de Marketing y Finanzas con presupuestos separados: una suscripción para cada uno.
- Entornos de producción y pruebas: suscripciones distintas, con políticas más estrictas en producción vía grupo de administración.
- Dar acceso a un contratista solo a los recursos de su proyecto: rol Colaborador en el grupo de recursos del proyecto.

## Comparaciones

| Concepto | Para qué sirve | No confundir con |
|---|---|---|
| Grupo de recursos | Agrupar por **ciclo de vida** y permisos | Suscripción (facturación) |
| Suscripción | **Facturación** y **límites** | Tenant (identidad) |
| Grupo de administración | Gobernar **varias suscripciones** | Grupo de recursos (dentro de una suscripción) |
| Tenant de Entra ID | **Identidades** (usuarios, grupos) | Suscripción (una suscripción confía en un tenant) |
| Región del grupo de recursos | Dónde viven los **metadatos** del grupo | Región de los recursos (pueden ser otras) |

## Conceptos que debo memorizar

> [!important]
> - Jerarquía: **Grupo de administración → Suscripción → Grupo de recursos → Recurso**. Herencia hacia abajo.
> - Un recurso está en **un solo** grupo. Los grupos **no se anidan**. Los recursos de un grupo **pueden estar en regiones distintas**.
> - **Borrar un grupo borra sus recursos.**
> - **Suscripción = facturación + límites.** Un tenant puede tener muchas suscripciones; cada suscripción confía en un solo tenant.
> - **Azure Resource Manager** es la capa de gestión que procesa toda petición, venga del portal, la CLI o una plantilla.

## Tips para AZ-900

> [!tip]
> - "Eliminar todos los recursos de un proyecto de una vez" → **borrar el grupo de recursos**.
> - "Facturas separadas por departamento" → **suscripciones** distintas.
> - "Aplicar lo mismo a varias suscripciones" → **grupo de administración** (ver [[Gobernanza en Azure]]).
> - "Superado el límite de vCPU" → límite de **suscripción**; solución: solicitar aumento o nueva suscripción.
> - Trampa: "Los recursos de un grupo deben estar en la misma región". **Falso**.
> - Trampa: "Un grupo de recursos puede contener otro grupo". **Falso**.
> - Trampa: "Un recurso puede pertenecer a dos grupos". **Falso**.
> - Trampa: "Una suscripción puede pertenecer a varios tenants". **Falso**; es al revés.

## Ejemplo de pregunta de examen

**Pregunta 1.** Tienes un grupo de recursos llamado `RG-Prueba` con una máquina virtual en *West Europe* y una cuenta de almacenamiento en *East US*. ¿Es válida esta configuración?

- A) No, todos los recursos de un grupo deben estar en la misma región.
- B) Sí, los recursos de un grupo pueden estar en regiones distintas.
- C) No, un grupo de recursos solo puede contener un tipo de recurso.
- D) Sí, pero solo si el grupo está en una tercera región.

**Respuesta: B.** La región del grupo solo determina dónde se guardan sus metadatos; sus recursos pueden estar donde convenga.
- A y C) Son restricciones que no existen.
- D) La región del grupo es irrelevante para las regiones de los recursos.

**Pregunta 2.** Tu empresa quiere recibir facturas separadas para el departamento de Ventas y el de Ingeniería. ¿Qué debes crear?

- A) Dos grupos de recursos
- B) Dos suscripciones
- C) Dos zonas de disponibilidad
- D) Dos etiquetas

**Respuesta: B.** La suscripción es la unidad de facturación.
- A) Los grupos de recursos organizan, pero comparten la factura de su suscripción.
- C) Las zonas son infraestructura física, no facturación.
- D) Las etiquetas permiten *desglosar* la factura, pero no generan facturas separadas.

## 🧠 Resumen para el examen

1. Jerarquía de cuatro niveles con herencia hacia abajo.
2. Recurso = cualquier elemento gestionable; vive en un solo grupo de recursos.
3. Grupo de recursos = contenedor lógico por ciclo de vida; no se anida; sus recursos pueden estar en varias regiones; borrarlo borra todo.
4. Suscripción = facturación y límites; muchas por tenant, un tenant por suscripción.
5. Azure Resource Manager procesa todas las operaciones sobre recursos.

---
