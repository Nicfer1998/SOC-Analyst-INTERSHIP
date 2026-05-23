# INC-20260520-024 — Rare and Potentially High-Risk Office Operations — **Verdadero Positivo Benigno**

# 📝 Resumen

El 20 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad baja al detectar operaciones administrativas de Exchange Online clasificadas como "raras y potencialmente de alto riesgo". Tras una investigación con KQL sobre `OfficeActivity` y el análisis de los campos de la cuenta ejecutora, se determinó que la operación fue realizada de forma completamente automática por el proceso interno de Exchange `NT SERVICE\MSExchangeAdminApiNetCore` sobre un buzón de arbitraje del sistema, sin intervención humana alguna.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260520-024 |
| **Incidente Sentinel** | #30924 |
| **Severidad** | **BAJA** |
| **Categoría** | Collection / Persistence |
| **Detección Inicial** | Microsoft Sentinel — Rare and potentially high-risk Office operations |
| **Cuenta ejecutora** | NT SERVICE\MSExchangeAdminApiNetCore |
| **Operación** | Set-Mailbox |
| **Objeto afectado** | SPO_Arbitration mailbox |

---

## 🛠 Proceso de Investigación

### Conceptos clave

**Exchange Online (ExO):** Servicio de correo corporativo de Microsoft 365.

**Buzones de arbitraje:** Buzones internos del sistema de Exchange, invisibles para los usuarios, utilizados para procesar flujos de aprobación, retención, compliance y tareas programadas internas. No reciben correos de usuarios.

**NT SERVICE\MSExchangeAdminApiNetCore:** Proceso interno de Exchange Online que ejecuta comandos administrativos vía API, tanto cuando son solicitados por un administrador humano como cuando Exchange los ejecuta de forma automática como parte de su mantenimiento.

### 1. Análisis de la Alerta

La alerta identificó una operación de tipo `Set-Mailbox` ejecutada sobre un objeto de sistema. Se investigó para determinar si detrás había un actor humano o un proceso automático.

### 2. Investigación con KQL

Se ejecutó una consulta sobre `OfficeActivity` para obtener el detalle completo del evento.

**Hallazgos:**

| Campo | Valor |
|---|---|
| **Operation** | Set-Mailbox |
| **AccountName** | NT SERVICE\MSExchangeAdminApiNetCore |
| **RecordType** | ExchangeAdmin |
| **UserType** | Admin |
| **AppPoolName** | MSExchangeAdminApiNetCore |
| **OfficeObjectId** | SPO_Arbitration_[ID] |
| **ResultStatus** | True |
| **OfficeTenantId** | $RestApiTenantId$ |
| **ElevationTime** | 2026-05-19T14:10:50Z |

### 3. Determinación: Humano vs. Automático

Para confirmar que no hubo intervención humana se analizaron tres campos clave:

- **`AccountName`** comienza con `NT SERVICE\` → proceso de sistema, no cuenta de usuario.
- **`AppPoolName: MSExchangeAdminApiNetCore`** → ejecutado por el pool de la API de Exchange, no por sesión interactiva.
- **`OfficeTenantId: $RestApiTenantId$`** → token de sistema interno, no token de sesión de usuario humano.

La combinación de estos tres campos confirma con certeza que fue un proceso automático.

### 4. Contexto de incidentes similares

Los incidentes similares #30901, #30854 y #30830 fueron todos cerrados como Low/Benign, confirmando que este patrón es recurrente y esperado en el entorno.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Consulta KQL sobre OfficeActivity | ✅ Completado | Identificación de la operación, cuenta ejecutora y objeto afectado. |
| Análisis de campos humano vs. automático | ✅ Completado | Confirmación mediante AccountName, AppPoolName y OfficeTenantId. |
| Revisión de incidentes similares | ✅ Completado | Patrón recurrente confirmado como comportamiento esperado. |
| Cierre del incidente | ✅ Completado | True Positive — Benign. Mantenimiento automático de Exchange. |

---

## 💡 Lecciones Aprendidas

1. **Cómo distinguir humano vs. automático en logs de Exchange:** Si `AccountName` comienza con `NT SERVICE\`, el `AppPoolName` es un proceso de Exchange y el `OfficeTenantId` es `$RestApiTenantId$`, la operación fue automática sin intervención humana.
2. **Por qué Sentinel alerta sobre servicios propios de Microsoft:** La regla detecta la operación, no la intención. Un atacante que compromete Exchange puede usar exactamente el mismo mecanismo para modificar buzones silenciosamente. Sentinel es conservador a propósito y el analista debe aportar el contexto.
3. **Buzones de arbitraje como ruido esperado:** Las operaciones de Exchange sobre buzones de arbitraje son mantenimiento interno rutinario. Documentarlos como patrón conocido permite cerrarlos más rápido en el futuro.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección del incidente.
- **Log Analytics (KQL)** — Consulta sobre `OfficeActivity`.
- **Exchange Online Admin Center** — Contexto sobre buzones de arbitraje y procesos internos.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Collection | Email Collection | T1114 |
| Persistence | Exchange Email Delegate Permissions | T1098.002 |
