---
tags: [cloud-native, fundamentos, arquitectura, 12-factor]
aliases: [Fundamentos Cloud Native, Qué es Cloud Native, Twelve-Factor App, 12 factores]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 01 - Fundamentos Cloud Native

> [!info] Objetivo del capítulo
> Entender **qué significa cloud native**, qué supuestos cambian respecto a una aplicación tradicional y por qué esos supuestos obligan a usar los patrones del resto de la carpeta.

## Contenido
- [[#1. ¿Qué es "cloud native"?]]
- [[#2. Lo que cambia respecto a una aplicación tradicional]]
- [[#3. Las 8 falacias de la computación distribuida]]
- [[#4. The Twelve-Factor App]]
- [[#5. Más allá de los 12 factores]]
- [[#6. Principios de diseño cloud native]]
- [[#7. Qué es (y qué no es) un patrón]]
- [[#🎯 Para entrevistas]]

---

## 1. ¿Qué es "cloud native"?

### La definición oficial (CNCF, v1.0, 2018)

La Cloud Native Computing Foundation la define, resumida, así:

> Las tecnologías cloud native permiten a las organizaciones **construir y ejecutar aplicaciones escalables** en entornos modernos y dinámicos como nubes públicas, privadas e híbridas. **Contenedores, service meshes, microservicios, infraestructura inmutable y APIs declarativas** ejemplifican este enfoque. Estas técnicas permiten sistemas **débilmente acoplados** que son **resilientes, gestionables y observables**. Combinadas con una **automatización robusta**, permiten a los ingenieros hacer cambios de alto impacto **con frecuencia y de forma predecible** con un esfuerzo mínimo.

Fuente: https://github.com/cncf/toc/blob/main/DEFINITION.md

### Traducción práctica

"Cloud native" **no** significa "está desplegado en la nube". Una VM con un monolito movida tal cual a Azure (*lift and shift*) está *en* la nube, pero no es cloud native. Cloud native describe **cómo está diseñada** la aplicación:

| Propiedad | Qué significa en la práctica |
|---|---|
| **Escalable** | Puedo añadir instancias para atender más carga, sin rediseñar |
| **Débilmente acoplada** | Puedo cambiar y desplegar una parte sin coordinar con las demás |
| **Resiliente** | La caída de una instancia, nodo o dependencia no tumba el sistema |
| **Gestionable** | Se configura y opera de forma declarativa y automatizada |
| **Observable** | Puedo saber qué está pasando dentro desde fuera (logs, métricas, trazas) |
| **Automatizada** | Construir, probar, desplegar y escalar no requiere pasos manuales |

> [!important] Idea central
> Cloud native = **diseñar asumiendo que todo falla y todo cambia**, y automatizar la respuesta a ello.

### ¿Cloud native = microservicios?

**No necesariamente.** Los microservicios son un ejemplo citado por la CNCF, pero un monolito modular, *stateless*, en contenedor, con configuración externa, health checks y despliegue automatizado cumple casi todas las propiedades anteriores. Muchos patrones de esta carpeta (Retry, Cache-Aside, Health Endpoint, Blue-Green, Feature Flags…) aplican igual a un monolito. Ver la advertencia *Monolith first* en [[🏗️ Diseño de Microservicios]].

---

## 2. Lo que cambia respecto a una aplicación tradicional

| Supuesto tradicional | Realidad cloud native | Consecuencia de diseño | Patrones |
|---|---|---|---|
| El servidor es estable y dura años | Las instancias se crean y destruyen continuamente | La app no puede guardar estado local | Stateless, External Config, Cache distribuida |
| Se escala comprando una máquina mayor (vertical) | Se escala añadiendo instancias (horizontal) | Las instancias deben ser intercambiables | Competing Consumers, Autoscaling |
| La red local es fiable | La red entre servicios falla, tiene latencia y se congestiona | Toda llamada remota puede fallar | Timeout, Retry, Circuit Breaker |
| Una base de datos, transacciones ACID | Cada servicio con su almacén | Consistencia eventual | Saga, Outbox, CQRS |
| Despliegue manual, ventanas de mantenimiento | Despliegues frecuentes y automatizados | Cada cambio debe ser seguro y reversible | Blue-Green, Canary, Feature Flags |
| Se depura entrando al servidor | Hay cientos de contenedores efímeros | Hay que instrumentar desde el diseño | Tracing, Correlation ID, logs estructurados |
| Perímetro de red = seguridad | No hay perímetro claro | Verificar cada petición | Zero Trust, mTLS, Workload Identity |

### Pets vs Cattle (mascotas vs ganado)

Analogía popularizada por Randy Bias (alrededor de 2012), basada en una charla de Bill Baker (Microsoft):

- **Pets (mascotas):** servidores con nombre propio, cuidados a mano; si enferman, se curan. Ejemplo: `SRV-SQL-PROD-01`, configurado manualmente durante años.
- **Cattle (ganado):** instancias idénticas, numeradas; si una enferma, **se reemplaza**. Ejemplo: `pedidos-7f9c4-xk2lp`, un Pod de Kubernetes.

Cloud native trata la infraestructura como ganado. Eso sólo es posible si la aplicación no depende de una instancia concreta.

### Infraestructura inmutable

En lugar de **modificar** un servidor en ejecución (instalar parches, cambiar ficheros), se **construye una imagen nueva** y se reemplaza la instancia antigua.

```
Mutable:    servidor v1 ──parche──► servidor v1.1 ──parche──► ¿qué tiene realmente? (deriva de configuración)
Inmutable:  imagen v1 ──► instancias v1   |   imagen v2 ──► instancias v2 (las v1 se destruyen)
```

**Ventajas:** reproducibilidad, rollback trivial (volver a la imagen anterior), sin *configuration drift*.
**Coste:** necesitas pipeline de build y despliegue automatizado; un parche urgente requiere reconstruir la imagen.

---

## 3. Las 8 falacias de la computación distribuida

Lista atribuida a L. Peter Deutsch y otros en Sun Microsystems (1994; la octava la añadió James Gosling en 1997). Son **supuestos falsos** que los programadores hacen de forma inconsciente. Casi todos los patrones de resiliencia y comunicación existen para corregir alguno.

| # | Falacia | Realidad | Patrón que lo mitiga |
|---|---|---|---|
| 1 | La red es fiable | Los paquetes se pierden, las conexiones se cortan | Retry, Circuit Breaker, mensajería persistente |
| 2 | La latencia es cero | Cada salto añade milisegundos; las cadenas de llamadas se acumulan | Cache-Aside, BFF/Aggregation, evitar *chatty I/O* |
| 3 | El ancho de banda es infinito | Mover datos cuesta tiempo y dinero | Claim Check, paginación, compresión |
| 4 | La red es segura | Cualquier tráfico puede ser interceptado | mTLS, Zero Trust |
| 5 | La topología no cambia | Las IPs y las instancias cambian constantemente | Service Discovery |
| 6 | Hay un único administrador | Varios equipos, proveedores y políticas | Contratos de API, Anti-Corruption Layer |
| 7 | El coste de transporte es cero | Serializar, cifrar y transmitir cuesta CPU y dinero (*egress*) | Diseño de granularidad, gRPC |
| 8 | La red es homogénea | Distintos protocolos, versiones y plataformas | Adapter, Gateway, contratos versionados |

> [!tip] Úsalas como checklist
> Antes de aprobar un diseño, recorre las 8 falacias y pregunta: "¿qué pasa si esto es falso aquí?".

---

## 4. The Twelve-Factor App

Metodología publicada por Adam Wiggins (Heroku) en 2011: https://12factor.net. Sigue siendo la base de los principios de cualquier app apta para contenedores.

| # | Factor | Regla | Cómo se aplica hoy (.NET / K8s) | Antipatrón que evita |
|---|---|---|---|---|
| I | **Codebase** | Un repositorio por app, muchos despliegues | Un repo → una imagen → dev/staging/prod | Ramas distintas por entorno |
| II | **Dependencies** | Declarar y aislar dependencias explícitamente | `.csproj` + NuGet; imagen con todo lo necesario | "En mi máquina funciona" |
| III | **Config** | La configuración en el entorno, no en el código | Variables de entorno, ConfigMap, Secret, Azure App Configuration | Cadenas de conexión en `appsettings.Production.json` dentro del repo |
| IV | **Backing services** | BD, colas, cachés son recursos adjuntos, intercambiables por configuración | Cambiar Redis local por Azure Cache for Redis sólo cambiando la URL | Código acoplado a una instancia concreta |
| V | **Build, release, run** | Separar estrictamente construir, combinar con config y ejecutar | CI construye la imagen; CD la despliega con config del entorno | Compilar en el servidor de producción |
| VI | **Processes** | Procesos *stateless* y *share-nothing* | Sesión en Redis, ficheros en Blob Storage | Sesión en memoria, ficheros en disco local |
| VII | **Port binding** | La app se expone por un puerto propio | Kestrel escucha en `:8080` | Depender de un IIS preinstalado |
| VIII | **Concurrency** | Escalar con más procesos | Más réplicas (HPA), no más hilos en una VM | Escalar sólo verticalmente |
| IX | **Disposability** | Arranque rápido, apagado elegante | Manejar `SIGTERM`, `IHostApplicationLifetime`, readiness probe | Perder peticiones en cada despliegue |
| X | **Dev/prod parity** | Entornos lo más parecidos posible | Docker Compose con las mismas dependencias que prod | SQLite en dev, SQL Server en prod |
| XI | **Logs** | Logs como flujo de eventos a `stdout` | Serilog/`ILogger` a consola en JSON; el clúster los recoge | Escribir a ficheros rotados dentro del contenedor |
| XII | **Admin processes** | Tareas administrativas como procesos puntuales | Migraciones como Kubernetes `Job` o paso del pipeline | Migrar la BD al arrancar cada réplica |

### Ejemplo: factores III, VI, IX y XI en ASP.NET Core

```csharp
var builder = WebApplication.CreateBuilder(args);

// III - Config: las variables de entorno sobrescriben appsettings.json
// (ConnectionStrings__Pedidos=... en el contenedor). Es el comportamiento por defecto.
var connStr = builder.Configuration.GetConnectionString("Pedidos");

// VI - Stateless: la caché/sesión vive fuera del proceso
builder.Services.AddStackExchangeRedisCache(o =>
    o.Configuration = builder.Configuration["Redis:ConnectionString"]);

// XI - Logs a stdout en formato estructurado
builder.Logging.ClearProviders();
builder.Logging.AddJsonConsole();

// IX - Disposability: tiempo para terminar peticiones en curso tras SIGTERM
builder.Services.Configure<HostOptions>(o => o.ShutdownTimeout = TimeSpan.FromSeconds(25));

builder.Services.AddHealthChecks();
var app = builder.Build();
app.MapHealthChecks("/health/ready");
app.Run();
```

> [!warning] Factor XII y las migraciones
> Ejecutar `dbContext.Database.Migrate()` al arrancar parece cómodo, pero con 5 réplicas arrancando a la vez tienes 5 procesos intentando migrar en paralelo. Mejor: un `Job` de Kubernetes, un *init container* con control de concurrencia, o un paso del pipeline.

---

## 5. Más allá de los 12 factores

Kevin Hoffman, en *Beyond the Twelve-Factor App* (O'Reilly, 2016), propuso ampliar la lista a 15. Los tres añadidos son relevantes hoy:

| Factor añadido | Qué pide |
|---|---|
| **API first** | Diseñar el contrato (OpenAPI, protobuf) antes de implementar; permite que equipos trabajen en paralelo |
| **Telemetry** | Métricas, trazas y health checks desde el diseño, no como añadido. Ver [[CN 10 - Observabilidad]] |
| **Authentication & Authorization** | Seguridad desde el inicio en cada servicio. Ver [[CN 09 - Seguridad]] |

Hoffman también renombra y reordena algunos factores originales (p. ej. "Dev/prod parity" pasa a "Environment parity" y "Config" a "Configuration, credentials and code"). Lo importante no es el número exacto, sino las ideas.

---

## 6. Principios de diseño cloud native

Estos principios, recogidos de las guías de arquitectura de Microsoft (*Azure Well-Architected Framework*), AWS y Google, son el "por qué" detrás de los patrones:

1. **Diseña para el fallo** (*design for failure*). Asume que cada dependencia caerá; decide qué hacer cuando ocurra.
2. **Haz que todo sea redundante**. Nada de puntos únicos de fallo: varias instancias, varias zonas.
3. **Minimiza la coordinación**. La coordinación (bloqueos, transacciones distribuidas, llamadas síncronas en cadena) limita la escalabilidad. Prefiere asincronía y consistencia eventual donde el negocio lo admita.
4. **Escala horizontalmente** (*scale out*, no *scale up*).
5. **Particiona** para superar límites (de una BD, de una cola, de una región).
6. **Diseña para la operación**: observabilidad, automatización, configuración externa.
7. **Usa servicios gestionados** cuando sea posible (PaaS sobre IaaS): delegar la operación de la base de datos o del broker suele ser más barato que operarlo tú.
8. **Diseña para evolucionar**: contratos versionados, servicios débilmente acoplados, despliegues independientes.
9. **Seguridad por defecto**: mínimo privilegio, sin secretos en el código, cifrado en tránsito.

### Los cinco pilares del Well-Architected Framework

Microsoft (Azure WAF) y AWS usan pilares muy parecidos para evaluar una arquitectura. Cada patrón de esta carpeta mejora uno o varios, normalmente a costa de otro:

| Pilar | Pregunta | Patrones típicos |
|---|---|---|
| **Fiabilidad** (*Reliability*) | ¿Sigue funcionando cuando algo falla? | Retry, Circuit Breaker, Bulkhead, Geode |
| **Seguridad** (*Security*) | ¿Está protegido? | Gatekeeper, Valet Key, Federated Identity |
| **Optimización de costes** (*Cost Optimization*) | ¿Pago sólo lo necesario? | Autoscaling, Compute Resource Consolidation, Static Content Hosting |
| **Excelencia operativa** (*Operational Excellence*) | ¿Puedo desplegar y operar con confianza? | GitOps, Blue-Green, Health Endpoint Monitoring |
| **Eficiencia del rendimiento** (*Performance Efficiency*) | ¿Responde bien bajo carga? | Cache-Aside, CQRS, Sharding, CDN |

AWS añade un sexto pilar, **Sostenibilidad** (2021).

---

## 7. Qué es (y qué no es) un patrón

Un **patrón** es una **solución probada a un problema recurrente en un contexto determinado**. La definición viene de Christopher Alexander (arquitectura de edificios, 1977) y la popularizó en software el libro *Design Patterns* (GoF, 1994).

Tres ideas que evitan malos usos:

- **Un patrón tiene contexto.** Saga resuelve consistencia entre servicios con bases de datos separadas; si tienes una sola base de datos, una transacción local es mejor.
- **Un patrón tiene coste.** Siempre añade complejidad. Si el problema no existe aún, el patrón es sobreingeniería.
- **Los patrones se combinan.** Casi nunca aparecen solos: Outbox + Inbox + Idempotencia; Retry + Circuit Breaker + Timeout; CQRS + Materialized View.

> [!tip] Regla práctica
> Para cada patrón que propongas, deberías poder decir: **"sin él, pasaría X; con él, pagamos Y; X es peor que Y porque Z"**. Si no puedes completar la frase, probablemente no lo necesitas todavía.

### Diferencia con los patrones GoF

| | Patrones GoF (Strategy, Factory…) | Patrones cloud native |
|---|---|---|
| Nivel | Código, dentro de un proceso | Arquitectura, entre procesos y máquinas |
| Problema | Acoplamiento entre clases | Red, fallos parciales, escala, despliegue |
| Implementación | En tu código | En tu código **o** en la infraestructura (mesh, gateway, broker, Kubernetes) |

Una tendencia importante: muchos patrones cloud native **se han movido a la plataforma**. Hace 10 años implementabas Service Discovery o Circuit Breaker en tu código (Netflix Eureka, Hystrix); hoy Kubernetes y un service mesh los ofrecen sin tocar la aplicación. Conocer el patrón sigue siendo necesario para **configurarlo bien**.

---

## Puntos clave

- Cloud native describe **cómo se diseña** una app, no dónde se despliega.
- Supuestos base: **instancias desechables, red no fiable, cambio constante**.
- Los **12 factores** son el mínimo para que una app funcione bien en contenedores.
- Las **8 falacias** explican por qué existen los patrones de resiliencia y comunicación.
- Todo patrón tiene **contexto y coste**; aplicarlo sin el problema es sobreingeniería.

## Errores comunes

- Llamar "cloud native" a un *lift and shift*.
- Guardar estado en memoria o en disco local del contenedor y luego sorprenderse al escalar.
- Ignorar `SIGTERM`: cada despliegue corta peticiones en curso.
- Adoptar microservicios para "ser cloud native" con un equipo de 4 personas.
- Copiar patrones de Netflix/Google sin tener sus problemas de escala.

## 🎯 Para entrevistas

- *"¿Qué es una aplicación cloud native?"* → Diseñada para entornos dinámicos: débilmente acoplada, resiliente, observable, escalable horizontalmente y desplegada de forma automatizada. Cita la definición CNCF y aclara que no es lo mismo que "estar en la nube".
- *"Nombra 3 de los 12 factores y por qué importan en Kubernetes."* → Config (ConfigMap/Secret), Processes stateless (escalado y reemplazo de Pods), Disposability (SIGTERM, rolling updates).
- *"¿Qué es la infraestructura inmutable?"* → Reemplazar en vez de modificar; rollback = volver a la imagen anterior.
- *"Menciona falacias de la computación distribuida."* → La red es fiable, latencia cero, topología fija… y qué patrón mitiga cada una.

## Referencias

- CNCF Cloud Native Definition: https://github.com/cncf/toc/blob/main/DEFINITION.md
- The Twelve-Factor App: https://12factor.net
- Microsoft, *Azure Well-Architected Framework*: https://learn.microsoft.com/azure/well-architected/
- Microsoft, *Architecting Cloud Native .NET Applications for Azure* (e-book): https://learn.microsoft.com/dotnet/architecture/cloud-native/
- Kevin Hoffman, *Beyond the Twelve-Factor App* (O'Reilly, 2016).

---
⬅️ [[CN 00 - Patrones Cloud Native - Índice|Índice]] · ➡️ [[CN 02 - Patrones de contenedores y Kubernetes]]
