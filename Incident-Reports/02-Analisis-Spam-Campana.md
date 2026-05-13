# INC-20260419-002 — Análisis de Correo Reportado: Campaña de Spam — **Mitigado**

# 📝 Resumen

El 19 de abril de 2026, un usuario reportó un correo sospechoso en su bandeja de entrada etiquetándolo como "Malware". Tras una investigación técnica exhaustiva utilizando **Microsoft Defender para Office 365** y **Exchange Online**, se determinó que el correo no contenía código malicioso, sino que formaba parte de una **Campaña de Spam** no solicitado (publicidad de cursos) dirigida a múltiples usuarios de la organización.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | 20260419-002 |
| **Severidad** | **MEDIA** |
| **Categoría** | Correo no solicitado / SPAM |
| **Detección Inicial** | Reporte preventivo del usuario a través de la herramienta de Microsoft |

---

## 🛠 Proceso de Investigación

### 1. Evaluación Inicial (Triage)
* **Revisión de actividad:** Se analizó la línea de tiempo del usuario en el portal de seguridad de Microsoft.
* **Resultado:** Se confirmó que el usuario **no interactuó ni hizo clic** en ningún enlace antes de reportar.
* **Análisis de Artefactos:** Inspección de URLs y archivos adjuntos.
    * **URLs:** Los enlaces no presentaban reputación maliciosa en bases de datos de inteligencia de amenazas.
    * **Adjuntos:** El correo no contenía archivos adjuntos.
* **Contenido:** Se identificó que el cuerpo del mensaje correspondía a una oferta de venta de cursos (SPAM).

### 2. Análisis Profundo y Recolección de Evidencia
* **Extracción de datos:** Se obtuvo el **Network Message ID** para rastrear la huella del correo en todo el Tenant.
* **Inspección manual:** Se descargó el archivo original (`.eml`) desde el portal de Exchange para revisar de forma segura los encabezados (*headers*) y los detalles del remitente (*sender*).
* **Hallazgos:** El remitente era una dirección externa sin historial previo de comunicación con la empresa.

### 3. Rastreo de Campaña (Alcance)
* **Acción:** Se utilizó la dirección del remitente para realizar un rastreo de mensajes (*Message Trace*) en Exchange Online.
* **Resultado:** Se confirmó la existencia de una **Campaña**; el mismo correo fue enviado a varios usuarios de la organización en un intervalo de tiempo corto.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
| :--- | :--- | :--- |
| **Búsqueda interna** | ✅ Completado | Uso del Network Message ID para localizar todas las instancias del correo en el Tenant. |
| **Remediación masiva** | ✅ Completado | Se movieron todos los correos identificados de esta campaña a la carpeta de **Correo no deseado (Junk)** de los usuarios afectados. |
| **Reporte de remitente** | ✅ Completado | Se reportó la dirección del remitente a Microsoft como **SPAM** para mejorar los filtros de reputación futuros. |
| **Feedback al usuario** | ✅ Completado | Se notificó al usuario que el correo era seguro pero publicitario, agradeciendo su proactividad. |

---

## 💡 Lecciones Aprendidas

1.  **Concientización del usuario:** El hecho de que el usuario reportara en lugar de hacer clic demuestra la efectividad de las capacitaciones en seguridad.
2.  **Limpieza proactiva:** El uso del **Network Message ID** es fundamental para realizar operaciones de "Búsqueda y Purga" (Search & Purge) y evitar que otros empleados interactúen con la misma campaña.
3.  **Intención vs. Reputación:** Aunque los enlaces no tengan reputación negativa inicial, el patrón de envío masivo indica una campaña que debe ser mitigada para evitar saturación y posibles riesgos futuros.

---

## 📊 Herramientas Utilizadas

* **Microsoft 365 Defender** (Submissions y seguimiento de actividad de usuario).
* **Exchange Online (EXO) Message Trace** (Identificación de otros destinatarios).
* **Threat Explorer / Content Search** (Localización y movimiento de correos).
