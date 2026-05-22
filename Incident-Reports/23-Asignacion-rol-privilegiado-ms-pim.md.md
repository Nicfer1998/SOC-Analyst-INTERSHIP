# INC-20260520-023 — User Assigned Privileged Role Involving Multiple Users (MS-PIM) — **Verdadero Positivo Benigno**

# 📝 Resumen

El 20 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad alta al detectar que dos cuentas distintas recibieron el rol de **Global Administrator** en el tenant a través de **Microsoft Privileged Identity Management (PIM)**. Tras una investigación con KQL sobre `AuditLogs` y verificación del tipo de cuentas involucradas, se confirmó que las asignaciones fueron realizadas por el sistema PIM como parte de un proceso administrativo controlado y legítimo.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260520-023 |
| **Incidente Sentinel** | #30922 |
| **Severidad** | **ALTA** |
| **Categoría** | Persistence / Privilege Escalation |
| **Detección Inicial** | Microsoft Sentinel — Threat Essentials: User Assigned Privileged Role |
| **Cuentas involucradas** | [AZ-Tota] / [pulsar365] |
| **Rol asignado** | Global Administrator |
| **Initiator** | MS-PIM |

---

## 🛠 Proceso de Investigación

### ¿Qué es MS-PIM y por qué importa?

**Privileged Identity Management (PIM)** es el sistema oficial de Microsoft Azure para gestionar, controlar y monitorear el acceso a roles privilegiados. Cuando una asignación de rol es iniciada por **MS-PIM**, significa que existe un proceso formal de solicitud y aprobación detrás de la acción, a diferencia de una asignación manual directa por un administrador.

### 1. Análisis del Timeline

El incidente correlacionó dos alertas simultáneas del mismo tipo a las 03:00 y 09:00 del 20 de mayo, lo que indicaba asignaciones múltiples en el mismo día.

### 2. Investigación con KQL sobre AuditLogs

Se ejecutó una consulta sobre `AuditLogs` filtrando por `Category == "RoleManagement"` y operaciones de tipo `Assign` para obtener el detalle completo de ambas asignaciones.

**Registro 1:**

| Campo | Valor |
|---|---|
| **OperationName** | Add member to role |
| **RoleName** | Global Administrator |
| **Target** | [pulsar365]@gpz.onmicrosoft.com |
| **Initiator** | MS-PIM |
| **Result** | Success |
| **Timestamp** | 2026-05-20T12:00:00Z |

**Registro 2:**

| Campo | Valor |
|---|---|
| **OperationName** | Add member to role |
| **RoleName** | Global Administrator |
| **Target** | AZ-[Tota]@gpz.onmicrosoft.com |
| **Initiator** | MS-PIM |
| **Result** | Success |
| **Timestamp** | 2026-05-20T06:00:00Z |

### 3. Análisis de las Cuentas

- **`AZ-[Tota]`:** El prefijo `AZ-` es una convención estándar de cuentas de administración dedicadas de Azure, separadas de la cuenta de usuario diaria. Su existencia indica que la organización sigue buenas prácticas de separación de cuentas administrativas.
- **`[pulsar365]`:** Cuenta de servicio en el tenant `gpz.onmicrosoft.com`.
- **Ambas iniciadas por MS-PIM:** Indica que hubo una solicitud formal de activación del rol, no una asignación directa manual sin control.

### 4. Validación

Se confirmó con los administradores que ambas asignaciones fueron realizadas de forma intencional como parte de sus tareas de administración del tenant.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Consulta KQL sobre AuditLogs | ✅ Completado | Identificación de ambas asignaciones, roles y cuentas involucradas. |
| Análisis del tipo de cuentas | ✅ Completado | Verificación de convención AZ- y cuenta de servicio. |
| Confirmación con administradores | ✅ Completado | Actividad confirmada como legítima. |
| Cierre del incidente | ✅ Completado | True Positive — Benign Activity. |

---

## 💡 Lecciones Aprendidas

1. **MS-PIM como señal positiva:** Cuando el initiator es MS-PIM, existe un proceso de solicitud y aprobación detrás. Esto reduce el riesgo respecto a una asignación manual directa, aunque no elimina la necesidad de verificación dado el nivel del rol (Global Administrator).
2. **Global Administrator siempre requiere revisión:** Independientemente del initiator, cualquier asignación del rol más alto del tenant debe ser investigada y documentada. La severidad Alta asignada por Sentinel es correcta.
3. **Convención de nombres como evidencia:** El prefijo `AZ-` en la cuenta indica buenas prácticas de administración. Reconocer estas convenciones acelera el triaje y da contexto inmediato sobre la naturaleza de la cuenta.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección y correlación del incidente.
- **Log Analytics (KQL)** — Consulta sobre `AuditLogs` para análisis de asignaciones de roles.
- **Azure Active Directory (Entra ID)** — Verificación del tipo y convención de las cuentas involucradas.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Persistence | Account Manipulation | T1098 |
| Privilege Escalation | Valid Accounts | T1078.004 |
