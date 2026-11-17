---
tags: [cloud-native, patrones, despliegue, gitops, cicd, configuracion]
aliases: [Blue-Green, Canary, Feature Flags, GitOps, Infrastructure as Code, External Configuration Store, Rolling Update, Shadow deployment, Progressive delivery]
up: "[[CN 00 - Patrones Cloud Native - Índice]]"
---

# CN 08 - Despliegue, configuración y entrega

> [!info] Objetivo del capítulo
> Desplegar **a menudo y sin miedo**: cambios pequeños, automatizados, observables y reversibles en minutos. Y gestionar la configuración fuera del código.

## Contenido
- [[#1. Infrastructure as Code]]
- [[#2. GitOps]]
- [[#3. Estrategias de despliegue]] (Recreate, Rolling, Blue-Green, Canary, Shadow, A/B)
- [[#4. Progressive Delivery]]
- [[#5. Feature Flags (Feature Toggles)]]
- [[#6. External Configuration Store]]
- [[#7. Configuración en Kubernetes]]
- [[#8. Cambios de base de datos sin downtime: Expand and Contract]]
- [[#9. Las métricas DORA]]

---

## 1. Infrastructure as Code

**Problema.** Infraestructura creada a mano en el portal: no reproducible, no revisable, con deriva entre entornos, imposible de recrear tras un desastre.

**Solución.** Describir la infraestructura en **ficheros versionados** que una herramienta aplica.

| Enfoque | Herramientas | Nota |
|---|---|---|
| **Declarativo** (estado deseado) | Terraform/OpenTofu, Bicep, ARM, CloudFormation, Pulumi (declarativo en lenguajes de programación), Crossplane | El estándar actual |
| **Imperativo** (pasos) | Scripts de Azure CLI/PowerShell | Útil para tareas puntuales, frágil para infraestructura completa |

**Prácticas:** módulos reutilizables, *plan* revisado en el PR antes de aplicar, estado remoto con bloqueo (Terraform), detección de *drift*, ningún cambio manual en producción. Es requisito para Deployment Stamps y para DR.

---

## 2. GitOps

**Problema.** Pipelines que ejecutan `kubectl apply` con credenciales de administrador del clúster: poca trazabilidad del estado real, credenciales potentes fuera del clúster, deriva si alguien cambia algo a mano.

**Solución.** Principios de GitOps (OpenGitOps, CNCF):
1. **Declarativo**: el estado deseado del sistema se describe declarativamente.
2. **Versionado e inmutable**: ese estado se guarda en Git (historial completo, revisión por PR).
3. **Obtenido automáticamente** (*pull*): agentes en el clúster descargan el estado deseado.
4. **Reconciliado continuamente**: los agentes corrigen cualquier desviación.

```
Dev ─PR─► repo app ─CI─► imagen v1.5 en ACR
                  └─► PR al repo de manifiestos (tag: v1.5)
                                   │
                    Argo CD / Flux (dentro del clúster) ──pull──┘
                                   └─► aplica y reconcilia el clúster
```

**Ventajas:** Git es la fuente de verdad y la auditoría; rollback = `git revert`; el clúster no expone credenciales al CI; la deriva se corrige sola.
**Costes:** gestionar secretos requiere una pieza más (Sealed Secrets, SOPS, External Secrets Operator); los repos de manifiestos crecen; la promoción entre entornos necesita convenciones claras.

Herramientas: **Argo CD**, **Flux** (ambos graduados en la CNCF). En AKS, la extensión GitOps usa Flux. Detalle: [[11 - Helm, Kustomize y CI-CD]].

---

## 3. Estrategias de despliegue

| Estrategia | Cómo funciona | Downtime | Rollback | Coste de infraestructura | Riesgo |
|---|---|---|---|---|---|
| **Recreate** | Para todo v1, arranca v2 | **Sí** | Lento (redesplegar v1) | 1× | Alto |
| **Rolling Update** | Reemplaza instancias poco a poco | No | Medio (otro rolling) | ~1× (+ `maxSurge`) | Medio: v1 y v2 conviven |
| **Blue-Green** | Dos entornos completos; se conmuta el tráfico de golpe | No | **Instantáneo** (volver a conmutar) | **2×** durante el despliegue | Bajo, pero todo el tráfico cambia a la vez |
| **Canary** | v2 recibe un % pequeño de tráfico que se aumenta si las métricas son buenas | No | Rápido (tráfico a 0 %) | ~1× + canary | **Bajo**: el fallo afecta a pocos usuarios |
| **Shadow / Mirror** | v2 recibe una **copia** del tráfico real; sus respuestas se descartan | No | No aplica (no sirve a usuarios) | ~2× | Mínimo para usuarios; cuidado con efectos secundarios |
| **A/B testing** | Versiones distintas por segmento de usuarios para **medir negocio** | No | — | Variable | Es experimentación, no una estrategia de seguridad |

### Rolling Update (Kubernetes por defecto)

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%         # cuántos Pods extra puede crear
    maxUnavailable: 0     # nunca bajar de la capacidad deseada
```

Requiere **readiness probe** correcta (si no, recibe tráfico antes de estar lista) y **compatibilidad hacia atrás**: durante unos minutos v1 y v2 atienden a la vez y comparten BD y mensajes.

### Blue-Green

```
             ┌─► Blue  (v1)  ← tráfico 100 %
Router ──────┤
             └─► Green (v2)  ← pruebas de humo
     (conmutar)   →  Green 100 %, Blue en espera para rollback
```

En Kubernetes: dos Deployments y cambiar el `selector` del Service; en Azure: *deployment slots* de App Service, *revisions* de Container Apps.
**El problema es la base de datos**: si v2 cambia el esquema de forma incompatible, no puedes volver a v1. Ver sección 8.

### Canary

```
v1 ████████████████████ 95 %
v2 █                      5 %   → métricas OK → 25 % → 50 % → 100 %
                                 → métricas mal → 0 % (rollback automático)
```

Necesita **división de tráfico por porcentaje** (service mesh, Gateway API con `weight`, Argo Rollouts, Flagger, Container Apps con *traffic splitting*) y **métricas comparables** entre versiones (tasa de error, latencia p99).

### Shadow

Útil para validar rendimiento o una reescritura con tráfico real. **Peligro:** si v2 escribe en BD, envía emails o cobra, la copia del tráfico produce efectos reales duplicados. Sólo para lecturas o con dependencias simuladas.

---

## 4. Progressive Delivery

Evolución del Canary: **automatizar** la promoción o el rollback según métricas.

```yaml
# Argo Rollouts (extracto)
strategy:
  canary:
    steps:
      - setWeight: 5
      - pause: { duration: 10m }
      - analysis: { templates: [{ templateName: tasa-exito }] }   # consulta Prometheus
      - setWeight: 25
      - pause: { duration: 10m }
      - setWeight: 100
```

Si el análisis falla (p. ej. tasa de éxito < 99 %), el rollout se aborta y vuelve a v1 sin intervención humana. Herramientas: **Argo Rollouts**, **Flagger**. Variante a gran escala: **ring deployment** (anillos: interno → 1 % → una región → todo), combinable con Deployment Stamps.

---

## 5. Feature Flags (Feature Toggles)

**Problema.** Desplegar código y **lanzar** una funcionalidad son decisiones distintas; acoplarlas obliga a ramas de larga duración y convierte cada despliegue en un lanzamiento arriesgado.

**Solución.** Envolver la funcionalidad en una **condición evaluada en tiempo de ejecución**, controlada por configuración externa. Se despliega el código "apagado" y se activa después, para todos o para un subconjunto.

Tipos (clasificación de Pete Hodgson en martinfowler.com):

| Tipo | Propósito | Vida |
|---|---|---|
| **Release toggle** | Ocultar código incompleto en `main` (*trunk-based development*) | Días o semanas — **elimínalo después** |
| **Experiment toggle** | A/B testing | Lo que dure el experimento |
| **Ops toggle** (*kill switch*) | Desactivar una funcionalidad costosa bajo carga | Puede ser permanente |
| **Permission toggle** | Funcionalidades premium o beta por usuario | Larga |

```csharp
// Microsoft.FeatureManagement + Azure App Configuration
builder.Configuration.AddAzureAppConfiguration(o => o
    .Connect(new Uri(builder.Configuration["AppConfig:Endpoint"]!), new DefaultAzureCredential())
    .UseFeatureFlags(f => f.SetRefreshInterval(TimeSpan.FromSeconds(30))));
builder.Services.AddFeatureManagement();

app.MapGet("/checkout", async (IFeatureManager features) =>
    await features.IsEnabledAsync("NuevoCheckout")
        ? Results.Ok(await NuevoCheckout())
        : Results.Ok(await CheckoutActual()));
```

Otras opciones: LaunchDarkly, Unleash, Flagsmith; **OpenFeature** (CNCF) como API estándar independiente del proveedor.

**Trade-offs.** ✅ Desacopla despliegue y lanzamiento, rollback sin redesplegar, lanzamientos graduales. ❌ **Deuda técnica**: flags que nadie retira multiplican caminos de código y combinaciones a probar. Pon fecha de caducidad a cada release toggle.

---

## 6. External Configuration Store

**Problema.** La configuración dentro del paquete de despliegue obliga a redesplegar para cambiarla, se duplica entre servicios y entornos, y mezcla secretos con código.

**Solución.** Guardar la configuración en un **almacén centralizado externo** que las aplicaciones leen al arrancar y, opcionalmente, refrescan en caliente.

| Tipo de dato | Almacén |
|---|---|
| Configuración y feature flags | Azure App Configuration, AWS AppConfig / Parameter Store, Consul, etcd |
| **Secretos** | **Azure Key Vault**, AWS Secrets Manager, HashiCorp Vault — ver [[CN 09 - Seguridad]] |

**Buenas prácticas:**
- Separar configuración (no sensible) de secretos.
- Etiquetas o claves por entorno (`Pedidos:Timeout` con label `prod`).
- **Validar la configuración al arrancar** (`ValidateOnStart()` en .NET) para fallar rápido en vez de en la primera petición.
- Refresco en caliente sólo para lo que lo soporte de verdad (un pool de conexiones no se reconfigura solo); usa una clave *sentinel* para refrescar de forma atómica.
- Caché local y valores por defecto: si el almacén no responde al arrancar, ¿qué haces?

```csharp
builder.Services.AddOptions<PedidosOptions>()
    .BindConfiguration("Pedidos")
    .ValidateDataAnnotations()
    .ValidateOnStart();       // la app no arranca con configuración inválida
```

---

## 7. Configuración en Kubernetes

Patrones de configuración de *Kubernetes Patterns*:

| Patrón | Mecanismo | Nota |
|---|---|---|
| **EnvVar Configuration** | `env` / `envFrom` desde ConfigMap/Secret | Simple; los cambios requieren reiniciar el Pod |
| **Configuration Resource** | ConfigMap/Secret montado como **volumen** | Los ficheros se actualizan solos (con retraso); la app debe recargarlos |
| **Immutable Configuration** | ConfigMap con `immutable: true` o con el hash en el nombre (`config-7f3a2`) | Cambiar config = nuevo despliegue → reproducible y con rollback. Kustomize `configMapGenerator` y Helm (checksum en anotaciones) lo facilitan |
| **Configuration Template** | Plantilla + valores procesados por un init container | Para configuraciones grandes que varían poco entre entornos |

Para secretos: **External Secrets Operator** o el **Secrets Store CSI Driver** (con el proveedor de Azure Key Vault) sincronizan desde el almacén externo. Ver [[05 - Configuración y almacenamiento]].

---

## 8. Cambios de base de datos sin downtime: Expand and Contract

**Problema.** Rolling, Blue-Green y Canary implican que **dos versiones** de la app usan la **misma** BD a la vez. Un cambio de esquema incompatible (renombrar una columna) rompe a una de ellas y hace imposible el rollback.

**Solución (*parallel change*, Fowler).** Dividir el cambio en pasos compatibles:

| Paso | Esquema | App | Ejemplo: renombrar `nombre` → `nombre_completo` |
|---|---|---|---|
| 1. **Expand** | Añadir lo nuevo sin quitar lo viejo | v1 sigue funcionando | Añadir columna `nombre_completo` (nullable) |
| 2. **Migrar** | Copiar datos | v2 escribe en **ambas**, lee la nueva | Backfill por lotes |
| 3. **Cambiar lecturas** | — | v3 lee y escribe sólo lo nuevo | — |
| 4. **Contract** | Eliminar lo viejo | — | Borrar `nombre`, cuando ya no haya ninguna versión que la use |

Cada paso es un despliegue independiente y reversible. Más lento, pero sin downtime ni riesgo de rollback imposible. Lo mismo aplica a **contratos de API y de eventos**: añadir es seguro; quitar requiere un periodo de convivencia.

---

## 9. Las métricas DORA

El programa DORA (*DevOps Research and Assessment*, ahora en Google Cloud) identifica métricas que se correlacionan con el rendimiento de entrega de software:

| Métrica | Qué mide |
|---|---|
| **Deployment frequency** | Con qué frecuencia se despliega a producción |
| **Lead time for changes** | Tiempo desde el commit hasta producción |
| **Change failure rate** | % de despliegues que causan un fallo en producción |
| **Time to restore service** (*failed deployment recovery time*) | Cuánto se tarda en recuperarse de un fallo |

Los patrones de este capítulo existen para mejorar las cuatro a la vez: desplegar más, más rápido, con menos fallos y recuperándose antes. Nota: los informes recientes de DORA han ido ajustando nombres y añadiendo métricas (p. ej. *rework rate*); los cuatro anteriores siguen siendo la referencia básica.

---

## Puntos clave

- IaC + GitOps: Git es la fuente de verdad; el clúster se reconcilia solo.
- **Rolling** por defecto; **Blue-Green** para rollback instantáneo; **Canary** para limitar el radio de impacto; **Shadow** para validar sin afectar (cuidado con escrituras).
- Progressive delivery = canary con análisis automático de métricas.
- **Feature flags** separan desplegar de lanzar; retíralos o se vuelven deuda.
- Configuración externa, **validada al arrancar**; secretos en un almacén específico.
- Cambios de BD y de contratos con **expand and contract**.

## Errores comunes

- Blue-Green con una migración de BD incompatible → rollback imposible.
- Canary sin métricas que comparar (sólo "parece que va bien").
- Feature flags eternos.
- Secretos en ConfigMaps o en el repo de GitOps sin cifrar.
- Cambiar un ConfigMap montado como variable de entorno y esperar que la app lo vea sin reiniciar.

## 🎯 Para entrevistas

- *"Blue-Green vs Canary"* → conmutación total con rollback instantáneo y 2× infra, vs exposición gradual con menor radio de impacto; ambos requieren compatibilidad de BD.
- *"¿Cómo renombras una columna sin downtime?"* → expand and contract en 4 pasos.
- *"¿Qué es GitOps y qué ventaja tiene frente a un pipeline push?"* → pull desde el clúster, reconciliación continua, Git como auditoría, sin credenciales del clúster en el CI.
- *"¿Qué riesgos tienen los feature flags?"* → deuda, combinatoria de pruebas, flags olvidados que cambian el comportamiento.

## Referencias

- Microsoft, *External Configuration Store*: https://learn.microsoft.com/azure/architecture/patterns/external-configuration-store
- OpenGitOps, *GitOps Principles*: https://opengitops.dev/
- Martin Fowler, *BlueGreenDeployment*, *CanaryRelease*, *ParallelChange*: https://martinfowler.com/bliki/
- Pete Hodgson, *Feature Toggles (aka Feature Flags)*: https://martinfowler.com/articles/feature-toggles.html
- Argo Rollouts: https://argoproj.github.io/rollouts/ · Flagger: https://flagger.app/
- OpenFeature: https://openfeature.dev/
- DORA: https://dora.dev/
- Microsoft, *Feature management in .NET*: https://learn.microsoft.com/azure/azure-app-configuration/feature-management-dotnet-reference

---
⬅️ [[CN 07 - Escalabilidad y rendimiento]] · ➡️ [[CN 09 - Seguridad]]
