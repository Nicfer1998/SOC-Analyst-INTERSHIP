# INC-20260515-017 — User Assigned Privileged Role Involving Multiple Users — Falso Positivo

# 📝 Resumen

El 15 de mayo de 2026, Microsoft Sentinel generó una alerta indicando la asignación de un rol privilegiado a múltiples usuarios en el directorio corporativo. La investigación de los logs de auditoría identificó que la asignación fue realizada manualmente por un Global Administrator sobre una cuenta que ya contaba con múltiples roles administrativos activos, descartando cualquier actividad maliciosa o escalada de privilegios no autorizada.

---

## 🔍 Detalles del incidente

| Campo | Valor |
| --- | --- |
| **ID del incidente** | INC-20260515-017 |
| **Severidad** | Media |
| **Categoría** | Falso positivo / Gestión administrativa legítima |
| **Alert Product** | Microsoft Sentinel |
| **Sentinel Incident** | #4577 |
| **Operación detectada** | Add member to role — Authentication Administrator |
| **Clasificación final** | False Positive |

---

## 🛠 Proceso de investigación

1. **Revisión del incidente:** La alerta indicaba que un rol privilegiado fue asignado a múltiples usuarios en el directorio. La descripción de la regla analítica especificaba: "Identifica cuando un rol privilegiado es asignado a un nuevo usuario. Si la asignación es inesperada, investigar."

2. **Análisis del log de auditoría:** Se consultó el log del evento en el SIEM. Los datos del registro mostraron:
   - **Operación:** `Add member to role`
   - **Rol asignado:** `Authentication Administrator`
   - **Initiator:** Usuario del directorio corporativo con cuenta activa
   - **Target:** Segundo usuario del directorio corporativo
   - El Initiator era una **persona real**, no una aplicación automática como en casos previos de Purview.

3. **Verificación de permisos del Initiator:** Se consultaron los roles activos del usuario que realizó la asignación en el portal de Azure AD (Microsoft Entra ID). El usuario contaba con los siguientes roles activos entre otros: `Global Administrator`, `User Administrator`, `Teams Administrator`, `Exchange Administrator`, `Billing Administrator`, `SharePoint Administrator`, `Helpdesk Administrator`. Al ser **Global Administrator**, tiene plena autorización para asignar cualquier rol del directorio.

4. **Verificación de roles del usuario que recibió el rol (Target):** Se consultaron los roles activos del usuario destino. El usuario ya contaba previamente con roles como: `Privileged Authentication Administrator`, `User Administrator`, `Teams Administrator`, `Intune Administrator`, `Exchange Administrator`, `Domain Name Administrator`, `Hybrid Identity Administrator`. El rol recibido (`Authentication Administrator`) es de menor jerarquía que el `Privileged Authentication Administrator` que el usuario ya poseía.

5. **Conclusión:** La asignación fue realizada por un administrador con autorización plena sobre un usuario que ya era administrador conocido con múltiples roles privilegiados. No existe escalada de privilegios inesperada ni actividad maliciosa.

---

## 🛡 Respuesta y remediación

- **Clasificación del evento:** False Positive — gestión administrativa legítima por Global Administrator.
- **Acción tomada:** Cierre del incidente con evidencia de los roles del Initiator y del Target en Azure AD.
- **Recomendación:** Si este tipo de asignaciones entre administradores conocidos es frecuente, considerar ajustar la regla analítica para excluir asignaciones realizadas por Global Administrators sobre usuarios que ya poseen roles privilegiados, reduciendo el ruido operativo.

---

## 💡 Análisis técnico: ¿Por qué saltó la alerta si fue legítimo?

La regla analítica que generó esta alerta monitorea **toda** asignación de roles privilegiados en el directorio, independientemente de quién la realice o sobre quién recaiga. Es una regla diseñada con intención conservadora — prefiere generar falsos positivos que dejar pasar una escalada de privilegios real sin detectar.

Esta filosofía es correcta en seguridad: es mejor investigar 10 asignaciones legítimas que dejar pasar 1 maliciosa. Sin embargo, en entornos donde los administradores realizan gestiones de roles frecuentemente, genera ruido operativo que debe gestionarse mediante excepciones o ajustes de la regla.

**Diferencia clave con el caso #11 (Purview Fusion):**
En el incidente #11, el Initiator era una **aplicación automática de Microsoft** (`PurviewRoleAssignmentMigrator`), lo que explicaba la ausencia de aprobación PIM. En este caso, el Initiator es un **usuario humano con rol de Global Administrator**, que tiene autorización directa para realizar asignaciones de roles sin requerir flujo PIM adicional, ya que PIM es opcional para Global Admins en muchas configuraciones de tenant.
