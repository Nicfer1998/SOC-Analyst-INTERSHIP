# INC-20260519-003 — Command and Control Incident on Multiple Endpoints — **Verdadero Positivo Benigno**

# 📝 Resumen

El 19 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad media al detectar que dos IPs externas, identificadas en feeds de Threat Intelligence como pertenecientes a botnets, establecieron conexiones de red con un host de la organización. Tras una investigación técnica con KQL sobre la tabla `ThreatIntelligenceIndicator` y verificación del estado del dispositivo en **Microsoft Defender**, se determinó que el host está expuesto a Internet de forma intencional, por lo que el tráfico recibido es ruido esperado y el incidente fue cerrado como Verdadero Positivo Benigno.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260519-003 |
| **Incidente Sentinel** | #30896 |
| **Severidad** | **MEDIA** |
| **Categoría** | Command & Control |
| **Detección Inicial** | Microsoft Sentinel — TI Map IP Entity to DeviceNetworkEvents |
| **Host afectado** | [hostname].[org-domain] |
| **IPs externas** | 3.130.168.2 / 205.210.31.131 |

---

## 🛠 Proceso de Investigación

### 1. Análisis del Timeline

El incidente agrupó 4 alertas distribuidas en dos semanas:

| Fecha | Alerta | Tipo |
|---|---|---|
| May 5 — 05:31 | Network session desde 3.130.168.2 | Media |
| May 5 — 07:31 | Network session desde 205.210.31.131 | Media |
| May 19 — 04:51 | TI Map IP → DeviceNetworkEvents | Media |
| May 19 — 06:51 | TI Map IP → DeviceNetworkEvents | Media |

- **Observación:** Las alertas del 19 de mayo corresponden a un match entre las IPs y los feeds de **Threat Intelligence de Microsoft (MSTIC)**, lo que indica que estas IPs son conocidas como maliciosas.

### 2. Threat Intelligence — Análisis de IPs con KQL

Se ejecutaron consultas KQL sobre `ThreatIntelligenceIndicator` para obtener el perfil de ambas IPs.

**Resultados:**

| Campo | IP 1 | IP 2 |
|---|---|---|
| **IP** | 3.130.168.2 | 205.210.31.131 |
| **ThreatType** | Botnet | Botnet |
| **ConfidenceScore** | 100/100 | 100/100 |
| **Fuente** | MSTIC HoneyPot | MSTIC HoneyPot |
| **Descripción** | Brute force attack | Brute force attack |
| **DstIpAddr** | 172.17.100.191 | 172.17.100.191 |

- **Hallazgo:** Ambas IPs tienen **ConfidenceScore de 100**, capturadas por honeypots de Microsoft. Son actores activos de botnet que realizan ataques de fuerza bruta de forma masiva contra múltiples targets en Internet.

### 3. Verificación del Estado del Dispositivo

Se formuló la hipótesis de que si el host está expuesto a Internet, recibir escaneos de botnets es comportamiento esperado y no implica compromiso.

- **Acción:** Búsqueda del dispositivo en **Microsoft Defender → Assets → Devices**.
- **Resultado:** El dispositivo aparece etiquetado como **"Internet Facing"**.
- **Conclusión:** El tráfico de botnet hacia un dispositivo expuesto a Internet es ruido normal. No hay evidencia de compromiso exitoso.

- **Referencia:** Los incidentes similares #30870 y #30850 fueron cerrados previamente como Benign por la misma razón. El incidente #30701 fue cerrado como True Positive (comprometido), lo que confirma que el monitoreo de este host es pertinente.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Query KQL sobre TI Indicators | ✅ Completado | Identificación de ambas IPs como Botnet con ConfidenceScore 100. |
| Verificación en Defender Assets | ✅ Completado | Confirmación del estado "Internet Facing" del dispositivo. |
| Revisión de incidentes similares | ✅ Completado | Contexto histórico confirma patrón de ruido esperado. |
| Cierre del incidente | ✅ Completado | Clasificado como True Positive — Benign (Internet Facing). |

---

## 💡 Lecciones Aprendidas

1. **El contexto del asset es determinante:** La misma alerta en un dispositivo interno representaría un incidente crítico. En un dispositivo Internet Facing, es ruido esperado. Verificar el estado del asset antes de escalar es una práctica esencial.
2. **TI Confidence Score 100:** Un score máximo de MSTIC indica certeza absoluta sobre la malicia de una IP. Esto es valioso para priorizar investigaciones, pero debe combinarse con el contexto del entorno.
3. **Historial de incidentes similares:** Revisar el historial de casos similares permite acelerar el proceso de investigación y evitar trabajo duplicado.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección y correlación del incidente.
- **Log Analytics (KQL)** — Consulta sobre `ThreatIntelligenceIndicator`.
- **Microsoft Defender for Endpoint** — Verificación del estado del dispositivo (Internet Facing).

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Command & Control | Application Layer Protocol | T1071 |
| Reconnaissance | Active Scanning | T1595 |
