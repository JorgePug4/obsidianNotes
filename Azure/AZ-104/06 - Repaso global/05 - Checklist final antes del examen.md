---
tags: [az-104, azure, repaso, checklist, examen]
modulo: Repaso global
---

# ✅ Checklist final antes del examen

## 1. Checklist de contenido por dominio

### Dominio 1 · Identidades y gobernanza (20-25 %)
- [ ] Diferencio roles de Entra ID y roles de Azure RBAC, y sé cuándo hace falta elevar acceso.
- [ ] Sé crear usuarios (individual, CSV, CLI), restaurarlos y qué es la ubicación de uso.
- [ ] Sé los tipos de grupo y de pertenencia, y que los dinámicos exigen P1.
- [ ] Sé asignar licencias por grupo y sus limitaciones.
- [ ] Sé invitar usuarios externos y restringir dominios.
- [ ] Sé configurar SSPR (ámbito, métodos, registro, writeback, licencias).
- [ ] Sé los roles integrados clave y qué **no** puede hacer Contributor.
- [ ] Sé leer un rol personalizado en JSON y qué significa NotActions.
- [ ] Sé calcular permisos efectivos con herencia, grupos, locks y Policy.
- [ ] Sé todos los efectos de Azure Policy y cuándo hace falta identidad administrada.
- [ ] Sé los dos tipos de lock y sus efectos secundarios.
- [ ] Sé los límites de etiquetas y cómo heredarlas.
- [ ] Sé las reglas para mover recursos entre RG y suscripciones.
- [ ] Sé la jerarquía de grupos de administración y sus límites.
- [ ] Sé crear presupuestos con alertas y qué hace Advisor.

### Dominio 2 · Almacenamiento (15-20 %)
- [ ] Sé elegir tipo de cuenta, rendimiento y redundancia según requisitos.
- [ ] Sé qué no se puede cambiar tras crear la cuenta.
- [ ] Sé configurar el firewall y sus efectos en el portal.
- [ ] Sé distinguir service endpoint y private endpoint.
- [ ] Sé los tres tipos de SAS y cómo revocar cada uno.
- [ ] Sé rotar claves y qué roles de datos hacen falta con Entra ID.
- [ ] Sé configurar CMK y cifrado de infraestructura.
- [ ] Sé los requisitos de object replication.
- [ ] Sé cuándo usar AzCopy y cuándo Storage Explorer.
- [ ] Sé los niveles de acceso, sus retenciones y la rehidratación.
- [ ] Sé escribir/leer una regla de ciclo de vida.
- [ ] Sé versionado, snapshots y soft delete (blobs y contenedores).
- [ ] Sé crear y montar recursos compartidos SMB/NFS y el asunto del puerto 445.
- [ ] Sé los tres orígenes de identidad de Azure Files y sus dos niveles de permisos.
- [ ] Sé los componentes de Azure File Sync y el orden de despliegue.

### Dominio 3 · Cómputo (20-25 %)
- [ ] Sé interpretar y modificar plantillas ARM y archivos Bicep.
- [ ] Sé los modos de implementación y cómo exportar plantillas.
- [ ] Sé crear VMs y los estados de energía y su facturación.
- [ ] Sé elegir tamaños y las consecuencias de redimensionar.
- [ ] Sé los tipos de disco, la caché y cómo cambiar/ampliar.
- [ ] Sé las cuatro formas de cifrado de discos y sus requisitos.
- [ ] Sé mover VMs entre RG, suscripción y región.
- [ ] Sé los SLA y cuándo usar availability set o zonas.
- [ ] Sé configurar un VMSS con autoescalado y su directiva de actualización.
- [ ] Sé generalizar y publicar imágenes en una galería.
- [ ] Sé los SKUs de ACR y cómo autenticar sin secretos.
- [ ] Sé crear ACI con red, volúmenes y restart policy.
- [ ] Sé configurar Container Apps con escalado a cero y revisiones.
- [ ] Sé qué incluye cada nivel de plan de App Service.
- [ ] Sé configurar dominios, certificados, backups, redes y slots de App Service.

### Dominio 4 · Redes (15-20 %)
- [ ] Sé calcular IPs útiles y los nombres de subred reservados.
- [ ] Sé configurar IPs públicas Standard y NAT Gateway.
- [ ] Sé configurar peering con gateway transit y sus límites.
- [ ] Sé crear UDRs y explicar la prioridad de rutas.
- [ ] Sé escribir reglas NSG y usar etiquetas de servicio y ASGs.
- [ ] Sé interpretar reglas efectivas y usar IP flow verify.
- [ ] Sé desplegar Bastion y elegir el SKU.
- [ ] Sé configurar service endpoints y private endpoints con su DNS.
- [ ] Sé crear zonas DNS públicas y privadas y cuándo usar alias.
- [ ] Sé configurar un Load Balancer con sondas y reglas, y diagnosticarlo.
- [ ] Sé elegir entre Load Balancer, Application Gateway, Front Door y Traffic Manager.

### Dominio 5 · Monitorización y mantenimiento (10-15 %)
- [ ] Sé la diferencia entre métricas y logs y sus retenciones.
- [ ] Sé crear configuraciones de diagnóstico con sus tres destinos.
- [ ] Sé que el agente actual es AMA con DCR.
- [ ] Sé leer una consulta KQL y elegir la tabla correcta.
- [ ] Sé crear los tres tipos de alerta y usar grupos de acciones y reglas de procesamiento.
- [ ] Sé qué aporta cada Insight.
- [ ] Sé qué herramienta de Network Watcher usar en cada caso.
- [ ] Sé elegir entre Recovery Services vault y Backup vault.
- [ ] Sé las políticas Standard y Enhanced y sus retenciones.
- [ ] Sé las formas de restaurar una VM y cuándo usar cada una.
- [ ] Sé el flujo completo de Azure Site Recovery.
- [ ] Sé configurar informes y alertas de copias de seguridad.

## 2. Checklist de práctica (laboratorios)

- [ ] He creado usuarios y grupos dinámicos y he asignado licencias por grupo.
- [ ] He asignado roles RBAC en varios ámbitos y usado "Comprobar acceso".
- [ ] He asignado una Azure Policy con parámetros y una tarea de remediación.
- [ ] He creado una cuenta de almacenamiento y he restringido el acceso por firewall.
- [ ] He generado SAS con y sin directiva almacenada, y las he revocado.
- [ ] He configurado ciclo de vida, versionado y soft delete en Blob.
- [ ] He montado un recurso compartido de Azure Files.
- [ ] He desplegado recursos con una plantilla ARM y con Bicep.
- [ ] He creado una VM, añadido un disco, cambiado su tamaño y la he desasignado.
- [ ] He creado un VMSS con reglas de autoescalado.
- [ ] He desplegado un contenedor en ACI y una app en Container Apps.
- [ ] He creado una Web App con slot y he hecho un swap.
- [ ] He configurado VNet, subredes, peering, NSG y ASG.
- [ ] He desplegado Bastion y me he conectado sin IP pública.
- [ ] He creado un private endpoint con su zona DNS privada.
- [ ] He montado un Load Balancer con sonda y he roto una instancia para verlo.
- [ ] He usado IP flow verify y Next hop.
- [ ] He creado una alerta con grupo de acciones y la he disparado.
- [ ] He protegido una VM con Azure Backup y he restaurado un archivo.
- [ ] He habilitado Site Recovery y he ejecutado un test failover con su limpieza.

## 3. Últimas 48 horas

- [ ] Releer [[02 - Guía de memorización (números, límites y nombres)]] (30 min).
- [ ] Releer [[04 - Errores y confusiones frecuentes]] (30 min).
- [ ] Hacer [[06 - Banco de preguntas de práctica]] y revisar solo los fallos.
- [ ] Hacer [[07 - Simulacro final AZ-104]] cronometrado.
- [ ] Repasar las tablas de [[03 - Tabla comparativa de servicios]].
- [ ] Revisar el **study guide oficial** en Microsoft Learn por si hubo cambios.
- [ ] Comprobar los requisitos técnicos del examen (si es en casa: espacio despejado, documento de identidad, cámara, conexión).

## 4. Estrategia durante el examen

> [!important] Tácticas que suman puntos
> 1. **Lee el requisito final antes que el escenario**: "minimizar coste", "mínimo esfuerzo administrativo", "privilegio mínimo", "sin tiempo de inactividad". Esa frase decide la respuesta.
> 2. **Descarta por violación de requisito**, no por intuición: si una opción expone una IP pública y el enunciado exige "sin exposición a Internet", cae aunque funcione.
> 3. **Las preguntas Sí/No en serie no permiten volver atrás**: léelas dos veces antes de contestar.
> 4. **En los casos prácticos**, lee primero las preguntas y luego busca solo los datos necesarios.
> 5. **Marca para revisar** las dudosas y sigue: el tiempo es ajustado (unos 100 minutos).
> 6. **No dejes preguntas en blanco**: no hay penalización por fallar.
> 7. **Si dos opciones parecen válidas**, elige la que use el servicio **nativo y más simple** de Azure y la que cumpla **todos** los requisitos, no solo el principal.
> 8. **Cuidado con los nombres antiguos**: "Azure AD" = Microsoft Entra ID; "Log Analytics Agent" suele ser la respuesta incorrecta hoy.
> 9. **Preguntas de laboratorio** (si aparecen): sigue el enunciado al pie de la letra; no hace falta optimizar.
> 10. **Reserva 10 minutos finales** para las marcadas.

## 5. El día del examen

- [ ] Dormir bien: los detalles memorizados se pierden con el cansancio.
- [ ] Llegar (o conectarse) con 20-30 minutos de margen.
- [ ] Documento de identidad válido.
- [ ] Nada de repasar contenido nuevo en la última hora: solo las tablas de números.

> [!tip] Si suspendes
> Se puede repetir tras 24 horas (y luego con esperas mayores). El informe de puntuación indica los dominios más débiles: vuelve a esas notas, rehaz sus laboratorios y repite el simulacro.

## Relacionado

- [[06 - Banco de preguntas de práctica]]
- [[07 - Simulacro final AZ-104]]
- [[01 - Guía del examen AZ-104]]
- [[00 - AZ-104 Índice general (MOC)]]
