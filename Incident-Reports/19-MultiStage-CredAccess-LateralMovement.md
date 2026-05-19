# INC-20260519-002 — Multi-stage Incident: Credential Access & Lateral Movement — **Verdadero Positivo Benigno**

# 📝 Resumen

El 19 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad media que correlacionó múltiples alertas: intentos de autenticación fallidos, conexiones RDP inusuales y dos cuentas administrativas involucradas, todo sobre un **Domain Controller**. Tras investigación en Microsoft Defender XDR y revisión de roles en Azure AD, se confirmó que la actividad fue legítima y autorizada por los administradores del sistema.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260519-002 |
| **Incidente Sentinel** | #16127 |
| **Severidad** | **MEDIA** |
| **Categoría** | Credential Access / Lateral Movement |
| **Detección Inicial** | Microsoft Sentinel — Multi-stage incident |
| **Host afectado** | [DC-hostname] (Domain Controller) |
| **IP involucrada** | 192.168.18.14 |
| **Cuentas involucradas** | EU\[admin-account-1] / EU\[admin-account-2] |

---

## 🛠 Proceso de Investigación

### 1. Análisis del Timeline

El incidente correlacionó tres alertas distribuidas en dos días:

| Fecha | Alerta | Severidad |
|---|---|---|
| May 18 — 04:48 | SecurityEvent — Multiple authentication failures | Baja |
| May 19 — 04:17 | Rare RDP Connections | Media |
| May 19 — 07:03 | Rare RDP Connections (segunda ocurrencia) | Media |

- **Hallazgo crítico:** El host afectado es un **Domain Controller**, lo que eleva automáticamente el nivel de atención independientemente de la severidad asignada por Sentinel. Actividad RDP inusual hacia un DC con dos cuentas admin es una señal de alto riesgo.
- **Patrón detectado:** Fallas de autenticación seguidas de conexiones RDP exitosas = patrón clásico de brute force + acceso exitoso.

### 2. Enriquecimiento — Roles de las Cuentas en Azure AD

Se investigaron los roles asignados a ambas cuentas en el portal de Azure AD.

**Cuenta 1 ([admin-account-1]):**
- Security Reader, Fabric Administrator, Global Reader, Places Administrator, Reports Reader.
- Perfil: Acceso amplio de lectura al tenant.

**Cuenta 2 ([admin-account-2]) — Perfil de mayor riesgo:**
- User Administrator, Exchange Administrator, License Administrator, Groups Administrator, Security Operator, Helpdesk Administrator.
- **Evaluación:** Esta cuenta tiene capacidad para crear usuarios, modificar grupos, acceder a Exchange y operar en seguridad. Una potencial compromisión representaría un impacto crítico sobre el tenant.

### 3. Validación con los Administradores

Dada la criticidad del host (Domain Controller) y el nivel de privilegios de las cuentas, se escaló para confirmar la legitimidad.

- **Resultado:** Ambos administradores confirmaron que la actividad fue parte de sus tareas habituales de administración del sistema.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Análisis del timeline en Sentinel | ✅ Completado | Correlación de las 3 alertas y evaluación del host afectado. |
| Investigación en Defender XDR | ✅ Completado | Revisión detallada de las conexiones RDP y autenticaciones. |
| Revisión de roles en Azure AD | ✅ Completado | Enriquecimiento del perfil de ambas cuentas administrativas. |
| Escalación y confirmación | ✅ Completado | Actividad confirmada como legítima por los administradores. |
| Cierre del incidente | ✅ Completado | Clasificado como True Positive — Benign Activity. |

---

## 💡 Lecciones Aprendidas

1. **El tipo de host importa:** Una alerta de severidad Media sobre un Domain Controller debe tratarse con la misma urgencia que una de severidad Alta. El contexto del asset eleva el riesgo real.
2. **Cuentas con múltiples roles críticos:** Ante cuentas con capacidad de User Administrator + Exchange Administrator + Security Operator, la verificación no es opcional, es obligatoria.
3. **Patrón temporal:** Auth failures → RDP inusual → segunda conexión RDP es una secuencia que, en ausencia de contexto, debe considerarse como compromiso potencial hasta que se demuestre lo contrario.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Correlación y timeline del incidente.
- **Microsoft Defender XDR** — Investigación detallada de conexiones.
- **Azure Active Directory (Entra ID)** — Revisión de roles asignados.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Credential Access | Brute Force | T1110 |
| Lateral Movement | Remote Desktop Protocol | T1021.001 |
| Initial Access | Valid Accounts | T1078 |
