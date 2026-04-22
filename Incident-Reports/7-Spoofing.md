# INC-20260421-007 — phishing con código de dispositivo y spoofing — remediado

# 📝 Resumen
Se investigó un incidente originado por un clic en un enlace sospechoso (`http://laptopfixrun.com/yy`). el análisis en sandbox reveló un intento de phishing avanzado utilizando la técnica de autorización por código de dispositivo (device code authorization) orientada a la toma de cuentas (account takeover). Mediante consultas en kql y revisión de cabeceras, se confirmó que el atacante no logró el acceso, pero se aplicaron medidas de contención sobre el usuario y la infraestructura de correo debido a un esquema de spoofing.

---

## 🔍 Detalles del incidente
| Campo | Valor |
|---|---|
| **ID del incidente** | 20260421-007 |
| **Severidad** | **alta** |
| **Categoría** | phishing avanzado / spoofing / intento de ATO |
| **Detección** | alerta por clic en url maliciosa y auto-reenvío de correo |

---

## 🛠 proceso de investigación
1. **Análisis dinámico (sandbox):** al detonar la url en un entorno seguro, se observó que la página generaba un código de inicio de sesión que se copiaba al portapapeles y luego redirigía al portal legítimo de microsoft (`https://login.microsoftonline.com/common/oauth2/deviceauth`). consultas en virustotal e inteligencia de amenazas confirmaron que es una táctica documentada para evadir mfa.
2. **Telemetría de identidad (kql):** se utilizó kql sobre la tabla `SigninLogs` para buscar el `UserPrincipalName` afectado, filtrando por `ResultType` de errores comunes (50126, 50053, etc.). la consulta extendió detalles del dispositivo y geolocalización, confirmando que la actividad del usuario mantenía su patrón habitual (misma ip, misma ubicación y mismo equipo corporativo). No hubo inicios de sesión anómalos posteriores al clic.
3. **Análisis de cabeceras y spoofing:** tras obtener el network message id, se descargó el correo para una inspección profunda. Los filtros de autenticación fallaron: DMARC (fail), SPF (soft fail) y DKIM (none). Se verificó que la ip remitente no coincidía con el dominio real de la empresa y que el dominio atacante fue registrado recientemente en 2026, confirmando la hipótesis de spoofing.

---

## 🛡 Respuesta y remediación
* **Protección de cuenta:** dado que el usuario interactuó con el enlace, se actuó bajo un enfoque de Zero Trust. Se forzó el cambio de contraseña y se exigió un nuevo registro de mfa ante una posible filtración.
* **Bloqueo de infraestructura atacante:** se procedió a crear una regla para enviar a cuarentena (junk) todos los correos futuros provenientes de la ip del remitente identificada en el análisis.

---

## 💡 Lecciones aprendidas
* **Nuevas técnicas de evasión:** los atacantes utilizan cada vez más el flujo de "device code" porque, al redirigir finalmente a una página real de microsoft, eluden la detección de muchos filtros tradicionales y engañan visualmente al usuario.
* **Importancia de dmarc:** este incidente subraya por qué una política estricta de dmarc (reject) es vital, ya que hubiera bloqueado el correo en la puerta de enlace (gateway) antes de llegar a la bandeja del usuario.
