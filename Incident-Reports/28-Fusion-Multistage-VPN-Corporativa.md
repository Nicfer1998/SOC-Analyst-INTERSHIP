# INC-20260522-028 — Possible Multistage Attack Detected by Fusion — **Verdadero Positivo Benigno**

# 📝 Resumen

El 22 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad alta mediante el motor de correlación **Fusion**, al detectar tres alertas encadenadas sobre la cuenta `forms.service@venchi.com`: Atypical Travel, Unfamiliar Sign-in Properties y Potential User Account Compromise. Tras una investigación exhaustiva que incluyó análisis de SigninLogs, Threat Intelligence sobre las IPs involucradas y verificación del perfil de la cuenta en Azure AD, se determinó que la IP sospechosa (`98.116.199.102`) es la **VPN corporativa de Venchi**, utilizada diariamente por múltiples empleados de la organización. El incidente fue cerrado como Verdadero Positivo Benigno.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260522-028 |
| **Incidente Sentinel** | #20467 |
| **Severidad** | **ALTA** |
| **Categoría** | Collection / Credential Access / Initial Access |
| **Detección Inicial** | Microsoft Sentinel — Fusion: Possible multistage attack |
| **Cuenta involucrada** | forms.service@venchi.com |
| **IPs involucradas** | 98.116.199.102 / 176.200.67.26 |
| **Fuentes** | Microsoft Defender XDR + Microsoft Entra ID Protection |

---

## 🛠 Proceso de Investigación

### ¿Qué es Fusion y cómo funciona?

**Fusion** es el motor de correlación de Microsoft Sentinel que analiza comportamiento, no nombres ni reglas simples. Detecta secuencias de eventos que coinciden con patrones conocidos de ataques reales (TTPs). En este caso correlacionó tres alertas individuales de severidad baja/media que juntas formaban el patrón de un **account takeover**:

```
Atypical Travel
    + Unfamiliar Sign-in Properties
    + Potential Account Compromise
    = Possible multistage attack
```

### 1. Timeline del Incidente

| Fecha | Alerta | Severidad |
|---|---|---|
| May 21 — 18:26 | Atypical Travel | Media |
| May 21 — 18:27 | Unfamiliar Sign-in Properties | Media |
| May 22 — 01:35 | Potential User Account Compromise | Alta |

### 2. Análisis de SigninLogs con KQL

Se ejecutó una consulta sobre `AADNonInteractiveUserSignInLogs` filtrando por `forms.service@venchi.com`.

**Hallazgos:**

| Timestamp | IP | Ubicación | Resultado |
|---|---|---|---|
| 08:57 AM | 5.91.112.213 → Monza, IT | 🇮🇹 Italia | 0 - Success |
| 08:58 AM | 176.200.67.26 → Pavia, IT | 🇮🇹 Italia | 0 - Success |
| 09:00 AM | 213.82.86.178 → Roma, IT | 🇮🇹 Italia | 0 - Success |
| 09:01 AM | 98.116.199.102 → New York, US | 🇺🇸 EEUU | 0 - Success |

- Todos los sign-ins fueron **exitosos**.
- El cambio abrupto de Italia a Nueva York en minutos disparó el **Atypical Travel**.

### 3. Threat Intelligence sobre las IPs

Se investigaron las dos IPs principales en **ipinfo.io**:

**IP `176.200.67.26`:**

| Campo | Valor |
|---|---|
| **ASN** | AS16232 — Telecom Italia S.p.A. |
| **Ubicación** | Milan, Lombardy, Italia |
| **VPN/Proxy** | ❌ No |
| **Tipo** | ISP móvil italiano |

**IP `98.116.199.102`:**

| Campo | Valor |
|---|---|
| **ASN** | AS701 — Verizon Business |
| **Ubicación** | New York City, EEUU |
| **VPN** | ✅ **Detectado** |
| **Privacy** | true |

### 4. Investigación del perfil de la cuenta

Se analizó el perfil de `forms.service@venchi.com` en Azure AD / Microsoft Defender.

**Nombre real:** VENCHI Forms Service No-Reply

| Campo | Valor | Observación |
|---|---|---|
| **MFA status** | Disabled | ⚠️ MFA deshabilitado |
| **Último cambio de contraseña** | Enero 23, 2023 | ⚠️ +3 años sin cambio |
| **Risk score** | 19 | ⚠️ Elevado por Entra ID |
| **Risky user** | "This user is at risk" | ⚠️ |
| **Group memberships** | 28 | Cuenta con acceso amplio |
| **Applications** | 16 | Uso extenso de M365 |
| **User type** | Member | No es service account técnico |
| **Interactive sign-ins** | Frecuentes | Cuenta usada por humanos |

**Conclusión clave:** A pesar del nombre "Forms Service No-Reply", esta **no es una cuenta de servicio automatizada**. Tiene licencias, MFA methods registrados, sign-ins interactivos frecuentes y 16 aplicaciones asignadas. Es una **cuenta funcional** utilizada activamente por personas.

### 5. Verificación de la VPN corporativa

Se ejecutó una consulta KQL filtrando todos los sign-ins desde `98.116.199.102` en las últimas 24 horas.

**Resultado:** Múltiples empleados de Venchi autenticándose exitosamente desde la misma IP:
- mattia.marmo@venchi.com
- niccolo.doni@venchi.com
- ivana.zhao@venchi.com

**Conclusión:** `98.116.199.102` es la **VPN corporativa de Venchi**. Sentinel nunca la había registrado como ubicación conocida y la trató como IP extranjera sospechosa.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Análisis de SigninLogs KQL | ✅ Completado | Identificación de todas las IPs y ubicaciones involucradas. |
| Threat Intelligence IPs | ✅ Completado | 176.200.67.26 = Telecom Italia. 98.116.199.102 = VPN Venchi. |
| Análisis del perfil en Azure AD | ✅ Completado | Cuenta funcional con MFA deshabilitado y contraseña de 3 años. |
| Verificación VPN corporativa | ✅ Completado | Múltiples empleados usan la misma IP diariamente. |
| Cierre del incidente | ✅ Completado | True Positive — Benign. Atypical Travel por VPN no registrada. |
| Recomendación estructural | ⏳ Pendiente | Agregar IP a Named Locations + habilitar MFA en la cuenta. |

---

## 💡 Lecciones Aprendidas

1. **Fusion correlaciona comportamiento, no nombres:** El motor no sabe que la cuenta se llama "service". Ve una secuencia de alertas que coincide con un patrón de account takeover y las correlaciona. Entender cómo funciona Fusion permite anticipar qué tipos de actividad va a generar este tipo de incidente.
2. **Una cuenta funcional ≠ service account técnico:** El nombre "Forms Service No-Reply" sugería automatización, pero el perfil revelaba una cuenta usada activamente por humanos. Los indicadores clave: licencias asignadas, interactive sign-ins frecuentes, MFA methods registrados y múltiples aplicaciones.
3. **Named Locations como mejora estructural:** La IP de la VPN corporativa no estaba registrada en Azure AD como ubicación conocida. Agregarla como Named Location en Conditional Access eliminaría esta clase de falsos positivos de forma permanente.
4. **MFA deshabilitado en cuenta con acceso amplio:** Una cuenta con acceso a 16 aplicaciones y 28 grupos sin MFA obligatorio es un riesgo real independientemente de este incidente. Debe ser corregido.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección y correlación Fusion del incidente.
- **Log Analytics (KQL)** — Consulta sobre `AADNonInteractiveUserSignInLogs`.
- **Azure Active Directory / Microsoft Defender** — Análisis del perfil de la cuenta.
- **ipinfo.io** — Threat Intelligence sobre IPs involucradas.
- **Microsoft Entra ID Protection** — Risk score y clasificación de cuenta riesgosa.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Initial Access | Valid Accounts | T1078 |
| Credential Access | Brute Force — Password Spraying | T1110.003 |
| Collection | Email Collection | T1114 |
