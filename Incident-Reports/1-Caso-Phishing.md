# INC-20260418-001 — Intento de phishing & Movimiento lateral — **Falso Positivo**


# 📝 Resumen

El 18 de abril de 2026, se detectó una alerta de seguridad categorizada inicialmente como inicio de sesión sospechoso y posible movimiento lateral. Tras una investigación técnica detallada utilizando Microsoft Defender para Office 365 y Exchange Online, el incidente fue reclasificado como un intento de Phishing con un Falso Positivo de movimiento lateral, causado por una regla de reenvío automático preexistente.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | 20260418-001 |
| **Severidad** | **CRITICO** |
| **Categoría** | Phishing / Intento de acceso no autorizado |
| **Detección Inicial** | Click en URL de phishing y anomalía en envío de correos internos |

---

## 🛠 Proceso de investigación

### 1. Análisis de phishing

La investigación comenzó cuando Microsoft Defender alertó que un usuario hizo clic en la URL: `zedsms.com/`.

- **Análisis:** Se identificó la URL como un sitio de Credential Harvesting (recolección de credenciales) que simula la página de inicio de sesión de Microsoft.
- **Hallazgos:** Defender confirmó el clic. Sin embargo, los logs de inicio de sesión (Sign-in logs) no mostraron accesos exitosos desde ubicaciones inusuales o dispositivos no permitidos inmediatamente después del evento.
- **Hipótesis:** El usuario accedió al sitio malicioso pero es probable que no haya ingresado sus credenciales.

### 2. Anomalía en el Flujo de Correo Interno

Se disparó una segunda alerta por "Envío Interno" sospechoso hacia la cuenta de Finanzas.

- **Investigación:** Al revisar el rol del usuario en el Tenant, se confirmó que pertenece al área de Finanzas. Esto justifica la comunicación con dicho buzón, pero no el volumen ni la frecuencia de los envíos detectados.

### 3. Análisis de causa raíz (RCA)

Para determinar si se trataba de un Movimiento Lateral (un atacante saltando de una cuenta a otra), se inspeccionó la configuración del buzón en Exchange Online (EXO).

- **Acción:** Revisión de reglas de flujo de correo y forwarding mediante el Centro de Administración de Exchange.
- **Resultado:** Se descubrió una Regla de Reenvío (Forwarding Rule) activa. Todo correo recibido en la cuenta personal del usuario se redirigía automáticamente a la cuenta compartida de Finanzas.
- **Conclusión:** La alerta de movimiento lateral fue un falso positivo provocado por esta redirección automática. No hubo actividad manual de un atacante.

---

## 🛡 Respuesta y remediación

| Acción | Estado | Descripción |
|---|---|---|
| Reseteo de contraseña | ✅ Completado | Cambio de contraseña forzado para invalidar cualquier credencial posiblemente filtrada. |
| Revocación de sesiones y MFA | ✅ Completado | Se cerraron todas las sesiones activas y se solicitó un nuevo desafío de MFA. |
| Bloqueo de URL | ✅ Completado | Se añadió `zedsms.com` a la lista de bloqueos del Tenant. |
| Comunicación con el usuario | ⏳ En curso | Entrevista con el usuario para confirmar si llegó a introducir datos en el sitio falso. |
| Revisión de reglas | ✅ Completado | Se validó que la regla de reenvío es parte del proceso de negocio del área. |

---

## 💡 Lecciones aprendidas

1. **El Contexto es Clave:** Conocer el departamento del usuario permitió descartar rápidamente una intrusión real y entender que el tráfico era operacional.
2. **Correlación de Eventos:** Las reglas de reenvío automático suelen imitar el comportamiento de un BEC (Business Email Compromise). Es vital verificar las reglas de Exchange antes de confirmar un movimiento lateral.
3. **Defensa Proactiva:** Aunque no se confirmó un inicio de sesión exitoso, el simple clic en un sitio de phishing exige aplicar Zero Trust: resetear credenciales y sesiones.

---

## 📊 Herramientas Utilizadas

- **Microsoft 365 Defender** — Línea de tiempo del incidente y rastreo de clics.
- **Entra ID (Azure AD)** — Análisis de logs de inicio de sesión.
- **Exchange Online Admin Center** — Reglas de flujo de correo.

## Solución estructural 

- El tenant debería tener documentadas todas las reglas de forwarding que existen por razones operativas. Cada una con su justificación, la fecha en que fue creada y el responsable. Cuando llega una alerta de este tipo, el analista tiene un punto de referencia inmediato para determinar si la regla es conocida o nueva. Si es conocida y está documentada, es falso positivo. Si no está en el inventario, es una señal de alerta real.
