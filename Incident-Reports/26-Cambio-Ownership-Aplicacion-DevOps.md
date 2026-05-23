# INC-20260520-026 — Changes to Application Ownership — **Verdadero Positivo Benigno**

# 📝 Resumen

El 20 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad baja al detectar tres eventos simultáneos de cambio de ownership sobre una aplicación registrada en Azure AD. Tras una investigación con KQL sobre `AuditLogs` y la verificación del rol del usuario en Azure AD, se determinó que la asignación fue realizada automáticamente por **Azure DevOps** como parte de un flujo de CI/CD, asignando al desarrollador responsable como owner de la aplicación de su equipo. El incidente fue cerrado como Verdadero Positivo Benigno.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260520-026 |
| **Incidente Sentinel** | #16180 |
| **Severidad** | **BAJA** |
| **Categoría** | Persistence / Privilege Escalation |
| **Detección Inicial** | Microsoft Sentinel — Changes to Application Ownership |
| **Usuario involucrado** | [username]@[org-domain] |
| **Initiator** | Azure DevOps |
| **Aplicación afectada** | [org-prefix]-[team-name] |

---

## 🛠 Proceso de Investigación

### ¿Por qué es sensible el ownership de una aplicación?

El **owner de una aplicación** registrada en Azure AD tiene capacidad para:
- Modificar los **permisos** de la aplicación
- Agregar **credenciales** (secrets o certificados)
- Otorgar **consentimiento** de la app para acceder a recursos del tenant

Esta es una técnica usada en ataques reales de tipo **App Consent Abuse**: un atacante compromete una cuenta, se hace owner de una app con permisos elevados y desde ahí opera con persistencia silenciosa sin necesitar credenciales de usuario.

### 1. Análisis del Timeline

Se generaron **3 alertas simultáneas** a las 09:22:13 del 20 de mayo, todas del mismo tipo, lo que indicaba un proceso automático en lote más que una acción manual individual.

### 2. Investigación con KQL sobre AuditLogs

Se ejecutó una consulta sobre `AuditLogs` para obtener el detalle completo de la operación.

**Hallazgos:**

| Campo | Valor |
|---|---|
| **ActivityDisplayName** | Add owner to application |
| **AADOperationType** | Assign |
| **AddedUser** | [username]@[org-domain] |
| **Identity** | Azure DevOps |
| **InitiatedBy** | Azure DevOps |
| **TargetAppName** | [org-prefix]-[team-name] |
| **TargetAccountName** | [username] |
| **Result** | Success |
| **Category** | ApplicationManagement |

- **`InitiatedBy: Azure DevOps`** → La asignación fue iniciada por el sistema de CI/CD, no por una persona manualmente. Es comportamiento típico cuando se crea o actualiza un pipeline/proyecto en DevOps.
- **`TargetAppName`** → Nombre de app consistente con un proyecto de desarrollo del equipo afectado.

### 3. Verificación del Rol del Usuario en Azure AD

Se consultó el perfil del usuario en Azure AD para evaluar si su rol justifica ser owner de esa aplicación.

- **Rol asignado:** `Application Developer`
- **Conclusión:** Un Application Developer siendo asignado automáticamente como owner de la aplicación de su equipo vía Azure DevOps es completamente consistente con su rol y responsabilidades.

### 4. Revisión de Incidentes Similares

| Incidente | Estado |
|---|---|
| #16101 | Closed — True Positive |
| #15808 | Closed — Benign |

El incidente #16101 fue cerrado como True Positive, lo que indica que en este entorno sí ocurren casos reales de cambios de ownership no autorizados. Esto justifica la investigación aunque el patrón actual sea benigno.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Consulta KQL sobre AuditLogs | ✅ Completado | Identificación del initiator, app afectada y usuario involucrado. |
| Verificación de rol en Azure AD | ✅ Completado | Rol Application Developer consistente con la acción. |
| Revisión de incidentes similares | ✅ Completado | Contexto histórico analizado, True Positive previo identificado. |
| Cierre del incidente | ✅ Completado | True Positive — Benign. Asignación automática legítima por Azure DevOps. |

---

## 💡 Lecciones Aprendidas

1. **El initiator determina el contexto:** Cuando `InitiatedBy` es Azure DevOps en lugar de una cuenta de usuario, indica un proceso automatizado de CI/CD. Reconocer los initiators automáticos legítimos acelera el triaje significativamente.
2. **El rol del usuario como evidencia de cierre:** En este caso no fue necesario contactar al usuario. El rol `Application Developer` en Azure AD, combinado con el nombre de la app y el equipo al que pertenece, fue suficiente evidencia para el cierre. El triaje eficiente usa toda la información disponible antes de escalar.
3. **True Positive previo = monitoreo prioritario:** Un incidente similar previo fue un True Positive real. Eso significa que esta regla de Sentinel ha demostrado ser efectiva en este entorno y debe mantenerse activa, aunque genere falsos positivos recurrentes.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección y correlación del incidente.
- **Log Analytics (KQL)** — Consulta sobre `AuditLogs`.
- **Azure Active Directory (Entra ID)** — Verificación del rol del usuario involucrado.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Persistence | Account Manipulation | T1098 |
| Privilege Escalation | Cloud Account | T1078.004 |
