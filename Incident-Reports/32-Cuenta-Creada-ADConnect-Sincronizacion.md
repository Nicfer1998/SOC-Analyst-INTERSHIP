# INC-20260526-032 — Account Created from Non-Approved Sources — **Verdadero Positivo Benigno**

# 📝 Resumen

El 26 de mayo de 2026, Microsoft Sentinel generó una alerta al detectar que una cuenta fue creada en Azure AD desde una fuente no aprobada. Tras investigar los AuditLogs directamente en el portal de Azure del cliente afectado, se determinó que la cuenta fue creada automáticamente por el proceso de sincronización **Azure AD Connect** — un programa que replica usuarios desde el Active Directory on-premise hacia Azure AD de forma automática. No hubo intervención humana. El incidente fue cerrado como Verdadero Positivo Benigno.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260526-032 |
| **Incidente Sentinel / ITSM** | Defence Center #122709 |
| **Severidad** | **MEDIA** |
| **Categoría** | Persistence |
| **Detección Inicial** | Microsoft Sentinel — Account created from non-approved sources |
| **Cuenta creada** | ConnectSyncProvisioning_[id] |
| **Cuenta iniciadora** | ADConnect-service |
| **Cliente** | [org-name] |

---

## 🛠 Proceso de Investigación

### Contexto — ¿Qué es Azure AD Connect?

Las organizaciones suelen tener dos sistemas de usuarios en paralelo:
- **Active Directory on-premise** → en sus servidores físicos
- **Azure Active Directory** → en la nube de Microsoft

**Azure AD Connect** es el programa que mantiene ambos sistemas sincronizados. Trabaja automáticamente en segundo plano — detecta usuarios nuevos o modificados en el AD físico y los replica en Azure AD sin intervención humana.

### 1. Análisis de las Entidades

| Kind | Valor |
|---|---|
| **Account 1** | ConnectSyncProvisioning_[id] |
| **Account 2** | [service-account]@[org-domain] |

- **ConnectSyncProvisioning** → nombre típico de cuentas creadas por el proceso de sincronización de Azure AD Connect.
- La presencia de la cuenta `ADConnect-service` en el tenant confirma que la organización tiene Azure AD Connect activo.

### 2. Verificación en AuditLogs del portal del cliente

Se accedió directamente al portal de Azure del cliente afectado y se navegó a:
**Microsoft Entra ID → Audit Logs → Activity: Add user**

Se encontraron múltiples registros del mismo día con el patrón:

| Hora | Status | Observación |
|---|---|---|
| Múltiples intentos | Failure | Microsoft.Online.* — intentos previos fallidos del sync |
| 7:12 AM | **Success** | Primera cuenta creada exitosamente |
| 9:36 AM | **Success** | Segunda cuenta creada exitosamente |

### 3. Confirmación del actor — Initiated by

Se expandió el registro exitoso de las 9:36 AM para verificar el actor:

| Campo | Valor |
|---|---|
| **Initiated by (actor) Type** | **Application** |
| **Display Name** | ConnectSyncProvisioning_[id] |
| **User Agent** | Vacío |
| **Status** | Success |

- **Type: Application** → proceso automático, no humano ✅
- **User Agent vacío** → confirma que no hubo sesión interactiva ✅
- **Display Name: ConnectSyncProvisioning** → Azure AD Connect ✅

### 4. Explicación de los intentos fallidos previos

Los múltiples registros con **Failure** iniciados por `Microsoft.Online.*` son intentos previos del proceso de sincronización que fallaron antes de completarse — comportamiento normal cuando Azure AD Connect intenta sincronizar un usuario que aún no existe en Azure AD y necesita varios intentos hasta completar el proceso.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Verificación de cuenta ADConnect-service | ✅ Completado | Confirma que Azure AD Connect está activo en el tenant. |
| Revisión de AuditLogs en portal del cliente | ✅ Completado | Acceso directo al tenant del cliente para mayor precisión. |
| Verificación de Initiated by | ✅ Completado | Type: Application — ConnectSyncProvisioning. Sin actor humano. |
| Cierre del incidente | ✅ Completado | True Positive — Benign. Sincronización automática legítima. |

---

## 💡 Lecciones Aprendidas

1. **Type: Application vs Type: User en Initiated by:** Esta distinción es clave para determinar si una acción en AuditLogs fue realizada por un proceso automático o por una persona. Application = robot. User = humano. Aplicable a cualquier operación en Azure AD.
2. **Los fallos previos son normales en AD Connect:** Ver múltiples registros Failure antes de un Success es el comportamiento esperado del proceso de sincronización. No indica un ataque — indica que el proceso necesitó varios ciclos para completarse.
3. **Verificar directamente en el portal del cliente:** Cuando KQL en el SIEM centralizado no filtra correctamente por tenant, acceder directamente al portal de Azure del cliente afectado da resultados más precisos y verificables.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección del incidente.
- **Azure Active Directory (Entra ID) — Portal del cliente** — Verificación directa de AuditLogs.
- **Log Analytics (KQL)** — Consulta inicial sobre AuditLogs.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Persistence | Create Account — Cloud Account | T1136.003 |
