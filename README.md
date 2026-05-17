# 🛡️ Cybersecurity & SOC Portfolio | José Nicolás Fernández

¡Bienvenido! Soy **Analista en Software** y actualmente me desempeño como **Analista de Seguridad / Formación Práctica en Respuesta a Incidentes**. Mi enfoque principal es la detección y mitigación de amenazas en entornos **Microsoft/Azure**, mientras finalizo mi **Ingeniería de Software** (Tesis en desarrollo).

---

## 🎯 Perfil Profesional
* 🎓 **Formación Académica:** Ingeniero de Software en formación (Universidad Siglo 21) con título intermedio de Analista de Software.
* 🛡️ **Especialización SOC:** Gestión activa de alertas de Phishing, inicios de sesión atípicos y actividad anómala.
* 🚀 **Certificaciones:** SOC Level 1 (TryHackMe), Google Cloud Security y Cisco Junior Cybersecurity.
* ⚙️ **Objetivo:** Obtener la certificación **Microsoft SC-200** para consolidar mis habilidades como Security Operations Analyst.

---

## 📁 casos de investigación (incident reports)
aquí documento mi metodología para resolver incidentes reales:

* **[INC-20260418-001] - phishing & regla de reenvío sospechosa**
    * *resumen:* investigación técnica sobre el robo de credenciales y descarte de movimiento lateral en exchange online.
    * 👉 [ver reporte](./Incident-Reports/1-Caso-Phishing.md)
      
* **[INC-20260419-002] - análisis de campaña de spam masiva**
    * *resumen:* respuesta ante reportes de malware, identificación de campañas mediante network message id y remediación masiva.
    * 👉 [ver reporte](./Incident-Reports/2-Analisis-Spam-Campana.md)
      
* **[INC-20260420-003] - inicios de sesión fallidos (contraseña vencida)**
    * *resumen:* análisis de logs de error 50126 en entra id para identificar falsos positivos causados por expiración.
    * 👉 [ver reporte](./Incident-Reports/3-Login-Fallido-Contrasena-Vencida.md)
      
* **[INC-20260421-004] - ubicación de inicio de sesión atípica (VPN)**
    * *resumen:* validación de alertas geográficas mediante el análisis de asn para identificar nodos vpn.
    * 👉 [ver reporte](./Incident-Reports/4-Acceso-inusual-Viaje-imposible.md)
      
* **[INC-20260421-005] - detección de malware en herramientas administrativas**
    * *resumen:* análisis de contexto y rol del usuario para identificar falsos positivos en herramientas de it.
    * 👉 [ver reporte](./Incident-Reports/5-Deteccion-de-malware.md)
      
* **[INC-20260421-006] - instalación de software no sancionado (shadow IT)**
    * *resumen:* gestión de incidentes de cumplimiento y escalamiento ante restricciones de firewall.
    * 👉 [ver reporte](./Incident-Reports/6-ShadowIT.md)
      
* **[INC-20260421-007] - phishing con código de dispositivo y spoofing**
    * *resumen:* investigación de técnicas de device code phishing y validación de spoofing en kql.
    * 👉 [ver reporte](./Incident-Reports/7-Spoofing.md)
      
* **[INC-20260421-008] - intentos de fuerza bruta distribuidos en active directory**
    * *resumen:* hunting proactivo utilizando kql para identificar fallos de autenticación.
    * 👉 [ver reporte](./Incident-Reports/8-Fuerza-Bruta.md)
      
* **[INC-20260422-009] - Detección de ransomware en actualización (Falso positivo)**
    * *Resumen:* Análisis de alerta EDR (CryptoGuard) originada por el comportamiento de un paquete nativo de Windows Update, resuelto mediante validación de heurística e identificadores del proveedor.
    * 👉 [Ver reporte](./Incident-Reports/9-Ransomware(CryptoGuard).md)
      
- **[INC-20260513-010] - Sophos PUA Cleanup Failed (Generic ML PUA)**
  * *Resumen:* Detección de archivo ZIP legacy marcado por heurística ML de Sophos. Verificado como legítimo mediante historial de incidentes similares previos.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/10-Sophos-PUA-Cleanup.md)

- **[INC-20260513-011] - Multistage Attack - Purview Role Assignment (Fusion)**
  * *Resumen:* Alerta de Fusion por asignación de roles privilegiados fuera de PIM. Identificado como migración automática legítima de Microsoft Purview mediante análisis de logs.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/11-Fusion-Purview-RoleAssignment.md)

- **[INC-20260513-012] - Local Admin Group Changes (False Positive)**
  * *Resumen:* Modificación de grupo local en vaxx01. Verificados ambos usuarios en Azure AD como miembros legítimos del área de logística.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/12-LocalAdminGroup-Changes.md)

- **[INC-20260513-013] - SAMR Reconnaissance desde ADCS**
  * *Resumen:* Reconocimiento SAMR desde GPZ-CA01 hacia GPZ-DC01. Confirmado como tráfico legítimo del servidor ADCS consultando el Domain Controller.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/13-SAMR-Recon-ADCS.md)

- **[INC-20260513-014] - vmnat.exe Suspicious Activity (False Positive)**
  * *Resumen:* Proceso vmnat.exe con conexión saliente detectado como amenaza. Hash verificado contra binario oficial de VMware. Confirmado legítimo.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/14-vmnat-Suspicious-Activity.md)

- **[INC-20260513-015] - NetSupport RAT via Malvertising (TRUE POSITIVE) 🔴**
  * *Resumen:* Usuario descargó fake Dynamics365 desde anuncio malicioso de Google. Instalación de NetSupport RAT con persistencia via tarea programada y C2 activo. Equipo aislado y sesiones revocadas.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/15-NetSupport-RAT-Malvertising.md)

- **[INC-20260515-016] - Command and Control — Botnet IPs vs Servidor DMZ**
  * *Resumen:* Dos IPs clasificadas como maliciosas en feeds de Threat Intelligence (ConfidenceScore 100 en MSTIC HoneyPot) contactaron servidor expuesto a internet. Tráfico de reconocimiento automatizado externo esperado para activos DMZ. Sin evidencia de compromiso.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/16-C2-BotnetIP-DMZ.md)
    
- **[INC-20260515-017] - User Assigned Privileged Role Involving Multiple Users**
  * *Resumen:* Asignación manual del rol Authentication Administrator por Global Administrator sobre cuenta ya con múltiples roles privilegiados activos. Gestión administrativa legítima verificada mediante consulta de roles en Azure AD.
  * 👉 [Ver reporte](https://github.com/Nicfer1998/SOC-Analyst-INTERSHIP/blob/main/Incident-Reports/17-Privileged-Role-Assignment-MultipleUsers.md)

---

## 🛠️ Habilidades Técnicas (Tech Stack)
* **SIEM & Monitoreo:** Azure Sentinel (Microsoft Sentinel), Splunk, Microsoft Defender.
* **Seguridad de Red:** Nmap, Snort, Sophos Firewall, Filtrado MAC/IP.
* **Identidad (IAM):** Active Directory y Azure AD (Entra ID).
* **Protocolos:** TCP/IP, DNS, VPN (IPsec/TLS), Syslog.
* **Scripting & Dev:** Python, PowerShell, Bash, Java.

---

## 💼 Experiencia Relevante
* **Respuesta a Incidentes (Azure/Microsoft):** Triaje de alertas semanales y ejecución de acciones correctivas inmediatas (Reset de MFA/Sesiones).
* **Desarrollo de Software:** Diseño de sistemas de gestión que redujeron un 50% los tiempos administrativos manuales.
* **Soporte Técnico Nivel 1:** Diagnóstico de redes xDSL/Wi-Fi y documentación exhaustiva de incidencias.

---

## 👤 Información y Contacto
* 📍 **Ubicación:** Córdoba, Argentina.
* 🗣️ **Idiomas:** Español (Nativo) e Inglés (Nivel B1 - Intermedio). PROGRESO CONTINUO
* 🔗 **Redes:** [LinkedIn](https://www.linkedin.com/in/nicfer1998) | [TryHackMe](https://tryhackme.com/p/nicfer1998) | [credly EMPRESARIAL](https://www.credly.com/users/nicfer1998/badges#credly) | [credly PERSONAL](https://www.credly.com/users/nicfer1998/badges#credly)
* ✉️ **Email:** Jose.Nicolas.Fernandez@outlook.es

