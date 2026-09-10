## ¿Qué es?

Red de servidores distribuidos globalmente (**edge nodes / PoPs**) que **cachean contenido estático** cerca del usuario final, reduciendo latencia.

## Conceptos fundamentales

- **Edge node / PoP (Point of Presence)**: servidor físico distribuido geográficamente que sirve el contenido cacheado.
- **Origin**: la fuente real del contenido (Storage Account, App Service, Web App).
- **Caching rules**: definen cuánto tiempo se guarda el contenido en el edge (TTL).
- **Purge**: fuerza a limpiar la caché antes de que expire el TTL.

## ¿Cómo funciona?

1. El usuario pide un archivo (ej. una imagen).
2. Si el edge node más cercano ya lo tiene cacheado → lo entrega directo (**cache hit**), rápido.
3. Si no lo tiene (**cache miss**) → lo pide al origen, lo entrega al usuario, y lo guarda en caché para la próxima vez.

## Puntos importantes

- Ideal para **contenido estático**: imágenes, CSS, JS, videos, descargas.
- **No sirve para contenido dinámico personalizado** por usuario (aunque hay configuraciones avanzadas para ciertos casos).
- Reduce carga en el origen y mejora la experiencia para usuarios geográficamente distantes.
- Puede combinarse con HTTPS personalizado y compresión automática.

## Ejemplo práctico

Una tienda online con clientes en Europa y Asia, pero su Storage Account está en East US. Sin CDN, un usuario en Tokio tiene alta latencia al cargar imágenes. Con CDN, la imagen se cachea en un PoP cercano a Tokio y carga mucho más rápido.

## Ventajas / Desventajas

|Ventajas|Desventajas|
|---|---|
|Reduce latencia globalmente|No apto para contenido altamente dinámico|
|Reduce carga en el origen|Requiere gestión de invalidación de caché (purge)|
|Mejora experiencia de usuario|Costo adicional por transferencia de datos|

## Errores comunes

- Usar CDN para contenido que cambia constantemente sin configurar bien el TTL → usuarios ven contenido desactualizado.
- Olvidar hacer **purge** después de actualizar un archivo crítico en el origen.
- Pensar que CDN sirve como balanceador de carga — no lo es, es una capa de caching/distribución de contenido.

## Buenas prácticas

- Configura TTLs adecuados según qué tan frecuente cambia el contenido.
- Usa versión en el nombre del archivo (`logo.v2.png`) en vez de depender solo de purge.
- Habilita compresión y HTTPS en el perfil del CDN.
