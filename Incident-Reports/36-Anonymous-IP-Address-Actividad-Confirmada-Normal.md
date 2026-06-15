# Caso 36 — Anonymous IP Address: Actividad de Usuario Confirmada como Normal

## Resumen narrativo

Sentinel generó dos alertas relacionadas con el acceso de un usuario desde una IP clasificada como anónima (hosting/VPN provider). La investigación reveló que la IP correspondía a infraestructura de red de la zona geográfica del usuario y que toda la actividad fue confirmada como normal por el propio usuario. El caso fue derivado para confirmación y cerrado como True Positive Benigno.

---

## Tabla de detalles del incidente

| Campo | Detalle |
|---|---|
| **Incident ID** | — |
| **Título** | Anonymous IP address / Potential user account compromise |
| **Clasificación** | ✅ True Positive Benigno |
| **Severidad** | Medium |
| **Primera actividad** | Jun 9, 2026 — 10:33 AM |
| **Última actividad** | Jun 9, 2026 — 11:04 AM |
| **Usuario afectado** | `[username]@[org-domain]` |
| **IP sospechosa** | `xxx.xxx.xx.xx` |
| **Aplicación** | One Outlook Web |
| **Ubicación declarada** | IT |
| **Fuente de detección** | Microsoft Sentinel / Microsoft Entra ID Protection |

---

## Proceso de investigación

### 1. Alertas del incidente

| Hora | Alerta |
|---|---|
| Jun 9 — 10:33 AM | Potential user account compromise |
| Jun 9 — 11:04 AM | Anonymous IP address |

---

### 2. Análisis de la IP sospechosa (xxx.xxx.xx.xx)

Consulta en IPinfo.io:

| Campo | Valor |
|---|---|
| **Location** |  Italia |
| **ASN** | AS136258 — BrainStorm Network, Inc |
| **Company** | Oneprovider.com - Florence Infrastructure |
| **AS Type** | Hosting |
| **Privacy** | true |
| **Tags** | Hosting, SSH, VPN, Webserver |

La IP está clasificada como Privacy: true y AS Type: Hosting — esto explica por qué Sentinel la detectó como anónima.

---

### 3. Análisis de usuarios desde esa IP

```kql
SigninLogs
| where IPAddress == "xxx.xxx.xx.xx"
| where TimeGenerated > ago(7d)
| summarize count() by UserPrincipalName
| order by count_ desc
```

Resultado: Solo aparece `[username]@[org-domain]` — ningún otro usuario de la organización.

Conclusión: No es proxy corporativo compartido. Es una IP usada específicamente por este usuario.

---

### 4. Análisis de resultados de autenticación

| Result | Cantidad | Significado |
|---|---|---|
| 0 — Success | Mayoría | Logins exitosos |
| 500121 — Authentication failed | 1 | Fallo puntual |
| 50074 — Strong Authentication required | 2 | MFA requerido |
| 50140 — Keep me signed in | Varios | Interrupción normal |

Patrón consistente con actividad normal — predominan los éxitos.

---

### 5. Derivación al cliente para confirmación

Dado que la IP es de tipo hosting/privacidad y los logs solo cubrían 7 días sin historial previo, se derivó al cliente para confirmación.

Respuesta: El usuario confirmó que toda la actividad es normal. La IP corresponde a infraestructura de red de su zona en Cosenza, Italia.

---

## Tabla de respuesta y remediación

| Acción | Responsable | Estado |
|---|---|---|
| Analizar IP en IPinfo | Analista SOC | ✅ Completado |
| Verificar usuarios desde esa IP | Analista SOC | ✅ Solo el usuario afectado |
| Analizar resultados de autenticación | Analista SOC | ✅ Predominan éxitos |
| Derivar al cliente para confirmación | Analista SOC | ✅ Completado |
| Confirmación del usuario | Cliente/IT | ✅ Actividad normal confirmada |

---

## Lecciones aprendidas

1. **Privacy: true en IPinfo no implica ataque** — ISPs de ciertas regiones de Italia usan infraestructura de hosting que hace que sus IPs aparezcan clasificadas como anónimas.

2. **Verificar cuántos usuarios acceden desde la IP es clave** — si solo aparece un usuario es probable que sea su IP personal. Si aparecen muchos puede ser proxy corporativo o atacante.

3. **La falta de historial de logs limita el análisis** — con solo 7 días de retención no se puede comparar si esa IP fue usada previamente. Mayor retención mejoraría la capacidad forense.

4. **La derivación al usuario es válida cuando los datos no son concluyentes** — no siempre hay suficiente evidencia técnica para cerrar sin confirmación humana.

---

## Herramientas utilizadas

- Microsoft Sentinel — Log Analytics, SigninLogs
- KQL — análisis de IPs y autenticaciones
- IPinfo.io — clasificación y geolocalización de IP

---

## MITRE ATT&CK

No aplica — actividad confirmada como legítima por el usuario.
