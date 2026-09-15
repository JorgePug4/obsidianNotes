---
tags: [az-900, azure, redes, subredes, cidr]
modulo: Redes
peso_examen: Medio
---

# Espacio de direcciones y subredes

## Concepto

El **espacio de direcciones** es el rango de IPs privadas que asignas a una [[01 - Azure Virtual Network]]. Se escribe en notación **CIDR**, por ejemplo `10.0.0.0/16`. Después divides ese rango en **subredes** más pequeñas, por ejemplo `10.0.1.0/24`.

**Problema que resuelve:** cada recurso necesita una IP única dentro de la red, y necesitas agrupar recursos por función para aplicar seguridad y organización.

**Para qué se utiliza:**
- Definir cuántas IPs tendrá disponibles tu VNet.
- Separar recursos por capas (frontend, backend, datos).
- Aplicar reglas de seguridad distintas a cada grupo.

## Características principales

- **Rangos privados (RFC 1918):** Azure recomienda usar `10.0.0.0/8`, `172.16.0.0/12` o `192.168.0.0/16`.
- **Notación CIDR:** el número tras la barra indica cuántos bits son fijos. Cuanto **menor** el número, **más** direcciones.
  - `/16` = 65.536 direcciones
  - `/24` = 256 direcciones
  - `/29` = 8 direcciones (el mínimo permitido en Azure)
- **Azure reserva 5 IPs en cada subred:** la primera (dirección de red), la segunda (gateway), la tercera y cuarta (DNS de Azure) y la última (broadcast). Un `/24` te da **251** IPs usables, no 256.
- **Una subred pertenece a una sola VNet** y su rango debe estar contenido en el espacio de direcciones de la VNet.
- **Las subredes de una misma VNet no pueden solaparse.**
- **Los rangos de dos VNets no deben solaparse** si quieres emparejarlas o conectarlas con VPN.
- Los recursos reciben una **IP privada** de su subred. Solo tienen **IP pública** si se la asignas.

> [!warning] Confusiones frecuentes
> - **IP privada vs. IP pública:** la privada solo es válida dentro de la VNet (y redes conectadas). La pública es alcanzable desde Internet y es un recurso separado que asignas a una NIC o a un balanceador.
> - **/16 vs /24:** el examen puede preguntar cuál ofrece más direcciones. **/16 tiene más** (65.536) que /24 (256).
> - **Subred ≠ VNet:** el NSG se puede asociar a una subred o a una NIC, nunca a la VNet completa.

## Casos de uso

- Una VNet `10.0.0.0/16` dividida en:
  - `10.0.1.0/24` → subred **web** (servidores públicos)
  - `10.0.2.0/24` → subred **app**
  - `10.0.3.0/24` → subred **datos** (sin acceso desde Internet)
- Una empresa ya usa `10.0.0.0/16` en su oficina. Para conectarla con Azure por VPN, crea la VNet con `10.1.0.0/16` y evita el solapamiento.

## Comparaciones

| Concepto | Qué es | Alcance |
|---|---|---|
| **Espacio de direcciones** | Rango total de la VNet (`10.0.0.0/16`) | Toda la VNet |
| **Subred** | Porción del espacio (`10.0.1.0/24`) | Grupo de recursos dentro de la VNet |
| **IP privada** | Dirección de un recurso dentro de la subred | Solo dentro de la VNet y redes conectadas |
| **IP pública** | Recurso independiente que expone algo a Internet | Internet |

## Conceptos que debo memorizar

> [!important]
> - CIDR: **menor sufijo = más IPs**. `/16` > `/24` > `/29`.
> - Azure reserva **5 IPs por subred**.
> - Subred mínima en Azure: **/29**.
> - Los rangos **no pueden solaparse**, ni entre subredes ni entre VNets que se vayan a conectar.
> - Por defecto un recurso tiene **solo IP privada**.

## Tips para AZ-900

> [!tip] Nivel de profundidad
> AZ-900 **no** pide calcular subredes ni máscaras. Basta con entender qué es CIDR, que `/16` es mayor que `/24` y que hay 5 IPs reservadas. Los cálculos detallados son de AZ-104 y AZ-700.

> [!tip] Palabras clave
> "rango de direcciones", "CIDR", "segmentar", "solapamiento", "IP privada".

> [!tip] Pregunta trampa
> "Una VNet con espacio `10.0.0.0/16` puede tener una subred `10.1.0.0/24`" → **Falso**. `10.1.x.x` está fuera del rango `10.0.x.x`.

## Ejemplo de pregunta de examen

**Pregunta 1.** ¿Cuál de estos rangos CIDR proporciona más direcciones IP?

- A) 10.0.0.0/16
- B) 10.0.0.0/24
- C) 10.0.0.0/28
- D) 10.0.0.0/29

**Respuesta correcta: A.** `/16` ofrece 65.536 direcciones.
- B da 256, C da 16 y D da 8. A menor número tras la barra, más direcciones.

**Pregunta 2.** Quieres emparejar (peering) dos VNets. ¿Qué requisito debe cumplirse?

- A) Ambas deben estar en la misma región.
- B) Sus espacios de direcciones no deben solaparse.
- C) Ambas deben tener una IP pública.
- D) Deben pertenecer al mismo grupo de recursos.

**Respuesta correcta: B.** El solapamiento impide el enrutamiento entre ellas.
- A es incorrecta: existe el peering global entre regiones.
- C es incorrecta: el peering usa la red privada de Microsoft.
- D es incorrecta: el peering funciona entre grupos de recursos e incluso suscripciones distintas.

**Pregunta 3.** Una máquina virtual se crea en una subred sin configuración adicional. ¿Qué tipo de dirección IP recibe automáticamente?

- A) Solo una IP pública
- B) Solo una IP privada
- C) Una IP pública y una privada
- D) Ninguna hasta que se configure el NSG

**Respuesta correcta: B.** Todo recurso en una subred recibe una IP privada de forma automática.
- A y C son incorrectas: la IP pública es un recurso opcional que hay que asignar.
- D es incorrecta: el NSG filtra tráfico, no asigna direcciones.

## 🧠 Resumen para el examen

1. El espacio de direcciones se define en **CIDR** con rangos privados.
2. **Menor número tras la barra = más IPs.**
3. Azure reserva **5 IPs** en cada subred.
4. Subred mínima: **/29**.
5. Las subredes deben caber dentro del espacio de la VNet y **no solaparse**.
6. Dos VNets con rangos solapados **no pueden emparejarse**.
7. Un recurso recibe **IP privada automática**; la pública es opcional.
