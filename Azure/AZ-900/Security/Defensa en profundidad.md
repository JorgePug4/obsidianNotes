---
tags: [AZ-900, Azure, Seguridad, ZeroTrust]
up: "[[00 - Índice - Seguridad]]"
---

# Defensa en profundidad

## 1. Concepto

**Defensa en profundidad** (*defense in depth*) es una estrategia de seguridad basada en **capas**. Cada capa protege por sí sola y, si un atacante supera una, la siguiente sigue frenándolo.

- **Problema que resuelve**: confiar en un único control (por ejemplo, solo un firewall) deja todo expuesto si ese control falla.
- **Para qué se usa**: reducir la probabilidad de que un ataque llegue a los datos y ganar tiempo para detectarlo.

Piensa en un castillo: foso, muralla, puerta, guardias y, al final, la cámara del tesoro. Los **datos** están en el centro.

## 2. Características principales

Microsoft define **7 capas**, de fuera hacia dentro:

| # | Capa | Qué protege | Ejemplo en Azure |
|---|---|---|---|
| 1 | **Seguridad física** | Acceso a los datacenters | Guardias, cámaras, tarjetas (lo gestiona Microsoft) |
| 2 | **Identidad y acceso** | Quién entra y qué puede hacer | Microsoft Entra ID, MFA, RBAC, Conditional Access |
| 3 | **Perímetro** | Ataques a gran escala desde Internet | Protección DDoS, Azure Firewall |
| 4 | **Red** | Comunicación entre recursos | NSG, segmentación de VNet, denegar por defecto |
| 5 | **Cómputo** | VMs y servicios de ejecución | Parches, Defender for Cloud, cifrado de discos |
| 6 | **Aplicación** | El código y sus secretos | Sin secretos en el código, Key Vault, desarrollo seguro |
| 7 | **Datos** | Lo que realmente quiere el atacante | Cifrado, control de acceso, backups |

Regla mnemotécnica (de fuera a dentro): **F-I-P-R-C-A-D** → *Física, Identidad, Perímetro, Red, Cómputo, Aplicación, Datos*.

> [!warning] Confusión frecuente: Defensa en profundidad vs Zero Trust
> - **Defensa en profundidad** = *muchas capas* de protección.
> - **Zero Trust** = *no confiar en nadie por defecto*, verificar siempre.
> Son complementarios, no excluyentes. El examen los presenta juntos.

### Zero Trust (va en el mismo objetivo del examen)

Modelo que asume que **ya hay una brecha** y que ninguna petición es de fiar por venir de la red corporativa. Tres principios:

1. **Verificar explícitamente** (autenticar y autorizar siempre, con toda la información disponible).
2. **Acceso con privilegio mínimo** (*least privilege*, acceso *just in time* y *just enough*).
3. **Asumir la brecha** (*assume breach*: segmentar, cifrar, monitorizar).

También conviene conocer la **tríada CIA**, que el examen usa como vocabulario:

- **Confidencialidad**: solo quien debe ve los datos.
- **Integridad**: los datos no se alteran sin permiso.
- **Disponibilidad** (*Availability*): los datos están accesibles cuando se necesitan.

## 3. Casos de uso

- Una empresa activa MFA (capa identidad), NSG en sus VNets (capa red) y cifrado en su base de datos (capa datos). Si roban una contraseña, el MFA y el resto de capas siguen protegiendo.
- Un empleado trabaja desde casa: con Zero Trust se le pide MFA y se comprueba que el dispositivo cumple políticas aunque use la VPN corporativa.

## 4. Comparaciones

| | Modelo clásico (perímetro) | Zero Trust |
|---|---|---|
| Confianza | Dentro de la red = confiable | Nadie es confiable por defecto |
| Verificación | Una vez, al entrar | En cada acceso |
| Privilegios | Amplios | Mínimos y temporales |
| Supuesto | La red es segura | Ya hay una brecha |

## 5. Conceptos que debo memorizar

> [!important]
> - Las **7 capas** y su orden, con los **datos en el centro**.
> - Microsoft es responsable de la **seguridad física**; tú de identidad, datos, etc. (modelo de responsabilidad compartida).
> - Zero Trust = **verificar explícitamente + privilegio mínimo + asumir la brecha**.
> - **CIA** = Confidencialidad, Integridad, Disponibilidad.

## 6. Tips para AZ-900

> [!tip]
> - Si la pregunta dice **"múltiples capas"** o **"si una falla, la siguiente protege"** → *defensa en profundidad*.
> - Si dice **"nunca confiar, verificar siempre"** o **"asumir que hay una brecha"** → *Zero Trust*.
> - Pregunta trampa: "¿qué capa protege contra ataques DDoS?" → **Perímetro**, no Red.
> - "¿Qué capa incluye MFA?" → **Identidad y acceso**.
> - "¿Qué está en el centro del modelo?" → **Datos**.

## 7. Ejemplo de pregunta de examen

**Pregunta 1.** Una empresa implementa MFA, grupos de seguridad de red y cifrado en reposo para que, si un control falla, los demás sigan protegiendo. ¿Qué estrategia aplica?

- A) Zero Trust
- B) Defensa en profundidad ✅
- C) Responsabilidad compartida
- D) Alta disponibilidad

*Correcta: B.* La idea de "varios controles en capas donde uno cubre el fallo de otro" define la defensa en profundidad. A trata de no confiar por defecto, no de capas; C describe qué gestiona Microsoft y qué gestiona el cliente; D es un concepto de resiliencia, no de seguridad.

**Pregunta 2.** ¿Cuál es el principio de Zero Trust que consiste en dar a cada usuario solo el acceso imprescindible durante el tiempo imprescindible?

- A) Verificar explícitamente
- B) Asumir la brecha
- C) Acceso con privilegio mínimo ✅
- D) Defensa perimetral

*Correcta: C.* A trata de autenticar siempre; B de diseñar suponiendo que el atacante ya está dentro; D es el modelo antiguo que Zero Trust reemplaza.

**Pregunta 3.** En el modelo de defensa en profundidad de Azure, ¿en qué capa se ubica la protección contra DDoS?

- A) Red
- B) Perímetro ✅
- C) Cómputo
- D) Física

*Correcta: B.* La capa perímetro frena ataques a gran escala desde Internet antes de que lleguen a la red interna. A gestiona el tráfico entre recursos (NSG); C protege VMs; D son los datacenters.

## 🧠 Resumen para el examen

1. Defensa en profundidad = capas; si una cae, la siguiente protege.
2. Siete capas: Física, Identidad, Perímetro, Red, Cómputo, Aplicación, Datos.
3. Los **datos** están en el centro.
4. DDoS y Azure Firewall → capa **Perímetro**; NSG → capa **Red**.
5. MFA, RBAC y Conditional Access → capa **Identidad**.
6. Zero Trust: verificar explícitamente, privilegio mínimo, asumir la brecha.
7. CIA = Confidencialidad, Integridad, Disponibilidad.
8. Microsoft cubre la capa física; el cliente, la mayoría del resto.

---
⬅️ [[00 - Índice - Seguridad|Volver al índice de Seguridad]]
