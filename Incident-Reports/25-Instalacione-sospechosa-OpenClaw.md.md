# INC-20260520-025 — Suspicious OpenClaw Installation — **Verdadero Positivo Benigno**

# 📝 Resumen

El 20 de mayo de 2026, Microsoft Sentinel generó una alerta de severidad baja al detectar la instalación del agente de IA open-source **OpenClaw** en un dispositivo corporativo. Tras la confirmación directa con el usuario propietario del dispositivo, se determinó que la instalación fue realizada de forma intencional por el propio usuario, quien cuenta con permisos para instalar software en su equipo. El caso fue cerrado como Verdadero Positivo Benigno.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260520-025 |
| **Incidente Sentinel / ITSM** | #4660 / Defence Center #117416 |
| **Severidad** | **BAJA** |
| **Categoría** | Collection |
| **Detección Inicial** | Microsoft Sentinel — Suspicious OpenClaw Installation |
| **Dispositivo afectado** | Dispositivo corporativo interno |
| **Usuario involucrado** | Empleado interno con permisos de instalación |

---

## 🛠 Proceso de Investigación

### ¿Qué es OpenClaw?

**OpenClaw** es un agente de IA open-source que, una vez instalado en un dispositivo, puede:
- Acceder a **datos del usuario** almacenados localmente
- Interactuar con **aplicaciones** instaladas en el sistema
- Acceder a **sesiones activas del navegador**, incluyendo cookies y tokens

En un entorno corporativo, estas capacidades representan un riesgo de seguridad significativo si la instalación no es autorizada, ya que podrían ser explotadas para **exfiltración silenciosa de datos** o acceso no autorizado a aplicaciones corporativas.

### 1. Evaluación Inicial

La alerta fue categorizada bajo la táctica de **Collection**, lo que es técnicamente correcto dado que OpenClaw tiene capacidad de acceder y recopilar información del sistema y del usuario.

Se identificaron dos escenarios posibles:
- **Escenario A:** Usuario instaló la herramienta por cuenta propia con permisos → Benigno
- **Escenario B:** Instalación no autorizada o forzada por malware → Crítico

### 2. Confirmación con el Usuario

Dado que el dispositivo pertenece a un empleado interno de la organización, se contactó directamente al usuario para confirmar la instalación.

- **Resultado:** El usuario confirmó haber instalado OpenClaw de forma intencional en su propio dispositivo.
- **Permisos:** El usuario cuenta con permisos para instalar software en su equipo corporativo.

### 3. Contexto del Entorno

Al tratarse de un dispositivo interno de la propia empresa donde se realiza la pasantía, la alerta fue generada por el SOC como parte del monitoreo estándar del entorno, sin implicar una amenaza externa.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Análisis de la alerta | ✅ Completado | Identificación del software instalado y evaluación del riesgo potencial. |
| Confirmación con el usuario | ✅ Completado | Usuario confirmó instalación intencional con permisos habilitados. |
| Cierre del incidente | ✅ Completado | True Positive — Benign Activity. |

---

## 💡 Lecciones Aprendidas

1. **Los agentes de IA como nuevo vector de alerta:** Las herramientas de IA open-source instaladas en dispositivos corporativos son un vector de seguridad emergente en 2025-2026. Los SOC están comenzando a incorporar reglas de detección específicas para este tipo de software, lo que refleja la evolución del panorama de amenazas.
2. **Capacidades técnicas ≠ intención maliciosa:** OpenClaw tiene capacidades que técnicamente podrían usarse para exfiltración. Pero tener esas capacidades no implica que se estén usando de forma maliciosa. El contexto del usuario y sus permisos determina la clasificación.
3. **Política de software de IA en entornos corporativos:** Este caso evidencia la necesidad de que las organizaciones definan políticas claras sobre el uso de agentes de IA en dispositivos corporativos, dado su acceso potencial a datos sensibles y sesiones activas.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección del incidente.
- **Microsoft Defender XDR** — Contexto del dispositivo afectado.
- **Portal ITSM [itsm-provider]** — Gestión y seguimiento del ticket.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Collection | Data from Local System | T1005 |
| Collection | Browser Session Hijacking | T1185 |
