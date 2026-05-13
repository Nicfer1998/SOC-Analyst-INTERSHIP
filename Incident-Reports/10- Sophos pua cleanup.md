# INC-20260513-010 — Sophos PUA Cleanup Failed (Generic ML PUA) — Falso Positivo

# 📝 Resumen

El 13 de mayo de 2026, Sophos Central generó una alerta de severidad media indicando que la limpieza automática de una aplicación potencialmente no deseada (PUA) había fallado sobre un archivo ZIP ubicado en una ruta de proyectos de ingeniería. Tras investigar el historial de incidentes similares y verificar el contexto del archivo, se determinó que se trata de un falso positivo generado por la heurística de machine learning del motor de Sophos.

---

## 🔍 Detalles del incidente

| Campo | Valor |
| --- | --- |
| **ID del incidente** | INC-20260513-010 |
| **Severidad** | Media |
| **Categoría** | Falso positivo / PUA heurística ML |
| **Alert Product** | Sophos Central |
| **Detección** | Generic ML PUA en archivo ZIP de proyecto legacy |
| **Táctica MITRE** | N/A |
| **Clasificación final** | BenignPositive |

---

## 🛠 Proceso de investigación

1. **Revisión de la descripción:** La alerta indicaba que Sophos detectó una aplicación clasificada como `Generic ML PUA` en un archivo comprimido `.zip` perteneciente a un proyecto de ingeniería antiguo. El cleanup automático no pudo completarse, requiriendo intervención manual.
2. **Análisis del tipo de detección:** La clasificación `Generic ML PUA` indica que no existe una firma específica para el archivo; la detección fue generada por el motor de inteligencia artificial de Sophos al identificar patrones comportamentales similares a software no deseado.
3. **Revisión de historial de incidentes:** Se consultaron incidentes previos similares dentro del mismo workspace. Se encontraron múltiples casos anteriores con el mismo archivo, misma ruta y misma clasificación, todos cerrados como `BenignPositive` con la nota "legit and known file".
4. **Conclusión:** El archivo es un artefacto legítimo de un proyecto de ingeniería, conocido y documentado en el entorno. La heurística ML de Sophos lo marcó por sus características estructurales (nombre con códigos alfanuméricos, ruta profunda) sin que represente una amenaza real.

---

## 🛡 Respuesta y remediación

- **Clasificación del evento:** BenignPositive — archivo legítimo conocido.
- **Acción tomada:** Cierre del incidente con referencia a casos previos como evidencia de legitimidad.
- **Recomendación:** Considerar la creación de una exclusión en Sophos Central para este archivo específico y evitar alertas recurrentes.

---

## 💡 Análisis técnico: ¿Por qué Sophos lo detectó?

El motor `Generic ML PUA` de Sophos no utiliza firmas fijas sino que analiza patrones mediante inteligencia artificial. Durante este incidente, el archivo ZIP fue marcado porque:

- Su nombre contiene códigos alfanuméricos que imitan convenciones de empaquetado de malware.
- La ruta de almacenamiento es profunda y con caracteres especiales, patrón habitual en amenazas reales.
- Al tratarse de un archivo legacy de 2017, no formaba parte de la lista de confianza actualizada del motor.

Sin embargo, el análisis de contexto y el historial de incidentes previos confirmaron que el archivo es inofensivo y pertenece a la operación legítima del negocio.
