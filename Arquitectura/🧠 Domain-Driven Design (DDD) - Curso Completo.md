---
tags: [ddd, arquitectura, software-design, clean-architecture, microservicios]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [DDD, Domain-Driven Design]
---

# 🧠 Domain-Driven Design (DDD)

> [!info] Introducido por **Eric Evans** en *Domain-Driven Design: Tackling Complexity in the Heart of Software* (2003, el "libro azul"). **Vaughn Vernon** lo hizo más accesible en *Implementing Domain-Driven Design* (2013, el "libro rojo").

## 1. ¿Qué es?

Domain-Driven Design (Diseño Guiado por el Dominio) **no es un framework ni una librería**; es una **filosofía y un conjunto de prácticas de desarrollo de software**.

Su premisa central: para construir sistemas de software complejos y exitosos, el diseño del código debe estar íntimamente ligado y modelado de acuerdo con las **reglas, procesos y estructuras del negocio** (el Dominio).

DDD tiene dos mitades que conviene distinguir desde el principio:

| Mitad | Pregunta que responde | Herramientas |
|---|---|---|
| **Diseño estratégico** | ¿Cómo dividimos el problema y organizamos equipos y sistemas? | Lenguaje Ubicuo, Bounded Contexts, Context Mapping |
| **Diseño táctico** | ¿Cómo escribimos el código dentro de cada parte? | Entidades, Value Objects, Agregados, Servicios, Eventos, Repositorios, Factories |

> [!tip] El diseño estratégico es el más valioso
> Muchos equipos aplican solo los patrones tácticos ("tenemos Entities y Repositories, hacemos DDD"). Sin Bounded Contexts y Lenguaje Ubicuo, eso es solo programación orientada a objetos con nombres nuevos.

### Conceptos relacionados

- [[Clean Architecture]] → cómo **aislar** el modelo de dominio de la infraestructura. DDD dice qué va en el centro; Clean dice cómo protegerlo.
- [[🏗️ Diseño de Microservicios]] → los Bounded Contexts son la mejor guía para definir los límites de los microservicios.
- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → los comandos operan sobre Agregados; las consultas leen modelos aparte.
- [[Event Sourcing]] → persistir Agregados como secuencia de Domain Events.
- [[Saga Pattern]] → coordinar operaciones que cruzan varios Bounded Contexts.
- [[Principios SOLID]] → los Agregados y Value Objects bien diseñados cumplen SRP y encapsulación.
- [[ACID en Bases de Datos|ACID]] → un Agregado define el límite de una transacción.

---

## 2. El espacio del problema: conceptos clave

Antes de escribir código, es obligatorio entender el negocio mediante tres conceptos base:

- **El Dominio**: la esfera de actividad principal de la empresa u organización. Es el problema del mundo real que el software intenta resolver (ej. logística en Uber, streaming en Spotify, pasarela de pagos en Stripe).
- **Subdominios**: el dominio general se divide en partes más pequeñas para su análisis:
    - **Core Domain (Núcleo)**: lo que hace única a la empresa y genera la ventaja competitiva. Aquí es donde se debe invertir el mejor talento y esfuerzo.
    - **Supporting Domain (Soporte)**: problemas relacionados con el negocio pero que no son el núcleo diferencial (ej. un sistema de inventario interno). Se construye, pero sin sobreinvertir.
    - **Generic Domain (Genérico)**: problemas comunes que cualquier empresa de cualquier rubro necesita resolver (ej. autenticación de usuarios, facturación). Suelen resolverse **comprando** software de terceros o usando servicios gestionados.
- **Expertos del Dominio (Domain Experts)**: las personas que entienden el negocio a la perfección (analistas, gerentes, usuarios finales). No son técnicos, pero saben exactamente cómo debe comportarse el sistema en el mundo real.

> [!example] Ejemplo: una tienda online
> - **Core**: el motor de recomendaciones y el pricing dinámico (lo que la diferencia).
> - **Supporting**: gestión de catálogo, gestión de devoluciones.
> - **Generic**: login, envío de emails, pasarela de pago (Stripe), facturación electrónica.

---

## 3. El diseño estratégico (la foto macro)

El diseño estratégico se enfoca en la arquitectura de alto nivel, la delimitación de responsabilidades y la comunicación entre equipos.

### A. Lenguaje Ubicuo (Ubiquitous Language)

Es un idioma común, riguroso y compartido de forma estricta entre los desarrolladores y los expertos del dominio.

- **Regla**: cero traducciones mentales. Si el negocio llama a un proceso *"Despacho de Orden"*, en el código la clase no puede llamarse `ShippingManager` o `LogisticsProcess`. Debe llamarse `DespachoOrden` u `OrderDispatch`.
- El lenguaje debe estar presente en las conversaciones, los diagramas, la documentación y, de forma idéntica, en los nombres de variables, clases y métodos del código fuente.
- Se construye y refina **continuamente** con los expertos; cuando aparece una ambigüedad en una reunión, es una señal de que el modelo necesita ajustarse.
- Herramienta habitual para descubrirlo: **Event Storming** (Alberto Brandolini), un taller donde negocio y desarrollo modelan el flujo con *post-its* de eventos, comandos y actores.

### B. Contextos Delimitados (Bounded Contexts)

Intentar crear un único modelo de datos unificado para toda una organización es el camino directo al fracaso. DDD soluciona esto dividiendo el dominio en fronteras lógicas e independientes llamadas **Bounded Contexts**.

Dentro de cada Bounded Context, el significado de una palabra es unívoco y absoluto:

- *Ejemplo clásico*: la palabra **Producto**.
    - En el contexto de **Ventas (Sales Context)**: un producto se define por su precio, margen de ganancia, promociones y comisiones.
    - En el contexto de **Soporte (Support Context)**: ese mismo producto físico o digital se define por sus manuales, versiones, tickets de fallo asociados y niveles de SLA.
- Intentar meter ambos enfoques en una única clase monolítica (`Product`) crea código acoplado, confuso e inmantenible. DDD separa el concepto en dos modelos distintos dentro de sus respectivos contextos.

> [!important] Bounded Context ≠ Subdominio
> El **subdominio** pertenece al *espacio del problema* (cómo está dividido el negocio). El **Bounded Context** pertenece al *espacio de la solución* (cómo dividimos el software). Lo ideal es que coincidan uno a uno, pero no siempre ocurre (ej. un sistema heredado que abarca dos subdominios).

Cada Bounded Context suele tener: su propio modelo, su propio Lenguaje Ubicuo, su propia base de datos y, idealmente, **su propio equipo**. Es la unidad natural para un [[🏗️ Diseño de Microservicios|microservicio]] (o un módulo en un monolito modular).

### C. Mapas de Contexto (Context Mapping)

Los Contextos Delimitados no viven aislados; necesitan interactuar. El *Context Mapping* define la relación técnica y organizativa entre ellos:

| Patrón | Descripción | Cuándo |
|---|---|---|
| **Partnership** (Asociación) | Dos equipos coordinan su planificación y evolucionan juntos | Contextos que deben lanzarse en conjunto |
| **Shared Kernel** (Núcleo compartido) | Dos contextos comparten un subconjunto de código o modelo. Requiere alta coordinación | Pequeño y estable; es el que más se abusa |
| **Customer-Supplier** (Upstream-Downstream) | El contexto **Proveedor** (Upstream) entrega datos al **Cliente** (Downstream). El cliente influye en la planificación del proveedor | Relación cooperativa entre equipos |
| **Conformist** (Conformista) | El Downstream **acepta el modelo** del Upstream tal cual, sin capacidad de influir | Integrar con un proveedor grande o externo (ej. una API de un banco) |
| **Anticorruption Layer (ACL)** | Una capa traductora que implementa el Downstream para evitar que el modelo caótico o externo de un Upstream **contamine** su propio modelo limpio | Integrar sistemas heredados o de terceros |
| **Open Host Service (OHS)** | El Upstream expone una API pública y estable para que cualquiera lo consuma de forma estándar | Un contexto con muchos consumidores |
| **Published Language** (Lenguaje publicado) | Un formato de intercambio bien documentado (JSON Schema, Protobuf, un estándar del sector) que suele acompañar al OHS | Integraciones estandarizadas |
| **Separate Ways** (Caminos separados) | Los contextos **no se integran**; cada uno resuelve lo suyo | Cuando integrar cuesta más de lo que aporta |
| **Big Ball of Mud** | Reconocer que una parte del sistema es un desorden sin modelo y **aislarla** con un ACL | Sistemas heredados |

```
   [ Ventas ] ──OHS + Published Language──►  [ Facturación ]
        │
        └──Customer-Supplier──► [ Inventario ]
                                       │
                          ACL ◄────────┘  ← protege de [ ERP heredado (Big Ball of Mud) ]
```

---

## 4. El diseño táctico (los bloques de construcción en código)

Una vez mapeada la estrategia, el diseño táctico nos provee de patrones específicos de Programación Orientada a Objetos para modelar la lógica de negocio pura dentro de un Bounded Context.

```
+-------------------------------------------------------------+
|                        AGREGADO                             |
|                                                             |
|   [ Entidad Raíz (Aggregate Root) ] <--- Punto de acceso    |
|               |                                             |
|               v                                             |
|       [ Otra Entidad ] ------> [ Objeto de Valor ]          |
+-------------------------------------------------------------+
```

### A. Entidades (Entities)

Son objetos que poseen una **identidad única y continua** a lo largo del tiempo, independientemente de que sus atributos cambien.

- **Características**:
    - Tienen un ID único (UUID, número autoincremental, DNI, etc.).
    - Son mutables (sus propiedades cambian, pero la identidad se mantiene).
    - Dos entidades son iguales únicamente si sus IDs son idénticos, sin importar si sus otros atributos coinciden o no.
- *Ejemplo*: un `Usuario` con ID `usr_9832`. Aunque cambie su nombre, correo y contraseña, sigue siendo el mismo usuario en el sistema.

### B. Objetos de Valor (Value Objects)

Son objetos que no tienen una identidad propia. Se definen única y exclusivamente por el **valor de sus atributos**.

- **Características**:
    - Son **inmutables**. Si necesitas modificar un Objeto de Valor, creas uno nuevo.
    - No tienen ID.
    - Dos Objetos de Valor son iguales si todos sus atributos tienen los mismos valores (igualdad estructural).
    - Encapsulan **lógica de validación** interna: un `Email` inválido no puede existir.
- *Ejemplo*: una `Direccion` (Calle, Altura, Código Postal) o una cantidad de `Dinero` (Monto, Divisa). Si tienes dos billetes de $100 USD, no te importa cuál es cuál; valen lo mismo por sus atributos.

```csharp
// En C# los 'record' encajan perfecto: inmutables e igualdad por valor
public sealed record Dinero(decimal Monto, string Divisa)
{
    public Dinero(decimal monto, string divisa) : this(ValidarMonto(monto), ValidarDivisa(divisa)) { }

    public Dinero Sumar(Dinero otro)
    {
        if (otro.Divisa != Divisa) throw new DivisasDistintasException();
        return new Dinero(Monto + otro.Monto, Divisa);
    }

    private static decimal ValidarMonto(decimal m) => m >= 0 ? m : throw new ArgumentException("Monto negativo");
    private static string ValidarDivisa(string d) => d.Length == 3 ? d.ToUpperInvariant() : throw new ArgumentException("Divisa ISO inválida");
}
```

> [!tip] Usa Value Objects en vez de primitivos
> `string email`, `decimal precio`, `int cantidad` son fuentes de bugs (*primitive obsession*). `Email`, `Precio`, `Cantidad` como Value Objects validan una sola vez y hacen el código autoexplicativo.

### C. Agregados (Aggregates) y Raíces de Agregado (Aggregate Roots)

Un Agregado es un **conglomerado de Entidades y Objetos de Valor asociados** que se tratan como una única unidad de cara al cambio de datos.

- **Regla de oro**: cada agregado posee una sola **Entidad Raíz (Aggregate Root)**.
- Cualquier código externo al agregado **solo puede interactuar y mantener referencias con la Raíz**. Nadie puede acceder o modificar los componentes internos de forma directa.
- Garantiza las **invariantes de negocio** (reglas de consistencia que siempre deben cumplirse de manera transaccional).
- **El Agregado es el límite de la transacción**: una transacción modifica **un solo Agregado**. Si una operación necesita tocar dos Agregados, se hace en dos transacciones coordinadas por Domain Events (consistencia eventual) o, entre servicios, con una [[Saga Pattern|Saga]].
- Entre Agregados se referencia **por ID**, no por objeto, para mantenerlos pequeños y desacoplados.
- *Ejemplo*: un coche. Las ruedas, el motor y las puertas son objetos internos, pero tú interactúas con el vehículo a través del tablero y el volante (la Raíz). No aceleras inyectando gasolina directamente al cilindro con la mano.
- *Ejemplo en código*: una `OrdenDeCompra` (Raíz) y sus `LineasDeOrden` (Entidades internas). Si quieres añadir un producto, invocas `orden.AgregarLinea(productoId, cantidad, precio)`. El Agregado recalcula el total y valida las reglas en una sola operación. No modificas las líneas directamente desde un controlador externo.

```csharp
public class OrdenDeCompra : AggregateRoot
{
    private readonly List<LineaDeOrden> _lineas = new();
    public IReadOnlyCollection<LineaDeOrden> Lineas => _lineas.AsReadOnly(); // sin setter público
    public EstadoOrden Estado { get; private set; }
    public Dinero Total { get; private set; }

    public void AgregarLinea(ProductoId productoId, int cantidad, Dinero precioUnitario)
    {
        if (Estado != EstadoOrden.Borrador)
            throw new OrdenNoModificableException(Id);          // invariante

        _lineas.Add(new LineaDeOrden(productoId, cantidad, precioUnitario));
        Total = _lineas.Aggregate(Dinero.Cero("USD"), (acc, l) => acc.Sumar(l.Subtotal));

        RegistrarEvento(new LineaAgregada(Id, productoId, cantidad)); // Domain Event
    }
}
```

> [!warning] Agregados grandes = problemas
> Un Agregado `Cliente` que contiene todos sus `Pedidos` obliga a cargar miles de pedidos para cambiar un teléfono, y genera conflictos de concurrencia constantes. Diseña Agregados **pequeños**: `Cliente` y `Pedido` son Agregados separados que se referencian por `ClienteId`.

### D. Servicios del Dominio (Domain Services)

A veces, ciertas operaciones de negocio no pertenecen naturalmente a una Entidad ni a un Objeto de Valor específico, sino que involucran a múltiples agregados o conceptos abstractos.

- **Características**:
    - No tienen estado (*stateless*).
    - Contienen lógica **puramente de negocio**, expresada en el Lenguaje Ubicuo.
    - *Ejemplo*: un `ServicioDeTransferencias` que interactúa con dos agregados `CuentaBancaria` para validar fondos y realizar el débito y el crédito.

> [!important] Domain Service ≠ Application Service
> - **Domain Service**: lógica de negocio que no encaja en un Agregado. Vive en el Dominio. Ej.: `CalculadoraDeComisiones`.
> - **Application Service** (caso de uso): **orquesta**: recibe un comando, carga el Agregado del repositorio, invoca sus métodos, guarda y publica eventos. **No contiene reglas de negocio.** Vive en la capa de Aplicación de [[Clean Architecture]]. Ej.: `CrearOrdenHandler`.
> Confundirlos lleva a servicios de aplicación llenos de `if` de negocio y Agregados vacíos.

### E. Eventos del Dominio (Domain Events)

Es una notificación que indica que **algo importante ocurrió en el negocio en el pasado**.

- **Características**:
    - Se nombran en **tiempo pasado** (ej. `OrdenPagada`, `UsuarioRegistrado`, `EnvioDespachado`).
    - Son **inmutables** y llevan los datos necesarios para que quien los reciba pueda reaccionar (IDs, fecha, importe).
    - Los emite el Agregado; la infraestructura los publica **después de confirmar la transacción** (ver [[Transactional Outbox]]).
    - Permiten desacoplar componentes o contextos delimitados distintos. Cuando ocurre un evento, otros sistemas pueden reaccionar de forma asíncrona (*Event-Driven Architecture*).
    - Son la base de [[Event Sourcing]].

### F. Repositorios (Repositories)

Son interfaces que encapsulan el mecanismo de persistencia (base de datos) para guardar y recuperar **únicamente Raíces de Agregado**.

- Para el Dominio, el Repositorio emula ser una **colección en memoria** de Agregados. El código de negocio no sabe ni le interesa si por detrás hay PostgreSQL, MongoDB o archivos de texto.
- La **interfaz** se define en el Dominio; la **implementación** (EF Core, Dapper) vive en Infraestructura ([[Clean Architecture]]).
- Un Repositorio por Agregado: `IOrdenRepository`, no `ILineaDeOrdenRepository`.

### G. Fábricas (Factories)

Cuando crear un Agregado o Value Object es complejo (muchas validaciones, varios pasos, generación de IDs), se encapsula en una **Factory**: un método estático (`Orden.Crear(...)`) o una clase dedicada. Garantizan que el objeto **nace válido**; un constructor público con setters no lo garantiza.

### H. Módulos (Modules)

Agrupaciones de conceptos relacionados dentro de un Bounded Context (namespaces / carpetas), nombradas con el Lenguaje Ubicuo. Son la herramienta para que un contexto grande siga siendo navegable.

---

## 5. El flujo de trabajo en DDD

```
[ 1. Descubrir el Dominio ]
        │ (Event Storming + Expertos del Dominio)
        ▼
[ 2. Trazar Contextos Delimitados ]
        │ (Fronteras, Lenguaje Ubicuo, Context Map)
        ▼
[ 3. Modelar el Diseño Táctico ]
        │ (Agregados, Entidades, Value Objects, Eventos)
        ▼
[ 4. Implementar con Clean Architecture ]
          (Aislar el Dominio de la Infraestructura)
```

---

## 6. ¿Cuándo aplicar DDD? (y cuándo NO)

### ✅ Cuándo aplicar

1. **Alta complejidad de negocio**: las reglas son enredadas, cambian frecuentemente y el core de la empresa depende de ellas.
2. **Sistemas a largo plazo**: proyectos que van a crecer y evolucionar durante años con múltiples equipos de desarrollo.
3. **Arquitecturas de microservicios**: mapear Bounded Contexts es la mejor técnica para definir los límites correctos de tus microservicios y evitar crear un "monolito distribuido".
4. **Acceso real a expertos del dominio**: sin ellos, no hay Lenguaje Ubicuo que construir.

### ❌ Cuándo NO aplicar

1. **Aplicaciones CRUD básicas**: si tu sistema solo guarda, edita y lee registros sin lógica de negocio compleja (ej. un administrador de tareas simple), aplicar DDD es sobreingeniería que ralentizará el desarrollo.
2. **Proyectos de corta vida / MVP simples**: en fases de validación rápida de mercado, la velocidad prima sobre la arquitectura rígida de negocio.
3. **Dominios genéricos**: para autenticación o facturación estándar, compra o usa un servicio; no modeles.

> [!tip] DDD no es todo o nada
> Puedes aplicar el diseño estratégico (Bounded Contexts, Lenguaje Ubicuo) en todo el sistema y reservar el diseño táctico completo solo para el **Core Domain**. Los subdominios de soporte pueden ser CRUD sin culpa.

---

## 7. Anti-patrones frecuentes

| Anti-patrón | Síntoma | Solución |
|---|---|---|
| **Modelo de dominio anémico** | Entidades que son solo getters/setters; toda la lógica en "servicios" | Mover el comportamiento a los Agregados |
| **Agregado gigante** | Un `Cliente` con todos sus pedidos, facturas y tickets | Dividir en Agregados pequeños referenciados por ID |
| **Obsesión por primitivos** | `string email`, `decimal precio` por todas partes | Value Objects |
| **Repositorio genérico para todo** | `IRepository<T>` usado sobre entidades internas | Un repositorio por Raíz de Agregado |
| **Un solo modelo para toda la empresa** | La clase `Product` con 80 propiedades | Bounded Contexts |
| **DDD sin expertos** | Los desarrolladores inventan el lenguaje | Event Storming con negocio |
| **Táctico sin estratégico** | "Hacemos DDD porque tenemos Entities" | Empezar por Bounded Contexts y Lenguaje Ubicuo |

---

## 8. Puntos clave

- DDD = **modelar el software según el negocio**, con un lenguaje compartido.
- **Estratégico** (Bounded Contexts, Context Map, Lenguaje Ubicuo) antes que **táctico**.
- **Entidad** = identidad; **Value Object** = valor e inmutable; **Agregado** = unidad de consistencia y de transacción.
- Los **Repositorios** solo devuelven Raíces de Agregado.
- Los **Domain Events** se nombran en pasado y desacoplan contextos.
- **Application Service orquesta; Domain Service y Agregado deciden.**
- Aplícalo donde hay **complejidad de negocio real**; en un CRUD es sobreingeniería.

## 9. Preguntas de repaso

1. ¿Cuál es la diferencia entre un subdominio y un Bounded Context?
2. ¿Por qué dos Value Objects con los mismos atributos son "el mismo" y dos Entidades no?
3. ¿Qué invariante protege el Agregado `OrdenDeCompra` del ejemplo?
4. ¿Cuándo usarías un Anticorruption Layer?
5. ¿Qué diferencia hay entre un Domain Service y un Application Service?
6. ¿Por qué un Agregado debería ser el límite de una transacción?
7. ¿Qué relación hay entre Bounded Contexts y microservicios?

## 🎯 Para entrevistas y exámenes

- Saber explicar **Entidad vs Value Object** con un ejemplo (Usuario vs Dinero).
- Saber explicar **Agregado y Raíz de Agregado**, y por qué el Agregado es el límite transaccional.
- Nombrar al menos 4 patrones de **Context Mapping** (Shared Kernel, Customer-Supplier, Conformist, ACL, OHS).
- Explicar el **modelo anémico** y por qué es un anti-patrón.
- Relación DDD ↔ Microservicios ↔ Clean Architecture.

## Referencias

- Eric Evans, *Domain-Driven Design: Tackling Complexity in the Heart of Software* (Addison-Wesley, 2003)
- Vaughn Vernon, *Implementing Domain-Driven Design* (Addison-Wesley, 2013) y *Domain-Driven Design Distilled* (2016, versión corta)
- Eric Evans, *DDD Reference* (resumen gratuito de los patrones): https://www.domainlanguage.com/ddd/reference/
- Alberto Brandolini, *Introducing EventStorming*: https://www.eventstorming.com/
- Microsoft Learn, *Using tactical DDD to design microservices*: https://learn.microsoft.com/azure/architecture/microservices/model/tactical-ddd
- Martin Fowler, *Bounded Context*: https://martinfowler.com/bliki/BoundedContext.html

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
