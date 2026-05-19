# INC-20260519-001 — Local Admin Group Changes — **Verdadero Positivo Benigno**

# 📝 Resumen

El 19 de mayo de 2026, Microsoft Sentinel generó una alerta de severidad alta al detectar que un usuario fue agregado manualmente a un grupo local con permisos elevados (RWX) en un host de la organización. Tras una investigación técnica utilizando **KQL en Log Analytics** y revisión del perfil del usuario en **Azure Active Directory**, el incidente fue confirmado como una actividad legítima y autorizada por el equipo de IT.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260519-001 |
| **Incidente Sentinel** | #30907 |
| **Severidad** | **ALTA** |
| **Categoría** | Privilege Escalation / Account Manipulation |
| **Detección Inicial** | Microsoft Sentinel — Cambio en grupo local de administración |
| **Host afectado** | [hostname].[org-domain] |
| **Usuario involucrado** | [username] |
| **Actor** | administrador (cuenta built-in) |

---

## 🛠 Proceso de Investigación

### 1. Análisis de la Alerta

Se recibió una alerta de Sentinel con 4 incidentes similares recientes (incidentes #30718, #30757, #30799, #30906), lo que indicaba un patrón repetitivo que justificaba una investigación detallada.

- **Query KQL ejecutada:** Consulta sobre `SecurityEvent` filtrando por `UserAccountAddedToLocalGroup` en el host afectado.
- **Hallazgos clave:**
  - `ActionType`: UserAccountAddedToLocalGroup
  - `LocalGroup`: DL_RWX_ALOF_Informatica
  - `Actor`: administrador
  - `Timestamp`: 2026-05-19T11:32:01Z

### 2. Enriquecimiento — Perfil del Usuario en Azure AD

Se investigó el perfil completo del usuario en el portal de Azure AD para evaluar el nivel de riesgo.

- **Hallazgos:** El usuario ya pertenecía a múltiples grupos con permisos RWX (Read/Write/Execute) y al grupo **FAM-VPNSSL-Admins**, lo que le otorga acceso administrativo a VPN.
- **Activo desde:** 2023
- **Evaluación:** Usuario con privilegios elevados preexistentes. La adición al nuevo grupo local resultaba redundante pero no anómala en ese contexto.

### 3. Validación con IT Owner

Ante el perfil de alto privilegio del usuario y el patrón de incidentes repetidos, se escaló al responsable de IT para confirmar la legitimidad de la acción.

- **Resultado:** El jefe de IT confirmó que la actividad fue realizada por el equipo de IT en el marco de sus operaciones habituales.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Consulta KQL en Log Analytics | ✅ Completado | Identificación del usuario, grupo, host y actor involucrados. |
| Revisión de perfil en Azure AD | ✅ Completado | Análisis de grupos y privilegios del usuario afectado. |
| Escalación a IT Owner | ✅ Completado | Confirmación por email de actividad legítima. |
| Cierre del incidente | ✅ Completado | Clasificado como True Positive — Benign Activity. |

---

## 💡 Lecciones Aprendidas

1. **Patrón de incidentes similares:** La existencia de múltiples incidentes del mismo tipo en días consecutivos puede indicar una operación de IT en curso, no necesariamente un ataque. Sin embargo, siempre debe verificarse.
2. **Contexto de privilegios:** Un usuario con acceso VPN administrativo y múltiples grupos RWX preexistentes que recibe un nuevo grupo local de los mismos permisos es una señal de escalada potencial que requiere validación obligatoria.
3. **Verificación siempre antes de cerrar:** Independientemente del contexto, confirmar con el IT Owner es el paso correcto. Un SOC analyst no asume, verifica.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección y timeline del incidente.
- **Log Analytics (KQL)** — Análisis detallado de eventos de seguridad.
- **Azure Active Directory (Entra ID)** — Enriquecimiento del perfil del usuario.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Privilege Escalation | Local Groups | T1069 |
| Persistence | Account Manipulation | T1098 |
