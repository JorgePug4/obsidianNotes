---
tags: [az-104, azure, redes, load-balancer, troubleshooting]
modulo: Redes virtuales
peso_examen: Alto
---

# Solución de problemas de balanceo de carga

## ¿Qué es?

Metodología para diagnosticar por qué un balanceador no distribuye tráfico: backend no saludable, conexiones rechazadas, SNAT agotado o configuración incompatible. Es un objetivo explícito del temario ("solucionar problemas de equilibrio de carga").

## Checklist de diagnóstico 🧠

```
1. ¿La sonda está sana?         → Métrica "Health probe status"; Insights del LB
2. ¿El NSG permite la sonda?    → Allow desde AzureLoadBalancer (168.63.129.16)
3. ¿La app escucha en el puerto de la sonda y responde 200? → probar en local (curl localhost)
4. ¿La regla usa el puerto correcto y el pool correcto?
5. ¿Las VMs están en el backend pool y encendidas?
6. ¿Coinciden los SKUs (LB Standard + IP Standard)?
7. ¿Hay UDR que desvíe el tráfico de retorno?
8. ¿Se agotan los puertos SNAT? → métrica "SNAT connection count" / "Allocated SNAT ports"
9. ¿El firewall del SO bloquea? (Windows Firewall, iptables)
10. ¿Tiempo de espera de inactividad demasiado corto? (4-30 min)
```

## Causas frecuentes y solución

| Síntoma | Causa probable | Solución |
|---|---|---|
| Todo el backend no saludable | NSG bloquea 168.63.129.16 / `AzureLoadBalancer` | Regla Allow desde la etiqueta `AzureLoadBalancer` |
| Sonda falla pero la app responde | Sonda HTTP a una ruta que devuelve 3xx/4xx | La sonda **solo acepta 200 OK**; usar una ruta de salud simple |
| Sonda TCP OK pero los usuarios reciben errores | La app está "escuchando" pero no sirve contenido | Usar sonda **HTTP/HTTPS** en lugar de TCP |
| Una sola VM recibe todo | Distribución **Client IP** activada o pruebas desde una sola IP | Revisar `load-distribution`; probar desde varios orígenes |
| No se puede añadir una VM al pool | SKU incompatible o VM en otra VNet/región | Igualar SKUs; el backend debe estar en la misma VNet (salvo cross-region) |
| Conexiones cortadas a los 4 minutos | **Idle timeout** por defecto | Subir el tiempo de espera (hasta 30 min) o usar keep-alives |
| Errores intermitentes de salida a Internet | **Agotamiento de puertos SNAT** | Reglas de salida con más puertos, **NAT Gateway**, o reducir conexiones |
| Tras añadir una UDR deja de funcionar | El tráfico de retorno se desvía a una NVA | Excluir el tráfico del LB o ajustar la UDR |
| El LB interno no responde desde on-premises | Falta ruta o la regla no tiene HA Ports | Revisar rutas y configuración del LB interno |
| La app solo funciona con Floating IP | Configuraciones tipo SQL AlwaysOn | Habilitar **Floating IP (Direct Server Return)** y configurar la IP del listener en el SO |

## Herramientas 🧠

| Herramienta | Uso |
|---|---|
| **Métricas del LB** | *Health probe status*, *Data path availability*, *SNAT connection count*, *Byte count* |
| **Insights del Load Balancer** (Azure Monitor) | Vista topológica con estado de sondas y flujos |
| **Resource Health** | Estado del recurso balanceador |
| **IP flow verify** (Network Watcher) | Comprobar si un NSG bloquea |
| **Connection troubleshoot / Connection Monitor** | Prueba extremo a extremo |
| **Effective security rules / routes** | Ver reglas y rutas aplicadas a la NIC |
| **VNet flow logs** | Flujos permitidos y denegados |
| **`curl`/`Test-NetConnection` dentro de la VM** | Verificar que la app escucha |

```bash
az monitor metrics list --resource $(az network lb show -g rg-net -n lb-web --query id -o tsv) --metric DipAvailability --interval PT1M -o table
az network watcher test-ip-flow -g rg-net --vm vm-web01 --direction Inbound --protocol TCP --local 10.0.1.4:80 --remote 168.63.129.16:60000
az network lb probe show -g rg-net --lb-name lb-web -n probe-http
az network lb address-pool show -g rg-net --lb-name lb-web -n bp-web --query backendIPConfigurations
```

## Ejemplo

Tras desplegar un LB Standard, la métrica *Health probe status* muestra 0 %. La sonda es HTTP en `/status`, pero la aplicación devuelve 302 hacia el login. Se cambia la sonda a una ruta `/health` que devuelve **200 OK** sin redirección y el backend pasa a saludable. Además, se añade al NSG una regla Allow desde `AzureLoadBalancer` que faltaba.

## Comparación de sondas

| Tipo de sonda | Comprueba | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **TCP** | Handshake en el puerto | Simple, cualquier protocolo | Servicios no HTTP |
| **HTTP** | Respuesta **200 OK** a una ruta | Detecta apps caídas aunque el puerto escuche | Aplicaciones web |
| **HTTPS** (Standard) | Igual con TLS | Extremo cifrado | Apps solo HTTPS |

## AZ-104 Exam Tips

- 🔥 🧠 Las sondas vienen de **168.63.129.16** y exigen `AzureLoadBalancer` permitido en el NSG.
- 🔥 🧠 Una sonda **HTTP solo considera sano un 200 OK**.
- 🧠 **Idle timeout** por defecto **4 minutos** (hasta 30).
- 🧠 Agotamiento de **puertos SNAT** → NAT Gateway o reglas de salida.
- 🧠 Los **SKUs deben coincidir** (LB e IP pública).
- 💻 Métricas *DipAvailability* / *Health probe status*, IP flow verify, Insights del LB.
- 📌 Problema de **filtrado** (NSG) vs **enrutamiento** (UDR) vs **aplicación** (no escucha).

## Errores comunes

- Apuntar la sonda a una ruta que redirige.
- Diagnosticar en el LB cuando el problema es el firewall del sistema operativo.
- Olvidar que las conexiones existentes pueden mantenerse aunque la sonda falle.

## Preguntas que podrían aparecer

**1.** El estado de la sonda HTTP es 0 % pero la aplicación responde correctamente en el navegador desde dentro de la VM. ¿Qué revisas primero?
- A) El SKU del balanceador · B) Que el NSG permita el origen AzureLoadBalancer y que la ruta de la sonda devuelva 200 OK · C) El idle timeout · D) Las reglas de salida

<details><summary>Respuesta</summary>

**B.** Las dos causas más habituales son el bloqueo de la sonda en el NSG y una ruta que no devuelve 200.
</details>

**2.** Una aplicación sufre errores intermitentes al abrir muchas conexiones salientes a Internet desde las VMs del backend. ¿Qué implementas?
- A) Una sonda TCP · B) Un NAT Gateway o reglas de salida con más puertos SNAT · C) Client IP affinity · D) Floating IP

<details><summary>Respuesta</summary>

**B.** El síntoma corresponde al agotamiento de puertos SNAT.
</details>

## Relacionado

- [[13 - Azure Load Balancer]]
- [[15 - Solución de problemas de conectividad de red]]
- [[07 - Reglas de seguridad efectivas]]
- [[07 - Network Watcher y Connection Monitor]]
- [[00 - Índice - Redes virtuales]]
