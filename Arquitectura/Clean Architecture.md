## ¿Qué es Clean Architecture?

La **Clean Architecture** es un estilo de arquitectura de software propuesto por Robert C. Martin (*Uncle Bob*) que busca crear sistemas:

- fáciles de mantener,
- independientes de frameworks,
- fáciles de probar,
- y desacoplados de detalles externos como bases de datos, UI o APIs.

La idea principal es que **las reglas de negocio sean el centro de la aplicación** y que todo lo demás dependa de ellas, no al revés.

---

# Estructura típica de Clean Architecture

## 1. Entities (Entidades)

Contienen las reglas de negocio más importantes.

### Ejemplos
- `User`
- `Order`
- `Product`

Son objetos del dominio puro y no dependen de nada externo.

---

## 2. Use Cases / Application

Contiene la lógica de aplicación.

### Ejemplos
- `CreateOrder`
- `ProcessPayment`
- `AuthenticateUser`

Aquí se define qué hace el sistema.

---

## 3. Interface Adapters

Adaptan la información entre el mundo exterior y los casos de uso.

### Ejemplos
- Controllers
- Presenters
- Repositories

Transforman datos para que las capas internas no dependan de tecnologías externas.

---

## 4. Frameworks & Drivers

La capa más externa.

### Ejemplos
- ASP.NET
- Entity Framework
- SQL Server
- APIs REST
- UI

Son detalles de infraestructura.

---

# Dependency Rule

Las dependencias siempre apuntan hacia adentro.

```text
Frameworks -> Adapters -> Use Cases -> Entities