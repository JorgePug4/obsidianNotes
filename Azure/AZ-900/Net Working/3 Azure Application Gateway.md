## ¿Qué es?

Un **balanceador de carga de capa 7 (HTTP/HTTPS)**, entiende el contenido de la petición web (URL, headers, cookies) y enruta según eso.

## Conceptos fundamentales

- **Listener**: escucha en un puerto/protocolo específico (HTTP/HTTPS).
- **Routing Rule**: decide a qué backend mandar según la regla (path-based o básica).
- **Backend Pool**: destino final (VMs, VMSS, App Service, IPs).
- **WAF (Web Application Firewall)**: SKU que agrega protección contra OWASP Top 10 (SQL injection, XSS, etc.).
- **SSL Termination**: descifra HTTPS en el Gateway, así las instancias backend no cargan con ese trabajo.
- **Cookie-based session affinity**: mantiene a un cliente conectado a la misma instancia backend.

## ¿Cómo funciona?

1. El cliente hace una petición HTTPS.
2. El listener la recibe y hace **SSL termination** (opcional).
3. La routing rule inspecciona la URL (ej. `/imagenes/*` → backend pool A, `/api/*` → backend pool B).
4. Envía la petición al backend correspondiente.

## Puntos importantes

- Capa 7 = **entiende contenido web**, a diferencia de Load Balancer (capa 4).
- **Path-based routing**: enruta según la ruta de la URL, algo que un Load Balancer normal no puede hacer.
- Con **WAF SKU**, protege contra ataques comunes sin tocar el código de la app.
- Autoescala según el SKU (v2).

## Ejemplo práctico

```
/tienda/*  → Backend Pool "eCommerce"
/blog/*    → Backend Pool "CMS"
```

Un solo Application Gateway enruta a dos aplicaciones distintas según la URL, algo imposible con Load Balancer básico.

## Comparación clave (cae seguido en exámenes/entrevistas)

||Load Balancer|Application Gateway|
|---|---|---|
|Capa OSI|4 (TCP/UDP)|7 (HTTP/HTTPS)|
|Enruta por URL|No|Sí|
|SSL Termination|No|Sí|
|WAF|No|Sí (SKU WAF)|
|Alcance|Regional|Regional|

## Ventajas / Desventajas

|Ventajas|Desventajas|
|---|---|
|Enrutamiento inteligente por contenido|Más caro que Load Balancer|
|WAF integrado|Solo protocolos web (HTTP/HTTPS/WebSocket)|
|SSL offloading|Mayor latencia que capa 4|

## Errores comunes

- Usarlo pensando que sirve para tráfico no-web (TCP genérico) — para eso es Load Balancer.
- Confundir Application Gateway (regional) con **Front Door** (global, capa 7 también, pero para múltiples regiones).
- No habilitar WAF pensando que el Gateway ya protege contra ataques por defecto.

## Buenas prácticas

- Usa **WAF SKU** si expones aplicaciones públicas críticas.
- Combina con **Front Door** cuando necesites failover/enrutamiento global entre regiones.
- Usa path-based routing para arquitecturas de microservicios detrás de un solo dominio.