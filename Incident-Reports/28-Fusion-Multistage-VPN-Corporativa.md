# INC-20260522-028 — Possible Multistage Attack Detected by Fusion — **Verdadero Positivo Benigno**

# 📝 Resumen

El 22 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad alta mediante el motor de correlación **Fusion**, al detectar tres alertas encadenadas sobre una cuenta de servicio funcional: Atypical Travel, Unfamiliar Sign-in Properties y Potential User Account Compromise. Tras una investigación exhaustiva que incluyó análisis de SigninLogs, Threat Intelligence sobre las IPs involucradas y verificación del perfil de la cuenta en Azure AD, se determinó que la IP sospechosa era la **VPN corporativa de la organización**, utilizada diariamente por múltiples empleados. El incidente fue cerrado como Verdadero Positivo Benigno.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260522-028 |
| **Incidente Sentinel** | #20467 |
| **Severidad** | **ALTA** |
| **Categoría** | Collection / Credential Access / Initial Access |
| **Detección Inicial** | Microsoft Sentinel — Fusion: Possible multistage attack |
| **Cuenta involucrada** | [service-account]@[org-domain] |
| **IPs involucradas** | [vpn-ip] (VPN corporativa) / [mobile-ip] (ISP móvil Italia) |
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

Se ejecutó una consulta sobre `AADNonInteractiveUserSignInLogs` filtrando por la cuenta afectada.

**Hallazgos:**

| Timestamp | IP | Ubicación | Resultado |
|---|---|---|---|
| 08:57 AM | [mobile-ip-1] | 🇮🇹 IT, Monza | 0 - Success |
| 08:58 AM | [mobile-ip-2] | 🇮🇹 IT, Pavia | 0 - Success |
| 09:00 AM | [office-ip] | 🇮🇹 IT, Roma | 0 - Success |
| 09:01 AM | [vpn-ip] | 🇺🇸 US, New York | 0 - Success |

- Todos los sign-ins fueron **exitosos**.
- El cambio abrupto de Italia a Nueva York en minutos disparó el **Atypical Travel**.

### 3. Threat Intelligence sobre las IPs

Se investigaron las dos IPs principales en **ipinfo.io**:

**IP Italia ([mobile-ip-2]):**

| Campo | Valor |
|---|---|
| **ASN** | Telecom Italia S.p.A. |
| **Ubicación** | Milan, Lombardy, Italia |
| **VPN/Proxy** | ❌ No |
| **Tipo** | ISP móvil italiano |

**IP Nueva York ([vpn-ip]):**

| Campo | Valor |
|---|---|
| **ASN** | Verizon Business |
| **Ubicación** | New York City, EEUU |
| **VPN** | ✅ Detectado |
| **Privacy** | true |

### 4. Investigación del perfil de la cuenta

Se analizó el perfil de la cuenta en Azure AD / Microsoft Defender.

| Campo | Valor | Observación |
|---|---|---|
| **MFA status** | Disabled | ⚠️ MFA deshabilitado |
| **Último cambio de contraseña** | +3 años sin cambio | ⚠️ |
| **Risk score** | 19 | ⚠️ Elevado por Entra ID |
| **Risky user** | "This user is at risk" | ⚠️ |
| **Group memberships** | 28 | Cuenta con acceso amplio |
| **Applications** | 16 | Uso extenso de M365 |
| **User type** | Member | No es service account técnico |
| **Interactive sign-ins** | Frecuentes | Cuenta usada por humanos |

**Conclusión clave:** A pesar del nombre que incluye "Service", esta **no es una cuenta de servicio automatizada**. Tiene licencias, MFA methods registrados, sign-ins interactivos frecuentes y 16 aplicaciones asignadas. Es una **cuenta funcional** utilizada activamente por personas.

### 5. Verificación de la VPN corporativa

Se ejecutó una consulta KQL filtrando todos los sign-ins desde la IP de Nueva York en las últimas 24 horas.

**Resultado:** Múltiples empleados de la organización autenticándose exitosamente desde la misma IP el mismo día.

**Conclusión:** La IP de Nueva York es la **VPN corporativa de la organización**. Sentinel nunca la había registrado como ubicación conocida y la trató como IP extranjera sospechosa.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Análisis de SigninLogs KQL | ✅ Completado | Identificación de todas las IPs y ubicaciones involucradas. |
| Threat Intelligence IPs | ✅ Completado | IP Italia = ISP móvil legítimo. IP NY = VPN corporativa. |
| Análisis del perfil en Azure AD | ✅ Completado | Cuenta funcional con MFA deshabilitado y contraseña antigua. |
| Verificación VPN corporativa | ✅ Completado | Múltiples empleados usan la misma IP diariamente. |
| Cierre del incidente | ✅ Completado | True Positive — Benign. Atypical Travel por VPN no registrada. |
| Recomendación estructural | ⏳ Pendiente | Agregar IP a Named Locations + habilitar MFA en la cuenta. |

---

## 💡 Lecciones Aprendidas

1. **Fusion correlaciona comportamiento, no nombres:** El motor no sabe cómo se llama la cuenta. Ve una secuencia de alertas que coincide con un patrón de account takeover y las correlaciona. Entender cómo funciona Fusion permite anticipar qué tipos de actividad van a generar este tipo de incidente.
2. **Una cuenta funcional ≠ service account técnico:** El nombre incluía "Service" pero el perfil revelaba una cuenta usada activamente por humanos. Los indicadores clave: licencias asignadas, interactive sign-ins frecuentes, MFA methods registrados y múltiples aplicaciones asignadas.
3. **Named Locations como mejora estructural:** La IP de la VPN corporativa no estaba registrada en Azure AD como ubicación conocida. Agregarla como Named Location en Conditional Access eliminaría esta clase de falsos positivos de forma permanente.
4. **MFA deshabilitado en cuenta con acceso amplio:** Una cuenta con acceso a 16 aplicaciones y 28 grupos sin MFA obligatorio es un riesgo real independientemente de este incidente.

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
