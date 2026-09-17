---
tags: [api-rest, http, backend, dotnet]
up: "[[Diseño Api Rest]]"
aliases: [Códigos de estado HTTP, HTTP Status Codes]
---

# Códigos de estado HTTP (Status Codes)

> [!info] Definidos en la **RFC 9110** (*HTTP Semantics*, 2022), que reemplaza a la RFC 7231. El registro oficial lo mantiene la IANA.

## ¿Qué es?

Un **código de estado** es un número de **tres dígitos** que el servidor devuelve en cada respuesta HTTP para indicar **cómo terminó la petición**. El primer dígito define la **familia**:

| Familia | Significado | Quién tiene la "culpa" |
|---|---|---|
| **1xx** | Informativo | — (raro en APIs) |
| **2xx** | ✅ Éxito | Nadie |
| **3xx** | ↪️ Redirección | Nadie; el recurso está en otro sitio |
| **4xx** | ❌ Error del **cliente** | Quien hizo la petición |
| **5xx** | 🔥 Error del **servidor** | El backend |

## ¿Para qué sirve?

- Permite al cliente **reaccionar sin leer el cuerpo**: reintentar (`503`), pedir login (`401`), mostrar "no encontrado" (`404`).
- Los **intermediarios** (proxies, CDN, [[API Gateway]]) los usan para cachear, reintentar o cortar tráfico.
- Los sistemas de **monitorización** los agregan: una subida de `5xx` es una alerta; una subida de `4xx` suele ser un cliente roto.
- Es parte del **contrato** de una [[Diseño Api Rest|API REST]] bien diseñada.

## Conceptos relacionados

- [[Diseño Api Rest|Diseño de APIs REST]] → los códigos son uno de los 4 pilares.
- [[Idempotencia]] → un `DELETE` repetido devuelve `404` la segunda vez y sigue siendo idempotente.
- [[Retry con Backoff Exponencial]] → qué códigos merecen reintento (`429`, `502`, `503`, `504`) y cuáles no (`4xx`).
- [[Circuit Breaker]] → los `5xx` y timeouts alimentan el contador de fallos.
- [[API Gateway]] → devuelve `401`, `429`, `502`, `503`, `504` en nombre de los servicios.
- [[Autenticación vs. Autorización]] → la diferencia exacta entre `401` y `403`.

## Los códigos que vas a usar todos los días

### 🟢 2xx: todo salió bien

| Código | Nombre | Cuándo | Ejemplo |
|---|---|---|---|
| **200** | OK | Respuesta estándar de éxito con cuerpo | `GET /productos` devuelve la lista |
| **201** | Created | Se **creó** un recurso. Incluir cabecera `Location` con la URL del nuevo recurso | `POST /usuarios` → `201` + `Location: /usuarios/57` |
| **202** | Accepted | La petición se **aceptó pero aún no se procesó** (procesamiento asíncrono) | `POST /informes` encola la generación; devuelve `202` + URL para consultar el estado |
| **204** | No Content | Éxito **sin cuerpo** de respuesta | `DELETE /productos/99` |

> [!tip] `200` vs `201` vs `204`
> `POST` que crea → `201`. `PUT`/`PATCH` que actualiza → `200` con el recurso actualizado, o `204` si no devuelves nada. `DELETE` → `204`.

### 🟡 3xx: redirecciones

| Código | Nombre | Cuándo |
|---|---|---|
| **301** | Moved Permanently | La URL cambió **para siempre**; el cliente debe actualizar sus enlaces. La nueva URL va en `Location` |
| **302 / 307** | Found / Temporary Redirect | Redirección **temporal**. `307` garantiza que el método no cambia (un `POST` sigue siendo `POST`) |
| **304** | Not Modified | El recurso **no ha cambiado** desde la última vez (caché condicional con `ETag` / `If-None-Match` o `Last-Modified` / `If-Modified-Since`). Ahorra ancho de banda: no se envía cuerpo |

### 🔴 4xx: el cliente hizo algo mal

| Código | Nombre | Cuándo | Ejemplo |
|---|---|---|---|
| **400** | Bad Request | La petición está **mal formada** o los datos son inválidos y no hay un código más específico | JSON con sintaxis rota, falta el campo `email`, tipo incorrecto |
| **401** | Unauthorized | **No estás autenticado**: no enviaste credenciales o el token es inválido/expiró. (El nombre es confuso; en realidad significa *Unauthenticated*) | `GET /perfil/compras` sin token |
| **403** | Forbidden | **Sí sabemos quién eres**, pero **no tienes permiso** para esta acción. Reautenticarse no ayuda | Cliente normal intenta `DELETE /usuarios/55` |
| **404** | Not Found | El recurso **no existe** (URL incorrecta o ID inexistente). También se usa para **ocultar** recursos a los que no tienes acceso | `GET /productos/999999` |
| **405** | Method Not Allowed | El recurso existe, pero **no acepta ese método**. Incluir cabecera `Allow` | `DELETE /productos` (sobre la colección) |
| **409** | Conflict | La petición **choca con el estado actual** del recurso | Crear un usuario con un email ya registrado; actualizar con una versión antigua (concurrencia optimista); repetir un `Idempotency-Key` en curso |
| **410** | Gone | El recurso **existió** y fue eliminado permanentemente | Un recurso borrado que quieres distinguir de "nunca existió" |
| **412** | Precondition Failed | Falló una condición `If-Match` / `If-Unmodified-Since` | Actualización con `ETag` desactualizado |
| **415** | Unsupported Media Type | El `Content-Type` enviado no se acepta | Enviar XML a una API que solo acepta JSON |
| **422** | Unprocessable Content | La sintaxis es correcta pero el contenido **viola reglas de validación o negocio**. (Antes "Unprocessable Entity", de WebDAV; ahora en RFC 9110) | `"edad": -5`, `"email": "esto-no-es-un-correo"`, fecha de fin anterior a la de inicio |
| **429** | Too Many Requests | El cliente **superó el límite de peticiones** (*rate limiting*). Incluir cabecera `Retry-After` | Más de 100 peticiones/minuto con la misma API key |

> [!warning] `400` vs `422`: la discusión eterna
> - **`400`**: no puedo ni leer tu petición (JSON roto, campo obligatorio ausente, tipo incorrecto).
> - **`422`**: la leo perfectamente, pero lo que dice no es válido para el negocio.
> Muchas APIs usan `400` para todo y es aceptable. Lo importante es **ser consistente** y devolver un cuerpo de error detallado (ver Problem Details más abajo). ASP.NET Core devuelve `400` por defecto en errores de *model validation*.

> [!warning] `401` vs `403`: pregunta de entrevista garantizada
> - **`401`**: "¿Quién eres? No lo sé." → Falta o falla la **autenticación**. El servidor debe incluir `WWW-Authenticate`.
> - **`403`**: "Sé quién eres, y no puedes." → Falla la **autorización**.
> Ver [[Autenticación vs. Autorización]].

### 🔥 5xx: el servidor falló

| Código | Nombre | Cuándo | Ejemplo |
|---|---|---|---|
| **500** | Internal Server Error | Error **genérico** no controlado en el backend | Excepción no capturada, `NullReferenceException`, división entre cero |
| **501** | Not Implemented | El servidor **no soporta** la funcionalidad pedida | Un método HTTP que el servidor no reconoce |
| **502** | Bad Gateway | Un **proxy o gateway** recibió una respuesta inválida del servidor de detrás | El [[API Gateway]] llama al microservicio y este devuelve basura o cierra la conexión |
| **503** | Service Unavailable | El servidor **no puede atender ahora**: sobrecarga, mantenimiento, [[Circuit Breaker]] abierto, [[Bulkhead]] lleno. Incluir `Retry-After` | Despliegue en curso, dependencia caída |
| **504** | Gateway Timeout | Un proxy o gateway **no recibió respuesta a tiempo** del servidor de detrás | El microservicio tardó más que el timeout del gateway |

> [!important] Nunca filtres detalles en un `500`
> El cuerpo de un `500` en producción no debe incluir *stack traces*, consultas SQL ni rutas internas. Devuelve un identificador de correlación (`traceId`) para buscar el error en los logs.

## ¿Cómo funciona? El cuerpo de la respuesta de error: Problem Details

El código dice *qué tipo* de error fue; el cuerpo debe decir *cuál exactamente*. El estándar es **Problem Details** (**RFC 9457**, que reemplaza a la RFC 7807), con `Content-Type: application/problem+json`:

```json
{
  "type": "https://api.tienda.com/errores/validacion",
  "title": "La petición contiene errores de validación",
  "status": 422,
  "detail": "El campo 'edad' debe ser mayor o igual que 0.",
  "instance": "/usuarios",
  "traceId": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
  "errors": {
    "edad": ["Debe ser mayor o igual que 0."],
    "email": ["No tiene un formato válido."]
  }
}
```

En **ASP.NET Core** viene de serie:

```csharp
builder.Services.AddProblemDetails();          // Program.cs
// ...
app.UseExceptionHandler();                     // convierte excepciones no controladas en 500 con Problem Details
app.UseStatusCodePages();                      // añade cuerpo Problem Details a 404, 405, etc.

// En un endpoint:
return Results.Problem(
    statusCode: StatusCodes.Status409Conflict,
    title: "El email ya está registrado",
    detail: $"Ya existe un usuario con el email {req.Email}.");

// Errores de validación:
return Results.ValidationProblem(errores);     // 400 + campo "errors"
```

## Ejemplo: mapa de códigos para un CRUD de productos

| Petición | Éxito | Errores posibles |
|---|---|---|
| `GET /productos` | `200` + lista | `401` sin token |
| `GET /productos/42` | `200` + producto | `404` no existe |
| `POST /productos` | `201` + `Location` | `400` JSON roto · `422` precio negativo · `409` SKU duplicado · `403` sin rol admin |
| `PUT /productos/42` | `200` o `204` | `404` · `412` ETag desactualizado · `422` |
| `DELETE /productos/42` | `204` | `404` · `403` |
| Cualquiera | — | `429` demasiadas peticiones · `500` bug · `503` mantenimiento |

## Puntos clave

- **2xx** éxito · **3xx** redirección · **4xx** culpa del cliente · **5xx** culpa del servidor.
- `201` para creación (con `Location`), `204` para éxito sin cuerpo, `202` para asíncrono.
- `401` = no autenticado; `403` = autenticado sin permiso.
- `400` = petición mal formada; `422` = válida sintácticamente pero rechazada por reglas.
- `409` para conflictos de estado y duplicados; `429` para *rate limiting*.
- `502`/`503`/`504` son los códigos de **infraestructura y resiliencia**; son los que se **reintentan**.
- Devuelve errores con **Problem Details** (RFC 9457).

## Errores comunes

- Devolver `200` con `{ "error": "no encontrado" }` en el cuerpo. Rompe cachés, monitorización y clientes.
- Usar `500` para errores de validación del cliente.
- Confundir `401` y `403`.
- No incluir `Location` en un `201` ni `Retry-After` en `429`/`503`.
- Exponer *stack traces* en producción.
- Inventar códigos propios (`299`, `499`); usa los estándar y detalla en el cuerpo.

## 🎯 Para entrevistas y exámenes

- *"¿Diferencia entre 401 y 403?"* → Autenticación vs autorización.
- *"¿Qué código devuelves al crear un recurso?"* → `201 Created` con `Location`.
- *"¿Qué código para una operación asíncrona?"* → `202 Accepted`.
- *"¿Qué códigos reintentarías automáticamente?"* → `429`, `502`, `503`, `504` (y timeouts). Nunca `4xx` salvo `429`.
- *"¿Diferencia entre 502 y 504?"* → Respuesta inválida del *upstream* vs sin respuesta a tiempo.

## Resumen rápido (mnemotecnia)

La analogía clásica de pedir una hamburguesa:

> - **200**: Aquí tienes tu hamburguesa.
> - **201**: Hamburguesa nueva hecha; está en la bandeja 57.
> - **202**: Tomé tu pedido; te aviso cuando esté.
> - **204**: Listo, retiré tu bandeja; no hay nada más que ver.
> - **304**: Es la misma hamburguesa de antes; usa la que tienes.
> - **400**: No entiendo lo que pides.
> - **401**: No puedes pedir hasta que te identifiques.
> - **403**: Sabemos quién eres, pero no puedes entrar a la cocina.
> - **404**: No vendemos hamburguesas aquí.
> - **409**: Ya pediste esa misma hamburguesa; tienes una en curso.
> - **422**: Entiendo el pedido, pero "hamburguesa con -2 panes" no existe.
> - **429**: Has pedido 50 hamburguesas en un minuto; espera.
> - **500**: La cocina se está incendiando.
> - **502**: El mesero fue a la cocina y le respondieron en un idioma desconocido.
> - **503**: La cocina está cerrada por mantenimiento; vuelve en 10 minutos.
> - **504**: El mesero fue a la cocina y nunca volvió.

## Referencias

- RFC 9110, *HTTP Semantics*, sección 15 *Status Codes*: https://www.rfc-editor.org/rfc/rfc9110#section-15
- RFC 9457, *Problem Details for HTTP APIs*: https://www.rfc-editor.org/rfc/rfc9457
- IANA, *HTTP Status Code Registry*: https://www.iana.org/assignments/http-status-codes/
- MDN, *HTTP response status codes*: https://developer.mozilla.org/docs/Web/HTTP/Status
- Microsoft Learn, *Handle errors in ASP.NET Core APIs*: https://learn.microsoft.com/aspnet/core/web-api/handle-errors

---
⬅️ [[Diseño Api Rest|Volver a Diseño de APIs REST]] · [[🗺️ Índice - Ingeniería de Software|Índice de Ingeniería de Software]]
