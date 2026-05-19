## Los Códigos Más Utilizados en APIs REST (Detalle a Detalle)

Vamos a desglosar los que **sí o sí** vas a usar y ver todos los días cuando programes o consumas una API:

### 🟢 Familia 2xx: Todo salió bien

#### `200 OK`

- **¿Para qué se usa?:** Es la respuesta estándar para peticiones exitosas.
    
- **Ejemplo clásico:** Haces un `GET /productos` y el servidor te regresa la lista de productos en un JSON. El código que acompaña ese JSON es un `200`.
    

#### `201 Created`

- **¿Para qué se usa?:** Específico para cuando haces una petición de creación (`POST`) y el nuevo recurso se guardó con éxito en la base de datos.
    
- **Ejemplo clásico:** Envías un `POST /usuarios` para registrar una nueva cuenta. El servidor te responde con un `201` y el JSON del usuario con su nuevo ID asignado.
    

#### `204 No Content`

- **¿Para qué se usa?:** La petición fue exitosa, pero el servidor no tiene nada que devolverte en el cuerpo de la respuesta (sin JSON).
    
- **Ejemplo clásico:** Haces un `DELETE /productos/99`. El producto se borró con éxito. No tiene sentido que el servidor te regrese el producto que acabas de destruir, así que te manda un `204` para decirte "Listo, ya quedó, no hay nada más que ver aquí".
    

### 🟡 Familia 3xx: Redirecciones

#### `301 Moved Permanently`

- **¿Para qué se usa?:** La URL a la que estás intentando acceder cambió de nombre o de lugar para siempre. El servidor te incluye la nueva URL en los datos de la respuesta para que tu app vaya hacia allá de ahora en adelante.
    

#### `304 Not Modified`

- **¿Para qué se usa?:** Es crucial para el rendimiento y la velocidad de internet (Caché). Le dice a tu app: _"Oye, el recurso que me pides no ha cambiado desde la última vez que me lo pediste. Usa la copia que tienes guardada en tu memoria"_. Ahorra tiempo y datos.
    

### 🔴 Familia 4xx: Errores tuyos (del programador o del usuario)

#### `400 Bad Request`

- **¿Para qué se usa?:** El servidor no puede procesar la petición porque está mal armada o los datos son inválidos. Es el equivalente a que falten campos obligatorios.
    
- **Ejemplo clásico:** Intentas registrar un usuario (`POST /usuarios`) pero olvidas enviar el campo `"email"`. El servidor te rechaza con un `400`.
    

#### `401 Unauthorized`

- **¿Para qué se usa?:** No estás autenticado en el sistema. El servidor no sabe quién eres porque no enviaste tus credenciales o tu Token de seguridad expiró.
    
- **Ejemplo clásico:** Intentas entrar a ver tu historial de compras (`GET /perfil/compras`), pero no has iniciado sesión.
    

#### `403 Forbidden`

- **¿Para qué se usa?:** El servidor **sí sabe quién eres**, pero no tienes permisos para hacer esa acción específica. No confundir con el 401.
    
- **Ejemplo clásico:** Iniciaste sesión con tu cuenta de cliente normal e intentas hacer un `DELETE /usuarios/55` (borrar a otro usuario). El servidor te dice: _"Sé quién eres, pero tú no eres administrador. Prohibido"_.
    

#### `404 Not Found`

- **¿Para qué se usa?:** El recurso solicitado no existe. Puede ser porque escribiste mal la URL o porque el ID que buscas ya no está en la base de datos.
    
- **Ejemplo clásico:** Haces un `GET /productos/999999` y ese producto no existe en la tienda.
    

#### `422 Unprocessable Entity`

- **¿Para qué se usa?:** Muy común en APIs modernas. Significa que la estructura de tu JSON está bien redactada, pero el contenido lógicamente no sirve.
    
- **Ejemplo clásico:** Envías el campo `"edad": -5` o `"email": "esto-no-es-un-correo"`. La API entiende el formato, pero el dato no pasa las reglas de negocio.
    

### 🔥 Familia 5xx: Errores del servidor (¡Alerta roja!)

#### `500 Internal Server Error`

- **¿Para qué se usa?:** El código comodín para cuando algo salió mal en el backend y el sistema no supo cómo manejar el problema. Es un error de código, una excepción no controlada o que la base de datos se cayó.
    
- **Ejemplo clásico:** Hay una división entre cero en el código del servidor o se olvidaron de poner un punto y coma y la app se congeló.
    

#### `503 Service Unavailable`

- **¿Para qué se usa?:** El servidor no está listo para manejar la petición en este momento. Suele pasar porque el servidor está saturado de visitas (tráfico masivo) o está apagado por mantenimiento programado.
    

## Resumen Rápido (Mnemotecnia)

Para que nunca los olvides, un viejo chiste de desarrolladores los resume a la perfección usando la analogía de pedir una hamburguesa:

> - **200:** Aquí tienes tu hamburguesa.
>     
> - **401:** No puedes pedir una hamburguesa hasta que pagues.
>     
> - **403:** Sabemos que pagaste, pero no tienes permitido entrar a la cocina por tu hamburguesa.
>     
> - **404:** No vendemos hamburguesas aquí.
>     
> - **500:** La cocina se está incendiando.
>