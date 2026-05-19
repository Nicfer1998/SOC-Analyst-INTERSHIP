# INC-20260519-005 — Mail Redirect via ExO Transport Rule — **Benigno Positivo — Sospechoso pero Esperado**

# 📝 Resumen

El 19 de mayo de 2026, Microsoft Sentinel generó una alerta de severidad media al detectar la modificación de una regla de transporte en **Exchange Online (ExO)** que redirige automáticamente correos entrantes de un usuario hacia otras cuentas internas de la organización. Tras una investigación con KQL sobre `OfficeActivity` y la confirmación con el equipo de IT, se determinó que la regla fue configurada intencionalmente como parte de las operaciones normales del negocio. El incidente fue cerrado como **BenignPositive — Suspicious but Expected**.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260519-005 |
| **Severidad** | **MEDIA** |
| **Categoría** | Collection / Exfiltration (potencial) |
| **Detección Inicial** | Microsoft Sentinel — Mail redirect via ExO transport rule |
| **Operación** | Set-TransportRule |
| **Nombre de la regla** | Copia de [área] a [destinatarios internos] |
| **Usuario que modificó** | [username]@[org-domain] |
| **IP origen** | 20.105.90.222 |
| **Destinatarios de la redirección** | Cuentas internas de [org-domain] |

---

## 🛠 Proceso de Investigación

### ¿Qué es Exchange Online (ExO) y una Transport Rule?

**Exchange Online (ExO)** es el servicio de correo corporativo de Microsoft 365. Una **Transport Rule** (regla de transporte) es una regla automática que instruye a Exchange a realizar una acción sobre los correos que cumplan ciertas condiciones — en este caso, **copiar/redirigir automáticamente** los correos entrantes a otras cuentas internas.

### 1. Análisis de la Alerta con KQL

Se ejecutó una consulta KQL sobre la tabla `OfficeActivity` filtrando por operaciones `New-TransportRule` y `Set-TransportRule` para obtener el detalle completo de la modificación.

**Hallazgos:**

| Campo | Valor |
|---|---|
| **TimeGenerated** | 2026-05-19T10:18:04Z |
| **Operation** | Set-TransportRule |
| **RedirectTo** | [cuenta-1]@[org]; [cuenta-2]@[org]; [cuenta-3]@[org] |
| **IPAddress** | 20.105.90.222 |
| **UserId** | [username]@[org-domain] |
| **AccountName** | [username] |

- **Evaluación inicial:** Una regla que copia automáticamente correos hacia otras cuentas puede corresponder tanto a una operación legítima de negocio como a una técnica de **exfiltración silenciosa** en un escenario de BEC (Business Email Compromise).

### 2. Revisión de Incidentes Similares

Se consultó el historial de incidentes similares en Sentinel para obtener contexto previo.

- **Incidente #65483 (5/15/2026):** Cerrado como **BenignPositive — Suspicious but Expected** con comentario: *"Legit admin activity"*.
- **Conclusión:** El patrón ya fue investigado y resuelto anteriormente, lo que refuerza la hipótesis de actividad legítima.

### 3. Confirmación con IT Owner

Dado que la táctica de Collection + Exfiltration en MITRE es aplicable a este patrón, se escaló para confirmar la legitimidad y comprender el propósito operativo de la regla.

- **Resultado:** IT confirmó que la regla fue configurada intencionalmente.
- **Propósito:** La regla garantiza que múltiples miembros del equipo reciban copias de comunicaciones relevantes para la operación del área, evitando puntos únicos de fallo en la gestión de información.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Query KQL sobre OfficeActivity | ✅ Completado | Identificación de la operación, destinatarios y origen. |
| Revisión de incidentes similares | ✅ Completado | Contexto previo confirma patrón recurrente benigno. |
| Escalación a IT Owner | ✅ Completado | Confirmación de actividad legítima y propósito operativo. |
| Cierre del incidente | ✅ Completado | BenignPositive — Suspicious but Expected. |

---

## 💡 Lecciones Aprendidas

1. **Transport Rules = vigilancia obligatoria:** Las reglas de transporte son una de las técnicas más comunes en ataques de BEC para exfiltrar correos de forma silenciosa. Toda modificación debe ser investigada, aunque el historial indique benignidad.
2. **"Suspicious but Expected" no es lo mismo que False Positive:** Sentinel está en lo correcto al alertar — el comportamiento es técnicamente sospechoso. La diferencia la da el contexto operativo confirmado.
3. **Inventario de Transport Rules:** Al igual que con las reglas de forwarding, las organizaciones deberían mantener un inventario documentado de todas las Transport Rules activas con su justificación, fecha de creación y responsable, para acelerar la investigación en futuros incidentes similares.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección y gestión del incidente.
- **Log Analytics (KQL)** — Consulta sobre `OfficeActivity`.
- **Exchange Online Admin Center (EXO)** — Contexto sobre reglas de transporte.
- **Microsoft Defender XDR** — Revisión del timeline del incidente.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Collection | Email Collection | T1114 |
| Exfiltration | Exfiltration Over Web Service | T1567 |
| Persistence | Email Forwarding Rule | T1114.003 |
