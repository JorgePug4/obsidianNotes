Garantiza la consistencia entre la base de datos y la publicación de eventos (evita el *Dual-Write Problem*).
* Guarda el cambio de estado y el evento a emitir en una tabla `Outbox` **dentro de la misma transacción de base de datos** (ej. `SaveChanges` en EF Core).
* Un *Background Worker* lee la tabla y asegura la entrega del mensaje al Broker.