---
tags: [az-104, azure, monitorizacion, backup, repaso]
modulo: Monitorización y mantenimiento
---

# 🎯 Repaso final · Monitorización y mantenimiento (10-15 %)

## 1. Los conceptos más importantes

1. **Métricas** automáticas, **93 días**, agregaciones y dimensiones; **logs** requieren configuración y se consultan con KQL. ([[01 - Azure Monitor - visión general]], [[02 - Métricas en Azure Monitor]])
2. **Configuración de diagnóstico**: sin ella no hay logs de recurso; destinos **Log Analytics, Storage, Event Hub**. ([[03 - Logs - Log Analytics y configuración de diagnóstico]])
3. **Agente actual = AMA + DCR**; el Log Analytics Agent está **retirado**. ([[03 - Logs - Log Analytics y configuración de diagnóstico]])
4. **KQL**: `Tabla | where | summarize | order by | render`; tablas Heartbeat, Perf, Event, AzureActivity. ([[04 - Consultas KQL]])
5. **Alertas**: métrica / log / registro de actividad; **grupo de acciones** para notificar y actuar; **reglas de procesamiento** para suprimir. ([[05 - Alertas, grupos de acciones y reglas de procesamiento]])
6. **Insights**: VM (AMA + DCR + Dependency Agent), Storage, Network. ([[06 - Azure Monitor Insights (VM, Storage, Network)]])
7. **Network Watcher**: diagnóstico puntual; **Connection Monitor** continuo; **VNet flow logs** sustituyen a los de NSG. ([[07 - Network Watcher y Connection Monitor]])
8. **Vaults**: **RSV** (VMs, Files, SQL/SAP en VM, MARS, **ASR**) vs **Backup vault** (discos, blobs, PostgreSQL, AKS); redundancia fija tras el primer elemento; soft delete 14 días. ([[08 - Azure Backup - Recovery Services vault y Backup vault]])
9. **Políticas**: Standard (diaria, snapshot 1-5 días) vs **Enhanced** (cada 4 h, snapshot 1-30 días, Trusted Launch). ([[09 - Directivas de copia de seguridad]])
10. **Restauración**: crear VM nueva / **reemplazar discos** / restaurar discos / **archivos individuales**; CRR con GRS. ([[10 - Operaciones de copia de seguridad y restauración]])
11. **ASR**: puntos crash cada **5 min**, retención por defecto **24 h**, **test failover** sin impacto + limpieza, plan de recuperación, failover → commit → reprotect → failback. ([[11 - Azure Site Recovery]])
12. **Informes**: Backup Reports requiere diagnóstico a Log Analytics; **Backup center** unifica todo. ([[12 - Informes y alertas de copias de seguridad]])

## 2. Tabla de decisión rápida

| Si el enunciado dice… | Respuesta |
|---|---|
| "quién eliminó el recurso" | Registro de actividad (90 días) |
| "consultar logs con KQL" | Configuración de diagnóstico → Log Analytics |
| "archivar 7 años barato" | Configuración de diagnóstico → cuenta de almacenamiento |
| "enviar a un SIEM externo" | Event Hub |
| "recopilar eventos de Windows/syslog" | AMA + DCR |
| "memoria y dependencias de las VMs" | VM Insights (+ Dependency Agent) |
| "CPU > 80 % durante 5 min" | Alerta de métrica |
| "cuando alguien borre una VM" | Alerta del registro de actividad |
| "más de N errores en los logs" | Alerta de búsqueda de registros |
| "no notificar durante el mantenimiento" | Regla de procesamiento de alertas |
| "enviar SMS y ejecutar runbook" | Grupo de acciones |
| "vigilar latencia de forma continua" | Connection Monitor |
| "qué regla NSG bloquea" | IP flow verify |
| "proteger VMs y Azure Files" | Recovery Services vault |
| "proteger discos, blobs o PostgreSQL" | Backup vault |
| "copias cada 4 horas" | Política Enhanced |
| "restaurar conservando nombre e IP" | Reemplazar discos existentes |
| "recuperar unos archivos" | Restauración de archivos individuales |
| "restaurar en la región secundaria" | Cross Region Restore (GRS) |
| "RPO de minutos, caída de región" | Azure Site Recovery |
| "probar el DR sin afectar a producción" | Test failover (+ limpieza) |
| "orden de arranque de una app multicapa" | Plan de recuperación |
| "informe de trabajos y almacenamiento" | Backup Reports (Log Analytics) |

## 3. Números que debo memorizar

| Dato | Valor |
|---|---|
| Retención de métricas de plataforma | 93 días |
| Retención del registro de actividad | 90 días |
| Retención de Log Analytics | 30 días incluidos, hasta 730 (archivo hasta 12 años) |
| Configuraciones de diagnóstico por recurso | 5 |
| Severidades de alerta | Sev 0 a Sev 4 |
| Soft delete de Backup | 14 días (configurable hasta 180) |
| Instant restore Standard / Enhanced | 1-5 días (2) / 1-30 días (7) |
| Frecuencia Enhanced | cada 4, 6, 8, 12 h o diaria |
| Retención máxima de copias de VM | 9999 días |
| Puntos crash-consistent de ASR | cada 5 minutos |
| Retención de puntos de ASR | 24 h por defecto (hasta 15 días) |
| Copias diarias del agente MARS | 3 |
| Retirada de NSG flow logs | 30/09/2027 |

## 4. Diferencias que más fácil puedo confundir

| Pareja | Diferencia |
|---|---|
| Métrica vs log | Número automático vs evento configurado |
| Activity log vs resource log | Plano de control vs plano de datos |
| AMA vs MMA | Actual (con DCR) vs retirado |
| Alerta de métrica vs de actividad vs de log | Umbral numérico vs operación vs consulta KQL |
| Grupo de acciones vs regla de procesamiento | A quién se avisa vs si se avisa |
| Insights vs workbook | Preconfigurado vs personalizado |
| Connection troubleshoot vs Connection Monitor | Puntual vs continuo |
| RSV vs Backup vault | Cargas clásicas + ASR vs discos/blobs/PostgreSQL/AKS |
| Backup vs Site Recovery | Puntos de restauración vs replicación y failover |
| Standard vs Enhanced (política) | Diaria vs cada 4 h |
| Instant restore vs retención del vault | Snapshot local rápido vs copia en el almacén |
| Test failover vs failover | Aislado sin impacto vs real |
| Commit vs reprotect vs failback | Confirmar vs invertir replicación vs volver |
| Soft delete vs retención | Papelera de 14 días vs política |
| Alertas integradas vs Azure Monitor | Solo correo vs cualquier acción |

## 5. Checklist de dominio

- [ ] Sé la diferencia entre métricas y logs y dónde vive cada uno.
- [ ] Sé crear configuraciones de diagnóstico con sus tres destinos.
- [ ] Sé que el agente actual es AMA con DCR y para qué sirve cada tabla.
- [ ] Sé leer una consulta KQL y elegir la correcta.
- [ ] Sé crear los tres tipos de alerta y asociar grupos de acciones.
- [ ] Sé cuándo usar una regla de procesamiento de alertas.
- [ ] Sé qué aporta cada Insight y sus requisitos.
- [ ] Sé qué herramienta de Network Watcher usar en cada caso.
- [ ] Sé elegir el tipo de almacén según la carga de trabajo.
- [ ] Sé las reglas de redundancia, soft delete y CRR.
- [ ] Sé configurar políticas Standard y Enhanced.
- [ ] Sé las cuatro formas de restaurar una VM y cuándo usar cada una.
- [ ] Sé el flujo completo de ASR y qué hace cada operación.
- [ ] Sé configurar informes y alertas de copias de seguridad.

## 6. Preguntas de repaso

**1.** Los registros de un Key Vault no aparecen en Log Analytics. ¿Qué falta?
- A) Un agente · B) Una configuración de diagnóstico · C) Una alerta · D) Application Insights

<details><summary>Respuesta</summary>**B.**</details>

---

**2.** ¿Qué agente recopila hoy los contadores de rendimiento del sistema operativo invitado?
- A) Log Analytics Agent · B) Azure Monitor Agent con DCR · C) Dependency Agent · D) MARS

<details><summary>Respuesta</summary>**B.**</details>

---

**3.** Quieres saber si el agente de una VM envía datos. ¿Qué tabla consultas?
- A) Perf · B) Event · C) Heartbeat · D) AzureMetrics

<details><summary>Respuesta</summary>**C.**</details>

---

**4.** Necesitas avisar cuando alguien cree una cuenta de almacenamiento en la suscripción. ¿Qué tipo de alerta?
- A) Métrica · B) Registro de actividad · C) Log search · D) Smart detection

<details><summary>Respuesta</summary>**B.**</details>

---

**5.** ¿Qué elemento define que se envíe un correo y se ejecute una Logic App cuando salta una alerta?
- A) La regla de alerta · B) El grupo de acciones · C) La regla de procesamiento · D) El workbook

<details><summary>Respuesta</summary>**B.**</details>

---

**6.** ¿Dónde se protegen los blobs de una cuenta de almacenamiento?
- A) Recovery Services vault · B) Backup vault · C) Key Vault · D) Storage account

<details><summary>Respuesta</summary>**B.**</details>

---

**7.** Proteges una VM y luego quieres cambiar la redundancia del vault a LRS. ¿Es posible?
- A) Sí, en cualquier momento · B) No, solo antes de proteger el primer elemento · C) Solo con soft delete desactivado · D) Solo con CRR

<details><summary>Respuesta</summary>**B.**</details>

---

**8.** Necesitas copias cada 4 horas y soporte de Trusted Launch. ¿Qué política?
- A) Standard · B) Enhanced · C) MARS · D) Operacional

<details><summary>Respuesta</summary>**B.**</details>

---

**9.** Una VM corrupta debe recuperarse conservando su IP y su nombre. ¿Qué opción?
- A) Crear VM nueva · B) Reemplazar discos existentes · C) Restaurar discos · D) Failover

<details><summary>Respuesta</summary>**B.**</details>

---

**10.** ¿Cada cuánto crea Site Recovery puntos coherentes con el bloqueo?
- A) 1 min · B) 5 min · C) 1 h · D) 24 h

<details><summary>Respuesta</summary>**B.**</details>

---

**11.** Tras un test failover, ¿qué debes hacer para no seguir pagando los recursos de prueba?
- A) Commit · B) Limpiar la conmutación por error de prueba · C) Reprotect · D) Failback

<details><summary>Respuesta</summary>**B.**</details>

---

**12.** Backup Reports aparece vacío. ¿Cuál es la causa más probable?
- A) Falta el rol Owner · B) No se ha creado la configuración de diagnóstico del vault hacia Log Analytics o no han pasado 24 h · C) El vault es de tipo Backup vault · D) Falta soft delete

<details><summary>Respuesta</summary>**B.**</details>

> [!tip] Última pasada
> **Métricas 93 días**, **sin diagnóstico no hay logs**, **AMA + DCR**, **alerta de actividad para "quién hizo qué"**, **RSV vs Backup vault**, **Enhanced = cada 4 h**, **reemplazar discos conserva identidad**, **ASR 5 min / 24 h / test failover + limpieza**.

Volver: [[00 - Índice - Monitorización y mantenimiento]] · [[00 - AZ-104 Índice general (MOC)]]
