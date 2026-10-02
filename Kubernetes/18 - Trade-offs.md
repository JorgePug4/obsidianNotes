---
tags: [kubernetes, tradeoffs, nivel/architect]
---
# 18 - Trade-offs

> En Kubernetes casi nunca hay una única respuesta correcta. Cada comparación: ventajas, desventajas, cuándo usar cada opción e impacto en **complejidad**, **coste** y **operación**.

## Contenido
- [[#Deployment vs StatefulSet]]
- [[#Base de datos dentro o fuera de Kubernetes]]
- [[#Ingress vs Gateway API]]
- [[#ConfigMap vs Secret]]
- [[#HPA vs VPA]]
- [[#Node affinity vs taints y tolerations]]
- [[#Helm vs Kustomize]]
- [[#Push CI-CD vs GitOps]]
- [[#Un clúster compartido vs varios clústeres]]
- [[#Kubernetes vs Serverless vs Azure Container Apps]]
- [[#Service mesh sí o no]]

---

## Deployment vs StatefulSet

| | **Deployment** | **StatefulSet** |
|---|---|---|
| Ventajas | Simple; rolling updates rápidos con surge; réplicas intercambiables; escala en paralelo | Identidad estable (`db-0`), DNS por Pod, un PVC por réplica, orden controlado, `partition` para canaries |
| Desventajas | Sin identidad ni disco por réplica | Actualizaciones lentas (una a una); PVCs que sobreviven (gestión manual); recuperación más lenta ante fallos de nodo |
| Cuándo | APIs, frontends, workers sin estado | Sistemas distribuidos con estado: bases de datos replicadas, Kafka, ZooKeeper, Elasticsearch |
| Complejidad | Baja | Media-alta (y la app debe saber replicarse) |
| Coste | Bajo | Discos por réplica, réplicas siempre encendidas |
| Operación | Mínima | Backups, upgrades, failover: mejor con un **operador** |

> [!tip] Regla práctica
> Si te preguntas "¿StatefulSet?", pregúntate antes "¿puedo sacar el estado del clúster (servicio gestionado) o usar un operador maduro?".

---

## Base de datos dentro o fuera de Kubernetes

| | **Servicio gestionado** (Azure SQL, Cosmos DB, RDS, Cloud SQL) | **En el clúster con operador** (CloudNativePG, Percona, Strimzi, MongoDB Operator) |
|---|---|---|
| Ventajas | HA, backups, PITR, parches y escalado incluidos; SLA; seguridad integrada | Portabilidad; mismo flujo GitOps; coste de infraestructura menor a gran escala; entornos efímeros baratos |
| Desventajas | Coste por instancia; dependencia del proveedor; menos control de versiones/extensiones | Tú eres el DBA: upgrades, backups, DR, rendimiento de discos, incidentes a las 3 AM |
| Cuándo | **Por defecto** en la nube y en producción | Requisitos on-premise/multi-nube, equipo con experiencia, muchas BD pequeñas, dev/test |
| Complejidad | Baja | Alta |
| Coste | Mayor en factura, menor en personas | Menor en factura, mayor en personas |

---

## Ingress vs Gateway API

| | **Ingress** (`networking.k8s.io/v1`) | **Gateway API** (`gateway.networking.k8s.io/v1`) |
|---|---|---|
| Ventajas | Simple, universal, estable, mucha documentación | Roles separados (infra/cluster/app), **traffic splitting por peso**, matching por cabeceras, redirecciones y reescrituras **portables**, TCP/UDP/gRPC, rutas entre namespaces controladas |
| Desventajas | Solo HTTP(S) básico; lo avanzado vía **anotaciones no portables**; congelado (no evoluciona); el controlador más popular (`ingress-nginx`) está retirado | Más objetos y conceptos; CRDs que hay que instalar; algunas funciones aún experimentales; madurez desigual entre implementaciones |
| Cuándo | Clústeres existentes, casos simples, controladores gestionados que solo soportan Ingress | **Proyectos nuevos**, canary/A-B, multi-equipo, necesidad de portabilidad |
| Complejidad | Baja | Media |
| Coste | Similar (depende del controlador) | Similar |
| Operación | Migraciones dolorosas si cambias de controlador (anotaciones) | Más fácil cambiar de implementación; delegación clara por roles |

> [!tip] Migración
> Herramienta oficial **`ingress2gateway`** para convertir Ingress (y algunas anotaciones) a Gateway API. Pueden convivir durante la transición.

---

## ConfigMap vs Secret

| | **ConfigMap** | **Secret** |
|---|---|---|
| Para | Configuración no sensible | Credenciales, claves, certificados |
| Almacenamiento | etcd en claro | etcd en Base64 (en claro sin cifrado en reposo); tmpfs en el nodo |
| Ventajas | Legible, fácil de diffear y revisar | RBAC separable, tratado como sensible por herramientas |
| Desventajas | Si metes secretos, los ve cualquiera con `get configmaps` | Falsa sensación de seguridad; no rota; ¿dónde está la fuente de verdad? |
| Cuándo | `LOG_LEVEL`, URLs, feature flags estáticos, ficheros de config | Contraseñas, tokens, TLS, pull secrets — idealmente **sincronizados desde un gestor** |
| Alternativa moderna | App Configuration / servicio de config para cambios en caliente | Key Vault + External Secrets / CSI; **Workload Identity** (sin secreto) |

---

## HPA vs VPA

| | **HPA** | **VPA** |
|---|---|---|
| Qué cambia | Nº de réplicas | Requests/limits de cada Pod |
| Ventajas | Rápido; aumenta disponibilidad; ideal para apps sin estado | Dimensiona bien; útil para cargas que no escalan horizontalmente (o con una réplica) |
| Desventajas | Requiere requests correctos; no ayuda si el cuello es la BD | Recrea Pods para aplicar (salvo *in-place resize*); conflicto con HPA en la misma métrica; reacción lenta |
| Cuándo | Carga variable en servicios sin estado | Dimensionamiento (`updateMode: Off`), *batch*, servicios singleton |
| Complejidad | Baja | Media |
| Coste | Ahorra en horas valle | Ahorra eliminando sobredimensionamiento |

**Combinación recomendada:** VPA en recomendación + HPA por CPU o métricas de negocio + Cluster Autoscaler.

---

## Node affinity vs taints y tolerations

| | **Node affinity / nodeSelector** | **Taints + tolerations** |
|---|---|---|
| Lo define | El **Pod** ("quiero ir a...") | El **nodo** ("no acepto a nadie salvo...") |
| Efecto | **Atrae** | **Repele** |
| Garantiza | Que *ese* Pod va a ciertos nodos | Que *otros* Pods no entran en esos nodos |
| No garantiza | Que otros Pods no ocupen esos nodos | Que tus Pods vayan allí (la toleration solo permite) |
| Cuándo | Requisitos de hardware/zona del Pod | Dedicar nodos (GPU, sistema, spot, equipos) |

**Para nodos dedicados, ambos:** taint en el nodo + toleration y affinity en los Pods.

---

## Helm vs Kustomize

| | **Helm** | **Kustomize** |
|---|---|---|
| Modelo | Plantillas Go + values | Base + overlays con parches (sin plantillas) |
| Ventajas | Empaquetado y versionado (charts, OCI), ecosistema enorme de charts de terceros, hooks, releases con historial y rollback | YAML "puro" legible, integrado en `kubectl`, ideal para variaciones por entorno, sin lógica oculta |
| Desventajas | Plantillas difíciles de leer y depurar; lógica en YAML; estado del release en Secrets | Sin empaquetado ni hooks ni historial; parches complejos para cambios grandes |
| Cuándo | Distribuir software (a otros equipos/clientes), instalar software de terceros, apps con muchas opciones | Tus propias apps con pocas diferencias entre entornos; GitOps |
| Complejidad | Media | Baja-media |

Muy habitual: **Helm para software de terceros** (Prometheus, cert-manager) y **Kustomize (o Helm sencillo) para tus apps**; Argo CD soporta ambos.

---

## Push CI-CD vs GitOps

| | **Push** (el pipeline ejecuta `helm upgrade`) | **GitOps** (Argo CD/Flux sincroniza desde git) |
|---|---|---|
| Ventajas | Simple, un único sistema, feedback inmediato en el pipeline | Git como fuente de verdad, detección y corrección de *drift*, sin credenciales del clúster en el CI, multi-clúster natural, auditoría |
| Desventajas | Credenciales de clúster en el CI; *drift* invisible; difícil a escala | Otro componente que operar; *debugging* en dos sitios; gestión de secretos en git |
| Cuándo | Equipos pequeños, pocos clústeres | Varios clústeres/entornos, requisitos de auditoría, plataformas internas |

---

## Un clúster compartido vs varios clústeres

| | **Compartido (multi-tenant)** | **Varios clústeres** |
|---|---|---|
| Ventajas | Mejor utilización, menos Control Planes, plataforma y actualizaciones centralizadas | Aislamiento fuerte, radio de explosión pequeño, versiones y configuración independientes |
| Desventajas | Vecinos ruidosos, RBAC/políticas complejas, una actualización afecta a todos | Más coste fijo y operación; herramientas multi-clúster |
| Cuándo | Equipos de la misma organización con guardarraíles (namespaces, cuotas, políticas) | Producción vs no producción, regulación, clientes distintos, regiones |

---

## Kubernetes vs Serverless vs Azure Container Apps

| | **AKS/EKS/GKE** | **Azure Container Apps** (o Cloud Run, App Runner) | **Functions / Lambda** | **App Service** |
|---|---|---|---|---|
| Unidad | Pods, cualquier carga | Contenedores (apps y jobs) | Funciones | Apps web |
| Control | Total (red, CRDs, operadores, DaemonSets, GPU) | Medio: sin acceso a la API de Kubernetes | Bajo | Bajo-medio |
| Escalado a cero | Con KEDA | **Nativo** (KEDA integrado) | Nativo | No |
| Despliegues avanzados | Todo (con herramientas) | Revisiones y tráfico por pesos integrados | Slots/versiones | Slots |
| Operación | **Alta** (plataforma que mantener) | Baja | Muy baja | Baja |
| Coste | Nodos siempre encendidos + equipo de plataforma | Por uso (consumo) o perfiles dedicados | Por invocación | Por plan |
| Cuándo | Muchos servicios y equipos, cargas heterogéneas, plataforma interna, requisitos de control/portabilidad | Microservicios y APIs en contenedores sin querer operar Kubernetes | Eventos cortos, *glue code* | Monolitos web y APIs sencillas |

> [!important] Coste total = factura + personas
> Un AKS "barato" en infraestructura puede costar una o dos personas dedicadas a operarlo. Para 3-10 servicios sin requisitos especiales, Container Apps suele ser la opción racional; los contenedores OCI permiten migrar a AKS más adelante.

---

## Service mesh sí o no

| | **Sin mesh** | **Con mesh** (Istio ambient, Linkerd, Cilium) |
|---|---|---|
| Ventajas | Menos piezas, menos latencia y recursos | mTLS automático, políticas L7 entre servicios, reintentos/timeouts uniformes, métricas y topología sin código, canary avanzado |
| Desventajas | mTLS y observabilidad dependen de cada app | Complejidad operativa real, upgrades delicados, curva de aprendizaje |
| Cuándo | La mayoría de equipos al empezar; resiliencia en las librerías de la app | Muchos servicios, requisitos de Zero Trust/mTLS, tráfico complejo, equipo de plataforma que la opere |

### Relacionado
- [[17 - Interview mode]] · [[19 - Kubernetes en producción]] · [[16 - Comparación de servicios de contenedores]]
