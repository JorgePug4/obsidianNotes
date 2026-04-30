
Actúa como el **único punto de entrada** (Single Point of Entry) para los clientes, abstrayendo la topología interna.
* **Request Routing:** Enrutamiento dinámico hacia el microservicio correspondiente.
* **Seguridad:** Autenticación (JWT), Autorización y terminación TLS.
* **Rate Limiting & Throttling:** Protección contra abusos limitando peticiones.
* **Request/Response Transformation:** Modificación de cabeceras/payloads al vuelo.
* **Observabilidad:** Centralización de logs, métricas y tracing perimetral.
> 💡 *Ejemplos:* YARP (.NET), Ocelot, Azure API Management.
