---
tags: [arquitectura, microservicios, api-rest, mensajeria, dotnet]
up: "[[🗺️ Índice - Ingeniería de Software]]"
aliases: [Idempotente, Idempotency]
---

# Idempotencia

> [!info] Concepto **transversal**: aparece en HTTP, en mensajería, en reintentos, en Sagas y en bases de datos. Entenderlo bien desbloquea la mitad de los patrones de sistemas distribuidos.

## ¿Qué es?

Una operación es **idempotente** cuando ejecutarla **una vez o N veces produce el mismo resultado** en el sistema.

```
f(x) = f(f(x)) = f(f(f(x)))
```

- `poner_saldo(100)` → idempotente: repetirla deja el saldo en 100.
- `sumar_saldo(100)` → **no** idempotente: repetirla suma 100 cada vez.
- `borrar_usuario(42)` → idempotente: la segunda vez no hay nada que borrar, pero el estado final es el mismo (el usuario no existe).

> [!important] Idempotencia ≠ "devuelve siempre la misma respuesta"
> Se refiere al **estado del servidor**, no a la respuesta. El primer `DELETE /usuarios/42` devuelve `204` y el segundo `404`, pero la operación sigue siendo idempotente: el estado final es idéntico.

## ¿Para qué sirve?

En sistemas distribuidos **la red no es fiable**: una petición puede enviarse, ejecutarse en el servidor y perderse la respuesta. El cliente no sabe si se ejecutó y **reintenta**. Si la operación no es idempotente, el reintento **duplica el efecto**: dos cobros, dos pedidos, dos emails.

La idempotencia es lo que hace **seguros**:

- los [[Retry con Backoff Exponencial|reintentos]] automáticos,
- la mensajería con entrega **al menos una vez** ([[Transactional Outbox]], RabbitMQ, Kafka, Service Bus),
- los pasos y compensaciones de una [[Saga Pattern|Saga]],
- las proyecciones de [[Event Sourcing]] que se reprocesan,
- los *webhooks* que un proveedor reenvía si no recibe `200`.

## Conceptos relacionados

- [[Diseño Api Rest|Diseño de APIs REST]] → qué métodos HTTP son idempotentes por definición.
- [[Status Code|Códigos de estado HTTP]] → `409 Conflict` y `422` para peticiones duplicadas o inválidas.
- [[Retry con Backoff Exponencial]] → solo reintenta operaciones idempotentes.
- [[Transactional Outbox]] → entrega at-least-once; el consumidor necesita un **Inbox** idempotente.
- [[Saga Pattern]] → compensaciones idempotentes.
- [[ACID en Bases de Datos|ACID]] → una restricción `UNIQUE` es la forma más simple de idempotencia en BD.

## ¿Cómo funciona?

### Idempotencia en HTTP

| Método | ¿Idempotente? | ¿Seguro (no modifica)? | Notas |
|---|---|---|---|
| `GET` | ✅ | ✅ | Solo lee |
| `HEAD`, `OPTIONS` | ✅ | ✅ | Metadatos |
| `PUT` | ✅ | ❌ | Reemplaza el recurso completo; repetirlo deja el mismo estado |
| `DELETE` | ✅ | ❌ | Repetirlo no borra "más" |
| `POST` | ❌ | ❌ | Por definición crea o dispara una acción; repetirlo puede duplicar |
| `PATCH` | ⚠️ Depende | ❌ | `{"estado": "pagado"}` es idempotente; `{"op": "incrementar"}` no |

> [!tip] Idempotencia definida por la especificación
> RFC 9110 define `GET`, `HEAD`, `PUT`, `DELETE`, `OPTIONS` y `TRACE` como idempotentes. Que **tu implementación** lo cumpla es tu responsabilidad: un `PUT` que añade una fila a un log en cada llamada rompe el contrato.

### Hacer idempotente un `POST`: la clave de idempotencia

El patrón estándar (usado por Stripe, PayPal, Adyen y descrito en el borrador IETF *Idempotency-Key Header*):

```
1. El cliente genera un identificador único (UUID) para la operación.
2. Lo envía en la cabecera:  Idempotency-Key: 7f3e2a1c-...
3. El servidor, antes de procesar, busca la clave:
   - No existe → procesa, guarda (clave, respuesta) y responde.
   - Existe → devuelve la respuesta guardada SIN volver a procesar.
4. La clave expira tras un tiempo (ej. 24 h).
```

```csharp
// Middleware / filtro simplificado en ASP.NET Core
public async Task<IResult> CrearPago(
    [FromHeader(Name = "Idempotency-Key")] string clave,
    CrearPagoRequest req,
    IIdempotencyStore store,
    IPagoService pagos)
{
    if (string.IsNullOrEmpty(clave))
        return Results.BadRequest("Falta la cabecera Idempotency-Key");

    var previa = await store.ObtenerAsync(clave);
    if (previa is not null)
        return Results.Json(previa.Cuerpo, statusCode: previa.StatusCode); // misma respuesta

    var pago = await pagos.CrearAsync(req);           // se ejecuta UNA sola vez
    await store.GuardarAsync(clave, 201, pago, TimeSpan.FromHours(24));
    return Results.Created($"/pagos/{pago.Id}", pago);
}
```

> [!warning] Condición de carrera
> Dos peticiones con la misma clave pueden llegar **simultáneamente**. El *store* debe garantizar atomicidad: `INSERT` con clave única (falla la segunda), `SET NX` en Redis, o un bloqueo por clave. Si la segunda llega mientras la primera aún se procesa, responde `409 Conflict` o espera.

### Idempotencia en consumidores de mensajes (Inbox pattern)

```
Mensaje llega con MessageId = "m-123"
   │
   ├─ ¿"m-123" está en la tabla ProcessedMessages?  ──sí──► ACK y descartar
   │
   └─ no ──► procesar + INSERT ProcessedMessages("m-123")  (misma transacción) ──► ACK
```

MassTransit, NServiceBus y Wolverine implementan este patrón como *Inbox* o *deduplicación*.

### Idempotencia natural por diseño

A veces no hace falta una clave: se diseña la operación para que **repetirla sea inocuo**.

| No idempotente | Idempotente equivalente |
|---|---|
| `UPDATE cuentas SET saldo = saldo - 50 WHERE id = 1` | `INSERT INTO movimientos (id, cuenta, importe) VALUES ('mov-9', 1, -50)` con `id` único; el saldo se calcula o se actualiza solo si el insert tuvo éxito |
| `INSERT INTO pedidos (...)` | `INSERT ... ON CONFLICT (numero_pedido) DO NOTHING` (PostgreSQL) / `MERGE` (SQL Server) |
| `enviar_email(usuario)` | `enviar_email_si_no_enviado(usuario, evento_id)` con registro de enviados |
| `estado = estado.siguiente()` | `estado = ENVIADO` (asignación absoluta) |

## Ejemplo completo: cobro con reintento

```
Cliente ──POST /pagos (Idempotency-Key: K1)──► Servidor: cobra 100 €, guarda K1 → 201
        ◄──── (respuesta se pierde por timeout) ────
Cliente ──POST /pagos (Idempotency-Key: K1)──► Servidor: K1 existe → devuelve el 201 guardado
                                                 (NO cobra otra vez)
```

Sin la clave, el segundo `POST` habría cobrado 200 €.

## Ventajas

- Hace **seguros los reintentos**, y por tanto todo el ecosistema de resiliencia.
- Permite mensajería **at-least-once**, mucho más simple y barata que *exactly-once*.
- Simplifica la recuperación tras fallos: "vuelve a ejecutar todo" es una estrategia válida.

## Desventajas / Limitaciones

- Requiere **almacenar claves o IDs procesados** (espacio, expiración, limpieza).
- Añade complejidad al cliente (debe generar y conservar la clave entre reintentos).
- Las condiciones de carrera exigen **atomicidad** en el almacén de claves.
- No todas las operaciones se pueden hacer idempotentes fácilmente (enviar un SMS ya enviado no se "des-envía"); ahí solo cabe la deduplicación previa.

## Comparación

| Concepto | Significado | Ejemplo |
|---|---|---|
| **Idempotente** | Repetir no cambia el estado final | `PUT`, `DELETE`, `SET x = 5` |
| **Seguro** (*safe*) | No modifica el estado | `GET`, `HEAD` (todo lo seguro es idempotente, no al revés) |
| **Determinista** | Misma entrada → misma salida | `sumar(2, 3)`; una función puede ser determinista y no idempotente (`incrementar`) |
| **Exactly-once** | El mensaje se procesa exactamente una vez | Muy difícil en distribuido; en la práctica = at-least-once + idempotencia |

## Puntos clave

- Idempotente = **repetir no duplica el efecto**.
- `GET`, `PUT`, `DELETE` son idempotentes; `POST` no lo es por defecto.
- **Clave de idempotencia** (`Idempotency-Key`) para hacer idempotente un `POST`.
- **Inbox / deduplicación por MessageId** en consumidores de mensajes.
- Cuidado con las **condiciones de carrera** en el almacén de claves.
- Es el **requisito previo** de Retry, Outbox y Saga.

## Errores comunes

- Reintentar un `POST` de pago sin clave de idempotencia.
- Guardar la clave **después** de procesar sin transacción: si se cae en medio, se procesa dos veces.
- Usar como clave algo que cambia entre reintentos (un timestamp).
- Confundir idempotente con "devuelve la misma respuesta".
- Asumir que el broker garantiza *exactly-once* y no deduplicar.

## 🎯 Para entrevistas y exámenes

- *"¿Qué métodos HTTP son idempotentes?"* → `GET`, `HEAD`, `PUT`, `DELETE`, `OPTIONS`. `POST` no. `PATCH` depende.
- *"¿Cómo harías idempotente un POST de pago?"* → Cabecera `Idempotency-Key` + almacén de respuestas con clave única.
- *"¿Por qué importa en mensajería?"* → Los brokers garantizan at-least-once; sin idempotencia hay duplicados.
- *"¿Es idempotente `DELETE` si la segunda vez devuelve 404?"* → Sí: se mide el estado del servidor, no la respuesta.

## Referencias

- RFC 9110, *HTTP Semantics*, sección 9.2.2 *Idempotent Methods*: https://www.rfc-editor.org/rfc/rfc9110#section-9.2.2
- IETF draft, *The Idempotency-Key HTTP Header Field*: https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/
- Stripe, *Idempotent requests*: https://docs.stripe.com/api/idempotent_requests
- Microsoft Learn, *Idempotent message processing*: https://learn.microsoft.com/azure/architecture/reference-architectures/containers/aks-mission-critical/mission-critical-data-platform#idempotent-message-processing

---
⬅️ [[🗺️ Índice - Ingeniería de Software|Volver al índice de Ingeniería de Software]]
