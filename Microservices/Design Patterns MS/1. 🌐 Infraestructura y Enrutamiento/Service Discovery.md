---
tags: [microservicios, patrones, infraestructura, kubernetes, dotnet]
up: "[[🏗️ Diseño de Microservicios]]"
---

# Service Discovery

> [!info] Categoría: **Infraestructura y enrutamiento**. Responde a la pregunta: *"¿en qué IP y puerto está ahora mismo el servicio de inventario?"*

## ¿Qué es?

**Service Discovery** es el mecanismo por el cual un servicio **localiza dinámicamente** la dirección de red (IP + puerto) de otro servicio al que necesita llamar, sin tenerla escrita en su configuración.

Se apoya en un **registro de servicios** (*Service Registry*): una base de datos viva de instancias disponibles, que se actualiza cuando las instancias arrancan, se detienen o dejan de responder.

## ¿Para qué sirve?

En un entorno cloud/contenedores las direcciones **cambian constantemente**:

- Las instancias se crean y destruyen con el **autoescalado**.
- Un pod que se reinicia recibe **otra IP**.
- Los despliegues *blue-green* o *canary* cambian qué instancias reciben tráfico.

Guardar `http://10.0.4.17:8080` en un `appsettings.json` es un **anti-patrón**: en cuanto la instancia muera, el cliente fallará hasta el próximo despliegue.

## Conceptos relacionados

- [[API Gateway]] → el Gateway es un gran consumidor de Service Discovery: necesita resolver a qué instancia reenviar cada petición.
- [[🏗️ Diseño de Microservicios]] → sección de *Health Checks*: el registro solo debe devolver instancias **sanas**.
- [[Circuit Breaker]] → complementario: Discovery encuentra instancias; el Circuit Breaker deja de llamar a las que fallan.
- [[04 - Azure Load Balancer|Azure Load Balancer]] / [[06 - Azure Application Gateway|Application Gateway]] → en el modelo *server-side* el balanceador oculta las instancias detrás de una IP estable.
- [[01 - Azure Virtual Network|Azure Virtual Network]] → el DNS privado de Azure resuelve nombres dentro de la red virtual.

## ¿Cómo funciona?

Hay dos decisiones de diseño: **quién registra** y **quién resuelve**.

### Registro: ¿quién anota la instancia en el registro?

| Patrón | Cómo | Ejemplo |
|---|---|---|
| **Auto-registro** (*Self-registration*) | La propia instancia se registra al arrancar y envía *heartbeats* | Eureka, Consul con agente en la app |
| **Registro por terceros** (*Third-party registration*) | La plataforma detecta la instancia y la registra | **Kubernetes** (el *kubelet* y el *control plane* actualizan los `Endpoints`), Registrator |

### Resolución: ¿quién consulta el registro?

#### Client-side discovery

```
Cliente ──(1) ¿dónde está "inventario"?──► Registro (Consul / Eureka)
   │   ◄──(2) [10.0.1.5:8080, 10.0.1.9:8080]──┘
   └──(3) elige una instancia (round-robin) y llama directamente
```

- ✅ Sin salto extra; el cliente puede aplicar balanceo inteligente.
- ❌ Cada lenguaje/framework necesita una librería cliente; acopla la app a la tecnología del registro.

#### Server-side discovery

```
Cliente ──(1) llama a http://inventario──► Balanceador / DNS ──(2) elige instancia──► Instancia
```

- ✅ El cliente no sabe nada: hace una llamada normal a un nombre estable.
- ❌ Un salto de red más; el balanceador debe ser altamente disponible.

> [!important] En Kubernetes, esto viene de serie
> Un `Service` de Kubernetes crea un **nombre DNS estable** (`inventario.default.svc.cluster.local`) y una IP virtual. `kube-proxy` reparte el tráfico entre los pods sanos. Es **server-side discovery con registro por terceros**, y en la mayoría de proyectos modernos **no hace falta añadir nada más**.

### Health checks: la pieza que hace que funcione

Un registro que devuelve instancias muertas es peor que ninguno. Por eso:

- Cada instancia expone endpoints de salud (`/health/live`, `/health/ready`).
- El registro (o Kubernetes con sus *probes*) los consulta periódicamente.
- Las instancias no sanas se **retiran del registro** hasta que se recuperen.

## Ejemplo

### .NET Aspire / `Microsoft.Extensions.ServiceDiscovery`

Desde .NET 8, Microsoft ofrece una abstracción de Service Discovery integrada con `HttpClient`. En el código se usa un **nombre lógico** con el esquema `https+http://`:

```csharp
builder.Services.AddServiceDiscovery();

builder.Services.AddHttpClient<InventarioClient>(client =>
{
    // "inventario" se resuelve en tiempo de ejecución:
    // en local vía configuración / Aspire, en producción vía DNS de Kubernetes
    client.BaseAddress = new Uri("https+http://inventario");
})
.AddServiceDiscovery();
```

Configuración local equivalente (sin Kubernetes):

```json
{
  "Services": {
    "inventario": {
      "https": [ "localhost:7101" ],
      "http":  [ "localhost:5101" ]
    }
  }
}
```

### Otras herramientas

| Herramienta | Tipo | Notas |
|---|---|---|
| **Kubernetes DNS + Services** | Server-side, terceros | El estándar de facto |
| **HashiCorp Consul** | Registro + health checks + KV | Muy usado fuera de Kubernetes; también hace *service mesh* |
| **etcd** | Almacén KV distribuido | Base del registro de Kubernetes |
| **Netflix Eureka** | Client-side | Ecosistema Spring Cloud; menos relevante hoy |
| **Steeltoe** | Cliente .NET | Integra Consul/Eureka en .NET |
| **Azure Container Apps / App Service** | Server-side gestionado | El nombre del servicio se resuelve automáticamente dentro del entorno |

## Ventajas

- Elimina direcciones **hardcodeadas** y despliegues por cambio de IP.
- Soporta **autoescalado** y despliegues sin interrupciones.
- Combinado con health checks, enruta solo a instancias **sanas**.
- Base para balanceo de carga, *canary releases* y *service mesh*.

## Desventajas / Limitaciones

- El registro es **infraestructura crítica**: si cae, nadie encuentra a nadie (debe ser distribuido y HA).
- **Latencia** de propagación: una instancia recién muerta puede seguir apareciendo unos segundos (mitigar con Circuit Breaker y reintentos).
- Client-side discovery **acopla** la aplicación al registro.
- Fuera de Kubernetes, hay que instalar y operar la herramienta.

## Comparación

| Enfoque | Quién resuelve | Salto extra | Acoplamiento del cliente | Cuándo |
|---|---|---|---|---|
| **URLs en configuración** | Nadie | No | Total (anti-patrón) | Nunca en producción dinámica |
| **Client-side** | La app | No | Alto (librería) | Necesitas balanceo personalizado en el cliente |
| **Server-side** | Balanceador / DNS | Sí | Ninguno | **Por defecto**, especialmente en Kubernetes |

## Puntos clave

- Resuelve direcciones **en tiempo de ejecución**, no en despliegue.
- Dos ejes: **auto-registro vs registro por terceros**; **client-side vs server-side**.
- **Kubernetes** lo resuelve nativamente con `Service` + DNS interno.
- Sin **health checks**, el registro devuelve instancias muertas.
- En .NET: `Microsoft.Extensions.ServiceDiscovery` con nombres lógicos `https+http://servicio`.

## Errores comunes

- IPs o URLs de instancias en `appsettings.json`.
- Registrar la instancia como disponible antes de que termine de arrancar (falta *readiness*).
- Un registro con una sola instancia (punto único de fallo).
- Añadir Consul/Eureka en Kubernetes sin necesidad, duplicando lo que ya ofrece la plataforma.

## 🎯 Para entrevistas y exámenes

- *"¿Client-side vs server-side discovery?"* → Quién consulta el registro: la aplicación o el balanceador.
- *"¿Cómo hace Service Discovery Kubernetes?"* → Objetos `Service` con DNS interno y `kube-proxy`; registro por terceros, resolución server-side.
- *"¿Por qué son imprescindibles los health checks?"* → Para retirar instancias no sanas del registro.

## Referencias

- Chris Richardson, *Service Discovery patterns*: https://microservices.io/patterns/server-side-discovery.html
- Kubernetes, *Service*: https://kubernetes.io/docs/concepts/services-networking/service/
- Microsoft Learn, *Service discovery in .NET*: https://learn.microsoft.com/dotnet/core/extensions/service-discovery

---
⬅️ [[🏗️ Diseño de Microservicios|Volver a Diseño de Microservicios]]
