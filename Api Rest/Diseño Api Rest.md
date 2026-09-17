---
tags: [api-rest, http, backend, diseño, dotnet]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [REST, API REST, Diseño de APIs REST]
---

# Diseño de APIs REST

> [!info] REST fue definido por **Roy Fielding** en su tesis doctoral (2000). No es un estándar ni un protocolo: es un **estilo arquitectónico** con un conjunto de restricciones. Las APIs que las cumplen se llaman **RESTful**.

## 1. ¿Qué es una API y qué significa REST?

- **API (Application Programming Interface)**: piensa en ella como el **mesero de un restaurante**. Tú (el cliente) miras el menú y pides un platillo. El mesero lleva tu orden a la cocina (el servidor), el chef prepara la comida y el mesero te la trae. No sabes cómo cocinó el chef, solo que interactuando con el mesero obtienes tu comida. Una API hace exactamente eso entre dos sistemas de software.

- **REST (Representational State Transfer)**: no es código ni un lenguaje; es un **estilo de arquitectura**, un conjunto de "reglas de buena conducta" para que crear APIs sea estándar, limpio y fácil de entender. El nombre significa que el cliente y el servidor intercambian **representaciones** (normalmente JSON) del **estado** de los **recursos**.

### Las 6 restricciones de REST (Fielding)

| Restricción | Significado | En la práctica |
|---|---|---|
| **Cliente-servidor** | Separación de responsabilidades | El frontend y el backend evolucionan por separado |
| **Sin estado** (*stateless*) | Cada petición lleva **toda** la información necesaria; el servidor no guarda sesión entre peticiones | El token JWT viaja en cada petición; permite escalar horizontalmente |
| **Cacheable** | Las respuestas indican si pueden cachearse | Cabeceras `Cache-Control`, `ETag`; respuestas `304` |
| **Interfaz uniforme** | Mismas convenciones para todos los recursos | URLs de recursos + métodos HTTP + códigos de estado + representaciones |
| **Sistema en capas** | Puede haber intermediarios (proxies, [[API Gateway]], CDN) sin que el cliente lo sepa | Balanceadores, cachés, gateways |
| **Código bajo demanda** (opcional) | El servidor puede enviar código ejecutable | Casi nunca se usa en APIs |

## ¿Para qué sirve REST?

- Es el estilo **dominante** para APIs web públicas y para exponer [[🏗️ Diseño de Microservicios|microservicios]] a frontends.
- Aprovecha toda la infraestructura de HTTP: cachés, proxies, CDN, herramientas de depuración, navegadores.
- Es **fácil de consumir** desde cualquier lenguaje: solo hace falta un cliente HTTP.

## Conceptos relacionados

- [[Status Code|Códigos de estado HTTP]] → detalle de cada código; uno de los 4 pilares.
- [[Idempotencia]] → qué métodos son idempotentes y cómo hacer seguro un `POST`.
- [[API Gateway]] → cómo se exponen y protegen las APIs REST en microservicios.
- [[🏗️ Diseño de Microservicios]] → REST vs gRPC vs mensajería.
- [[Autenticación vs. Autorización]] → base para entender `401`/`403` y OAuth 2.0.
- [[Clean Architecture]] → los controladores son adaptadores; no contienen lógica de negocio.
- [[CQRS (Command Query Responsibility Segregation)|CQRS]] → `GET` = queries; `POST`/`PUT`/`PATCH`/`DELETE` = commands.

## 2. Los 4 pilares de una API REST

### A. Los recursos (endpoints)

En REST, todo es un **recurso** (usuarios, productos, pedidos, facturas). Cada recurso se identifica con una URL única llamada **endpoint**.

**Reglas de nombrado:**

| Regla | ❌ Mal | ✅ Bien |
|---|---|---|
| Sustantivos en **plural**, nunca verbos | `/obtenerTodosLosProductos`, `/crearUsuario` | `/productos`, `/usuarios` |
| **Minúsculas** y guiones (`kebab-case`) | `/OrdenesDeCompra`, `/ordenes_compra` | `/ordenes-compra` |
| **Jerarquía** para relaciones (máximo 2 o 3 niveles) | `/lineas?pedido=45` (aceptable) | `/pedidos/45/lineas` |
| El **método** indica la acción, no la URL | `POST /productos/45/eliminar` | `DELETE /productos/45` |
| Acciones que no encajan en CRUD: sustantivo del **proceso** o **sub-recurso** | `POST /pedidos/45/cancelar` (tolerable, muy común) | `POST /pedidos/45/cancelaciones` o `PATCH /pedidos/45` con `{"estado":"cancelado"}` |
| Sin extensiones ni barra final | `/productos.json`, `/productos/` | `/productos` (el formato va en `Accept`) |

### B. Los métodos HTTP (las acciones)

Si el endpoint es el *sustantivo*, el método HTTP es el *verbo*.

| Método | Acción | CRUD | ¿Idempotente? | ¿Seguro? | Éxito típico |
|---|---|---|---|---|---|
| **GET** | Leer | **R**ead | ✅ | ✅ | `200` |
| **POST** | Crear / disparar una acción | **C**reate | ❌ | ❌ | `201` (crear), `200`/`202` (acción) |
| **PUT** | Reemplazar el recurso **completo** | **U**pdate | ✅ | ❌ | `200` / `204` |
| **PATCH** | Modificar **parcialmente** | **U**pdate | ⚠️ depende | ❌ | `200` / `204` |
| **DELETE** | Eliminar | **D**elete | ✅ | ❌ | `204` |

> [!important] `PUT` vs `PATCH`
> - **`PUT /productos/45`** con `{"nombre": "Laptop"}` → el producto queda **solo** con nombre; el precio y el resto de campos se **borran** (o la petición se rechaza por incompleta). Reemplazo total.
> - **`PATCH /productos/45`** con `{"precio": 999}` → **solo** cambia el precio.
> Formatos estándar para PATCH: **JSON Merge Patch** (RFC 7396, el JSON parcial de arriba) y **JSON Patch** (RFC 6902, lista de operaciones `add`/`remove`/`replace`). ASP.NET Core soporta ambos.

Ver [[Idempotencia]] para entender por qué `PUT` y `DELETE` son seguros de reintentar y `POST` no.

### C. Los códigos de estado (Status Codes)

El servidor responde siempre con un número de tres dígitos que resume cómo salió la petición:

- **2xx (éxito)**: `200 OK`, `201 Created`, `204 No Content`, `202 Accepted`.
- **3xx (redirección)**: `301`, `304 Not Modified`.
- **4xx (error del cliente)**: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Content`, `429 Too Many Requests`.
- **5xx (error del servidor)**: `500 Internal Server Error`, `502`, `503`, `504`.

👉 Detalle completo, ejemplos y el formato **Problem Details** para errores en [[Status Code]].

### D. El formato de datos (JSON)

Las APIs REST modernas se comunican casi exclusivamente con **JSON** (JavaScript Object Notation): ligero, legible por humanos y máquinas.

```json
{
  "id": 142,
  "nombre": "Laptop Gamer",
  "precio": 1299.99,
  "disponible": true,
  "etiquetas": ["gaming", "portátil"],
  "creadoEn": "2026-09-01T14:30:00Z"
}
```

**Convenciones:**

- `Content-Type: application/json` en peticiones con cuerpo; `Accept: application/json` para indicar qué se espera.
- Nombres de propiedades en **`camelCase`** de forma consistente (System.Text.Json lo hace por defecto en ASP.NET Core).
- Fechas en **ISO 8601 / RFC 3339** y en **UTC** (`2026-09-01T14:30:00Z`).
- Dinero como **string decimal** o entero en centavos, nunca `float`.
- Envolver las colecciones en un objeto (`{ "items": [...], "total": 120 }`) para poder añadir metadatos sin romper clientes.

## 3. Anatomía de una petición y una respuesta

```
Cliente                                                     Servidor
  │                                                            │
  │  POST /productos HTTP/1.1                                  │
  │  Host: api.tienda.com                                      │
  │  Authorization: Bearer eyJhbGciOi...                       │
  │  Content-Type: application/json                            │
  │  Idempotency-Key: 7f3e2a1c-9b...                           │
  │                                                            │
  │  { "nombre": "Laptop Gamer", "precio": 1299.99 }           │
  │ ─────────────────────────────────────────────────────────► │
  │                                          1. Valida el token│
  │                                          2. Valida el JSON │
  │                                          3. Aplica reglas  │
  │                                          4. Guarda en BD   │
  │ ◄───────────────────────────────────────────────────────── │
  │  HTTP/1.1 201 Created                                      │
  │  Location: /productos/142                                  │
  │  Content-Type: application/json                            │
  │  ETag: "a1b2c3"                                            │
  │                                                            │
  │  { "id": 142, "nombre": "Laptop Gamer", "precio": 1299.99 }│
```

1. **Petición (Request)**: método + URL + **headers** (autenticación, formato, correlación) + **body** (solo en `POST`/`PUT`/`PATCH`).
2. **Procesamiento**: autenticación, validación, lógica de negocio (idealmente en la capa de aplicación de [[Clean Architecture]], no en el controlador), persistencia.
3. **Respuesta (Response)**: **status code** + headers + **body** (el recurso creado con su ID).

## 4. Ejemplo práctico: API de una biblioteca

| Método y ruta | Qué hace | Éxito | Errores típicos |
|---|---|---|---|
| `GET /libros` | Lista libros (paginada, filtrable) | `200` | `401` |
| `GET /libros/45` | Un libro | `200` | `404` |
| `POST /libros` | Crea un libro | `201` + `Location: /libros/46` | `400`, `422`, `409` (ISBN duplicado) |
| `PUT /libros/45` | Reemplaza el libro 45 completo | `200` / `204` | `404`, `422`, `412` |
| `PATCH /libros/45` | Cambia solo algunos campos | `200` / `204` | `404`, `422` |
| `DELETE /libros/45` | Elimina el libro 45 | `204` | `404`, `403` |
| `GET /libros/45/prestamos` | Préstamos de ese libro | `200` | `404` |
| `POST /libros/45/prestamos` | Registra un préstamo | `201` | `409` (ya prestado) |

La URL base (`/libros`, `/libros/45`) se mantiene casi idéntica; **lo que cambia el comportamiento es el método HTTP**.

### Minimal API en ASP.NET Core

```csharp
var libros = app.MapGroup("/libros").RequireAuthorization();

libros.MapGet("/", async (int page = 1, int pageSize = 20, string? autor = null, ILibroQueries q = null!) =>
    Results.Ok(await q.ListarAsync(page, pageSize, autor)));

libros.MapGet("/{id:int}", async (int id, ILibroQueries q) =>
    await q.ObtenerAsync(id) is { } libro ? Results.Ok(libro) : Results.NotFound());

libros.MapPost("/", async (CrearLibroRequest req, ILibroService svc) =>
{
    var id = await svc.CrearAsync(req);
    return Results.Created($"/libros/{id}", new { id });
});

libros.MapDelete("/{id:int}", async (int id, ILibroService svc) =>
    await svc.EliminarAsync(id) ? Results.NoContent() : Results.NotFound());
```

## 5. Buenas prácticas que distinguen una API profesional

### Versionado

Las APIs cambian; los clientes no se actualizan a la vez. Opciones:

| Estrategia | Ejemplo | Pros / contras |
|---|---|---|
| **En la URL** | `/v1/libros`, `/v2/libros` | La más común y visible; "ensucia" la URL según los puristas |
| **Cabecera personalizada** | `Api-Version: 2` | URL limpia; menos descubrible |
| **Media type** | `Accept: application/vnd.tienda.v2+json` | La más "REST"; la más incómoda |
| **Query string** | `/libros?api-version=2` | Usada por Azure; fácil de probar |

Regla: **cambios compatibles** (añadir campos opcionales) no requieren versión nueva; **cambios incompatibles** (quitar o renombrar campos, cambiar tipos) sí. En .NET: paquete `Asp.Versioning.Http`.

### Paginación, filtrado y ordenación

Nunca devuelvas "toda la tabla".

```
GET /libros?page=2&pageSize=20&autor=Tolkien&sort=-anio,titulo
```

```json
{
  "items": [ ... ],
  "page": 2,
  "pageSize": 20,
  "totalItems": 143,
  "totalPages": 8,
  "links": { "next": "/libros?page=3&pageSize=20", "prev": "/libros?page=1&pageSize=20" }
}
```

Para colecciones muy grandes o que cambian mucho, usa **paginación por cursor** (`?after=eyJpZCI6MTQyfQ`) en vez de `page`/`offset`: es estable y eficiente.

### Idempotencia en `POST`

Para operaciones críticas (pagos, pedidos) acepta la cabecera `Idempotency-Key` para que un reintento no duplique el efecto. Detalle en [[Idempotencia]].

### Caché y concurrencia con ETags

- El servidor devuelve `ETag: "a1b2c3"` (hash de la versión del recurso).
- Lectura condicional: `If-None-Match: "a1b2c3"` → `304 Not Modified` si no cambió.
- Escritura condicional: `If-Match: "a1b2c3"` → `412 Precondition Failed` si alguien lo modificó antes (**concurrencia optimista**).

### Errores consistentes

Usa **Problem Details** (RFC 9457) en todos los errores. Ver [[Status Code]].

### Seguridad

- **HTTPS siempre**.
- **OAuth 2.0 / OpenID Connect** con tokens **JWT** en `Authorization: Bearer`. Nunca credenciales en la URL.
- **Rate limiting** (`429` + `Retry-After`), normalmente en el [[API Gateway]].
- **Validar toda entrada**; no confiar en el cliente.
- No exponer IDs secuenciales si permiten enumerar recursos ajenos (usar GUIDs o comprobar autorización por recurso).

### Documentación: OpenAPI

**OpenAPI** (antes Swagger) describe la API en un archivo YAML/JSON del que se generan documentación interactiva, clientes y pruebas. ASP.NET Core genera el documento con `AddOpenApi()` / `MapOpenApi()` (desde .NET 9) o con Swashbuckle / NSwag.

### HATEOAS y el Modelo de Madurez de Richardson

Leonard Richardson propuso 4 niveles para medir "cuán REST" es una API:

| Nivel | Característica | Ejemplo |
|---|---|---|
| **0** | Un solo endpoint, todo por `POST` (RPC sobre HTTP) | `POST /api` con `{"accion": "obtenerLibro"}` |
| **1** | **Recursos** con URLs propias | `/libros/45` |
| **2** | Recursos + **métodos HTTP** y **códigos de estado** correctos | `GET /libros/45` → `200`; `DELETE` → `204` |
| **3** | **HATEOAS**: las respuestas incluyen **enlaces** a las acciones posibles | `"links": [{"rel": "prestar", "href": "/libros/45/prestamos", "method": "POST"}]` |

La mayoría de las APIs del mundo real están en el **nivel 2** y funcionan perfectamente. HATEOAS (nivel 3) es el ideal de Fielding pero pocas APIs lo implementan por completo.

## 6. Comparación con otros estilos

| Estilo | Cuándo brilla | Limitaciones |
|---|---|---|
| **REST** | APIs públicas, CRUD sobre recursos, aprovechar caché HTTP | *Over-fetching* / *under-fetching*; muchas llamadas para pantallas complejas |
| **GraphQL** | Frontends que necesitan datos de muchas entidades en una llamada, con forma variable | Caché HTTP difícil; complejidad en el servidor; riesgo de consultas costosas |
| **gRPC** | Comunicación **interna** entre microservicios, alto rendimiento, streaming, contratos estrictos | Binario (poco legible); soporte limitado en navegadores |
| **WebSockets / SSE** | Tiempo real, push del servidor | No es request/response; otro modelo mental (ver [[Go routines]] para el lado servidor) |
| **Eventos / mensajería** | Desacoplamiento, procesos asíncronos | Consistencia eventual; ver [[🏗️ Diseño de Microservicios]] |

## Puntos clave

- REST = **recursos** (sustantivos en plural) + **métodos HTTP** (verbos) + **códigos de estado** + **JSON**.
- **Sin estado**: cada petición lleva su autenticación.
- `PUT` reemplaza, `PATCH` modifica parcialmente, `POST` crea o dispara.
- `201` con `Location` al crear; `204` al borrar; Problem Details en errores.
- Producción real: **versionado, paginación, filtrado, idempotencia, ETags, rate limiting, OpenAPI**.
- Nivel 2 de Richardson es el estándar práctico.

## Errores comunes

- Verbos en las URLs (`/getUsers`, `/deleteProduct/5`).
- Todo por `POST` y todo con `200` (nivel 0 de Richardson).
- Devolver `200` con `{"success": false}`.
- Exponer las entidades de base de datos directamente como JSON (acopla la API al esquema; usa DTOs).
- Sin paginación: `GET /pedidos` devuelve 2 millones de filas.
- Guardar estado de sesión en el servidor (rompe *stateless* y el escalado horizontal).
- Romper clientes con cambios incompatibles sin versionar.

## 🎯 Para entrevistas y exámenes

- *"¿Qué es REST?"* → Estilo arquitectónico de Fielding con 6 restricciones; la clave es *stateless* e interfaz uniforme.
- *"¿PUT vs PATCH?"* → Reemplazo total vs parcial; PUT es idempotente, PATCH depende.
- *"¿Qué código al crear un recurso?"* → `201 Created` + `Location`.
- *"¿Cómo versionarías una API?"* → URL (`/v1`) es lo más común; cabecera o media type son alternativas.
- *"¿REST vs gRPC?"* → REST externo y universal; gRPC interno y de alto rendimiento.
- *"¿Qué es HATEOAS?"* → Respuestas con enlaces a las acciones disponibles; nivel 3 de Richardson.

## Referencias

- Roy Fielding, *Architectural Styles and the Design of Network-based Software Architectures* (tesis, 2000), capítulo 5: https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- RFC 9110, *HTTP Semantics*: https://www.rfc-editor.org/rfc/rfc9110
- Martin Fowler, *Richardson Maturity Model*: https://martinfowler.com/articles/richardsonMaturityModel.html
- Microsoft Learn, *Web API design best practices*: https://learn.microsoft.com/azure/architecture/best-practices/api-design
- Microsoft, *REST API Guidelines*: https://github.com/microsoft/api-guidelines
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
