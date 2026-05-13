# INC-20260421-006 — Instalación de software no sancionado (shadow IT) — Escalado

# 📝 Resumen
Se detectó la instalación de una aplicación no aprobada en el equipo de un usuario final. La investigación determinó que el software no es malicioso, pero infringe las políticas corporativas. Debido a restricciones de red que impiden el acceso remoto directo, se escaló el caso para su eliminación manual o justificación formal.

---

## 🔍 Detalles del incidente
| Campo | Valor |
|---|---|
| **ID del incidente** | 20260421-006 |
| **Severidad** | **media** |
| **Categoría** | Shadow IT / Violación de políticas |
| **Detección** | Alerta de aplicación no aprobada descubierta en el endpoint |

---

## 🛠 Proceso de investigación
1. **Análisis de reputación:** Se analizó el software en bases de datos de inteligencia de amenazas. Se comprobó que es un programa benigno y no presenta comportamiento de malware.
2. **Evaluación de cumplimiento:** Se verificó el catálogo de software permitido por la empresa. La aplicación detectada no figura en la lista de herramientas autorizadas para los empleados.

---

## 🛡 Respuesta y remediación
* **Intento de mitigación remota:** Se intentó ingresar al equipo del usuario mediante RDP para desinstalar el software directamente.
* **Bloqueo de infraestructura:** Se comprobó que las políticas del firewall bloquean las conexiones RDP entrantes, imposibilitando la acción remota del analista.
* **Escalamiento:** Se generó un reporte en el sistema de tickets dirigido al área de soporte local. Se solicitó contactar al usuario para eliminar el archivo o, en su defecto, que un supervisor confirme formalmente la necesidad operativa del software para añadirlo a las excepciones.
