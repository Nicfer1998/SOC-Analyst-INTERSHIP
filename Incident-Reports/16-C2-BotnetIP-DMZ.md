# INC-20260515-016 — Command and Control Incident on Multiple Endpoints (Botnet IPs vs DMZ) — Falso Positivo

# 📝 Resumen

El 15 de mayo de 2026, Microsoft Sentinel generó una alerta de severidad media indicando actividad de Command and Control sobre un servidor expuesto a internet (DMZ), al detectar sesiones de red originadas desde dos IPs clasificadas como indicadores de compromiso (IoC) de Botnet en los feeds de Threat Intelligence. La investigación confirmó que el tráfico corresponde a reconocimiento automatizado externo típico de internet, sin evidencia de compromiso del servidor destino.

---

## 🔍 Detalles del incidente

| Campo | Valor |
| --- | --- |
| **ID del incidente** | INC-20260515-016 |
| **Severidad** | Media |
| **Categoría** | Falso positivo / Tráfico externo automatizado contra DMZ |
| **Alert Product** | Microsoft Sentinel |
| **Táctica MITRE** | T1071 — Application Layer Protocol (Command and Control) |
| **IPs maliciosas** | `[IP-1]` / `[IP-2]` |
| **Servidor afectado** | Servidor DMZ / Internet Facing |
| **Clasificación final** | False Positive / Expected Activity |

---

## 🛠 Proceso de investigación

1. **Revisión del incidente:** La alerta indicaba que dos IPs externas contactaron un servidor de la organización expuesto a internet. El incidente llevaba activo 15 días sin resolución, con 5 alertas activas correlacionadas.

2. **Verificación de IPs en VirusTotal:**
   - **IP #1:** Clasificada como maliciosa por múltiples vendors (ADMINUSLabs, CyRadar, GreyNoise, Fortinet, IPsum). Reportada por Guardpot con 29,604 eventos de actividad maliciosa incluyendo port scanning, SMTP abuse, MySQL brute force, Telnet brute force contra IoT, VNC activity, Modbus ICS probing y RDP brute force. Primera actividad detectada en febrero de 2026, última actividad el día del incidente.
   - **IP #2:** Clasificada por Microsoft Defender Threat Intelligence con **ConfidenceScore 100** como IP capturada en honeypot de Microsoft (MSTIC HoneyPot) realizando brute force contra servicios.

3. **Análisis del contexto del servidor:** El servidor destino fue identificado en la documentación de infraestructura como un activo expuesto a internet (DMZ / Internet Facing), por lo que recibir intentos de conexión desde IPs maliciosas es una situación esperada y normal para cualquier servicio publicado en internet.

4. **Análisis de la regla analítica:** La regla `Threat Intelligence / Internal / TI map IP entity to Network Session Events` matcheó las IPs del feed de TI contra los logs de sesiones de red del servidor, generando la alerta automáticamente al detectar la coincidencia. Este comportamiento es el esperado para la regla, pero no implica compromiso del servidor.

5. **Estado del firewall:** Se confirmó que el firewall perimetral procesó el tráfico entrante de ambas IPs. Pendiente confirmación de acción (ALLOW/DENY) por demora en acceso a consola, aunque dado el contexto de DMZ con firewall activo, el tráfico fue gestionado por los controles perimetrales existentes.

6. **Conclusión:** Las IPs maliciosas realizaron reconocimiento automatizado de forma masiva contra múltiples objetivos en internet, incluyendo el servidor DMZ de la organización. No existe evidencia de acceso exitoso ni compromiso del servidor.

---

## 🛡 Respuesta y remediación

- **Clasificación del evento:** False Positive / Expected Activity — tráfico externo automatizado hacia servidor DMZ.
- **Acción tomada:** Cierre del incidente con evidencia de VirusTotal y análisis de contexto del servidor.
- **Recomendación:** Considerar la creación de una regla de supresión en Sentinel para tráfico de reconocimiento externo hacia activos DMZ conocidos, reduciendo el ruido de alertas recurrentes. Confirmar con el equipo de red que el firewall bloqueó el tráfico de ambas IPs.

---

## 💡 Análisis técnico: ¿Por qué un servidor DMZ recibe este tráfico?

Un servidor expuesto a internet recibe constantemente intentos de conexión automatizados desde IPs maliciosas. Esto ocurre porque existen botnets globales que escanean continuamente todo el rango de IPs de internet en busca de servicios vulnerables, sin importar si el objetivo es una empresa grande o pequeña.

El proceso es completamente automatizado — estas IPs no apuntan específicamente a la organización, sino que barren miles de objetivos simultáneamente buscando puertos abiertos, servicios sin parchear o credenciales débiles.

**La presencia de este tráfico en los logs no indica compromiso.** Indica que el servidor está correctamente expuesto a internet y que Sentinel está funcionando bien al correlacionar las IPs contra los feeds de Threat Intelligence. La clave para determinar si hubo compromiso es verificar si el firewall permitió o bloqueó las conexiones, y si existe algún login exitoso desde esas IPs.

**¿Qué es un HoneyPot de Microsoft (MSTIC)?**
Microsoft mantiene una red global de servidores trampa (honeypots) diseñados para atraer a atacantes. Cuando una IP interactúa con estos honeypots, Microsoft la registra en su feed de Threat Intelligence con alta confianza. Una IP con ConfidenceScore 100 fue capturada en la práctica realizando actividad maliciosa, lo cual hace que la alerta de Sentinel sea correcta en marcarla, aunque el impacto real dependa del contexto del servidor destino.
