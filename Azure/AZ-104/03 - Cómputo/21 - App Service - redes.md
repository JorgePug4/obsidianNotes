---
tags: [az-104, azure, computo, app-service, redes, vnet]
modulo: Cómputo
peso_examen: Alto
---

# App Service · redes

## ¿Qué es?

Las opciones para controlar **quién puede llamar a la app** (tráfico de entrada) y **a qué puede llegar la app** (tráfico de salida), integrándola con redes virtuales.

## Conceptos clave

### Entrada (inbound) 🧠

| Opción | Qué hace | Nota |
|---|---|---|
| **Restricciones de acceso (access restrictions)** | Lista ordenada de reglas **Allow/Deny** por **IP/CIDR**, **etiqueta de servicio** o **subred (service endpoint)** con prioridad | Se aplica al sitio principal y, por separado, al sitio **SCM/Kudu** |
| **Private endpoint** | IP privada en tu VNet para la app; deshabilita el acceso público si se configura así | Requiere **Basic o superior** (PaaS); usa zona DNS `privatelink.azurewebsites.net` |
| **Service endpoints** | Permitir solo una subred concreta | Alternativa ligera al private endpoint |
| **App Service Environment (ASE)** | Despliegue aislado dentro de tu VNet | Nivel **Isolated** |

### Salida (outbound) 🧠

| Opción | Qué hace | Nota |
|---|---|---|
| **Integración con red virtual (VNet integration)** | La app envía tráfico a una **subred delegada** (`Microsoft.Web/serverFarms`) para alcanzar recursos privados | Requiere **Basic/Standard o superior** (planes dedicados); una subred por plan |
| **Enrutar todo el tráfico saliente** (`WEBSITE_VNET_ROUTE_ALL` / opción del portal) | Envía **todo** el tráfico saliente por la VNet (no solo RFC 1918) | Necesario para NAT Gateway, firewall o forzar rutas |
| **Direcciones IP de salida** | Conjunto de IPs compartidas del plan | Cambian al escalar entre niveles; usar **NAT Gateway** o ASE para IP fija |
| **Hybrid Connections** ➕ | Acceso a un host:puerto on-premises con un agente | Sin VPN |

### Otros
- **Sitio SCM/Kudu** (`<app>.scm.azurewebsites.net`): tiene sus **propias restricciones de acceso**; se puede heredar la configuración del sitio principal.
- **HTTPS Only** y **certificados de cliente** también son control de entrada.
- El **private endpoint** no cubre el acceso saliente; se combina con VNet integration.
- **DNS**: con VNet integration, la app usa el DNS de la VNet si se configura (`WEBSITE_DNS_SERVER`).

## Cómo funciona

```
          Internet ──► [Restricciones de acceso] ──► App Service ──► [VNet integration] ──► subred delegada ──► SQL/Storage/VM privados
                       o [Private endpoint] (IP privada en la VNet)
```

```bash
# Restricciones de acceso
az webapp config access-restriction add -g rg-web -n app-contoso --rule-name "oficina" --action Allow --ip-address 203.0.113.0/24 --priority 100
az webapp config access-restriction add -g rg-web -n app-contoso --rule-name "solo-agw" --action Allow --service-tag AzureFrontDoor.Backend --priority 200
az webapp config access-restriction show -g rg-web -n app-contoso
# Integración con VNet (subred delegada)
az network vnet subnet create -g rg-web --vnet-name vnet-web -n snet-appsvc --address-prefixes 10.0.3.0/27 --delegations Microsoft.Web/serverFarms
az webapp vnet-integration add -g rg-web -n app-contoso --vnet vnet-web --subnet snet-appsvc
az webapp config appsettings set -g rg-web -n app-contoso --settings WEBSITE_VNET_ROUTE_ALL=1
# Private endpoint
az network private-endpoint create -g rg-web -n pe-app --vnet-name vnet-web --subnet snet-pe \
  --private-connection-resource-id $(az webapp show -g rg-web -n app-contoso --query id -o tsv) \
  --group-id sites --connection-name app-conn
# Ver IPs de salida
az webapp show -g rg-web -n app-contoso --query "{out:outboundIpAddresses,possible:possibleOutboundIpAddresses}"
```

Portal: Web App → **Redes** → *Tráfico entrante*: Restricciones de acceso, Puntos de conexión privados; *Tráfico saliente*: Integración de red virtual, Hybrid Connections.

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| La app solo debe ser accesible desde la oficina | **Restricción de acceso** Allow con el CIDR público de la oficina y Deny implícito |
| Solo se debe acceder a través de Front Door / Application Gateway | Restricción con **etiqueta de servicio** (`AzureFrontDoor.Backend`) o la IP del gateway |
| La app no debe tener ningún endpoint público | **Private endpoint** + deshabilitar acceso público |
| La app debe consultar una base de datos con private endpoint | **VNet integration** (salida) |
| Todo el tráfico saliente debe pasar por el firewall corporativo | VNet integration + **Route All** + UDR hacia la NVA/Azure Firewall |
| IP de salida fija para una lista blanca de un tercero | **NAT Gateway** en la subred de integración (o ASE) |
| Proteger también Kudu/SCM | Configurar restricciones del **sitio SCM** |
| Aislamiento completo en mi VNet | **App Service Environment** (Isolated) |

> [!warning] Confusión clásica
> **VNet integration = salida** (la app llama a recursos privados). **Private endpoint = entrada** (los clientes llegan a la app por IP privada). Muchas preguntas se resuelven identificando la dirección del tráfico.

## Ejemplo

Una API en App Service debe consumir una base de datos Azure SQL con private endpoint y solo debe ser accesible desde una Application Gateway. Solución: **VNet integration** con una subred delegada `/27` (para la salida hacia SQL) y **restricciones de acceso** que permitan únicamente la IP privada/pública de la Application Gateway, con la regla Deny implícita para el resto. El sitio SCM se restringe a la IP de la oficina.

## Comparaciones

| Función | Dirección | Nivel mínimo | Cuándo utilizarla |
|---|---|---|---|
| **Restricciones de acceso** | Entrada | Todos (incluidos Free) | Filtrado por IP o etiqueta de servicio |
| **Private endpoint** | Entrada | Basic+ | Acceso privado, sin exposición pública |
| **Service endpoint** | Entrada | Basic+ | Permitir una subred concreta |
| **VNet integration** | Salida | Basic+ | Alcanzar recursos privados |
| **Hybrid Connections** | Salida | Basic+ | Un host:puerto on-premises sin VPN |
| **ASE (Isolated)** | Ambas | Isolated | Aislamiento total y escala |

## AZ-104 Exam Tips

- 🔥 📌 **VNet integration = salida; Private endpoint = entrada.**
- 🔥 🧠 La subred de VNet integration debe estar **delegada a `Microsoft.Web/serverFarms`** y ser exclusiva del plan.
- 🧠 Las **restricciones de acceso** son reglas ordenadas por **prioridad** con Deny implícito al final si hay alguna Allow.
- 🧠 El sitio **SCM/Kudu** tiene reglas propias.
- 🧠 Las **IPs de salida** son compartidas y cambian al cambiar de nivel: para IP fija, NAT Gateway o ASE.
- 💻 `az webapp config access-restriction add`, `az webapp vnet-integration add`, private endpoint.
- ⚠️ `WEBSITE_VNET_ROUTE_ALL=1` para enviar **todo** el tráfico saliente por la VNet.

## Errores comunes

- Usar VNet integration esperando que oculte la app de Internet.
- Olvidar la delegación de la subred.
- Proteger el sitio principal y dejar Kudu abierto.

## Preguntas que podrían aparecer

**1.** Una Web App debe conectarse a una base de datos que solo acepta tráfico desde una red virtual. ¿Qué configuras?
- A) Private endpoint para la app · B) Integración con red virtual (VNet integration) · C) Restricciones de acceso · D) Hybrid Connections

<details><summary>Respuesta</summary>

**B.** La integración con VNet dirige el tráfico **saliente** de la app a la red virtual.
</details>

**2.** Necesitas que la aplicación solo sea accesible mediante una dirección IP privada de tu red virtual, sin endpoint público. ¿Qué implementas?
- A) VNet integration · B) Restricciones de acceso por IP · C) Private endpoint · D) Service endpoint

<details><summary>Respuesta</summary>

**C.** El private endpoint da a la app una IP privada y permite deshabilitar el acceso público.
</details>

**3.** Tras aplicar restricciones de acceso al sitio principal, un atacante podría seguir accediendo a la consola de Kudu. ¿Qué falta?
- A) Habilitar HTTPS Only · B) Configurar las restricciones de acceso del sitio SCM · C) Cambiar el plan · D) Activar Always On

<details><summary>Respuesta</summary>

**B.** El sitio SCM/Kudu tiene su propio conjunto de reglas, que puede heredar las del sitio principal.
</details>

## Relacionado

- [[18 - Azure App Service - creación y configuración]]
- [[10 - Private Endpoint y Private Link]]
- [[09 - Service Endpoints]]
- [[04 - Rutas definidas por el usuario (UDR) y NVA]]
- [[00 - Índice - Cómputo]]
