# 🛡️ Cybersecurity & SOC Portfolio | José Nicolás Fernández

¡Bienvenido! Soy **Analista en Software** y actualmente me desempeño como **Analista de Seguridad / Formación Práctica en Respuesta a Incidentes**. Mi enfoque principal es la detección y mitigación de amenazas en entornos **Microsoft/Azure**, mientras finalizo mi **Ingeniería de Software** (Tesis en desarrollo).

---

## 🎯 Perfil Profesional
* 🎓 **Formación Académica:** Ingeniero de Software en formación (Universidad Siglo 21) con título intermedio de Analista de Software.
* 🛡️ **Especialización SOC:** Gestión activa de alertas de Phishing, inicios de sesión atípicos y actividad anómala.
* 🚀 **Certificaciones:** SOC Level 1 (TryHackMe), Google Cloud Security y Cisco Junior Cybersecurity.
* ⚙️ **Objetivo:** Obtener la certificación **Microsoft SC-200** para consolidar mis habilidades como Security Operations Analyst.

---

## 📁 Casos de Investigación (Incident Reports)
Aquí documento mi metodología para resolver incidentes reales:

* **[INC-20260418-001] - Phishing & Regla de Reenvío Sospechosa**
    * *Resumen:* Investigación técnica sobre el robo de credenciales y descarte de movimiento lateral en Exchange Online.
    * 👉 [Ver reporte](./Incident-Reports/01-Caso-Phishing.md)

* **[INC-20260419-002] - Análisis de Campaña de Spam Masiva**
    * *Resumen:* Respuesta ante reportes de malware, identificación de campañas mediante Network Message ID y remediación masiva.
    * 👉 [Ver reporte](./Incident-Reports/02-Analisis-Spam-Campana.md)

* **[INC-20260420-003] - Inicios de Sesión Fallidos (Contraseña Vencida)**
    * *Resumen:* Análisis de logs de error 50126 en Entra ID para identificar falsos positivos causados por expiración.
    * 👉 [Ver reporte](./Incident-Reports/03-Login-Fallido-Contrasena-Vencida.md)

* **[INC-20260421-004] - Ubicación de Inicio de Sesión Atípica (VPN)**
    * *Resumen:* Validación de alertas geográficas mediante el análisis de ASN para identificar nodos VPN.
    * 👉 [Ver reporte](./Incident-Reports/04-Acceso-inusual-Viaje-imposible.md)

* **[INC-20260421-005] - Detección de Malware en Herramientas Administrativas**
    * *Resumen:* Análisis de contexto y rol del usuario para identificar falsos positivos en herramientas de IT.
    * 👉 [Ver reporte](./Incident-Reports/05-Deteccion-de-malware.md)

* **[INC-20260421-006] - Instalación de Software No Sancionado (Shadow IT)**
    * *Resumen:* Gestión de incidentes de cumplimiento y escalamiento ante restricciones de firewall.
    * 👉 [Ver reporte](./Incident-Reports/06-ShadowIT.md)

* **[INC-20260421-007] - Phishing con Código de Dispositivo y Spoofing**
    * *Resumen:* Investigación de técnicas de Device Code Phishing y validación de spoofing en KQL.
    * 👉 [Ver reporte](./Incident-Reports/07-Spoofing.md)

* **[INC-20260421-008] - Intentos de Fuerza Bruta Distribuidos en Active Directory**
    * *Resumen:* Hunting proactivo utilizando KQL para identificar fallos de autenticación.
    * 👉 [Ver reporte](./Incident-Reports/08-Fuerza-Bruta.md)

* **[INC-20260422-009] - Detección de Ransomware en Actualización (Falso Positivo)**
    * *Resumen:* Análisis de alerta EDR (CryptoGuard) originada por el comportamiento de un paquete nativo de Windows Update, resuelto mediante validación de heurística e identificadores del proveedor.
    * 👉 [Ver reporte](./Incident-Reports/09-Ransomware(CryptoGuard).md)

* **[INC-20260513-010] - Sophos PUA Cleanup Failed (Generic ML PUA)**
    * *Resumen:* Detección de archivo ZIP legacy marcado por heurística ML de Sophos. Verificado como legítimo mediante historial de incidentes similares previos.
    * 👉 [Ver reporte](./Incident-Reports/10-Sophos-PUA-Cleanup.md)

* **[INC-20260513-011] - Multistage Attack — Purview Role Assignment (Fusion)**
    * *Resumen:* Alerta de Fusion por asignación de roles privilegiados fuera de PIM. Identificado como migración automática legítima de Microsoft Purview mediante análisis de logs.
    * 👉 [Ver reporte](./Incident-Reports/11-Fusion-Purview-RoleAssignment.md)

* **[INC-20260513-012] - Local Admin Group Changes (False Positive)**
    * *Resumen:* Modificación de grupo local en host corporativo. Verificados ambos usuarios en Azure AD como miembros legítimos del área correspondiente.
    * 👉 [Ver reporte](./Incident-Reports/12-LocalAdminGroup-Changes.md)

* **[INC-20260513-013] - SAMR Reconnaissance desde ADCS**
    * *Resumen:* Reconocimiento SAMR desde servidor ADCS hacia Domain Controller. Confirmado como tráfico legítimo del servidor de certificados consultando el DC.
    * 👉 [Ver reporte](./Incident-Reports/13-SAMR-Recon-ADCS.md)

* **[INC-20260513-014] - vmnat.exe Suspicious Activity (False Positive)**
    * *Resumen:* Proceso vmnat.exe con conexión saliente detectado como amenaza. Hash verificado contra binario oficial de VMware. Confirmado legítimo.
    * 👉 [Ver reporte](./Incident-Reports/14-vmnat-Suspicious-Activity.md)

* **[INC-20260513-015] - NetSupport RAT via Malvertising (TRUE POSITIVE) 🔴**
    * *Resumen:* Usuario descargó fake Dynamics365 desde anuncio malicioso de Google. Instalación de NetSupport RAT con persistencia via tarea programada y C2 activo. Equipo aislado y sesiones revocadas.
    * 👉 [Ver reporte](./Incident-Reports/15-NetSupport-RAT-Malvertising.md)

* **[INC-20260515-016] - Command and Control — Botnet IPs vs Servidor DMZ**
    * *Resumen:* Dos IPs clasificadas como maliciosas en feeds de Threat Intelligence (ConfidenceScore 100 en MSTIC HoneyPot) contactaron servidor expuesto a internet. Tráfico de reconocimiento automatizado externo esperado para activos DMZ. Sin evidencia de compromiso.
    * 👉 [Ver reporte](./Incident-Reports/16-C2-BotnetIP-DMZ.md)

* **[INC-20260515-017] - User Assigned Privileged Role Involving Multiple Users**
    * *Resumen:* Asignación manual del rol Authentication Administrator por Global Administrator sobre cuenta ya con múltiples roles privilegiados activos. Gestión administrativa legítima verificada mediante consulta de roles en Azure AD.
    * 👉 [Ver reporte](./Incident-Reports/17-Privileged-Role-Assignment-MultipleUsers.md)

* **[INC-20260519-018] - Local Admin Group Changes — Actividad Autorizada**
    * *Resumen:* Usuario con múltiples grupos RWX y acceso VPN administrativo agregado manualmente a grupo local por cuenta built-in. Patrón repetido en incidentes recientes. Confirmado como actividad legítima del equipo de IT.
    * 👉 [Ver reporte](./Incident-Reports/18-LocalAdminGroup-Changes-Authorized.md)

* **[INC-20260519-019] - Multi-stage: Credential Access & Lateral Movement sobre Domain Controller**
    * *Resumen:* Correlación de auth failures + RDP inusual sobre un Domain Controller con dos cuentas admin de alto privilegio. Actividad confirmada como legítima por los administradores del sistema.
    * 👉 [Ver reporte](./Incident-Reports/19-MultiStage-CredAccess-LateralMovement.md)

* **[INC-20260519-020] - Command and Control — Botnet IPs vs Dispositivo Internet Facing**
    * *Resumen:* IPs de Botnet con ConfidenceScore 100 (MSTIC HoneyPot) contactaron host de la organización. Verificado en Microsoft Defender que el dispositivo está etiquetado como Internet Facing. Tráfico externo automatizado esperado. Sin evidencia de compromiso.
    * 👉 [Ver reporte](./Incident-Reports/20-C2-BotnetIP-InternetFacing.md)

* **[INC-20260519-021] - Unfamiliar Sign-in Properties — Aplicación de RRHH en Azure**
    * *Resumen:* Login desde dispositivo no gestionado en Países Bajos (Azure NL) con deviceId vacío e isManaged false. Identificado como acceso generado por aplicación de Recursos Humanos hosteada en Azure. Patrón confirmado en múltiples usuarios. Propuesta de exclusión en regla analítica.
    * 👉 [Ver reporte](./Incident-Reports/21-UnfamiliarSignin-HRApp.md)

* **[INC-20260519-022] - Mail Redirect via ExO Transport Rule**
    * *Resumen:* Regla de transporte en Exchange Online que copia correos entrantes hacia cuentas internas. Tácticas de Collection y Exfiltration identificadas por Sentinel. Confirmado como configuración administrativa intencional. BenignPositive — Suspicious but Expected.
    * 👉 [Ver reporte](./Incident-Reports/22-MailRedirect-ExO-TransportRule.md)

* **[INC-20260520-023] - User Assigned Privileged Role Involving Multiple Users (MS-PIM)**
    * *Resumen:* Dos cuentas recibieron el rol de Global Administrator vía MS-PIM en el mismo día. Verificado como proceso administrativo controlado mediante análisis de AuditLogs. Initiator confirmado como MS-PIM, indicando solicitud formal de activación de rol.
    * 👉 [Ver reporte](./Incident-Reports/23-Asignacion-Rol-Privilegiado-MS-PIM.md)

* **[INC-20260520-024] - Rare and Potentially High-Risk Office Operations (Exchange Automático)**
    * *Resumen:* Operación Set-Mailbox sobre buzón de arbitraje ejecutada por NT SERVICE\MSExchangeAdminApiNetCore. Confirmado como mantenimiento automático interno de Exchange Online mediante análisis de AccountName, AppPoolName y OfficeTenantId. Sin intervención humana.
    * 👉 [Ver reporte](./Incident-Reports/24-Operaciones-Office-Raras-Exchange-Automatico.md)

* **[INC-20260520-025] - Suspicious OpenClaw Installation**
    * *Resumen:* Instalación del agente de IA open-source OpenClaw detectada en dispositivo corporativo. Software con capacidad de acceder a datos, aplicaciones y sesiones del navegador. Confirmado como instalación intencional por el propio usuario con permisos habilitados.
    * 👉 [Ver reporte](./Incident-Reports/25-Instalacion-Sospechosa-OpenClaw.md)

* **[INC-20260520-026] - Changes to Application Ownership (Azure DevOps)**
    * *Resumen:* Tres eventos simultáneos de cambio de ownership sobre aplicación registrada en Azure AD. Initiator identificado como Azure DevOps en flujo automático de CI/CD. Rol Application Developer del usuario confirmado como consistente con la acción. True Positive previo en incidente similar refuerza la importancia del monitoreo.
    * 👉 [Ver reporte](./Incident-Reports/26-Cambio-Ownership-Aplicacion-DevOps.md)

* **[INC-20260522-027] - Brute Force Attack on Active Directory**
    * *Resumen:* 42 intentos de autenticación fallidos en menos de 4 minutos sobre una cuenta de dominio desde IP interna. Velocidad incompatible con acción humana — posible aplicación con credenciales vencidas. Cuenta bloqueada por política del dominio. Escalado a IT para identificar el proceso origen en el workstation afectado.
    * 👉 [Ver reporte](./Incident-Reports/27-Brute-Force-Active-Directory.md)

* **[INC-20260522-028] - Fusion: Possible Multistage Attack — VPN Corporativa**
    * *Resumen:* Incidente Fusion High correlacionando Atypical Travel + Unfamiliar Sign-in + Potential Compromise sobre cuenta funcional mal clasificada como service account. IP sospechosa en Nueva York identificada como VPN corporativa mediante verificación de múltiples usuarios autenticándose desde la misma IP. Falso positivo por VPN no registrada como Named Location en Azure AD.
    * 👉 [Ver reporte](./Incident-Reports/28-Fusion-Multistage-VPN-Corporativa.md)

* **[INC-20260522-029] - Malicious Network Traffic Blocked — C2/Generic-A (En Investigación)**
    * *Resumen:* Dispositivo interno desconocido intentó comunicarse con servidor C2/Generic-A en 6 ocasiones entre 01:42 y 06:12 AM. Tráfico bloqueado por firewall Sophos. Dispositivo sin agente Sophos, sin registro DNS y fuera del dominio — posible móvil personal infectado conectado al WiFi corporativo. Escalado a IT para identificación via DHCP lease history.
    * 👉 [Ver reporte](./Incident-Reports/29-Malicious-Traffic-C2-Sophos-Bloqueado.md)

* **[INC-20260526-030] - Fusion: Possible Multistage Attack — Credenciales Comprometidas**
    * *Resumen:* Incidente Fusion High correlacionando Atypical Travel + Unfamiliar Sign-in + Potential Compromise. IP de Ucrania con un único usuario intentó autenticarse con credenciales válidas — bloqueado por MFA (Error 50074). IP corporativa verificada con 234 usuarios distintos. Credenciales comprometidas confirmadas. Sesiones revocadas y contraseña forzada desde Microsoft Defender.
    * 👉 [Ver reporte](./Incident-Reports/30-Fusion-Multistage-Credenciales-Comprometidas.md)

* **[INC-20260526-031] - Nueva Credencial Agregada a Aplicación — Integración SSO Keycloak**
    * *Resumen:* Global Administrator agregó credencial de tipo Password (Keycloak) a aplicación IAM corporativa para configurar integración SSO. IP conocida del administrador, UserAgent real y cuenta administrativa dedicada confirmaron actividad legítima. Técnica idéntica a App Consent Abuse pero con contexto administrativo verificado.
    * 👉 [Ver reporte](./Incident-Reports/31-Nueva-Credencial-Aplicacion-SSO-Keycloak.md)

* **[INC-20260526-032] - Cuenta Creada desde Fuente No Aprobada — Azure AD Connect**
    * *Resumen:* Cuenta creada automáticamente por el proceso de sincronización Azure AD Connect (ConnectSyncProvisioning). Verificado directamente en AuditLogs del portal del cliente — campo "Initiated by" de Type: Application sin intervención humana. Los intentos fallidos previos corresponden al comportamiento normal del ciclo de sincronización.
    * 👉 [Ver reporte](./Incident-Reports/32-Cuenta-Creada-ADConnect-Sincronizacion.md)
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
