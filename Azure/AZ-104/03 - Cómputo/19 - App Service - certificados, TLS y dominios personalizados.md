---
tags: [az-104, azure, computo, app-service, tls, dns, certificados]
modulo: Cómputo
peso_examen: Alto
---

# App Service · certificados, TLS y dominios personalizados

## ¿Qué es?

Configurar que una Web App responda en un **dominio propio** (`www.contoso.com`) en lugar de `<app>.azurewebsites.net`, y que lo haga por **HTTPS** con un **certificado TLS** válido.

## ¿Para qué sirve?

- Presentar la marca en la URL.
- Cumplir requisitos de seguridad (HTTPS obligatorio, TLS 1.2+).

## Conceptos clave

### Dominio personalizado 🧠
- Requiere plan **Basic o superior** (Shared permite dominio en Windows, pero sin TLS propio).
- **Validación de propiedad** con registros DNS en el registrador del dominio:

| Tipo de registro | Para qué | Valor |
|---|---|---|
| **A** | Dominio raíz (`contoso.com`) | Dirección IP entrante de la app |
| **CNAME** | Subdominio (`www.contoso.com`) | `<app>.azurewebsites.net` |
| **TXT** (`asuid.<subdominio>`) | Verificar la propiedad | **Domain verification ID** de la app |

- El **dominio raíz (apex)** no admite CNAME según el estándar DNS: se usa **A + TXT** o un **alias** si el DNS lo soporta (Azure DNS permite registros **alias** al recurso).
- **Azure DNS**: si la zona está en Azure, se puede crear el registro desde el propio asistente.
- **App Service Domain** ➕: comprar el dominio directamente en Azure (gestiona DNS automáticamente).

### Certificados TLS/SSL 🧠

| Opción | Coste | Renovación | Limitaciones |
|---|---|---|---|
| **Certificado administrado por App Service** (App Service Managed Certificate) | **Gratis** | Automática | No admite **comodín** en todos los casos, no exportable, requiere que el dominio esté ya configurado y verificado; no soporta dominios sin CNAME/A válidos ni redes privadas |
| **App Service Certificate** | De pago (comprado en Azure) | Automática con Key Vault | Se almacena en **Key Vault** |
| **Importar desde Key Vault** | Según origen | Manual/automática | Requiere acceso de la app al Key Vault |
| **Cargar un certificado privado (.pfx)** | Externo | Manual | Debe cumplir requisitos (clave de 2048 bits, cadena completa, contraseña) |
| **Certificado público (.cer)** | — | — | Solo para que la app confíe en servicios externos |

- **Enlace TLS (TLS binding)**: asociar el certificado al dominio con tipo **SNI SSL** (compartido, habitual) o **IP-based SSL** (IP dedicada, requiere Standard+ y genera un coste adicional; cambia la IP entrante).
- **Requisitos del certificado**: emitido por una CA de confianza, coincidir con el nombre de dominio, no caducado, con clave privada (.pfx) para el binding.
- **Configuración de seguridad**: **HTTPS Only** (redirige HTTP→HTTPS), **versión mínima de TLS** (1.0/1.1/1.2/1.3), **certificados de cliente** (mutual TLS: Require/Allow/Ignore).
- El certificado se asocia **por app y por slot**; los slots tienen su propio hostname.

## Cómo funciona

```
1. Crear en el DNS del dominio:
     www   CNAME  app-contoso.azurewebsites.net
     asuid.www TXT  <Domain verification ID>
2. App Service → Dominios personalizados → Agregar → validar → Agregar
3. App Service → Certificados → Certificado administrado → crear para www.contoso.com
4. Dominios personalizados → Agregar enlace TLS → SNI SSL
5. Configuración → Solo HTTPS = Activado, TLS mínimo 1.2
```

```bash
# Verificar y añadir dominio
az webapp config hostname add --webapp-name app-contoso --resource-group rg-web --hostname www.contoso.com
# Certificado administrado gratuito
az webapp config ssl create --resource-group rg-web --name app-contoso --hostname www.contoso.com
# Cargar un .pfx y enlazarlo
az webapp config ssl upload --resource-group rg-web --name app-contoso --certificate-file cert.pfx --certificate-password '<pwd>'
az webapp config ssl bind --resource-group rg-web --name app-contoso --certificate-thumbprint <thumbprint> --ssl-type SNI
# Seguridad
az webapp update --resource-group rg-web --name app-contoso --https-only true
az webapp config set --resource-group rg-web --name app-contoso --min-tls-version 1.2
az webapp show --resource-group rg-web --name app-contoso --query "{ip:outboundIpAddresses,verificationId:customDomainVerificationId}"
```

```powershell
Set-AzWebApp -ResourceGroupName rg-web -Name app-contoso -HostNames @("www.contoso.com","app-contoso.azurewebsites.net")
New-AzWebAppSSLBinding -ResourceGroupName rg-web -WebAppName app-contoso -Name www.contoso.com -CertificateFilePath cert.pfx -CertificatePassword '<pwd>' -SslState SniEnabled
```

## Configuración relevante para el examen

| Escenario | Solución |
|---|---|
| Añadir `www.contoso.com` a una app | **CNAME** a `<app>.azurewebsites.net` + **TXT** `asuid.www` con el ID de verificación |
| Añadir el dominio raíz `contoso.com` | **A** a la IP entrante + **TXT** `asuid` (o registro alias en Azure DNS) |
| Certificado gratuito y autorrenovable | **App Service Managed Certificate** |
| Certificado comodín `*.contoso.com` | Comprar App Service Certificate, importar de Key Vault o cargar .pfx |
| Necesito una IP dedicada para el certificado | **IP-based SSL** (Standard+) |
| Forzar HTTPS | **HTTPS Only** |
| Solo TLS 1.2 o superior | **Versión mínima de TLS** |
| Autenticación con certificado de cliente | **Client certificates: Require** |
| El plan es Free y no deja añadir dominio | Escalar a **Basic** o superior |

## Ejemplo

Contoso publica su web en `app-contoso.azurewebsites.net` y quiere `www.contoso.com` con HTTPS. En su proveedor DNS crean `www CNAME app-contoso.azurewebsites.net` y `asuid.www TXT <verification id>`. En el portal añaden el dominio personalizado, crean un **certificado administrado gratuito** para ese host, lo enlazan con **SNI SSL** y activan **Solo HTTPS** con TLS mínimo 1.2.

## Comparaciones

| Tipo de certificado | Uso | Ventajas | Cuándo utilizarlo |
|---|---|---|---|
| **Administrado (gratis)** | Dominios estándar | Sin coste, renovación automática | Casos simples |
| **App Service Certificate** | Comprado en Azure | Gestión en Key Vault, renovación | Empresas que compran en Azure |
| **Key Vault import** | Certificado corporativo | Centralizado, rotación | PKI propia |
| **Carga manual (.pfx)** | Certificado externo | Cualquier CA | Certificados ya adquiridos |

| Enlace | Uso | Nota |
|---|---|---|
| **SNI SSL** | Varios certificados por IP | Estándar actual, sin coste extra |
| **IP-based SSL** | IP dedicada | Clientes muy antiguos sin SNI; requiere Standard+ |

## AZ-104 Exam Tips

- 🔥 🧠 **CNAME** para subdominio, **A** para el dominio raíz, **TXT `asuid`** para verificar la propiedad.
- 🔥 🧠 Dominio personalizado y certificado propio: **Basic o superior**.
- 🧠 El certificado **gratuito administrado** se renueva solo pero **no se exporta** ni cubre todos los escenarios comodín.
- 🧠 **SNI SSL** por defecto; **IP-based SSL** da IP dedicada (Standard+).
- 💻 `az webapp config hostname add`, `az webapp config ssl create/upload/bind`, HTTPS Only.
- ⚠️ El dominio apex no admite CNAME: A o alias.

## Errores comunes

- Crear el CNAME sin el TXT de verificación y fallar la validación.
- Intentar un dominio personalizado en el nivel Free.
- Olvidar activar HTTPS Only después de enlazar el certificado.

## Preguntas que podrían aparecer

**1.** Necesitas asignar `shop.contoso.com` a una Web App. ¿Qué registros DNS creas?
- A) Solo un registro A · B) Un CNAME a `<app>.azurewebsites.net` y un TXT `asuid.shop` con el ID de verificación · C) Un MX · D) Un SRV

<details><summary>Respuesta</summary>

**B.** El CNAME dirige el tráfico y el TXT valida la propiedad del dominio.
</details>

**2.** ¿Qué nivel mínimo de plan permite usar un certificado TLS propio con un dominio personalizado?
- A) Free · B) Shared · C) Basic · D) Premium

<details><summary>Respuesta</summary>

**C.** Basic es el primer nivel con soporte de certificados TLS propios.
</details>

## Relacionado

- [[18 - Azure App Service - creación y configuración]]
- [[17 - App Service Plan (niveles y escalado)]]
- [[11 - Azure DNS (zonas públicas)]]
- [[Azure Key Vault]] (AZ-900)
- [[00 - Índice - Cómputo]]
