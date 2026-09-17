---
tags: [az-104, azure, monitorizacion, alertas, action-groups]
modulo: Monitorización y mantenimiento
peso_examen: Muy alto
---

# Alertas, grupos de acciones y reglas de procesamiento

## ¿Qué es?

Una **alerta** detecta proactivamente una condición (una métrica supera un umbral, aparece un evento en el registro de actividad, una consulta KQL devuelve resultados) y ejecuta **acciones** a través de un **grupo de acciones**. Las **reglas de procesamiento de alertas** modifican qué ocurre con las alertas generadas (suprimirlas o añadir acciones).

## Conceptos clave

### Tipos de regla de alerta 🧠

| Tipo | Se basa en | Latencia | Ejemplo |
|---|---|---|---|
| **Alerta de métrica** | Métricas de plataforma o personalizadas | ~1 min | CPU > 80 % durante 5 min |
| **Alerta de búsqueda de registros (log search)** | Consulta **KQL** sobre logs | Minutos | Más de 10 errores en 5 min |
| **Alerta del registro de actividad** | Eventos del plano de control | ~1 min | Alguien eliminó una VM; **Service Health**; **Resource Health** |
| **Alerta inteligente / Smart detection** ➕ | Detección automática de anomalías | Variable | Anomalías en Application Insights |
| **Alerta de prueba web / disponibilidad** ➕ | Application Insights | Minutos | La URL no responde |

### Anatomía de una regla 🧠
- **Ámbito (scope)**: recurso, grupo de recursos o suscripción (puede ser multi-recurso para alertas de métrica del mismo tipo y región).
- **Condición**: señal, **agregación** (Average, Max…), **operador**, **umbral** (estático o **dinámico**, basado en aprendizaje), **granularidad** (periodo de agregación) y **frecuencia de evaluación**.
- **Acciones**: uno o varios **grupos de acciones**.
- **Detalles**: nombre, descripción, **severidad (Sev 0 crítica … Sev 4 verbosa)**, habilitada, **resolución automática**.
- **Estado de la alerta**: *New → Acknowledged → Closed*; **estado de supervisión** (Fired/Resolved).
- **Costes**: las reglas de métrica y de log tienen coste por regla/serie; las del registro de actividad son gratuitas.

### Grupo de acciones (action group) 🧠
Define **a quién se notifica** y **qué se ejecuta**:
- **Notificaciones**: correo electrónico, SMS, notificación push de la app de Azure, voz, **correo a un rol de ARM** (Owner, Contributor, Monitoring Reader…).
- **Acciones**: **Webhook**, **Logic App**, **Azure Function**, **Automation Runbook**, **ITSM**, **Event Hub**, webhook seguro.
- Se **reutiliza** en muchas reglas; tiene nombre corto para SMS.
- Límites orientativos: 1000 acciones por tipo, límites de tasa por SMS/voz.

### Reglas de procesamiento de alertas (alert processing rules) 🧠
- **Suprimir notificaciones** durante una ventana (mantenimiento planificado) para un ámbito.
- **Aplicar un grupo de acciones** a todas las alertas de un ámbito sin editar cada regla.
- Se programan **una vez o de forma recurrente** (diaria, semanal) y se filtran por recurso, severidad, etiqueta, nombre de alerta, etc.
- Antes se llamaban *action rules*.

## Cómo funciona

```
Señal (métrica/log/activity) ──► Regla de alerta (condición) ──► Alerta (severidad, estado)
                                                                    │
                                                    Regla de procesamiento (suprimir / añadir acciones)
                                                                    │
                                                          Grupo de acciones ──► correo, SMS, webhook, runbook…
```

```bash
# Grupo de acciones
az monitor action-group create -g rg-mon -n ag-ops --short-name OpsTeam \
  --action email admin admin@contoso.com --action sms movil 34 600000000
# Alerta de métrica
az monitor metrics alert create -g rg-mon -n cpu-alta --scopes <vmId> \
  --condition "avg Percentage CPU > 80" --window-size 5m --evaluation-frequency 1m \
  --severity 2 --action ag-ops --description "CPU alta en vm-web01"
# Alerta del registro de actividad (eliminación de VMs)
az monitor activity-log alert create -g rg-mon -n vm-eliminada --scope /subscriptions/<subId> \
  --condition category=Administrative and operationName=Microsoft.Compute/virtualMachines/delete \
  --action-group ag-ops
# Alerta de búsqueda de registros
az monitor scheduled-query create -g rg-mon -n errores-app --scopes <lawId> \
  --condition "count 'Heartbeat | where TimeGenerated > ago(5m)' < 1" \
  --evaluation-frequency 5m --window-size 5m --severity 1 --action-groups <agId>
# Regla de procesamiento (suprimir por la noche)
az monitor alert-processing-rule create -g rg-mon -n mantenimiento --rule-type RemoveAllActionGroups \
  --scopes /subscriptions/<subId> --schedule-recurrence-type Daily --schedule-start-time 22:00 --schedule-end-time 07:00
az monitor metrics alert list -g rg-mon -o table
```

```powershell
$ag = New-AzActionGroupReceiver -Name admin -EmailReceiver -EmailAddress admin@contoso.com
Set-AzActionGroup -ResourceGroupName rg-mon -Name ag-ops -ShortName OpsTeam -Receiver $ag
Add-AzMetricAlertRuleV2 -Name cpu-alta -ResourceGroupName rg-mon -TargetResourceId $vm.Id `
  -Condition (New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" -Operator GreaterThan -Threshold 80 -TimeAggregation Average) `
  -WindowSize 00:05:00 -Frequency 00:01:00 -Severity 2 -ActionGroupId $agId
```

Portal: Monitor → **Alertas** → Reglas de alerta → Crear; Monitor → **Grupos de acciones**; Monitor → Alertas → **Reglas de procesamiento de alertas**.

## Configuración relevante para el examen

| Escenario | Configuración |
|---|---|
| Avisar si la CPU supera el 80 % durante 5 minutos | Alerta de **métrica** (Average, 5 min) |
| Avisar cuando alguien elimine una VM | Alerta del **registro de actividad** (operación delete) |
| Avisar si aparecen más de 10 errores en los logs | Alerta de **búsqueda de registros** (KQL) |
| Avisar de incidencias de Azure que afectan a mis servicios | Alerta de **Service Health** (tipo registro de actividad) |
| No recibir notificaciones durante la ventana de mantenimiento | **Regla de procesamiento** con supresión programada |
| Añadir un grupo de acciones a todas las alertas de una suscripción | **Regla de procesamiento** |
| Ejecutar un runbook que apague una VM al saltar la alerta | Grupo de acciones con **Automation Runbook** |
| Notificar a todos los propietarios de la suscripción | Acción **correo a rol de ARM** |
| Umbral difícil de fijar | **Umbrales dinámicos** |
| Alerta para varias VMs con una sola regla | Regla de métrica **multi-recurso** (mismo tipo y región) |

## Ejemplo

Operaciones crea el grupo de acciones `ag-ops` (correo al equipo + SMS al responsable de guardia + runbook de reinicio). Define: alerta de métrica *Percentage CPU > 85 % (Average, 10 min, Sev 2)* sobre todas las VMs de producción; alerta de registro de actividad para `Microsoft.Compute/virtualMachines/delete` (Sev 1); y una **regla de procesamiento** que suprime notificaciones todos los domingos de 02:00 a 06:00 durante la ventana de parcheo.

## Comparaciones

| Tipo de alerta | Señal | Coste | Cuándo utilizarla |
|---|---|---|---|
| **Métrica** | Números | Por serie temporal | Rendimiento y umbrales |
| **Log search** | KQL | Por regla y frecuencia | Patrones complejos, texto, correlación |
| **Registro de actividad** | Operaciones | **Gratis** | Auditoría, Service Health, Resource Health |
| **Smart detection** | Anomalías | Incluido | Aplicaciones (App Insights) |

| Elemento | Responde a |
|---|---|
| **Regla de alerta** | ¿Cuándo se dispara? |
| **Grupo de acciones** | ¿A quién se avisa y qué se ejecuta? |
| **Regla de procesamiento** | ¿Se notifica realmente o se suprime? |

## 💻 Laboratorio: alerta de eliminación de VM

1. Crear el grupo de acciones `ag-lab` con tu correo y confirmar la suscripción.
2. Crear una alerta del **registro de actividad** para la operación `Microsoft.Compute/virtualMachines/delete` en el ámbito de la suscripción.
3. Crear una **regla de procesamiento** que suprima notificaciones de 22:00 a 07:00.
4. Eliminar una VM de prueba y comprobar el correo y la alerta en Monitor → Alertas.
5. Crear también una alerta de métrica de CPU > 70 % y ver los estados (New/Acknowledged/Closed).

## AZ-104 Exam Tips

- ⭐ **Regla de alerta = condición**; **grupo de acciones = notificación/acción**; **regla de procesamiento = supresión o acciones globales**.
- 🔥 🧠 Alerta de **métrica** para rendimiento; de **registro de actividad** para operaciones (y Service Health); de **búsqueda de registros** para KQL.
- 🔥 🧠 Severidades **Sev 0 (crítica) a Sev 4**.
- 🧠 Los grupos de acciones se **reutilizan** entre reglas.
- 🧠 Las reglas de procesamiento sirven para **ventanas de mantenimiento**.
- 💻 `az monitor action-group create`, `az monitor metrics alert create`, `activity-log alert create`, `alert-processing-rule create`.
- ⚠️ Una alerta no actúa por sí sola: sin grupo de acciones no notifica nada.

## Errores comunes

- Crear la regla sin asociar grupo de acciones.
- Usar alerta de métrica cuando el enunciado habla de "alguien eliminó/creó" (es del registro de actividad).
- Deshabilitar reglas una a una en vez de usar una regla de procesamiento.

## Preguntas que podrían aparecer

**1.** Necesitas recibir un correo cuando cualquier usuario elimine una máquina virtual en la suscripción. ¿Qué tipo de alerta creas?
- A) De métrica · B) Del registro de actividad · C) De búsqueda de registros · D) Smart detection

<details><summary>Respuesta</summary>

**B.** Las operaciones del plano de control se detectan con alertas del registro de actividad.
</details>

**2.** Durante la ventana de mantenimiento mensual no quieres recibir notificaciones de ninguna alerta de un grupo de recursos, sin desactivar las reglas. ¿Qué configuras?
- A) Un grupo de acciones vacío · B) Una regla de procesamiento de alertas con supresión programada · C) Severidad 4 · D) Umbrales dinámicos

<details><summary>Respuesta</summary>

**B.** Las reglas de procesamiento permiten suprimir notificaciones en ventanas programadas.
</details>

**3.** ¿Qué elemento define que se envíe un SMS y se ejecute un runbook cuando salta una alerta?
- A) La regla de alerta · B) El grupo de acciones · C) La regla de procesamiento · D) La configuración de diagnóstico

<details><summary>Respuesta</summary>

**B.** El grupo de acciones contiene notificaciones y acciones.
</details>

## Relacionado

- [[02 - Métricas en Azure Monitor]]
- [[04 - Consultas KQL]]
- [[12 - Informes y alertas de copias de seguridad]]
- [[16 - Administración de costes (presupuestos, alertas y Advisor)]]
- [[00 - Índice - Monitorización y mantenimiento]]
