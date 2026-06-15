# Caso 34 — Privileged Accounts Sign-in Failure Spikes: Token Expirado en Outlook Mobile

## Resumen narrativo

Sentinel generó una alerta de alta severidad por picos de fallos de inicio de sesión en una cuenta con privilegios elevados. Al investigar, se identificó que los fallos correspondían a renovaciones automáticas fallidas del token de sesión de Outlook Mobile en el dispositivo móvil del usuario — causadas por políticas de Conditional Access que invalidaron el refresh token. No se identificó actividad maliciosa.

---

## Tabla de detalles del incidente

| Campo | Detalle |
|---|---|
| **Incident ID** | 31197 |
| **Título** | Privileged Accounts - Sign in Failure Spikes involving one user |
| **Clasificación** | ✅ True Positive Benigno |
| **Severidad** | High |
| **Primera actividad** | May 26, 2026 — 05:55 AM |
| **Usuario afectado** | `[username]@[org-domain]` |
| **Roles del usuario** | Dynamics 365 Administrator, Power Platform Administrator |
| **IPs involucradas** | `[corp-ip-1]` (proxy corporativo), `[mobile-ip]` (IP personal móvil) |
| **Aplicación** | Outlook Mobile — One Outlook Web |
| **Error principal** | 70043 — Refresh token expirado/inválido por Conditional Access |
| **Dispositivo** | iPhone (iOS 18) |
| **Ubicación** | Italia |
| **Fuente de detección** | Microsoft Sentinel (Analytics Rule) |

---

## Proceso de investigación

### 1. Primera lectura de la alerta

La alerta indicaba picos de fallos de autenticación en una cuenta con privilegios elevados. Los roles de la cuenta (Dynamics 365 Administrator y Power Platform Administrator) justificaban la severidad High.

Se identificaron dos IPs en las entidades del incidente:
- `[corp-ip-1]`
- `[mobile-ip]`

Los logins eran de tipo `NonInteractiveUserSignInLogs` — no era el usuario escribiendo su contraseña, sino aplicaciones autenticándose en segundo plano.

---

### 2. Análisis de IP corporativa (`[corp-ip-1]`)

Se ejecutó query en Sentinel para ver qué usuarios accedían desde esa IP:

```kql
SigninLogs
| where IPAddress == "[corp-ip-1]"
| summarize count() by UserPrincipalName
| order by count_ desc
```

**Resultado:** Múltiples usuarios de la organización con alto volumen de eventos (57 a 91 por usuario), todos con `ResultType == 0` (éxito).

**Conclusión:** IP de proxy/NAT corporativo — comportamiento normal.

---

### 3. Análisis de IP móvil (`[mobile-ip]`)

```kql
SigninLogs
| where IPAddress == "[mobile-ip]"
| summarize count() by UserPrincipalName
| order by count_ desc
```

**Resultado:** Solo dos cuentas:
- `[username]@[org-domain]` — 16 eventos
- `[personal-email]@icloud.com` — 5 eventos

**Conclusión:** La IP pertenece al dispositivo móvil personal del usuario. Los eventos de iCloud confirman que es su iPhone personal.

---

### 4. Análisis de resultados por tipo de error

```kql
SigninLogs
| where IPAddress in ("[corp-ip-1]", "[mobile-ip]")
| where UserPrincipalName == "[username]@[org-domain]"
| summarize count() by Result
| order by count_ desc
```

**Resultados:**

| Result | Cantidad | Significado |
|---|---|---|
| 0 — Success | 634 | Logins exitosos — IP habitual del usuario |
| 50140 — Keep me signed in | 7 | Interrupción normal de sesión |
| **70043 — Refresh token expired** | **spike** | ⚠️ Causa del pico de fallos |

---

### 5. Causa raíz identificada

El error **70043** indica que el refresh token de la sesión de Outlook Mobile fue invalidado por políticas de **Conditional Access (sign-in frequency)**. Cuando el token vence, la app intenta renovarlo automáticamente en segundo plano múltiples veces en poco tiempo — generando el pico de fallos que disparó la alerta.

```
Política de Conditional Access → invalida refresh token
        ↓
Outlook Mobile en iPhone intenta renovar automáticamente
        ↓
Múltiples fallos en corto período
        ↓
Sentinel detecta el pico → genera alerta High
        ↓
Causa: comportamiento legítimo de la app, no ataque
```

---

### 6. Verificación de cuenta — creada en 2017

La cuenta `[username]@[org-domain]` fue creada en enero de 2017 — es una cuenta legítima con historial largo de actividad normal. No se detectaron logins desde IPs nuevas ni desde países inusuales para el usuario.

---

## Tabla de respuesta y remediación

| Acción | Responsable | Estado |
|---|---|---|
| Verificar que las dos IPs son legítimas | Analista SOC | ✅ Confirmado |
| Identificar la causa del pico de fallos (error 70043) | Analista SOC | ✅ Confirmado |
| Informar al usuario que debe reautenticarse en Outlook Mobile | Soporte IT | ✅ Recomendado |
| Revisar configuración de sign-in frequency en Conditional Access | Equipo IT | ⚠️ Opcional — si el pico es recurrente |

---

## Lecciones aprendidas

1. **El error 70043 no indica ataque** — es un error de expiración de refresh token por Conditional Access. Es importante diferenciarlo de errores de credenciales incorrectas (50126) o de bloqueo por MFA (50074).

2. **NonInteractiveUserSignInLogs genera ruido diferente al de logins interactivos** — los fallos aquí son de apps renovando tokens, no de usuarios escribiendo contraseñas incorrectas.

3. **Verificar el historial de IPs antes de escalar** — la IP corporativa con 634 éxitos del mismo usuario desmonta inmediatamente la hipótesis de compromiso de credenciales.

4. **Cuentas con roles privilegiados generan alertas de mayor severidad** — aunque la actividad sea benigna, los roles de Dynamics 365 Admin y Power Platform Admin justifican la investigación completa.

---

## Herramientas utilizadas

- Microsoft Sentinel — Log Analytics, SigninLogs
- KQL — queries de análisis de IPs y errores
- Azure AD / Microsoft Entra ID — perfil del usuario

---

## MITRE ATT&CK

No aplica — actividad legítima. Sin técnicas de ataque identificadas.
