---
tags: [az-104, azure, computo, vmss, escalado]
modulo: Cómputo
peso_examen: Muy alto
---

# Virtual Machine Scale Sets (VMSS)

## ¿Qué es?

Un **conjunto de escalado de máquinas virtuales** crea y administra un grupo de VMs **idénticas** con **balanceo de carga** y **escalado automático** (horizontal). Azure añade o quita instancias según la demanda o una programación.

## ¿Para qué sirve?

- Aplicaciones web y de proceso que deben crecer y decrecer con la carga.
- Grandes despliegues de cómputo (hasta 1000 instancias).
- Alta disponibilidad con distribución automática en FD/zonas.

## Conceptos clave

### Modos de orquestación 🧠

| | **Uniform** | **Flexible** |
|---|---|---|
| Modelo | VMs idénticas desde un perfil de VM | VMs estándar de Azure |
| API | API de VMSS (instancias) | API de VM estándar |
| Tamaños | Un solo tamaño | **Mezcla de tamaños** y de Spot/regular |
| Distribución | FD dentro de la región/zonas | **Fault domains explícitos** (1, 2, 3 o max spreading) |
| Máximo instancias | 1000 (600 con imagen propia) | **1000** |
| Casos | Cargas homogéneas sin estado | Cargas heterogéneas, quorum, mezcla Spot |
| Recomendado hoy | — | **Flexible es el modo recomendado por Microsoft** para nuevas cargas |

### Escalado
- **Manual**: fijar el número de instancias.
- **Automático (autoscale)** basado en **reglas**:
  - **Métrica**: CPU %, cola de mensajes, métrica personalizada, métricas del propio host o del invitado (requiere agente).
  - Elementos de una regla 🧠: métrica, **agregación** (Average, Max…), **operador y umbral** (p. ej. > 70 %), **duración/ventana** (p. ej. 10 minutos), **acción** (aumentar/disminuir en N o hasta N), **período de recuperación (cool-down)** (p. ej. 5 minutos).
  - **Escalar horizontalmente (scale out)** y **reducir horizontalmente (scale in)** son reglas separadas.
  - **Instancias mínimas, máximas y predeterminadas**.
  - **Perfiles**: por defecto, **basado en programación (recurrencia)** o por intervalo de fechas fijo.
  - **Política de reducción (scale-in policy)**: Default, NewestVM, OldestVM.
- **Escalado predictivo** ➕ (preview/GA según región) para cargas con patrón.
- **Flexible orchestration** también soporta autoscale.

### Otros conceptos
- **Upgrade policy** 🧠: **Manual** (tú actualizas cada instancia), **Automatic** (Azure actualiza todas a la vez), **Rolling** (por lotes con comprobación de estado, requiere health probe o extensión Application Health).
- **Automatic OS image upgrades**: mantiene las instancias con la última imagen.
- **Health probe / Application Health Extension**: necesario para rolling upgrades y para reparación automática de instancias (**automatic instance repair**).
- **Zonas**: un scale set puede abarcar **varias zonas** (zone balancing). Con Flexible se combinan zonas y FD.
- **Balanceo**: se asocia a un **Load Balancer** (capa 4) o **Application Gateway** (capa 7).
- **Overprovisioning** (Uniform): Azure crea instancias extra temporalmente y elimina las que fallan, para acelerar el despliegue.
- **Spot**: instancias de bajo coste con desalojo.
- **Extensiones y cloud-init** para configurar las instancias.
- **Custom images / Compute Gallery** para escalar rápido con la app ya instalada.

## Cómo funciona

```bash
az vmss create --resource-group rg-web --name vmss-web --orchestration-mode Flexible \
  --image Ubuntu2204 --instance-count 2 --vm-sku Standard_D2s_v5 \
  --zones 1 2 3 --load-balancer lb-web --upgrade-policy-mode Rolling \
  --admin-username azureadmin --generate-ssh-keys

# Escalado manual
az vmss scale --resource-group rg-web --name vmss-web --new-capacity 5
# Autoscale
az monitor autoscale create --resource-group rg-web --resource vmss-web --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-web --min-count 2 --max-count 10 --count 2
az monitor autoscale rule create --resource-group rg-web --autoscale-name autoscale-web \
  --condition "Percentage CPU > 70 avg 10m" --scale out 2
az monitor autoscale rule create --resource-group rg-web --autoscale-name autoscale-web \
  --condition "Percentage CPU < 30 avg 10m" --scale in 1
# Actualizar instancias (Manual upgrade policy)
az vmss update-instances --resource-group rg-web --name vmss-web --instance-ids "*"
az vmss list-instances --resource-group rg-web --name vmss-web -o table
az vmss reimage --resource-group rg-web --name vmss-web --instance-id 1
```

```powershell
New-AzVmss -ResourceGroupName rg-web -VMScaleSetName vmss-web -OrchestrationMode Flexible -Zone 1,2,3 -Credential (Get-Credential)
Update-AzVmss -ResourceGroupName rg-web -VMScaleSetName vmss-web -SkuCapacity 5
Add-AzVmssExtension ...
```

Portal: **Conjuntos de escalado de máquinas virtuales** → Crear → orquestación, zonas, tamaño, imagen, instancias, balanceador, directiva de actualización, **Escalado** (manual/personalizado con reglas) y **Mantenimiento**.

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| Añadir 2 instancias cuando la CPU media supere el 70 % durante 10 min | Regla de scale out: métrica CPU, Average, > 70, 10 min, aumentar en 2 |
| Evitar oscilaciones (flapping) | **Cool-down** suficiente y umbrales separados (out > 70 / in < 30) |
| Más instancias en horario laboral | Perfil de **recurrencia** (schedule-based) |
| Mezclar tamaños y VMs Spot en el mismo conjunto | Orquestación **Flexible** |
| Actualizar la imagen sin cortar el servicio | Upgrade policy **Rolling** + health probe |
| Reemplazar instancias no saludables automáticamente | **Automatic instance repair** + Application Health Extension |
| Escalar según la longitud de una cola de Service Bus | Regla con métrica de otro recurso |
| Instalar la app en cada instancia nueva | Extensión Custom Script / cloud-init / imagen de Compute Gallery |
| HA máxima | Varias **zonas** + Load Balancer estándar |

## Ejemplo

Una tienda online sufre picos a mediodía. Se crea un VMSS Flexible con 3 zonas, mínimo 2 y máximo 12 instancias, detrás de un Application Gateway. Reglas: out +2 si CPU media > 70 % durante 10 min (cool-down 5 min); in -1 si CPU media < 30 % durante 10 min. Además, un perfil programado fija un mínimo de 6 instancias de 12:00 a 16:00 de lunes a viernes. Las instancias se crean desde una imagen de Compute Gallery con la app preinstalada.

## Comparaciones

| Opción | Uso | Ventajas | Cuándo utilizarla |
|---|---|---|---|
| **VMSS Flexible** | Grupo de VMs heterogéneo | Mezcla tamaños/Spot, API de VM, FD explícitos | Recomendado para nuevas cargas |
| **VMSS Uniform** | VMs idénticas | Modelo clásico, overprovisioning | Cargas homogéneas existentes |
| **VMs individuales + availability set** | Pocas VMs estables | Control total | Sin necesidad de escalado |
| **App Service (escalado)** | Web PaaS | Sin gestionar SO | Aplicaciones web |
| **Container Apps / AKS** | Contenedores | Escalado por eventos | Microservicios |

## 💻 Laboratorio: VMSS con autoscale

1. Crear `vmss-lab` (Flexible, Ubuntu, 2 instancias, zonas 1-3) con cloud-init que instale nginx y un Load Balancer público.
2. Comprobar el acceso por la IP del balanceador y ver las instancias con `az vmss list-instances`.
3. Configurar autoscale: min 2, max 6, out +1 si CPU > 60 % durante 5 min, in -1 si CPU < 25 % durante 5 min.
4. Generar carga (`stress` o bucle) en una instancia y observar el escalado en Azure Monitor.
5. Cambiar la directiva de actualización a Rolling y actualizar la imagen.

## AZ-104 Exam Tips

- 🔥 🧠 **Flexible** permite **mezclar tamaños y Spot** y usa la API de VM; **Uniform** usa un perfil idéntico. Ambos hasta **1000 instancias**.
- 🔥 🧠 Una regla de autoscale = **métrica + agregación + operador + umbral + duración + acción + cool-down**.
- 🧠 Scale out y scale in son **reglas separadas**; define **min, max y default**.
- 🧠 Upgrade policy: **Manual / Automatic / Rolling** (Rolling necesita sonda de estado).
- 💻 `az vmss create/scale/update-instances`, `az monitor autoscale rule create`.
- 📌 Escalado **horizontal** (instancias, VMSS) vs **vertical** (tamaño, resize).
- ⚠️ Con umbrales demasiado juntos se producen oscilaciones; el examen premia cool-down y separación.

## Errores comunes

- Definir solo la regla de scale out y preguntarse por qué nunca baja.
- Usar Uniform cuando se necesitan tamaños mixtos.
- Olvidar la sonda de estado con Rolling upgrades.

## Preguntas que podrían aparecer

**1.** Necesitas un conjunto de escalado que combine instancias Spot y regulares de distintos tamaños. ¿Qué modo de orquestación eliges?
- A) Uniform · B) Flexible · C) Availability set · D) Proximity placement group

<details><summary>Respuesta</summary>

**B.** Solo Flexible permite instancias heterogéneas y mezcla de prioridad Spot/regular.
</details>

**2.** Tu VMSS escala de 2 a 10 instancias y vuelve a 2 constantemente cada pocos minutos. ¿Qué ajustas?
- A) El tamaño de la VM · B) El período de recuperación (cool-down) y la separación entre umbrales de scale out e in · C) La región · D) La imagen

<details><summary>Respuesta</summary>

**B.** Umbrales demasiado próximos y cool-down corto provocan oscilaciones.
</details>

**3.** ¿Qué directiva de actualización aplica una nueva imagen por lotes comprobando el estado de las instancias?
- A) Manual · B) Automatic · C) Rolling · D) Reimage

<details><summary>Respuesta</summary>

**C.** Rolling actualiza en lotes y requiere una sonda de estado o la extensión Application Health.
</details>

## Relacionado

- [[09 - Alta disponibilidad - Availability Sets y Availability Zones]]
- [[13 - Azure Load Balancer]]
- [[12 - Imágenes y Azure Compute Gallery]]
- [[05 - Alertas, grupos de acciones y reglas de procesamiento]]
- [[00 - Índice - Cómputo]]
