# INC-20260513-011 — Multistage Attack: Privileged Role Assignment Outside PIM (Fusion) — Falso Positivo

# 📝 Resumen

El 13 de mayo de 2026, el motor Fusion de Microsoft Sentinel generó una alerta de severidad alta correlacionando dos eventos de asignación de roles privilegiados fuera del flujo estándar de Privileged Identity Management (PIM). La investigación de los logs de auditoría reveló que ambas asignaciones fueron realizadas por una aplicación automática de Microsoft como parte de una migración planificada de roles en Microsoft Purview, descartando cualquier actividad maliciosa.

---

## 🔍 Detalles del incidente

| Campo | Valor |
| --- | --- |
| **ID del incidente** | INC-20260513-011 |
| **Severidad** | Alta |
| **Categoría** | Falso positivo / Migración automática de roles |
| **Alert Product** | Microsoft Sentinel (Fusion) |
| **Tácticas MITRE** | Persistence, Privilege Escalation |
| **Clasificación final** | BenignPositive |

---

## 🛠 Proceso de investigación

1. **Análisis del timeline:** Se registraron dos eventos en minutos consecutivos: asignación del rol `Purview Workload Content Administrator` a dos cuentas del directorio corporativo. Ambas asignaciones ocurrieron de madrugada, fuera del horario laboral habitual.
2. **Revisión del campo Initiator en los logs de auditoría:** Al consultar los AuditLogs mediante KQL, se identificó que el campo `Initiator` en ambos registros correspondía a `PurviewRoleAssignmentMigrator`, una aplicación de servicio de Microsoft con su `ServicePrincipalId` asociado.
3. **Validación de cuentas afectadas:** Las cuentas que recibieron el rol fueron verificadas contra la documentación interna de la organización, confirmándose como usuarios legítimos con roles coherentes (IT Manager y cuenta de servicio MDR).
4. **Consulta de incidentes similares:** Se revisaron incidentes previos con entidades similares, encontrando dos casos anteriores (`#15709` Low y `#15633` High) cerrados como `Closed - Benign`.
5. **Conclusión:** La actividad fue generada por una migración automática de permisos ejecutada por Microsoft al actualizar el esquema de roles de Purview. Las aplicaciones de sistema de Microsoft no pasan por PIM al realizar estas operaciones, lo cual es un comportamiento esperado y documentado.

---

## 🛡 Respuesta y remediación

- **Clasificación del evento:** BenignPositive — migración automática de Microsoft Purview.
- **Acción tomada:** Cierre del incidente con referencia a los logs de auditoría y casos previos como evidencia.
- **Recomendación:** Considerar la creación de una regla de supresión en Sentinel para excluir el `PurviewRoleAssignmentMigrator` como initiator en alertas de asignación de roles.

---

## 💡 Análisis técnico: ¿Qué es PIM y por qué se saltó?

**Privileged Identity Management (PIM)** es el sistema de control de acceso Just-In-Time de Azure AD. Obliga a que la asignación de roles privilegiados pase por un flujo de aprobación, justificación y límite temporal.

Sin embargo, las aplicaciones de sistema de Microsoft como `PurviewRoleAssignmentMigrator` operan como Service Principals y ejecutan cambios de directorio de forma programática, sin posibilidad técnica de pasar por el flujo de aprobación humana de PIM. Esto es una limitación conocida del diseño de la plataforma, no una evasión de controles de seguridad.

El motor Fusion de Sentinel correlacionó los dos eventos como un patrón de ataque multietapa (reconocimiento + escalada de privilegios) por la proximidad temporal y la naturaleza de las operaciones, sin disponer del contexto de que el actor era una aplicación interna de Microsoft.
