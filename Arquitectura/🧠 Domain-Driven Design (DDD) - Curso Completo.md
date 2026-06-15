## 1. Introducción a DDD
Domain-Driven Design (Diseño Guiado por el Dominio) no es un framework ni una librería; es una **filosofía y un conjunto de prácticas de desarrollo de software** introducida por Eric Evans en 2003. 

Su premisa central es que para construir sistemas de software complejos y exitosos, el diseño del código debe estar íntimamente ligado y modelado de acuerdo con las reglas, procesos y estructuras del negocio (el Dominio).

---

## 2. El Espacio del Problema: Conceptos Clave
Antes de escribir código, es obligatorio entender el negocio mediante tres conceptos base:

* **El Dominio:** Es la esfera de actividad principal de la empresa u organización. Es el problema del mundo real que el software intenta resolver (ej. logística en Uber, streaming en Spotify, pasarela de pagos en Stripe).
* **Subdominios:** El dominio general se divide en partes más pequeñas para su análisis:
    * **Core Domain (Núcleo):** Lo que hace única a la empresa y genera la ventaja competitiva. Aquí es donde se debe invertir el mejor talento y esfuerzo.
    * **Supporting Domain (Soporte):** Problemas relacionados con el negocio pero que no son el núcleo diferencial (ej. un sistema de inventario interno).
    * **Generic Domain (Genérico):** Problemas comunes que cualquier empresa de cualquier rubro necesita resolver (ej. la autenticación de usuarios o la pasarela de facturación). Suelen resolverse comprando software de terceros.
* **Expertos del Dominio (Domain Experts):** Son las personas que entienden el negocio a la perfección (analistas, gerentes, usuarios finales). No son técnicos, pero saben exactamente cómo debe comportarse el sistema en el mundo real.

---

## 3. El Diseño Estratégico (La Foto Macro)
El diseño estratégico se enfoca en la arquitectura de alto nivel, la delimitación de responsabilidades y la comunicación entre equipos.

### A. Lenguaje Ubicuo (Ubiquitous Language)
Es un idioma común, riguroso y compartido de forma estricta entre los desarrolladores y los expertos del dominio. 
* **Regla:** Cero traducciones mentales. Si el negocio llama a un proceso *"Despacho de Orden"*, en el código la clase no puede llamarse `ShippingManager` o `LogisticsProcess`. Debe llamarse `DespachoOrden` u `OrderDispatch`.
* El lenguaje debe estar presente en las conversaciones, los diagramas, la documentación y, de forma idéntica, en los nombres de variables, clases y métodos del código fuente.

### B. Contextos Delimitados (Bounded Contexts)
Intentar crear un único modelo de datos unificado para toda una organización es el camino directo al fracaso. DDD soluciona esto dividiendo el dominio en fronteras lógicas e independientes llamadas **Bounded Contexts**.

Dentro de cada Bounded Context, el significado de una palabra es unívoco y absoluto:
* *Ejemplo clásico:* La palabra **Producto**.
    * En el contexto de **Ventas (Sales Context)**: Un producto se define por su precio, margen de ganancia, promociones y comisiones.
    * En el contexto de **Soporte (Support Context)**: Ese mismo producto físico o digital se define por sus manuales, versiones, tickets de fallo asociados y niveles de SLA.
* Intentar meter ambos enfoques en una única clase monolítica (`Product`) crea código acoplado, confuso e inmantenible. DDD separa el concepto en dos modelos distintos dentro de sus respectivos contextos.

### C. Mapas de Contexto (Context Mapping)
Los Contextos Delimitados no viven aislados; necesitan interactuar. El *Context Mapping* define la relación técnica y organizativa entre ellos:
* **Shared Kernel (Núcleo Compartido):** Dos contextos comparten un subconjunto de código o base de datos. Requiere alta coordinación.
* **Customer-Supplier / Upstream-Downstream:** El contexto "Proveedor" (Upstream) entrega datos al "Cliente" (Downstream). Los cambios del proveedor impactan al cliente.
* **Anticorruption Layer (ACL - Capa Anticorrupción):** Una capa traductora que implementa el contexto Downstream para evitar que el modelo caótico o externo de un sistema Upstream contamine su propio modelo limpio.
* **Open Host Service (OHS) / Published Language:** El contexto Upstream expone una API pública y estable (un lenguaje publicado, como JSON/REST) para que cualquiera lo consuma de forma estándar.

---

## 4. El Diseño Táctico (Los Bloques de Construcción en Código)
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
* **Características:** * Tienen un ID único (UUID, número autoincremental, DNI, etc.).
    * Son mutables (sus propiedades cambian, pero la identidad se mantiene).
    * Dos entidades son iguales únicamente si sus IDs son idénticos, sin importar si sus otros atributos coinciden o no.
* *Ejemplo:* Un `Usuario` con ID `usr_9832`. Aunque cambie su nombre, correo y contraseña, sigue siendo el mismo usuario en el sistema.

### B. **Objetos de Valor (Value Objects)**
Son objetos que no tienen una identidad propia. Se definen única y exclusivamente por el **valor de sus atributos**.
* **Características:**
    * Son **inmutables**. Si necesitas modificar un Objeto de Valor, debes destruir el objeto actual y crear uno nuevo por completo.
    * No tienen ID.
    * Dos Objetos de Valor son estructuralmente idénticos si todos sus atributos tienen los mismos valores.
    * Engloban lógica de validación interna.
* *Ejemplo:* Una `Direccion` (Calle, Altura, Código Postal) o una cantidad de `Dinero` (Monto, Divisa). Si tienes dos billetes de $100 USD, no te importa cuál es cuál; valen lo mismo por sus atributos.

### **C. Agregados (Aggregates)** y Raíces de Agregados (Aggregate Roots)
Un Agregado es un **conglomerado de Entidades y Objetos de Valor asociados** que se tratan como una única unidad de cara al cambio de datos.

* **Regla de Oro:** Cada agregado posee una sola **Entidad Raíz (Aggregate Root)**. 
* Cualquier código externo al agregado **solo puede interactuar y mantener referencias con la Raíz**. Nadie puede acceder o modificar los componentes internos de forma directa.
* Garantiza las **invariantes de negocio** (reglas de consistencia que siempre deben cumplirse de manera transaccional).
* *Ejemplo:* Un coche. Las ruedas, el motor y las puertas son objetos internos, pero tú interactúas con el vehículo a través del tablero y el volante (la Raíz del Agregado). No aceleras inyectando gasolina directamente al cilindro con la mano.
* *Ejemplo en código:* Una `OrdenDeCompra` (Raíz) y sus `LineasDeOrden` (Entidades internas). Si quieres añadir un producto, invocas `orden.agregarLinea(producto, cantidad)`. El sistema calcula internamente el precio total y valida el stock general en una sola transacción. No modificas las líneas directamente desde un controlador externo.

### D. Servicios del Dominio (Domain Services)
A veces, ciertas operaciones de negocio no pertenecen naturalmente a una Entidad ni a un Objeto de Valor específico, sino que involucran a múltiples agregados o conceptos abstractos.
* **Características:**
    * No tienen estado (Stateless).
    * Orquestan lógica puramente de negocio.
    * *Ejemplo:* Un `ProcesadorDeTransferencias` que interactúa con dos agregados `CuentaBancaria` para validar fondos, realizar el débito y el crédito correspondientes.

### E. Eventos del Dominio (Domain Events)
Es una notificación que indica que **algo importante ocurrió en el negocio en el pasado**.
* **Características:**
    * Se nombran en tiempo pasado (ej: `OrdenPagada`, `UsuarioRegistrado`, `EnvioDespachado`).
    * Permiten desacoplar componentes o contextos delimitados distintos. Cuando ocurre un evento, otros sistemas pueden reaccionar de forma asíncrona (Event-Driven Architecture).

### F. Repositorios (Repositories)
Son interfaces que encapsulan el mecanismo de persistencia (Base de datos) para guardar y recuperar **únicamente Raíces de Agregados**.
* Para el Dominio, el Repositorio emula ser una colección en memoria de Agregados. El código de negocio no sabe ni le interesa si por detrás hay un PostgreSQL, MongoDB o archivos de texto.

---

## 5. El Flujo de Trabajo en DDD

```
[ 1. Descubrir el Dominio ]
        │ (Event Storming + Expertos)
        ▼
[ 2. Trazar Contextos Delimitados ]
        │ (Fronteras y Ubiquitous Language)
        ▼
[ 3. Modelar el Diseño Táctico ]
        │ (Entidades, Objetos de Valor, Agregados)
        ▼
[ 4. Implementar Arquitectura Limpia ]
          (Aislar el Dominio de la Infraestructura)
```

---

## 6. ¿Cuándo Aplicar DDD? (Y cuándo NO)

### ✅ Cuándo aplicar:
1.  **Alta complejidad de negocio:** Las reglas son enredadas, cambian frecuentemente y el core de la empresa depende de ellas.
2.  **Sistemas a largo plazo:** Proyectos que van a crecer y evolucionar durante años con múltiples equipos de desarrollo.
3.  **Arquitecturas de Microservicios:** Mapear Bounded Contexts es la mejor técnica científica para definir los límites correctos de tus microservicios y evitar crear un "monolito distribuido".

### ❌ Cuándo NO aplicar (Anti-patrones):
1.  **Aplicaciones CRUD básicas:** Si tu sistema solo guarda, edita y lee registros sin lógica de negocio compleja (ej. un administrador de tareas simple), aplicar DDD es sobreingeniería extrema que ralentizará el desarrollo de forma innecesaria.
2.  **Proyectos de corta vida / MVP simples:** En fases de validación rápida de mercado, la velocidad prima sobre la arquitectura rígida de negocio.

---
#ddd #arquitectura #software-design #clean-architecture