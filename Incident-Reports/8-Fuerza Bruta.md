# INC-20260421-008 — Intentos de fuerza bruta distribuidos en Active Directory — Flso Positivo

# 📝 Resumen
Durante una sesión de threat hunting, se detectaron múltiples intentos de inicio de sesión fallidos (Event ID 4625) distribuidos desde varias direcciones IP internas hacia una cuenta con privilegios administrativos (`mp-admin`). Mediante el uso de consultas avanzadas en KQL, se determinó que los fallos correspondían a autenticaciones de red (NTLM) con credenciales desactualizadas. El incidente fue clasificado como un falso positivo operativo (credenciales cacheadas/servicios mal configurados) y no como un ataque real.

---

## 🔍 Detalles del incidente
| Campo | Valor |
|---|---|
| **ID del incidente** | 20260421-008 |
| **severidad** | **Media** |
| **categoría** | Anomalía de autenticación / fuerza bruta sospechada |
| **detección** | Hunting proactivo mediante scripts de kql en microsoft sentinel |

---

## 🛠 Proceso de investigación
1. **Ejecución de threat hunting (kql):** se ejecutó una consulta personalizada sobre la tabla `SecurityEvent` para monitorear el evento 4625. la consulta incluyó una función de parseo (`case`) para traducir los subestados hexadecimales de windows a razones de fallo legibles por humanos.
2. **Análisis de resultados:** los logs revelaron múltiples fallos con el motivo "Unknown user name or bad password". 
3. **Evaluación de contexto de red:** se verificó que las direcciones IP de origen (`192.168.12.22`, `192.168.90.225`, etc.) pertenecían a la red interna (LAN). además, el `LogonTypeName` fue identificado como `3 - Network` usando el protocolo `NTLM`.
4. **Hipótesis confirmada:** el patrón de una cuenta de administrador fallando por red desde múltiples IPs internas simultáneamente es el comportamiento clásico de un servicio, tarea programada o unidad de red mapeada que intenta autenticarse usando una contraseña antigua que ya fue rotada.

---

## 🛡 Respuesta y remediación
* **Clasificación:** se descartó la presencia de un actor de amenazas interno realizando fuerza bruta o movimiento lateral.
* **Remediación operativa:** se derivó un ticket al equipo de infraestructura/sysadmins con el listado exacto de las estaciones de trabajo, para que actualicen la credencial de la cuenta `mp-admin` en los servicios locales que están generando el ruido.

---

## 💡 Lecciones aprendidas
* **Valor del parseo en kql:** traducir códigos de error nativos de windows (como `0xC000006A`) directamente en la consulta de kql reduce drásticamente el tiempo de triaje (MTTR), permitiendo al analista entender la causa raíz sin tener que buscar los códigos manualmente.
* **Diferenciación de ataques:** entender los tipos de inicio de sesión de windows (logon type 3 vs logon type 2 o 10) es fundamental para distinguir entre un ataque interactivo por RDP y un proceso automatizado de fondo en la red local.
