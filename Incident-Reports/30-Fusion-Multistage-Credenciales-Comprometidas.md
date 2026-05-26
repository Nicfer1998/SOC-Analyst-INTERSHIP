# INC-20260526-030 — Fusion: Possible Multistage Attack — Credenciales Comprometidas — **Verdadero Positivo**

# 📝 Resumen

El 26 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad alta mediante el motor de correlación **Fusion**, al detectar tres alertas encadenadas sobre una cuenta de usuario: Atypical Travel, Unfamiliar Sign-in Properties y Potential User Account Compromise. Tras una investigación con KQL sobre `SigninLogs` y verificación de las IPs involucradas, se determinó que una de las IPs correspondía a la **VPN corporativa de la organización** (234 usuarios distintos en 24 horas) y la otra a un **intento de acceso desde Ucrania bloqueado por MFA**. Las credenciales del usuario estaban comprometidas. Se tomaron acciones de contención inmediatas: revocación de sesiones y forzado de cambio de contraseña desde Microsoft Defender.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260526-030 |
| **Incidente Sentinel / ITSM** | #65924 / Defence Center #122458 |
| **Severidad** | **ALTA** |
| **Categoría** | Credential Access / Initial Access |
| **Detección Inicial** | Microsoft Sentinel — Fusion: Possible multistage attack |
| **Usuario involucrado** | [username]@[org-domain] |
| **IPs involucradas** | [vpn-ip] (VPN corporativa) / [attacker-ip] (Ucrania) |
| **Fuentes** | Microsoft Defender XDR + Microsoft Entra ID Protection |

---

## 🛠 Proceso de Investigación

### ¿Qué es Fusion y cómo funciona?

**Fusion** es el motor de correlación de Microsoft Sentinel que analiza comportamiento y detecta secuencias de eventos que coinciden con patrones conocidos de ataques reales. En este caso correlacionó tres alertas que juntas formaban el patrón de un account takeover:

```
Atypical Travel
    + Unfamiliar Sign-in Properties
    + Potential Account Compromise
    = Possible multistage attack
```

### 1. Análisis de las IPs con KQL

Se ejecutó una consulta sobre `SigninLogs` para verificar cuántos usuarios distintos usaban cada IP involucrada:

```kql
SigninLogs
| where IPAddress == "[ip]"
| summarize dcount(Identity)
```

**Resultados:**

| IP | Usuarios distintos | Conclusión |
|---|---|---|
| [vpn-ip] | **234** | ✅ VPN corporativa |
| [attacker-ip] | **1** | 🔴 IP sospechosa |

### 2. Análisis del intento desde Ucrania

Se expandió el registro del login desde [attacker-ip] para obtener el detalle completo.

| Campo | Valor |
|---|---|
| **ResultType** | 50074 |
| **ResultSignature** | FAILURE |
| **ResultDescription** | Strong Authentication is required |
| **Location** | UA — Ucrania |
| **UserAgent** | Vacío → herramienta automatizada |
| **Identity** | [username] — Consultor externo |

**Error 50074** → El atacante tenía las credenciales correctas pero MFA bloqueó el acceso. El UserAgent vacío indica uso de herramienta automatizada de credential stuffing, no un humano navegando.

### 3. Verificación de la VPN corporativa

La IP [vpn-ip] fue verificada como VPN corporativa mediante el alto volumen de usuarios distintos autenticándose desde ella — 234 identidades en 24 horas. Esto explica las alertas de Atypical Travel y Unfamiliar Sign-in, ya que Sentinel no tenía registrada esa IP como ubicación conocida.

### 4. Acciones de contención

Confirmado el compromiso de credenciales, se tomaron acciones inmediatas directamente desde Microsoft Defender sin esperar confirmación del cliente:
- **Revocación de todas las sesiones activas**
- **Forzado de cambio de contraseña**

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Query dcount por IP | ✅ Completado | 234 usuarios en VPN corporativa vs 1 en IP atacante |
| Análisis del log de Ucrania | ✅ Completado | Error 50074 — MFA bloqueó el acceso |
| Verificación UserAgent vacío | ✅ Completado | Herramienta automatizada confirmada |
| Revocación de sesiones | ✅ Completado | Ejecutado desde Microsoft Defender |
| Forzado de cambio de contraseña | ✅ Completado | Ejecutado desde Microsoft Defender |
| Notificación al cliente | ✅ Completado | Mail enviado con resumen del incidente |
| Recomendación estructural | ⏳ Pendiente | Registrar IP de VPN corporativa como Named Location en Azure AD |

---

## 💡 Lecciones Aprendidas

1. **`dcount(Identity)` como técnica de verificación de VPN:** Contar las identidades distintas que usan una IP en 24 horas es una técnica rápida y efectiva para determinar si una IP es VPN corporativa o individual. Una IP con cientos de usuarios distintos no puede ser un atacante individual.
2. **Error 50074 — MFA como última línea de defensa:** El atacante tenía las credenciales correctas. Lo único que impidió el acceso fue MFA. Este caso demuestra el valor crítico de MFA habilitado en todas las cuentas, incluyendo consultores externos.
3. **UserAgent vacío en intentos fallidos:** La ausencia de UserAgent en un login fallido desde un país sospechoso indica uso de herramientas automatizadas de credential stuffing — no un humano que olvidó su contraseña.
4. **Acción directa desde Defender sin esperar al cliente:** Ante evidencia clara de credenciales comprometidas, revocar sesiones y forzar cambio de contraseña directamente reduce el tiempo de exposición sin necesidad de esperar confirmación.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección y correlación Fusion del incidente.
- **Log Analytics (KQL)** — Consulta sobre `SigninLogs` con `dcount`.
- **Microsoft Defender** — Revocación de sesiones y forzado de cambio de contraseña.
- **Microsoft Entra ID Protection** — Risk score y clasificación de cuenta riesgosa.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Credential Access | Brute Force — Credential Stuffing | T1110.004 |
| Initial Access | Valid Accounts | T1078 |
| Defense Evasion | Use Alternate Authentication Material | T1550 |
