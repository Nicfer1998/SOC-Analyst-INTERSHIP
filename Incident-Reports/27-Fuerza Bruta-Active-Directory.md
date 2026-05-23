# INC-20260522-027 — Brute Force Attack on Active Directory — **En Investigación**

# 📝 Resumen

El 22 de mayo de 2026, el portal ITSM de Relatech generó un incidente al detectar una alerta de Microsoft Sentinel clasificada como **Brute Force Attack on Active Directory**. La regla detecta 10 o más intentos de autenticación fallidos en una ventana de 10 minutos, excluyendo intentos hacia dominios inexistentes. Tras la investigación inicial, se identificó que la cuenta `DONGNOCCHI\ibonora` recibió 42 intentos fallidos desde la IP interna `10.248.2.55` (workstation `SDC-FIREFDG02`) en menos de 4 minutos, resultando en el bloqueo de la cuenta. El incidente fue escalado al encargado de IT al finalizar el turno y queda pendiente de resolución.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260522-027 |
| **Incidente Sentinel / ITSM** | #8814 / Defence Center #118938 |
| **Severidad** | **MEDIA** |
| **Categoría** | Credential Access |
| **Detección Inicial** | Microsoft Sentinel — Brute force attack on Active Directory |
| **Cuenta afectada** | DONGNOCCHI\ibonora |
| **IP origen** | 10.248.2.55 (interna) |
| **Workstation** | SDC-FIREFDG02 |
| **Estado** | 🔄 En investigación — escalado a IT |

---

## 🛠 Proceso de Investigación

### 1. Análisis de la Alerta

La regla de detección tiene una característica importante: **excluye intentos hacia dominios inexistentes**. Esto significa que los 42 intentos fueron contra una cuenta **real y válida** del dominio, no escaneos aleatorios.

### 2. Investigación con KQL

Se ejecutó una consulta sobre `SecurityEvent` para obtener el detalle completo de los intentos.

**Hallazgos:**

| Campo | Valor |
|---|---|
| **Account** | DONGNOCCHI\ibonora |
| **Attempts** | 42 intentos |
| **StartTime** | 2026-05-22T08:57:01Z |
| **EndTime** | 2026-05-22T09:00:41Z |
| **Duración** | ~3 minutos 40 segundos |
| **IPAddress** | 10.248.2.55 |
| **WorkstationName** | SDC-FIREFDG02 |
| **Reason** | User logon with account locked |

- **42 intentos en menos de 4 minutos** → velocidad incompatible con un usuario humano escribiendo contraseñas manualmente. Sugiere una aplicación o script intentando autenticarse en loop.
- **La cuenta quedó bloqueada** → los intentos activaron la política de bloqueo de cuenta del dominio.
- **IP interna** → el origen es dentro de la red corporativa, no un ataque externo.

### 3. Hipótesis

**Hipótesis A — Aplicación con credenciales vencidas:**
> Una aplicación o servicio configurado con las credenciales de `ibonora` intenta autenticarse repetidamente porque la contraseña expiró o fue cambiada. Es el escenario más común y benigno.

**Hipótesis B — Movimiento lateral:**
> Un atacante con acceso a `SDC-FIREFDG02` está intentando comprometer la cuenta `ibonora` mediante fuerza bruta interna.

### 4. Estado al cierre del turno

El incidente fue escalado al encargado de IT con toda la evidencia recopilada. Pendiente de:
- Identificar qué aplicación o proceso en `SDC-FIREFDG02` genera los intentos
- Desbloquear la cuenta `ibonora` una vez identificada la causa raíz
- Confirmar si fue una aplicación con credenciales vencidas o actividad maliciosa

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Consulta KQL sobre SecurityEvent | ✅ Completado | Identificación de cuenta, IP, workstation y número de intentos. |
| Análisis de velocidad de intentos | ✅ Completado | 42 intentos en ~4 minutos sugiere proceso automatizado. |
| Escalación a IT Owner | ✅ Completado | Handoff al turno siguiente con evidencia documentada. |
| Identificación de causa raíz | 🔄 Pendiente | IT debe identificar el proceso en SDC-FIREFDG02. |
| Desbloqueo de cuenta ibonora | 🔄 Pendiente | Una vez confirmada la causa raíz. |

---

## 💡 Lecciones Aprendidas

1. **Velocidad de intentos como indicador clave:** 42 intentos en menos de 4 minutos es físicamente imposible para un humano. Cuando la velocidad es tan alta, lo primero que hay que considerar es una aplicación con credenciales desactualizadas, no necesariamente un ataque.
2. **IP interna no significa seguro:** El origen interno del ataque indica que o hay un dispositivo comprometido dentro de la red o una aplicación mal configurada. Ambos escenarios requieren investigación.
3. **Handoff documentado:** Los incidentes no siempre se cierran en un turno. Documentar el estado exacto y las hipótesis al escalar es tan importante como la investigación misma.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección del incidente.
- **Log Analytics (KQL)** — Consulta sobre `SecurityEvent`.
- **Portal ITSM (Relatech)** — Gestión y seguimiento del ticket.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Credential Access | Brute Force | T1110 |
| Credential Access | Password Spraying | T1110.003 |
