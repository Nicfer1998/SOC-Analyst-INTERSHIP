# Caso 35 — First Credential Added to Application: Sophos VPN SSO

## Resumen narrativo

Sentinel generó una alerta al detectar que se agregó la primera credencial (secret/certificado) a la aplicación **Sophos VPN SSO** en Azure AD — donde previamente no existía ninguna. Este tipo de actividad puede indicar persistencia maliciosa post-compromiso, pero tras investigar se confirmó que fue una acción legítima de un administrador realizando tareas de configuración planificadas de la VPN corporativa.

---

## Tabla de detalles del incidente

| Campo | Detalle |
|---|---|
| **Incident ID** | — |
| **Título** | First access credential added to application or service principal where no credential was present |
| **Clasificación** | ✅ True Positive Benigno |
| **Severidad** | High |
| **Fecha/Hora** | Jun 9, 2026 — 08:14 AM |
| **Usuario** | `[admin-user]@[org-domain]` |
| **IP origen** | `[corp-ip]` (xx.xxx.xxx.xxx) |
| **Aplicación afectada** | Sophos VPN SSO |
| **App ID** | `[app-id]` |
| **Tipo de credencial** | Password (secret) |
| **Key name** | VPN - Cernusco |
| **Key usage** | Verify |
| **User Agent** | Windows 10 x64, Chrome 148 |
| **Fuente de detección** | Microsoft Sentinel (Analytics Rule) |
| **Workspace** | `[workspace-name]` |

---

## Proceso de investigación

### 1. Primera lectura de la alerta

La alerta detectó la operación:
> `Update application – Certificates and secrets management`

Agregar una credencial a una aplicación donde no había ninguna es una señal de alerta porque es una técnica conocida de persistencia — un atacante que compromete una cuenta admin puede agregar sus propias credenciales a una app para mantener acceso aunque se cambien contraseñas.

Datos iniciales relevantes:
- La aplicación es **Sophos VPN SSO** — aplicación de autenticación de VPN corporativa
- El usuario que realizó la acción: `[admin-user]@[org-domain]`
- IP de origen: `xx.xxx.xxx.xxx`

---

### 2. Análisis de la IP de origen

```kql
SigninLogs
| where IPAddress == "xx.xxx.xxx.xxx"
| summarize count() by UserPrincipalName
| order by count_ desc
```

**Resultado:** Solo aparece `[admin-user]@[org-domain]` con **604 eventos** — todos o casi todos exitosos (ResultType 0).

**Conclusión:** Es la IP habitual de trabajo del administrador. No es una IP nueva ni sospechosa.

---

### 3. Análisis de resultados de autenticación

```kql
SigninLogs
| where IPAddress == "xx.xxx.xxx.xxx"
| where UserPrincipalName == "[admin-user]@[org-domain]"
| summarize count() by Result
```

| Result | Cantidad | Significado |
|---|---|---|
| 0 — Success | 634 | Logins exitosos — actividad normal |
| 50140 — Keep me signed in | 7 | Interrupción normal |
| 50126 — Invalid credentials | 2 | ⚠️ Fallos de credenciales |

Los 2 fallos (50126) ocurrieron **antes** del evento que generó la alerta — probablemente typos antes de autenticarse correctamente. No están relacionados con la acción sobre la app.

---

### 4. Contexto de la aplicación

La aplicación **Sophos VPN SSO** existe desde **2017** en el tenant — no es una app nueva ni desconocida. La acción fue agregar una credencial para un key llamado **"VPN - Cernusco"**, lo que sugiere una configuración de VPN para una ubicación específica.

La presencia de cuentas `test.vpn.sso@[org-domain]` y `testvpn@[org-domain]` activas desde la misma IP confirma que el equipo estaba realizando configuraciones y pruebas de la integración SSO de Veeam.

---

### 5. Confirmación con el cliente

Se consultó al equipo IT del cliente si había un cambio planificado en la configuración de Sophos VPN SSO el 9 de junio.

**Respuesta:** Confirmaron que estaban realizando configuración de la integración SSO para la VPN de la sede de Cernusco. La acción fue deliberada y planificada.

---

## Tabla de respuesta y remediación

| Acción | Responsable | Estado |
|---|---|---|
| Verificar legitimidad de la IP de origen | Analista SOC | ✅ Confirmado — IP habitual del admin |
| Verificar que la aplicación Sophos VPN SSO es legítima | Analista SOC | ✅ Confirmado — existe desde 2017 |
| Confirmar cambio planificado con el cliente | Analista SOC | ✅ Confirmado por equipo IT |
| Verificar que no hay otras credenciales agregadas no autorizadas | Analista SOC | ✅ Sin hallazgos adicionales |

---

## Lecciones aprendidas

1. **Agregar credenciales a apps es una técnica de persistencia real** — T1098.001. Por eso la alerta se genera aunque la actividad sea legítima. Siempre requiere verificación.

2. **El historial de la IP es el primer indicador clave** — 604 eventos previos del mismo admin desde esa IP desmonta inmediatamente la hipótesis de cuenta comprometida desde IP nueva.

3. **Verificar con el cliente antes de escalar** — cambios de configuración en apps de SSO/VPN son comunes en entornos corporativos. Un ticket de cambio aprobado o confirmación del equipo IT cierra el caso.

4. **La antigüedad de la aplicación importa** — una app creada en 2017 con historial de uso es completamente diferente a una app recién creada por un atacante.

---

## Herramientas utilizadas

- Microsoft Sentinel — Log Analytics, AuditLogs, SigninLogs
- KQL — análisis de IPs y autenticaciones
- Azure AD / Microsoft Entra ID — perfil de la app y del usuario

---

## MITRE ATT&CK

No aplica en este caso — actividad administrativa legítima confirmada. La técnica de referencia para contexto sería T1098.001 (Account Manipulation: Additional Cloud Credentials) si hubiera sido malicioso.
