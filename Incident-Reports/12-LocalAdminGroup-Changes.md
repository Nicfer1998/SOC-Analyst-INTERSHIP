# INC-20260513-012 — Local Admin Group Changes — Falso Positivo

# 📝 Resumen

El 13 de mayo de 2026, Microsoft Sentinel generó una alerta de severidad alta indicando modificaciones en el grupo de administradores locales de un endpoint corporativo. La investigación de los logs reveló que la acción fue realizada por la cuenta de administrador del dominio, afectando a dos usuarios que fueron verificados como miembros legítimos de la organización, siendo la asignación parte de una gestión administrativa planificada.

---

## 🔍 Detalles del incidente

| Campo | Valor |
| --- | --- |
| **ID del incidente** | INC-20260513-012 |
| **Severidad** | Alta |
| **Categoría** | Falso positivo / Cambio administrativo legítimo |
| **Alert Product** | Microsoft Sentinel |
| **Táctica MITRE** | Persistence |
| **Regla analítica** | cloud-directory-018x |
| **Clasificación final** | False Positive |

---

## 🛠 Proceso de investigación

1. **Revisión del incidente:** La alerta indicaba que se habían realizado cambios en grupos locales de un endpoint del dominio. Se identificaron dos entidades de tipo usuario y el equipo afectado como entidades del incidente.
2. **Consulta de logs mediante KQL:** Se ejecutó la query asociada a la regla analítica sobre los logs del workspace. Los resultados mostraron dos registros de tipo `UserAccountAddedToLocalGroup` generados en el mismo segundo exacto, con el mismo actor (`administrador`) y el mismo grupo destino.
3. **Análisis del grupo destino:** El grupo al que fueron agregados los usuarios no corresponde al grupo de administradores locales del sistema, sino a un grupo de distribución del área operativa de logística (`DL_RX_[REDACTED]_Logistica_REF_[REDACTED]`).
4. **Verificación de identidades:** Ambos usuarios fueron buscados en el portal de Azure AD (Microsoft Entra ID). Ambos aparecen como miembros activos del directorio corporativo con cuentas válidas.
5. **Revisión de incidentes similares:** Casos previos con la misma regla analítica (#30718 High y #30563 Medium) fueron cerrados como `Closed - Benign` y `Closed - False Positive` respectivamente.
6. **Conclusión:** La simultaneidad de ambas asignaciones, el actor legítimo y el grupo operativo de destino indican una gestión administrativa planificada, no una actividad maliciosa.

---

## 🛡 Respuesta y remediación

- **Clasificación del evento:** False Positive — asignación administrativa legítima.
- **Acción tomada:** Cierre del incidente con evidencia de los logs y verificación de identidades en Azure AD.
- **Recomendación:** Si estas asignaciones son recurrentes en el área de logística, considerar ajustar el umbral de la regla analítica o crear una exclusión para el grupo de distribución afectado.

---

## 💡 Análisis técnico: ¿Por qué saltó la alerta?

La regla analítica `cloud-directory-018x` monitorea cualquier modificación en grupos locales de equipos del dominio, independientemente del tipo de grupo. Al detectar dos operaciones `UserAccountAddedToLocalGroup` en tiempo real, Sentinel las categorizó bajo la táctica de **Persistence**, ya que agregar usuarios a grupos locales es una técnica utilizada por atacantes para mantener acceso en un equipo comprometido.

Sin embargo, en este caso el contexto operativo demostró que la actividad era completamente legítima: el actor era la cuenta de administrador del dominio, los usuarios afectados eran empleados válidos, y el grupo de destino era un grupo funcional del negocio, no un grupo de privilegios del sistema.
