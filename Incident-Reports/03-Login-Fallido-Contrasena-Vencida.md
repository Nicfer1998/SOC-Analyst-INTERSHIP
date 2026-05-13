# INC-20260420-003 — Inicios de Sesión Fallidos Masivos — **Falso Positivo**

# 📝 Resumen Ejecutivo

El 20 de abril de 2026, se detectó una alerta de seguridad por múltiples intentos fallidos de inicio de sesión en la cuenta de un usuario. Tras investigar los logs en **Microsoft Entra ID**, se determinó que los intentos provenían de dispositivos legítimos del usuario intentando autenticarse con una contraseña expirada. El incidente fue cerrado como un **Falso Positivo** de origen operativo.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | 20260420-003 |
| **Severidad** | **MEDIA** |
| **Categoría** | Acceso fallido / Fuerza bruta (Sospecha) |
| **Detección Inicial** | Alerta automática por múltiples errores de autenticación en corto tiempo |

---

## 🛠 Proceso de Investigación

### 1. Análisis de Logs de Inicio de Sesión (Sign-in Logs)
Se ingresó al apartado de usuario en **Microsoft Entra ID** para revisar el historial de accesos:
* **Código de Error:** Se identificó de forma recurrente el error **50126** (Error al validar credenciales debido a una contraseña inválida).
* **Verificación de Dispositivos:** Se comprobó que los intentos provenían de equipos registrados y conocidos por la organización (Laptop corporativa y dispositivo móvil del usuario).
* **Geolocalización:** El origen de la IP coincidía con la ubicación habitual de trabajo del usuario, descartando accesos desde regiones sospechosas.

### 2. Revisión de política de vredenciales
Se procedió a verificar el estado de la cuenta:
* **Último Cambio de Contraseña:** Se detectó que el último cambio ocurrió hace exactamente 91 días.
* **Causa Raíz:** La política de rotación de la empresa (90 días) había expirado la contraseña. El dispositivo móvil del usuario seguía intentando sincronizar el correo automáticamente con la credencial antigua, disparando la alerta de seguridad.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
| :--- | :--- | :--- |
| **Validación de identidad** | ✅ Completado | Se contactó al usuario para confirmar si estaba teniendo problemas de acceso. |
| **Reseteo de contraseña** | ✅ Completado | Se asistió al usuario en el cambio de contraseña a través del portal de autoservicio. |
| **Sincronización de equipos** | ✅ Completado | Se solicitó al usuario actualizar la nueva credencial en sus aplicaciones de escritorio y móviles para detener los logs fallidos. |

---

## 💡 Lecciones aprendidas

1.  **Diferenciación Técnica:** Es crucial distinguir entre un ataque de "Spray" o "Brute Force" y una desincronización de credenciales. La clave está en el **origen (IP/Dispositivo)** y el código de error específico.
2.  **Patrones de Expiración:** Mantener presente la duración de las políticas de la empresa (30/60/90 días) permite al analista hipotetizar rápidamente sobre la causa de fallos masivos en días específicos del mes.
3.  **Higiene de Aplicaciones:** Se recomienda instruir a los usuarios sobre la importancia de actualizar contraseñas en dispositivos móviles inmediatamente después de un cambio en el sistema para evitar bloqueos de cuenta automáticos.

---

## 📊 Herramientas Utilizadas

* **Microsoft Entra ID (Azure AD)** — Análisis de Sign-in logs y estados de dispositivo.
* **Microsoft 365 Defender** — Gestión de alertas de identidad.
