---
tags: [az-104, azure, redes, dns, zonas-publicas]
modulo: Redes virtuales
peso_examen: Alto
---

# Azure DNS (zonas públicas)

## ¿Qué es?

**Azure DNS** hospeda **zonas DNS públicas** en la infraestructura de Microsoft: aloja los registros de tu dominio (`contoso.com`) y responde a las consultas de Internet con alta disponibilidad y baja latencia mediante **anycast**.

⚠️ Azure DNS **no registra dominios** (eso es **App Service Domains** o un registrador externo); solo los **hospeda**.

## ¿Para qué sirve?

- Gestionar los registros DNS del dominio corporativo desde Azure, con RBAC, plantillas y auditoría.
- Publicar servicios (web, correo, verificaciones).

## Conceptos clave

- **Zona DNS**: contenedor de los registros de un dominio. El nombre de la zona debe ser único **dentro del grupo de recursos**.
- **Delegación** 🧠: para que la zona sea autoritativa hay que cambiar los **servidores de nombres (NS)** en el **registrador** del dominio por los 4 que Azure asigna (`ns1-xx.azure-dns.com`, `-dns.net`, `-dns.org`, `-dns.info`).
- **Tipos de registro** 🧠: **A** (IPv4), **AAAA** (IPv6), **CNAME** (alias a otro nombre), **MX** (correo), **TXT** (SPF, verificaciones), **NS**, **SOA** (creados con la zona), **SRV**, **PTR** (zonas inversas), **CAA**.
- **Conjunto de registros (record set)**: varios registros del mismo nombre y tipo (por ejemplo, dos A para `www`).
- **TTL**: por conjunto de registros; valores bajos para cambios frecuentes.
- **Registros alias** 🧠: apuntan directamente a un **recurso de Azure** (IP pública, Traffic Manager, Front Door, CDN, o a otro record set de la misma zona). Ventajas: se actualizan solos si cambia la IP, y permiten usar el **dominio raíz (apex)** apuntando a Traffic Manager/Front Door (donde CNAME no está permitido).
- **Zonas inversas (PTR)** y **zonas hijas** (delegación interna).
- **Límites**: 250 zonas por suscripción (ampliable), 10 000 record sets por zona (ampliable).
- No admite transferencias de zona (AXFR/IXFR) ni DNSSEC en todas las regiones (DNSSEC está disponible en fase reciente; verifica en Learn).
- RBAC: **DNS Zone Contributor** para gestionar zonas y registros.

## Cómo funciona

```
Registrador (GoDaddy, etc.)  ──NS──►  ns1-01.azure-dns.com …  (delegación)
                                        │
                                  Zona contoso.com en Azure DNS
                                    www  A     20.50.1.10
                                    @    ALIAS → IP pública / Front Door
                                    mail MX    10 mail.contoso.com
                                    @    TXT   "v=spf1 include:..."
```

```bash
az network dns zone create -g rg-net -n contoso.com
az network dns zone show -g rg-net -n contoso.com --query nameServers   # NS a poner en el registrador
az network dns record-set a add-record -g rg-net -z contoso.com -n www -a 20.50.1.10
az network dns record-set cname set-record -g rg-net -z contoso.com -n shop -c app-contoso.azurewebsites.net
az network dns record-set mx add-record -g rg-net -z contoso.com -n @ -e mail.contoso.com -p 10
az network dns record-set txt add-record -g rg-net -z contoso.com -n @ -v "v=spf1 include:spf.protection.outlook.com -all"
# Registro alias al recurso de IP pública
az network dns record-set a create -g rg-net -z contoso.com -n @ --target-resource $(az network public-ip show -g rg-net -n pip-web --query id -o tsv)
az network dns record-set list -g rg-net -z contoso.com -o table
az network dns record-set a update -g rg-net -z contoso.com -n www --set ttl=300
```

```powershell
New-AzDnsZone -ResourceGroupName rg-net -Name contoso.com
New-AzDnsRecordSet -ResourceGroupName rg-net -ZoneName contoso.com -Name www -RecordType A -Ttl 3600 `
  -DnsRecords (New-AzDnsRecordConfig -IPv4Address 20.50.1.10)
Get-AzDnsZone -ResourceGroupName rg-net -Name contoso.com | Select-Object -ExpandProperty NameServers
```

## Configuración relevante para el examen

| Escenario | Registro |
|---|---|
| `www` apunta a una IP pública | **A** (o **alias** al recurso de IP pública) |
| Subdominio a una Web App | **CNAME** a `<app>.azurewebsites.net` |
| Dominio raíz a Front Door o Traffic Manager | **Registro alias** (CNAME no está permitido en el apex) |
| Verificar propiedad de dominio para App Service | **TXT** `asuid.<subdominio>` |
| Correo de Microsoft 365 | **MX** + **TXT** (SPF) + **CNAME** (autodiscover) |
| La zona no responde desde Internet | Falta la **delegación NS** en el registrador |
| Cambio de IP frecuente | **Alias** al recurso (se actualiza solo) o TTL bajo |
| Nombres internos privados | **Zona privada** ([[12 - Azure Private DNS y resolución de nombres]]) |

## Ejemplo

Contoso compra `contoso.com` en un registrador externo. Crea la zona en Azure DNS, copia los 4 servidores NS y los configura en el registrador. Publica `www` con un registro **alias** hacia la IP pública del Application Gateway (para que se actualice solo si cambia), el apex con alias hacia Front Door, los MX de Microsoft 365 y los TXT de verificación. La propagación de la delegación tarda hasta 48 horas.

## Comparaciones

| Servicio | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Azure DNS (público)** | Hospedar dominios de Internet | Anycast, RBAC, ARM, alias | Dominios corporativos |
| **Azure Private DNS** | Nombres internos de VNets | Resolución privada, autorregistro | Redes virtuales |
| **App Service Domains** | Comprar dominios | Integración automática con Azure DNS | Comprar y usar en Azure |
| **Traffic Manager** | Enrutamiento por DNS | Failover y geo-enrutamiento | Multi-región |
| **DNS externo (registrador)** | Hospedaje fuera | Ya existente | Cuando no se migra a Azure |

## 💻 Laboratorio: zona pública

1. Crear la zona `<tuapellido>demo.com` (no hace falta ser propietario para practicar; no resolverá en Internet).
2. Añadir registros A, CNAME, MX y TXT y revisar el TTL.
3. Crear un registro **alias** apuntando a una IP pública y comprobar que cambia solo al cambiar la IP.
4. Consultar los servidores NS de la zona.
5. Con `nslookup -type=NS <zona> ns1-xx.azure-dns.com` comprobar la respuesta autoritativa.

## AZ-104 Exam Tips

- ⭐ Azure DNS **hospeda**, no **registra** dominios.
- 🔥 🧠 Para que funcione hay que **delegar los NS** en el registrador.
- 🔥 🧠 **Registros alias** para el **apex** y para recursos de Azure que cambian de IP.
- 🧠 Tipos clave: A, AAAA, CNAME, MX, TXT, NS, SOA, SRV, PTR, CAA.
- 🧠 CNAME **no se permite en el apex** del dominio.
- 💻 `az network dns zone create`, `record-set a/cname/mx/txt add-record`, consultar nameServers.
- 📌 Zona pública (Internet) vs zona privada (VNets).

## Errores comunes

- Crear la zona y esperar que resuelva sin delegar los NS.
- Intentar un CNAME en `@`.
- Confundir Azure DNS con el servicio de resolución interna de las VNets.

## Preguntas que podrían aparecer

**1.** Creas una zona DNS pública en Azure para `contoso.com` pero las consultas desde Internet siguen resolviendo en el proveedor anterior. ¿Qué falta?
- A) Crear registros A · B) Actualizar los servidores de nombres (NS) en el registrador del dominio · C) Vincular la zona a una VNet · D) Habilitar el autorregistro

<details><summary>Respuesta</summary>

**B.** Sin delegación NS, Azure DNS no es autoritativo para el dominio.
</details>

**2.** Necesitas que `contoso.com` (dominio raíz) apunte a un perfil de Azure Front Door. ¿Qué registro usas?
- A) CNAME · B) A con IP fija · C) Registro alias · D) MX

<details><summary>Respuesta</summary>

**C.** El apex no admite CNAME; el registro alias resuelve el problema y se actualiza automáticamente.
</details>

## Relacionado

- [[12 - Azure Private DNS y resolución de nombres]]
- [[19 - App Service - certificados, TLS y dominios personalizados]]
- [[17 - Comparación de balanceadores (Load Balancer, Application Gateway, Front Door, Traffic Manager)]]
- [[00 - Índice - Redes virtuales]]
