---
tags: [microservicios, patrones, infraestructura, seguridad, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
---

# API Gateway

> [!info] Categoría: **Infraestructura y enrutamiento**. Es la "puerta principal" de una arquitectura de microservicios.

## ¿Qué es?

Un **API Gateway** es un componente que actúa como **único punto de entrada** (*Single Point of Entry*) para los clientes externos. Recibe todas las peticiones, las **enruta** al microservicio correcto y aplica de forma **centralizada** tareas transversales: autenticación, límites de tráfico, registro, transformación de peticiones.

Los clientes no conocen la topología interna: hablan con `api.miempresa.com` y el Gateway decide a qué servicio interno va cada petición.

## ¿Para qué sirve?

Sin Gateway, cada cliente (web, móvil, terceros) tendría que:

- conocer la dirección de cada uno de los 30 microservicios,
- autenticarse contra cada uno,
- hacer 5 llamadas para pintar una pantalla,
- adaptarse cada vez que un servicio cambia de nombre o se divide.

Con Gateway, todo eso se resuelve en un solo sitio.

### Responsabilidades habituales

| Responsabilidad | Descripción |
|---|---|
| **Enrutamiento** (*routing*) | `/api/pedidos/*` → servicio Pedidos; `/api/catalogo/*` → servicio Catálogo |
| **Autenticación / autorización** | Validar el **JWT** (OAuth2 / OIDC) una sola vez y pasar la identidad a los servicios internos |
| **Terminación TLS** | El cliente habla HTTPS con el Gateway; internamente puede usarse HTTP o mTLS |
| **Rate limiting y throttling** | Limitar peticiones por cliente/API key para evitar abusos (responde `429`) |
| **Transformación** | Añadir/quitar cabeceras, adaptar formatos, versionar la API |
| **Agregación** | Combinar respuestas de varios servicios en una sola (patrón *API Composition*) |
| **Caché** | Cachear respuestas de lectura frecuentes |
| **Observabilidad** | Logs, métricas y *tracing* de todo el tráfico entrante en un solo punto |
| **Resiliencia perimetral** | [[Circuit Breaker]], [[Retry con Backoff Exponencial|reintentos]] y timeouts hacia los servicios |

## Conceptos relacionados

- [[Service Discovery]] → el Gateway necesita saber **dónde** está cada servicio; lo consulta al registro o al DNS del orquestador.
- [[Circuit Breaker]], [[Bulkhead]] → se aplican en el Gateway para proteger a los servicios internos.
- [[Diseño Api Rest|Diseño de APIs REST]] → el Gateway expone la API pública; su diseño sigue los mismos principios.
- [[Status Code|Códigos de estado HTTP]] → el Gateway devuelve `401`, `429`, `502`, `503`, `504` en nombre de los servicios.
- [[06 - Azure Application Gateway|Azure Application Gateway]] → balanceador de capa 7 de Azure; **no es** un API Gateway completo (ver comparación).
- [[04 - Azure Load Balancer|Azure Load Balancer]] → balanceador de capa 4.
- [[Azure Key Vault]] → dónde guardar los certificados TLS y secretos del Gateway.
- [[🏗️ Diseño de Microservicios]] → sección de seguridad.

## ¿Cómo funciona?

```
   📱 App móvil    🌐 SPA web     🤝 Partner
        │              │              │
        └──────────────┼──────────────┘
                       ▼
            ┌─────────────────────┐
            │     API GATEWAY     │  TLS · JWT · Rate limit · Logs
            └──┬────────┬─────────┘
               │        │         │
          ┌────▼──┐ ┌───▼────┐ ┌──▼───────┐
          │Pedidos│ │Catálogo│ │Usuarios  │
          └───────┘ └────────┘ └──────────┘
```

1. El cliente envía `GET https://api.tienda.com/pedidos/42` con `Authorization: Bearer <jwt>`.
2. El Gateway termina TLS, **valida el token** (firma, expiración, audiencia) y comprueba la cuota del cliente.
3. Busca la ruta `/pedidos/*` → servicio `pedidos`, resuelve su dirección vía [[Service Discovery]].
4. Reenvía la petición, quizá añadiendo cabeceras (`X-User-Id`, `X-Correlation-Id`).
5. Recibe la respuesta, la registra y la devuelve al cliente.

### Variante: Backend for Frontend (BFF)

En lugar de un Gateway único, se crea **un Gateway por tipo de cliente** (uno para móvil, otro para web, otro para terceros). Cada BFF agrega y adapta los datos a las necesidades exactas de su interfaz. Evita el "Gateway monolítico" que crece sin control.

## Ejemplo

### YARP en .NET (Gateway ligero, código propio)

**YARP** (*Yet Another Reverse Proxy*) es la librería de Microsoft para construir *reverse proxies* en ASP.NET Core:

```json
// appsettings.json
{
  "ReverseProxy": {
    "Routes": {
      "pedidos": {
        "ClusterId": "pedidos-cluster",
        "AuthorizationPolicy": "authenticated",
        "RateLimiterPolicy": "por-cliente",
        "Match": { "Path": "/api/pedidos/{**catch-all}" },
        "Transforms": [ { "PathRemovePrefix": "/api/pedidos" } ]
      }
    },
    "Clusters": {
      "pedidos-cluster": {
        "Destinations": {
          "d1": { "Address": "http://pedidos-svc:8080/" }
        }
      }
    }
  }
}
```

```csharp
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));
builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddRateLimiter(/* ... */);

var app = builder.Build();
app.UseAuthentication();
app.UseRateLimiter();
app.MapReverseProxy();
```

### Otras opciones

| Tipo | Ejemplos |
|---|---|
| **Librería .NET** | **YARP** (recomendado por Microsoft), Ocelot (comunidad, menos activo) |
| **Producto gestionado en cloud** | **Azure API Management**, Amazon API Gateway, Google Apigee |
| **Open source / self-hosted** | Kong, Traefik, KrakenD, Envoy Gateway, NGINX |
| **Kubernetes** | Ingress Controllers y la **Gateway API** de Kubernetes |

## Ventajas

- **Simplifica los clientes**: una sola URL, una sola autenticación.
- **Centraliza** las políticas transversales: menos código duplicado en cada servicio.
- **Oculta la topología** interna: puedes dividir o fusionar servicios sin romper clientes.
- **Reduce el número de llamadas** (*chattiness*) gracias a la agregación.
- Punto natural para **monitorizar** y **proteger** todo el tráfico entrante.

## Desventajas / Limitaciones

- **Punto único de fallo** (*SPOF*): si cae el Gateway, cae todo. Requiere alta disponibilidad (varias réplicas).
- **Cuello de botella** potencial de rendimiento y latencia añadida (un salto de red más).
- **Riesgo de acoplamiento**: si el Gateway acumula lógica de negocio se convierte en un "mini-monolito" que hay que desplegar cada vez que cambia un servicio.
- Un componente más que **operar, versionar y asegurar**.

## Comparación

| Componente | Capa | Qué hace | Qué NO hace |
|---|---|---|---|
| **Load Balancer** (L4) | Transporte | Reparte conexiones TCP entre instancias | No entiende HTTP ni rutas |
| **Reverse Proxy / LB L7** ([[06 - Azure Application Gateway\|App Gateway]], NGINX) | Aplicación | Enruta por *path*/host, terminación TLS, WAF | Normalmente no gestiona API keys, cuotas, versiones ni portal de desarrolladores |
| **API Gateway** | Aplicación | Todo lo anterior + autenticación, rate limiting por consumidor, transformación, agregación, analítica | No gestiona el tráfico **entre** servicios internos |
| **Service Mesh** (Istio, Linkerd) | Aplicación (interno) | mTLS, reintentos, observabilidad del tráfico **servicio-a-servicio** vía *sidecars* | No es la puerta de entrada externa |

> [!tip] Gateway y Service Mesh no compiten
> El Gateway gobierna el tráfico **norte-sur** (de fuera hacia dentro). El Service Mesh gobierna el tráfico **este-oeste** (entre servicios). Muchas arquitecturas usan ambos.

## Puntos clave

- **Único punto de entrada** que enruta y centraliza políticas transversales.
- Responsabilidades: **routing, auth, TLS, rate limiting, transformación, agregación, observabilidad**.
- Variante **BFF**: un Gateway por tipo de cliente.
- Riesgos: **SPOF**, latencia, acoplamiento si acumula lógica de negocio.
- En .NET: **YARP**; en Azure: **API Management**.

## Errores comunes

- Meter lógica de negocio en el Gateway.
- Desplegar una sola instancia del Gateway.
- Validar el JWT en el Gateway y **no** volver a validar nada dentro (los servicios internos deben al menos verificar que la petición viene del Gateway o confiar en una red cerrada / mTLS).
- Confundir un Application Gateway / Ingress con un API Gateway completo.
- Un Gateway único para todos los clientes que termina sirviendo mal a todos (usar BFF).

## 🎯 Para entrevistas y exámenes

- *"¿Qué problemas resuelve un API Gateway?"* → Múltiples endpoints, autenticación repetida, exceso de llamadas, acoplamiento del cliente a la topología.
- *"¿Cuál es su principal riesgo?"* → Ser punto único de fallo y cuello de botella.
- *"¿Qué es BFF?"* → Backend for Frontend: un Gateway por tipo de cliente.
- *"API Gateway vs Service Mesh"* → Norte-sur vs este-oeste.
- En **AZ-900**: Azure API Management es el servicio de API Gateway gestionado; Application Gateway es un balanceador de capa 7 con WAF.

## Referencias

- Chris Richardson, *Pattern: API Gateway / Backends for Frontends*: https://microservices.io/patterns/apigateway.html
- Microsoft Learn, *Gateway Routing / Offloading / Aggregation patterns*: https://learn.microsoft.com/azure/architecture/patterns/gateway-routing
- YARP: https://microsoft.github.io/reverse-proxy/
- Azure API Management: https://learn.microsoft.com/azure/api-management/

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
