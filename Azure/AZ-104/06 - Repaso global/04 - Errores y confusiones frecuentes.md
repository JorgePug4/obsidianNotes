---
tags: [az-104, azure, repaso, errores, trampas]
modulo: Repaso global
---

# ⚠️ Errores y confusiones frecuentes

Las trampas que más se repiten en AZ-104, agrupadas por dominio. Cada una en formato **"lo que parece" → "lo que es"**.

## Identidad y gobernanza

1. **"Global Administrator lo puede todo en Azure"** → Los roles de **Entra ID** no dan permisos sobre suscripciones. Necesita **elevación de acceso** o una asignación RBAC.
2. **"Contributor puede hacer todo salvo borrar"** → Contributor **no asigna roles** ni **gestiona bloqueos**.
3. **"Asignar Reader en el RG reduce el Contributor de la suscripción"** → Los permisos **se suman**; nunca se restan.
4. **"NotActions deniega"** → Solo **excluye del propio rol**; si otro rol lo concede, el usuario lo tiene.
5. **"El Owner puede borrar cualquier cosa"** → Un **lock** lo impide; hay que quitarlo primero.
6. **"Una Policy Deny borra los recursos que incumplen"** → Solo los marca **no conformes** e impide nuevas creaciones.
7. **"Append sirve para etiquetar recursos existentes"** → Append solo actúa **al crear**; para existentes, **Modify + remediación**.
8. **"Las etiquetas del grupo de recursos se heredan"** → **No**; se heredan con una Policy de tipo **Modify**.
9. **"El presupuesto detiene el gasto"** → Solo **alerta**; para actuar, grupo de acciones + automatización.
10. **"Puedo poner varios grupos en SSPR Selected"** → Solo **uno** (que puede contener grupos anidados).
11. **"Las licencias por grupo llegan a los grupos anidados"** → Solo a **miembros directos**.
12. **"Mover un recurso conserva sus permisos"** → Las asignaciones **en el recurso** se pierden.
13. **"Puedo mover una suscripción a otro tenant sin consecuencias"** → Se **borran** todas las asignaciones RBAC.
14. **"Un grupo de administración agrupa recursos"** → Agrupa **suscripciones**.

## Almacenamiento

15. **"Premium admite GRS"** → Premium solo **LRS y ZRS**.
16. **"GRS permite leer la copia secundaria"** → Solo **RA-GRS / RA-GZRS**.
17. **"La redundancia es una copia de seguridad"** → Replica también los **borrados**.
18. **"Puedo revocar una SAS emitida"** → Solo si usa **directiva almacenada**; si no, hay que **regenerar la clave**.
19. **"Contributor puede leer blobs con Entra ID"** → Necesita un **rol de datos**; con Contributor puede **listar claves** y usar esa vía.
20. **"Habilitar el service endpoint ya restringe el acceso"** → Falta poner el **firewall del recurso en Deny** y añadir la regla de red virtual.
21. **"El service endpoint funciona desde on-premises"** → **No**; para eso, **private endpoint**.
22. **"Con un private endpoint ya está todo hecho"** → Falta la **zona DNS privada** y deshabilitar el acceso público.
23. **"Un private endpoint cubre toda la cuenta"** → Es **por subrecurso** (blob, file, sqlServer…).
24. **"Archive se puede leer directamente"** → Está **offline**: hay que **rehidratar** (hasta 15 h).
25. **"Cambiar de nivel es gratis"** → Hay **cargos por eliminación anticipada** (Cool 30, Cold 90, Archive 180 días).
26. **"El ciclo de vida actúa al instante"** → Se ejecuta **una vez al día** y puede tardar 24 h.
27. **"Soft delete de blobs recupera contenedores"** → Son **dos configuraciones distintas**.
28. **"NFS funciona en cuentas Standard"** → Solo en **Premium FileStorage**.
29. **"Con la clave de la cuenta tengo permisos por usuario en Azure Files"** → La clave es **superusuario**; para permisos se necesita **identidad**.

## Cómputo

30. **"Apagar la VM desde Windows deja de facturar"** → Hay que **desasignar** (Stop desde el portal/CLI).
31. **"Los datos del disco D: son persistentes"** → Es el **disco temporal**: se pierde al desasignar o redimensionar.
32. **"Puedo añadir una VM existente a un availability set"** → **No**; hay que recrearla.
33. **"Una VM puede estar en una zona y en un availability set"** → Son **excluyentes**.
34. **"El modo Complete solo añade recursos"** → **Elimina** lo que no está en la plantilla.
35. **"Bicep se puede pegar en el editor de plantillas del portal"** → El portal solo acepta **JSON**.
36. **"La plantilla exportada se despliega tal cual"** → Hay que **parametrizar** y **no incluye secretos**.
37. **"Cualquier tamaño admite discos Premium"** → Solo los que llevan **"s"**.
38. **"Puedo cambiar el tipo de disco con la VM encendida"** → Hay que **desasignar**.
39. **"ADE funciona en cualquier VM"** → No en series **A/Basic**, ni con **Ultra/Premium SSD v2**, ni junto a encryption at host.
40. **"Mover una VM cambia su región"** → No; para eso, **Azure Resource Mover**.
41. **"ACI escala automáticamente"** → **No**; para eso, **Container Apps**.
42. **"Container Apps escala a cero con una regla de CPU"** → Las reglas de **CPU/memoria no escalan a cero**; hacen falta reglas **HTTP/KEDA**.
43. **"Basic admite deployment slots"** → Slots desde **Standard** (5) y Premium (20).
44. **"Las app settings se quedan en su ranura"** → Solo si están marcadas como **slot setting**.
45. **"VNet integration oculta la app de Internet"** → VNet integration es **salida**; para entrada privada, **private endpoint**.
46. **"El certificado gratuito de App Service cubre comodines"** → Tiene limitaciones; para comodín, otro tipo de certificado.
47. **"Un CNAME sirve para el dominio raíz"** → El apex necesita **A** o un **registro alias**.

## Redes

48. **"Un /24 tiene 254 IPs útiles"** → **251**: Azure reserva **5**.
49. **"El peering es transitivo"** → **No**: A↔B y B↔C no dan A↔C.
50. **"Con crear el peering en un lado basta"** → Hay que crearlo en **ambos** (si no, estado *Initiated*).
51. **"El NSG se asocia a la VNet"** → Solo a **subred** o **NIC**.
52. **"4096 tiene más prioridad que 100"** → **Menor número = mayor prioridad**.
53. **"Basta con permitirlo en el NSG de la subred"** → Si la NIC tiene NSG, **también** debe permitirlo.
54. **"Hay que crear la regla de vuelta"** → El NSG es **stateful**.
55. **"El ASG filtra tráfico"** → Solo **agrupa NICs**; filtra el NSG.
56. **"Una UDR filtra tráfico"** → **Enruta**; filtrar es del NSG.
57. **"La NVA reenvía sin más"** → Necesita **IP forwarding** en la NIC y en el SO.
58. **"El DNS de Azure resuelve entre VNets emparejadas"** → **No**: hace falta una **zona privada vinculada**.
59. **"Puedo tener varios vínculos con autorregistro"** → **Uno por VNet**.
60. **"Azure DNS registra dominios"** → Solo los **hospeda**.
61. **"La IP pública Standard acepta tráfico por defecto"** → Está **cerrada**: necesita NSG.
62. **"El balanceador funciona sin permitir las sondas"** → Hay que permitir **AzureLoadBalancer** (168.63.129.16).
63. **"Una sonda HTTP acepta un 302"** → Solo considera sano el **200 OK**.
64. **"Load Balancer puede enrutar por URL"** → Es **capa 4**; eso es **Application Gateway**.
65. **"Traffic Manager balancea el tráfico"** → Solo responde **DNS**; el cliente conecta directo.
66. **"Bastion puede ir en cualquier subred"** → Debe llamarse **AzureBastionSubnet** y ser **/26**.

## Monitorización y backup

67. **"Los logs de recurso se recopilan solos"** → Requieren **configuración de diagnóstico**.
68. **"El agente de Log Analytics sigue siendo el recomendado"** → Está **retirado**: usar **AMA + DCR**.
69. **"Las métricas se guardan un año"** → **93 días**; para más, exportar.
70. **"Una alerta de métrica detecta que alguien borró una VM"** → Eso es una alerta del **registro de actividad**.
71. **"La alerta notifica por sí sola"** → Necesita un **grupo de acciones**.
72. **"Para silenciar alertas hay que desactivar las reglas"** → Se usa una **regla de procesamiento** con supresión.
73. **"El vault protege cualquier carga"** → **RSV** para VMs/Files/SQL/SAP/MARS/ASR; **Backup vault** para discos/blobs/PostgreSQL/AKS.
74. **"Puedo cambiar la redundancia del vault cuando quiera"** → Solo **antes del primer elemento protegido**.
75. **"Cross Region Restore funciona por defecto"** → Requiere **GRS** y habilitarlo.
76. **"Standard permite copias cada 4 horas"** → Eso es **Enhanced**.
77. **"Restaurar crea siempre una VM nueva"** → También se pueden **reemplazar los discos** (conserva nombre e IP).
78. **"MARS hace copias de VMs completas"** → Protege **archivos, carpetas y estado del sistema**.
79. **"Backup y Site Recovery son lo mismo"** → Backup **recupera datos**; ASR **mantiene el servicio** con RPO de minutos.
80. **"El test failover afecta a producción"** → Se hace en una **red aislada**; hay que **limpiar** después.
81. **"Backup Reports funciona sin configuración"** → Necesita enviar los datos del vault a **Log Analytics** (~24 h).

## Cómo usar esta lista

> [!tip] Estrategia de repaso
> Léela dos veces: una semana antes y la mañana del examen. Marca las 10 que más te sorprendan y conviértelas en tarjetas. En el examen, cuando una respuesta "parezca obvia", comprueba si está en esta lista.

## Relacionado

- [[01 - Resumen general de AZ-104]]
- [[02 - Guía de memorización (números, límites y nombres)]]
- [[05 - Checklist final antes del examen]]
- [[06 - Banco de preguntas de práctica]]
- [[00 - AZ-104 Índice general (MOC)]]
