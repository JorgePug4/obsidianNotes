---
tags: [kubernetes, entrevista, nivel/senior, nivel/architect]
---
# 17 - Interview mode

> Preguntas por nivel. Cada una: **respuesta corta** (lo que dirías en 20 s), **respuesta profunda**, **ejemplo**, **qué evalúa** el entrevistador y **errores a evitar**. Despliega solo después de responder en voz alta.

## Contenido
- [[#🟢 Junior - fundamentos]]
- [[#🔵 Mid - arquitectura y troubleshooting]]
- [[#🟣 Senior - diseño, escalabilidad, seguridad y producción]]
- [[#🟠 Staff y Architect - decisiones y trade-offs]]
- [[#Cómo responder bien]]

---

## 🟢 Junior - fundamentos

> [!question]- 1. ¿Qué es un Pod y por qué no se despliegan contenedores directamente?
> **Corta:** la unidad mínima de despliegue: uno o varios contenedores que comparten IP, puertos y volúmenes y se programan juntos en un nodo.
> **Profunda:** el Pod da una unidad atómica de *scheduling*, red y ciclo de vida para procesos acoplados (app + sidecar). Es efímero: si muere, no "resucita"; un controlador crea **otro** con otra IP. Por eso se gestionan con Deployments y se accede a ellos vía Services.
> **Ejemplo:** una API con un sidecar de Fluent Bit que lee sus logs de un `emptyDir` compartido.
> **Evalúa:** modelo mental básico. **Evita:** "un Pod es un contenedor" o "un Pod es una VM".

> [!question]- 2. ¿Deployment vs ReplicaSet?
> **Corta:** el ReplicaSet mantiene N réplicas; el Deployment gestiona ReplicaSets para hacer actualizaciones y rollbacks.
> **Profunda:** cada cambio en la plantilla crea un ReplicaSet nuevo; el Deployment traslada réplicas según `maxSurge`/`maxUnavailable` y conserva los antiguos (`revisionHistoryLimit`) para `rollout undo`.
> **Evalúa:** jerarquía de objetos. **Evita:** decir que se crean ReplicaSets a mano.

> [!question]- 3. ¿Por qué utilizarías un Service?
> **Corta:** para tener una IP y un DNS estables delante de Pods efímeros, con balanceo entre los que están Ready.
> **Profunda:** el Service selecciona Pods por labels; el EndpointSlice controller mantiene sus IPs Ready; kube-proxy/eBPF programa en cada nodo el DNAT de la IP virtual a un Pod. Tipos: ClusterIP, NodePort, LoadBalancer, headless, ExternalName.
> **Ejemplo:** `orders` llama a `http://payments` sin saber cuántos Pods hay ni dónde.
> **Evalúa:** service discovery. **Evita:** "para exponer a Internet" (eso es solo un caso).

> [!question]- 4. ¿ConfigMap vs Secret?
> **Corta:** ConfigMap para configuración no sensible, Secret para sensible; ambos se consumen como variables o ficheros.
> **Profunda:** los Secrets **no están cifrados por defecto** (Base64); su ventaja es RBAC separado, montaje en tmpfs y tratamiento especial. Para protegerlos: cifrado en reposo con KMS, RBAC, gestor externo (Key Vault + External Secrets/CSI), Workload Identity para evitar secretos.
> **Evalúa:** conciencia de seguridad. **Evita:** "los Secrets están cifrados".

> [!question]- 5. ¿Qué es un namespace? ¿Aísla la red?
> **Corta:** partición lógica para nombres, RBAC, cuotas y políticas. **No** aísla la red por defecto.
> **Profunda:** el aislamiento real requiere NetworkPolicies, ResourceQuotas, RBAC por namespace y, para seguridad fuerte, nodos o clústeres separados.
> **Evita:** tratarlo como frontera de seguridad.

> [!question]- 6. ¿Declarativo vs imperativo?
> **Corta:** imperativo = órdenes (`kubectl scale`); declarativo = estado deseado en YAML (`kubectl apply`) que Kubernetes reconcilia.
> **Profunda:** el declarativo es reproducible, versionable y base de GitOps; lo imperativo sirve para explorar, depurar y generar YAML (`--dry-run=client -o yaml`).

---

## 🔵 Mid - arquitectura y troubleshooting

> [!question]- 7. ¿Qué ocurre cuando creas un Deployment?
> **Corta:** el API Server valida y guarda en etcd; el Deployment controller crea un ReplicaSet; este crea Pods; el scheduler los asigna a nodos; el kubelet los arranca vía el runtime; al pasar readiness entran en el Service.
> **Profunda:** incluye AuthN → RBAC → admission (mutating/validating) → etcd. Ningún componente llama a otro: todos observan la API (*watch*) y reconcilian. CNI da red, CSI monta volúmenes. Diagrama en [[02 - Fundamentos y arquitectura#Qué ocurre cuando creas un Deployment]].
> **Evalúa:** visión de extremo a extremo. **Evita:** saltarte el scheduler o decir que el scheduler arranca los contenedores.

> [!question]- 8. ¿Qué sucede cuando un Pod muere?
> **Corta:** si muere el **contenedor**, el kubelet lo reinicia (con backoff); si desaparece el **Pod**, su controlador crea otro nuevo.
> **Profunda:** el Pod nuevo tiene otro nombre e IP; el EndpointSlice se actualiza; si el nodo murió, hay ~5 min de tolerancia antes del desalojo. Los Pods sin controlador no se recrean. En StatefulSets se recrea con la **misma identidad** y su PVC.
> **Evalúa:** distinguir contenedor/Pod/nodo. **Evita:** "Kubernetes lo reinicia" sin precisar quién y qué.

> [!question]- 9. ¿Cómo funciona el scheduler?
> **Corta:** filtra los nodos donde el Pod cabe y cumple restricciones, puntúa los válidos y asigna el mejor.
> **Profunda:** filtrado por requests vs allocatable, nodeSelector/affinity requerida, taints, puertos, volúmenes/zonas; puntuación por affinity preferida, spread, balance de recursos, imágenes presentes; *binding* escribiendo `nodeName`. Si ninguno vale: `Pending` + `FailedScheduling` → Cluster Autoscaler o preemption. Usa **requests**, no uso real.
> **Evita:** decir que mira la CPU actual de los nodos.

> [!question]- 10. ¿Cómo se comunican dos Pods?
> **Corta:** cada Pod tiene IP propia y todos se alcanzan sin NAT gracias al CNI; en la práctica se comunican a través de Services y DNS.
> **Profunda:** mismo nodo: bridge/veth; distinto nodo: overlay (VXLAN) o enrutamiento nativo en la VNet. Pod → `api.ns.svc.cluster.local` → CoreDNS → ClusterIP → DNAT a un Pod Ready. Balanceo por conexión (cuidado con gRPC). NetworkPolicies pueden restringirlo.

> [!question]- 11. ¿Liveness vs Readiness?
> **Corta:** readiness decide si recibe **tráfico**; liveness si hay que **reiniciar**. Startup protege el arranque lento.
> **Profunda:** readiness fallida → fuera del EndpointSlice, sin reinicio; liveness fallida → reinicio. Nunca dependencias externas en liveness (cascada de reinicios). Readiness con dependencias solo con criterio. Detalle en [[07 - Salud, fiabilidad y despliegues#Liveness vs Readiness vs Startup]].
> **Ejemplo:** API que tarda 90 s en arrancar sin startupProbe y con liveness a los 30 s → CrashLoopBackOff infinito.

> [!question]- 12. ¿Requests vs Limits?
> **Corta:** requests = garantía usada por el scheduler; limits = techo aplicado por el kernel (CPU: throttling; memoria: OOMKill).
> **Profunda:** determinan la QoS (Guaranteed/Burstable/BestEffort) y el orden de desalojo; el HPA calcula la utilización sobre requests. Debate sobre límites de CPU ([[06 - Scheduling y recursos#Requests y limits]]).
> **Evita:** decir que el límite de CPU mata el contenedor.

> [!question]- 13. ¿Cómo investigarías un CrashLoopBackOff?
> **Corta:** `describe pod` para ver Last State y exit code, `logs --previous` para ver por qué murió, y eventos.
> **Profunda:** exit 137 → OOM; 1 → excepción (config, secreto, dependencia); 127 → comando inexistente; reinicios por liveness → eventos de probe. Si no hay logs: `kubectl debug --copy-to` con `sleep`, reproducir en local con las mismas variables. Mitigar con `rollout undo` si es un despliegue nuevo.
> **Evalúa:** método, no adivinanzas. **Evita:** "borro el Pod a ver si se arregla".

> [!question]- 14. ¿Ingress vs Service?
> **Corta:** el Service es L4 (IP estable y balanceo a Pods); el Ingress es una regla L7 HTTP (host, ruta, TLS) que un controlador aplica y que apunta a Services.
> **Profunda:** un único LoadBalancer para el controlador en vez de uno por servicio; TLS centralizado. Hoy, Gateway API es el sucesor (roles separados, pesos, cabeceras) y `ingress-nginx` está retirado.

> [!question]- 15. ¿Deployment vs StatefulSet?
> **Corta:** Deployment para réplicas intercambiables sin estado; StatefulSet para réplicas con identidad estable, DNS por Pod y un disco por réplica.
> **Profunda:** orden de arranque/actualización, `volumeClaimTemplates`, headless Service, PVCs que persisten tras borrar. Y la pregunta de fondo: ¿debería esa BD estar en el clúster? Operadores o servicios gestionados ([[18 - Trade-offs#Deployment vs StatefulSet]]).

---

## 🟣 Senior - diseño, escalabilidad, seguridad y producción

> [!question]- 16. ¿Cómo realizarías un deployment sin downtime?
> **Corta:** rolling update con `maxUnavailable: 0`, readiness real, cierre ordenado (preStop + SIGTERM), PDB, cambios compatibles hacia atrás y verificación con rollback automático.
> **Profunda:** carrera endpoint/SIGTERM y cómo evitarla; *expand/contract* para BD; `minReadySeconds`; canary con análisis de métricas (Argo Rollouts/Flagger); capacidad para el surge. Checklist en [[07 - Salud, fiabilidad y despliegues#Checklist de despliegue sin downtime]].
> **Evalúa:** que hayas sufrido los 502 de los despliegues. **Evita:** "Kubernetes ya lo hace solo".

> [!question]- 17. ¿HPA vs VPA? ¿Y el Cluster Autoscaler?
> **Corta:** HPA cambia el **número** de Pods; VPA el **tamaño** (requests); el Cluster Autoscaler el número de **nodos**.
> **Profunda:** HPA sobre requests; VPA en `Off` para dimensionar (conflicto si ambos actúan sobre CPU); CA reacciona a Pods Pending y retira nodos infrautilizados respetando PDBs; KEDA para colas y escala a cero; Karpenter/NAP para elegir el tipo de nodo. Cadena: requests buenos → todo lo demás funciona.

> [!question]- 18. ¿Cómo asegurarías Kubernetes?
> **Corta:** por capas: acceso al API, RBAC mínimo, admisión con Pod Security `restricted` y políticas, NetworkPolicies, secretos externos con Workload Identity, cadena de suministro y detección en runtime.
> **Profunda y ejemplo:** [[09 - Seguridad#Checklist de seguridad]]. Mencionar permisos peligrosos (`create pods`, `pods/exec`, `list secrets`) y el escenario de RCE.
> **Evita:** listas de herramientas sin amenazas.

> [!question]- 19. Tu servicio tiene p99 de 2 s en horas punta, CPU media al 40 %. ¿Qué miras?
> **Corta:** throttling de CPU, saturación de dependencias y desequilibrio de carga, con métricas y trazas.
> **Profunda:** la media oculta: `cfs_throttled` alto con límites bajos; conexiones a BD agotadas (pool); thread pool starvation (.NET); un Pod caliente por conexiones persistentes; GC; HPA en su máximo; vecinos ruidosos en el nodo. Trazas para ver el *span* lento.
> **Evalúa:** pensamiento de SRE.

> [!question]- 20. ¿Cómo gestionas la configuración y los secretos en varios entornos?
> **Corta:** misma imagen en todos; valores por entorno con Helm values o Kustomize overlays en git; secretos en Key Vault sincronizados con External Secrets; identidad sin secretos para servicios cloud.
> **Profunda:** *build once, deploy many*; checksum annotations para recargar; `immutable`; separar quién puede leer Secrets; rotación.

> [!question]- 21. Un nodo cae en producción. ¿Qué pasa con tu app y qué deberías tener configurado?
> **Corta:** sus Pods se recrean en otros nodos tras ~5 min; la app sigue si había réplicas en otros nodos/zonas.
> **Profunda:** topology spread por zona + ≥ 3 réplicas + PDB + readiness hacen que el impacto sea solo capacidad reducida; el CA repone nodos; discos RWO tardan en desacoplarse (StatefulSets); `tolerationSeconds` ajustable para reaccionar antes.

---

## 🟠 Staff y Architect - decisiones y trade-offs

> [!question]- 22. ¿Cómo diseñarías Kubernetes para millones de requests?
> **Corta:** el clúster es la parte fácil: entrada global con CDN, varias regiones, servicios sin estado con autoescalado, caché, asincronía con colas y una capa de datos que escale; todo medido con SLOs y pruebas de carga.
> **Profunda:** [[14 - Arquitecturas reales#Preguntas de arquitectura]]. Límites del clúster (nodos, Pods por nodo, IPs, CoreDNS, conntrack, API Server); multi-clúster; *backpressure* y rate limiting; coste por request.
> **Evalúa:** que identifiques los cuellos de botella reales (datos, red, dependencias). **Evita:** "pongo maxReplicas: 1000".

> [!question]- 23. ¿Kubernetes o Azure Container Apps (o serverless) para un nuevo producto?
> **Corta:** depende del tamaño del equipo, la heterogeneidad de cargas, el control que necesitas y la capacidad de operar una plataforma.
> **Profunda:** tabla en [[18 - Trade-offs#Kubernetes vs Serverless vs Azure Container Apps]]. Coste total incluye personas. Estrategia: empezar en una plataforma gestionada con contenedores estándar (portables) y migrar a AKS si se necesita control.

> [!question]- 24. ¿Un clúster grande compartido o muchos clústeres pequeños?
> **Corta:** compartido = eficiencia y operación centralizada; muchos = aislamiento y menor radio de explosión. Normalmente: pocos clústeres por entorno/región y multi-tenancy por namespaces con guardarraíles.
> **Profunda:** requisitos regulatorios, ruido entre vecinos, actualizaciones, coste del Control Plane, herramientas (Argo CD ApplicationSets, Fleet/Azure Kubernetes Fleet Manager), límites de escala.

> [!question]- 25. ¿Cómo plantearías el disaster recovery?
> **Corta:** definir RTO/RPO, clúster reproducible como código + GitOps, datos con replicación/backup del servicio gestionado, Velero para estado del clúster si hace falta, y ensayos periódicos.
> **Profunda:** activo-pasivo vs activo-activo, DNS/Front Door para failover, secretos y registros replicados, dependencias externas. Ver [[19 - Kubernetes en producción#Backup y disaster recovery]].

> [!question]- 26. ¿Cómo reducirías el coste de un clúster?
> **Corta:** ajustar requests a la realidad, autoescalar nodos con consolidación, spot para lo interrumpible, apagar no-producción y medir coste por equipo.
> **Profunda:** VPA en recomendación, Karpenter/NAP, bin packing, reservas/Savings Plans, rightsizing de node pools, menos LBs públicos, logs con retención por nivel, Kubecost/OpenCost para *showback*.

---

## Cómo responder bien

> [!tip] Estructura
> 1. **Respuesta corta** primero (demuestra que sabes).
> 2. **Mecanismo**: qué pasa por dentro.
> 3. **Trade-off o matiz**: "depende de..." con criterio concreto.
> 4. **Experiencia**: "en un proyecto nos pasó X; lo resolvimos con Y".

> [!warning] Errores que delatan poca experiencia
> - Responder con nombres de herramientas sin explicar el problema que resuelven.
> - Afirmaciones absolutas ("siempre usa StatefulSet para BD", "nunca pongas límites").
> - Confundir Ingress/Service, liveness/readiness, requests/limits, Docker/containerd.
> - No mencionar observabilidad ni rollback al hablar de despliegues.
> - Información desactualizada: PodSecurityPolicy, dockershim, `ingress-nginx` como recomendación, `extensions/v1beta1`.

### Relacionado
- [[16 - Troubleshooting]] · [[18 - Trade-offs]] · [[21 - Roadmap y checklist]]
