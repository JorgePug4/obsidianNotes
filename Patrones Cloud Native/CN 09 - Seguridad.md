---
tags: [cloud-native, patrones, seguridad, zero-trust, identidad]
aliases: [Federated Identity, Gatekeeper, Workload Identity, Quarantine pattern, mTLS, Seguridad cloud native]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 09 - Seguridad

> [!info] Objetivo del capítulo
> Patrones para proteger aplicaciones distribuidas cuando **no existe un perímetro** fiable: cada servicio, cada identidad y cada petición deben verificarse.
> Relacionado: [[Confianza cero (Zero Trust)]], [[Microsoft Entra ID]], [[Azure Key Vault]], [[09 - Seguridad]] (Kubernetes).

## Contenido
- [[#1. Zero Trust]]
- [[#2. Federated Identity]]
- [[#3. Gatekeeper]]
- [[#4. Workload Identity (identidad sin secretos)]]
- [[#5. External Secret Store]]
- [[#6. mTLS e identidad de servicio]]
- [[#7. Valet Key]]
- [[#8. Quarantine]]
- [[#9. Defensa en profundidad en Kubernetes]]
- [[#10. Seguridad de la cadena de suministro]]

---

## 1. Zero Trust

**Problema.** El modelo "castillo y foso" asume que lo que está dentro de la red es de confianza. En la nube, con microservicios, trabajo remoto y servicios de terceros, un atacante que entra en un componente se mueve lateralmente sin obstáculos.

**Solución.** Principios (Microsoft; NIST SP 800-207):
1. **Verificar explícitamente**: autenticar y autorizar cada petición con toda la información disponible (identidad, dispositivo, ubicación, servicio).
2. **Mínimo privilegio**: acceso justo y a tiempo (*JIT/JEA*), permisos acotados.
3. **Asumir la brecha**: segmentar, cifrar de extremo a extremo, monitorizar para detectar y limitar el radio de impacto.

**En una arquitectura cloud native se traduce en:** identidad para cada servicio (no sólo para usuarios), mTLS entre servicios, políticas de red por defecto *deny*, tokens validados en cada servicio (no sólo en el gateway), sin secretos de larga duración.

---

## 2. Federated Identity

**Problema.** Cada aplicación con su propia base de usuarios y contraseñas: los usuarios tienen muchas credenciales, cada app reimplementa (mal) la autenticación, y dar de baja a un empleado requiere tocar todos los sistemas.

**Solución.** **Delegar la autenticación** en un proveedor de identidad (IdP) externo de confianza. La aplicación recibe un **token** firmado con *claims* y lo valida.

```
Usuario ─► App ──redirige──► IdP (Entra ID, Auth0, Keycloak, Okta)
                              └─ autentica (MFA, SSO)
Usuario ◄── token (JWT, OIDC) ──┘
App valida firma, emisor, audiencia y caducidad → usa los claims para autorizar
```

**Protocolos:** **OpenID Connect** (autenticación, sobre OAuth 2.0), **OAuth 2.0** (autorización delegada), SAML 2.0 (entornos empresariales clásicos).

**Flujos a conocer:**
| Escenario | Flujo |
|---|---|
| App web / SPA / móvil con usuario | **Authorization Code + PKCE** |
| Servicio a servicio sin usuario | **Client Credentials** (mejor con Workload Identity, sin secreto) |
| Servicio que llama a otro **en nombre del usuario** | *On-Behalf-Of* (Entra ID) / *Token Exchange* (RFC 8693) |
| Dispositivos sin navegador | Device Code |

Los flujos **Implicit** y **Resource Owner Password** están desaconsejados (OAuth 2.0 Security Best Current Practice, RFC 9700).

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(o =>
    {
        o.Authority = "https://login.microsoftonline.com/{tenantId}/v2.0";
        o.Audience = "api://pedidos";              // rechaza tokens emitidos para otra API
    });
builder.Services.AddAuthorization(o =>
    o.AddPolicy("pedidos.escribir", p => p.RequireClaim("scp", "Pedidos.Write")));
```

> [!warning] Validar la audiencia
> Un token emitido para la API A **no** debe ser aceptado por la API B. Sin validar `aud`, un servicio comprometido puede reutilizar tokens contra otros (*confused deputy*).

---

## 3. Gatekeeper

**Problema.** Si los servicios que manejan datos sensibles están expuestos directamente, un fallo en su validación permite un ataque directo, y comprometer el host expone credenciales y datos.

**Solución.** Un **intermediario dedicado** que valida y sanea **todas** las peticiones antes de pasarlas a los servicios de confianza. No tiene acceso a datos ni a credenciales de almacenamiento; si lo comprometen, el atacante no obtiene nada valioso.

```
Internet ─► Gatekeeper (validación, saneado, autenticación, WAF)  ─► red privada ─► Servicio con datos
              sin secretos de BD                                     (sólo acepta tráfico del gatekeeper)
```

**Implementaciones:** WAF (Azure Front Door / Application Gateway WAF), API Management con políticas de validación, un proxy inverso dedicado. Es una aplicación concreta de **Gateway Offloading** con foco en seguridad.

**Trade-offs:** latencia adicional y un componente crítico que debe escalar y ser HA. No sustituye a la validación en cada servicio (defensa en profundidad).

---

## 4. Workload Identity (identidad sin secretos)

**Problema.** Para que un servicio acceda a Key Vault, Storage o SQL, tradicionalmente necesita un **secreto** (cadena de conexión, client secret). Ese secreto hay que guardarlo, rotarlo y protegerlo; acaba en ficheros, variables de entorno y, a veces, en Git.

**Solución.** Dar al servicio una **identidad gestionada por la plataforma** y obtener tokens de corta duración automáticamente, **sin secretos que gestionar**.

| Plataforma | Mecanismo |
|---|---|
| Azure (App Service, Functions, Container Apps, VMs) | **Managed Identity** |
| AKS | **Microsoft Entra Workload ID**: la ServiceAccount de Kubernetes se federa con una identidad de Entra ID mediante OIDC (sustituye a la antigua *AAD Pod Identity*) |
| AWS EKS | IRSA / EKS Pod Identity |
| GKE | Workload Identity Federation |
| CI/CD (GitHub Actions → Azure) | Federación OIDC, sin client secret |

```csharp
// El mismo código funciona en local (az login / Visual Studio) y en Azure (Managed Identity / Workload ID)
var credential = new DefaultAzureCredential();
var secrets = new SecretClient(new Uri("https://kv-pedidos.vault.azure.net/"), credential);
var blobs   = new BlobServiceClient(new Uri("https://stpedidos.blob.core.windows.net/"), credential);
```

> [!tip] Producción
> `DefaultAzureCredential` prueba varias fuentes en cadena; en producción, Microsoft recomienda usar una credencial concreta (`ManagedIdentityCredential`, `WorkloadIdentityCredential`) para evitar comportamiento inesperado y latencia al arrancar.

Con Entra ID también puedes conectar a **Azure SQL, Service Bus, Cosmos DB, Redis**… sin contraseña. Es el patrón de seguridad con **mejor relación beneficio/coste** de este capítulo.

---

## 5. External Secret Store

**Problema.** Secretos en `appsettings.json`, en variables del pipeline, en ConfigMaps o en el repositorio.

**Solución.** Un almacén específico de secretos con control de acceso, auditoría, versionado y rotación: **Azure Key Vault**, HashiCorp Vault, AWS Secrets Manager.

**Formas de consumirlo:**
- Directamente desde la app (SDK + Workload Identity), con caché.
- Proveedor de configuración: `builder.Configuration.AddAzureKeyVault(...)`.
- En Kubernetes: **Secrets Store CSI Driver** (monta los secretos como ficheros) o **External Secrets Operator** (sincroniza a `Secret` de Kubernetes).
- Referencias de Key Vault en App Service / Container Apps.

**Recuerda:** un `Secret` de Kubernetes sólo está codificado en **Base64**, no cifrado; activa el cifrado de etcd en reposo y restringe con RBAC quién puede leerlos. Lo ideal es **no tener secretos** (sección 4) y, para los inevitables, **rotación automática**.

---

## 6. mTLS e identidad de servicio

**Problema.** El tráfico interno sin cifrar puede ser leído o suplantado por cualquiera dentro de la red. Y una IP no es una identidad (cambia constantemente).

**Solución.** **TLS mutuo**: ambos extremos presentan certificados, de modo que el tráfico va cifrado **y** cada lado verifica la identidad del otro. La identidad del servicio se codifica en el certificado (estándar **SPIFFE**: `spiffe://cluster.local/ns/pedidos/sa/api-pedidos`).

Gestionar certificados a mano para cientos de servicios es inviable; lo resuelve un **service mesh** (Istio, Linkerd: emisión y rotación automáticas) o **SPIRE**. Sobre esa identidad se definen **políticas de autorización** ("sólo `api-pedidos` puede llamar a `pagos` en `POST /cobros`").

---

## 7. Valet Key

Descrito en [[CN 05 - Patrones de gestión de datos#11. Valet Key]]. Desde el punto de vista de seguridad: **token de mínimo privilegio** (un objeto, una operación, minutos de vida), preferiblemente firmado con una identidad (SAS de delegación de usuario) en lugar de con la clave de la cuenta, y revocable.

---

## 8. Quarantine

**Problema.** Artefactos externos (imágenes de contenedor de terceros, paquetes, ficheros subidos por usuarios) pueden contener vulnerabilidades o malware. Usarlos directamente es un riesgo.

**Solución.** Patrón del catálogo de Microsoft: todo artefacto externo pasa por una **zona de cuarentena** donde se **valida** (escaneo de vulnerabilidades, antivirus, firma, licencias) antes de estar disponible para su consumo.

```
Imagen pública ─► Registro de cuarentena ─► escaneo (Trivy, Defender for Containers) ─► firma ─► Registro aprobado ─► clúster
Fichero de usuario ─► contenedor "pendiente" ─► antivirus (Defender for Storage) ─► contenedor "limpio" ─► uso
```

En Kubernetes se refuerza con **políticas de admisión** que sólo permiten imágenes del registro aprobado y firmadas (Azure Policy / Gatekeeper OPA, Kyverno, Ratify).

---

## 9. Defensa en profundidad en Kubernetes

Ningún control es suficiente solo; se apilan capas:

| Capa | Control | Patrón (*Kubernetes Patterns*) |
|---|---|---|
| Proceso | `runAsNonRoot`, `readOnlyRootFilesystem`, sin capacidades (`drop: [ALL]`), `seccompProfile: RuntimeDefault`, **Pod Security Standards** (*restricted*) | Process Containment |
| Red | `NetworkPolicy` *default deny* + permitir sólo lo necesario; mTLS | Network Segmentation |
| Configuración | Secretos externos, cifrado de etcd, no montar el token de la ServiceAccount si no se usa | Secure Configuration |
| Acceso | RBAC mínimo, una ServiceAccount por workload, nada de `cluster-admin` en apps | Access Control |
| Admisión | Políticas que rechazan Pods inseguros o imágenes no aprobadas | — |

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities: { drop: ["ALL"] }
  seccompProfile: { type: RuntimeDefault }
```

> [!note] .NET y usuario no root
> Las imágenes oficiales de .NET 8+ incluyen un usuario `app` sin privilegios (`USER $APP_UID`) y escuchan por defecto en el puerto **8080** (no 80) precisamente para poder ejecutarse sin root.

---

## 10. Seguridad de la cadena de suministro

**Problema.** El código que despliegas incluye cientos de dependencias e imágenes base que no has escrito. Ataques como SolarWinds (2020) o la puerta trasera de `xz` (2024) atacan la cadena, no la aplicación.

**Prácticas:**
- **SBOM** (*Software Bill of Materials*) de cada imagen (SPDX, CycloneDX).
- **Escaneo** de dependencias e imágenes en el CI (Dependabot, Trivy, Grype, Microsoft Defender).
- **Firma** de imágenes y verificación en la admisión (Sigstore/cosign, Notation).
- **Imágenes base mínimas** (*distroless*, *chiseled* de Ubuntu para .NET): menos paquetes = menos CVEs.
- Fijar versiones por **digest** (`image@sha256:...`), no por `latest`.
- Marco de referencia: **SLSA** (*Supply-chain Levels for Software Artifacts*).

---

## Puntos clave

- **Zero Trust**: verificar siempre, mínimo privilegio, asumir la brecha.
- **Federated Identity**: OIDC/OAuth con un IdP; Authorization Code + PKCE para usuarios; valida `aud`.
- **Workload Identity / Managed Identity**: la forma más eficaz de eliminar secretos.
- Secretos inevitables → **Key Vault** + rotación; un Secret de K8s es Base64, no cifrado.
- **mTLS** con identidad SPIFFE, normalmente vía service mesh.
- **Gatekeeper** y **Quarantine** para aislar lo que viene de fuera.
- Defensa en profundidad: contenedor no root, NetworkPolicy, RBAC, admisión, cadena de suministro.

## Errores comunes

- Validar el token sólo en el gateway y confiar ciegamente dentro.
- Client secrets en variables de entorno del pipeline en lugar de federación OIDC.
- Aceptar tokens sin validar la audiencia.
- Contenedores ejecutándose como root con el sistema de ficheros escribible.
- Usar imágenes `:latest` de Docker Hub sin escanear.
- Tokens en `localStorage` en SPAs (expuestos a XSS) → BFF con cookies `HttpOnly`.

## 🎯 Para entrevistas

- *"¿Cómo accede tu servicio en AKS a Key Vault sin guardar ningún secreto?"* → Entra Workload ID: ServiceAccount federada con una identidad de Entra; `WorkloadIdentityCredential`; RBAC de Key Vault.
- *"¿Qué flujo OAuth usarías para una SPA?"* → Authorization Code + PKCE, idealmente con un BFF que guarde los tokens en el servidor.
- *"¿Qué es Zero Trust aplicado a microservicios?"* → identidad por servicio, mTLS, autorización en cada salto, NetworkPolicy deny por defecto.
- *"¿Qué es el patrón Gatekeeper?"* → intermediario que valida y sanea sin acceso a datos ni secretos.

## Referencias

- Microsoft, *Federated Identity*, *Gatekeeper*, *Valet Key*, *Quarantine*: https://learn.microsoft.com/azure/architecture/patterns/
- NIST SP 800-207, *Zero Trust Architecture*: https://csrc.nist.gov/pubs/sp/800/207/final
- RFC 9700, *OAuth 2.0 Security Best Current Practice* (2025): https://datatracker.ietf.org/doc/rfc9700/
- Microsoft, *Use Microsoft Entra Workload ID with AKS*: https://learn.microsoft.com/azure/aks/workload-identity-overview
- Kubernetes, *Pod Security Standards*: https://kubernetes.io/docs/concepts/security/pod-security-standards/
- SPIFFE: https://spiffe.io/ · SLSA: https://slsa.dev/ · Sigstore: https://www.sigstore.dev/

---
⬅️ [[CN 08 - Despliegue, configuración y entrega]] · ➡️ [[CN 10 - Observabilidad]]
