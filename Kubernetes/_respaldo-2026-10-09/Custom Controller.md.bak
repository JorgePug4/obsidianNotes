---
tags:
  - kubernetes
  - extensibility
aliases:
  - Custom Controller
  - Custom Controllers
  - Controlador personalizado
  - Operator
  - Operators
  - Kubernetes Operator
  - Operator Pattern
---

# Custom Controller

### Qué es
Un **Custom Controller** (controlador personalizado) es un programa que implementa el **bucle de reconciliación** de Kubernetes para recursos propios o para agregar comportamiento a recursos existentes. Funciona igual que los controladores integrados del [[Kube-controller-manager]], pero lo escribe y despliega un tercero, normalmente como un [[Workloads#Deployment|Deployment]] dentro del clúster.

### Para qué sirve
Da vida a los objetos definidos con una [[Custom Resource Definition]]. Su ciclo es:
1. **Observar** (*watch*) en el API Server los objetos que le interesan (ej. `Backup`, `Ingress`, `HTTPRoute`).
2. **Comparar** el estado deseado (`spec`) con el estado real.
3. **Actuar** creando, modificando o borrando recursos (Pods, Services, Jobs, balanceadores externos...) y **reportar** el resultado en el `status` del objeto.

Cuando una CRD y su controlador encapsulan el conocimiento operativo de una aplicación compleja (instalar, escalar, respaldar, actualizar una base de datos), el conjunto se conoce como **Operator**.

Ejemplos conocidos:
- Un [[Networking#Ingress Controller|Ingress Controller]] (NGINX, Traefik) es un controlador que convierte objetos [[Networking#Ingress|Ingress]] en configuración de proxy.
- El plano de control de Istio en una [[Networking#Service Mesh|Service Mesh]].
- Operadores de bases de datos (CloudNativePG, Strimzi para Kafka), cert-manager, Argo CD.

> [!tip] Herramientas para construirlos
> - **Kubebuilder** y **Operator SDK** (Go, basados en la librería `controller-runtime`).
> - **Kopf** (Python), **Java Operator SDK**, **kube-rs** (Rust).

> [!warning] La reconciliación debe ser idempotente
> El controlador puede ejecutar la reconciliación muchas veces para el mismo objeto (reinicios, re-sincronizaciones periódicas). Cada ejecución debe producir el mismo resultado sin duplicar recursos. Usa `ownerReferences` para que los recursos creados se borren junto con el objeto padre, y *finalizers* para limpiar recursos externos.

### Ejemplo

**Comandos útiles:**
```bash
# Crear el esqueleto de un operador con Kubebuilder
kubebuilder init --domain example.com --repo github.com/jorge/backup-operator
kubebuilder create api --group apps --version v1 --kind Backup

# Instalar las CRDs y ejecutar el controlador localmente contra el clúster
make install
make run

# Ver el controlador desplegado y sus logs
kubectl get deploy -n backup-operator-system
kubectl logs -n backup-operator-system deploy/backup-operator-controller-manager
```

**Estructura mínima de la función de reconciliación (Go con `controller-runtime`):**
```go
func (r *BackupReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. Leer el Custom Resource
    var backup appsv1.Backup
    if err := r.Get(ctx, req.NamespacedName, &backup); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // 2. Construir el recurso deseado (un CronJob que ejecute el respaldo)
    cronJob := buildCronJob(&backup)
    _ = ctrl.SetControllerReference(&backup, cronJob, r.Scheme)

    // 3. Crear o actualizar hasta alcanzar el estado deseado (idempotente)
    if _, err := controllerutil.CreateOrUpdate(ctx, r.Client, cronJob, func() error { return nil }); err != nil {
        return ctrl.Result{}, err
    }

    // 4. Reportar el estado
    backup.Status.Ready = true
    return ctrl.Result{}, r.Status().Update(ctx, &backup)
}
```

### Relacionado
- [[Custom Resource Definition]]
- [[Kube-controller-manager]]
- [[Networking#Ingress Controller|Ingress Controller]]
- [[Networking#Kubernetes Gateway API|Kubernetes Gateway API]]
- [[Networking#Service Mesh|Service Mesh]]
- [[Helm]]

> [!info] 📚 Estudio guiado
> Capítulo: [[02 - Fundamentos y arquitectura]] · Índice: [[00 - Kubernetes - Índice]]
