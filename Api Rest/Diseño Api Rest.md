# 1. ¿Qué es una API y qué significa REST?

Primero, bajemos los conceptos a la Tierra.

- **API (Application Programming Interface):** Piensa en ella como el **mesero de un restaurante**. Tú (el cliente) miras el menú y le pides un platillo. El mesero lleva tu orden a la cocina (el servidor), el chef prepara la comida y el mesero te la trae de vuelta. Tú no sabes cómo el chef cocinó la carne, solo sabes que interactuando con el mesero obtienes tu comida. Una API hace exactamente eso pero entre dos sistemas de software.
    
- **REST (Representational State Transfer):** No es un código ni un lenguaje de programación; es un **estilo de arquitectura**. Es un conjunto de "reglas de buena conducta" que los desarrolladores acordaron seguir para que crear APIs sea estándar, limpio y fácil de entender para cualquiera. Cuando una API sigue estas reglas, la llamamos **RESTful**.
    

## 2. Los 4 Pilares de una API REST

Para entender cualquier API REST, solo necesitas dominar estos cuatro elementos básicos:

### A. Los Recursos (Endpoints)

En REST, todo es un "recurso" (usuarios, productos, fotos, facturas). Cada recurso se identifica con una dirección URL única llamada **Endpoint**.

- _Regla de oro:_ Los endpoints siempre deben usar **sustantivos en plural**, nunca verbos.
    

> ❌ _Mal diseñado:_ `[https://api.tienda.com/obtenerTodosLosProductos](https://api.tienda.com/obtenerTodosLosProductos)`
> 
> _Bien diseñado:_ `[https://api.tienda.com/productos](https://api.tienda.com/productos)`

### B. Los Métodos HTTP (Las Acciones)

Si el endpoint es el _sustantivo_, el método HTTP es el _verbo_. Le dice al servidor qué quieres hacer con ese recurso. Los cuatro más comunes se conocen como **CRUD**:

| **Método HTTP** | **Acción**       | **Equivalente CRUD** | **Ejemplo de uso**                                    |
| --------------- | ---------------- | -------------------- | ----------------------------------------------------- |
| **GET**         | Leer / Recuperar | **R**ead             | Traer la lista de productos o un producto específico. |
| **POST**        | Crear            | **C**reate           | Crear un nuevo producto en la base de datos.          |
| **PUT / PATCH** | Actualizar       | **U**pdate           | Modificar el precio de un producto existente.         |
| **DELETE**      | Eliminar         | **D**elete           | Borrar un producto.                                   |

### C. Los Códigos de Estado (Status Codes)

Cuando le haces una petición al servidor, este siempre te responde con un número de tres dígitos. Es su forma rápida de decirte cómo salieron las cosas:

- **2xx (Éxito):** Todo salió bien. El más famoso es el **`200 OK`** (petición exitosa) o **`201 Created`** (se creó un recurso).
    
- **3xx (Redirección):** El recurso se movió a otro lado.
    
- **4xx (Errores del Cliente):** Tú hiciste algo mal. Por ejemplo: **`400 Bad Request`** (mandaste datos incompletos), **`401 Unauthorized`** (olvidaste iniciar sesión) o el legendario **`404 Not Found`** (ese endpoint o recurso no existe).
    
- **5xx (Errores del Servidor):** Tú lo hiciste bien, pero el servidor se rompió por dentro. El clásico es **`500 Internal Server Error`**.
    

### D. El Formato de Datos (JSON)

Las APIs REST modernas se comunican casi exclusivamente usando **JSON** (JavaScript Object Notation). Es un formato de texto muy ligero que tanto los humanos como las computadoras pueden leer sin esfuerzo. Se ve así:

JSON

```
{
  "id": 142,
  "nombre": "Laptop Gamer",
  "precio": 1299.99,
  "disponible": true
}
```

## 3. Anatomía de una Petición (Request / Response)

Cuando programas y te conectas a una API, el flujo se ve exactamente así en la práctica:

**1.El Cliente envía la Petición (Request):**Cliente -.

Servidor">

Tu aplicación junta el **Endpoint** (ej. `/productos`), el **Método** (ej. `POST`), los **Headers** (metadatos como tokens de seguridad) y el **Body** (el JSON con los datos del nuevo producto) y lo envía por internet.

**2.El Servidor procesa la orden:**Dentro del Servidor.

La API recibe los datos, valida que el usuario tenga permisos, se conecta a la base de datos, guarda el producto y genera un resultado.

**3.El Servidor devuelve la Respuesta (Response):**Servidor -.

Cliente">

El servidor te responde con un **Status Code** (ej. `201 Created`) y un **Body** en JSON que suele confirmar el objeto creado con su ID único generado por la base de datos.

## 4. Diseñando una API: Ejemplo Práctico

Imagina que vamos a diseñar la API para administrar los libros de una biblioteca. Siguiendo las reglas REST, nuestra estructura de endpoints y métodos quedaría así de limpia:

- `GET /libros` $\rightarrow$ Trae la lista de todos los libros.
    
- `GET /libros/45` $\rightarrow$ Trae únicamente el libro con el ID 45.
    
- `POST /libros` $\rightarrow$ Crea un nuevo libro (enviando el JSON en el cuerpo).
    
- `PUT /libros/45` $\rightarrow$ Reemplaza o actualiza toda la información del libro 45.
    
- `DELETE /libros/45` $\rightarrow$ Elimina el libro 45 de la biblioteca.
    

¿Notas la elegancia? La URL de la base (`/libros` o `/libros/45`) se mantiene casi idéntica; lo que cambia el comportamiento por completo es el método HTTP que uses.