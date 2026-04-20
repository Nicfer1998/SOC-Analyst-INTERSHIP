# INC-20260421-004 — Ubicación de Inicio de Sesión Atípica — **Falso Positivo (VPN)**

# 📝 Resumen
El 21 de abril de 2026, se generó una alerta por un inicio de sesión desde una ubicación geográfica inusual (EE. UU.). Tras el análisis de los logs de **Microsoft Entra ID** y la infraestructura de red, se confirmó que el tráfico provenía de la **VPN Corporativa** de la empresa. El incidente se cerró como un Falso Positivo.

---

## 🔍 Detalles del incidente
| Campo | Valor |
|---|---|
| **ID del incidente** | 20260421-004 |
| **Severidad** | **BAJA** |
| **Categoría** | Acceso inusual / Anomalía geográfica |
| **Detección** | Login detectado desde IP de centro de datos externa |

---

## 🛠 Proceso de investigación
1. **Análisis de IP:** Se realizó un *IP Lookup* de la dirección sospechosa. Se identificó que el proveedor (ASN) pertenece a un servicio de infraestructura en la nube, típico de nodos de salida de VPN.
2. **Correlación de usuarios:** Al filtrar por la IP sospechosa, se observó que múltiples colaboradores de la organización presentaban inicios de sesión exitosos desde la misma dirección en el mismo lapso de tiempo.
3. **Consistencia de dispositivo:** El *User Agent* y el *Device ID* coincidían con el equipo corporativo registrado del usuario, manteniendo la coherencia con su actividad habitual.

---

## 🛡 Respuesta y remediación
* **Validación de red:** Se confirmó con el equipo de IT/Infraestructura que la IP pertenece a un segmento autorizado para el trabajo remoto.
* **Ajuste de política:** Se recomendó marcar este rango de IPs como "Ubicaciones de Confianza" (Named Locations) en el acceso condicional para reducir el ruido en el monitoreo.

---

## 💡 Lecciones Aprendidas
* **Identificación de infraestructura:** Diferenciar entre una IP residencial y una de Data Center es el primer paso para descartar falsos positivos por VPN.
* **Contexto organizacional:** Como analista, conocer las herramientas de conectividad que usa la empresa (VPN, Proxies, Zscaler, etc.) es vital para evitar escalaciones innecesarias.
