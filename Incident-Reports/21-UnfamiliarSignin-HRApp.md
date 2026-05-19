# INC-20260519-004 — Unfamiliar Sign-in Properties — **Verdadero Positivo Benigno**

# 📝 Resumen

El 19 de mayo de 2026, Microsoft Sentinel generó una alerta de severidad media al detectar que el inicio de sesión de un usuario presentaba propiedades completamente desconocidas: ASN, navegador, dispositivo, IP, ubicación, EASId y TenantIPsubnet eran todos inusuales simultáneamente. Tras una investigación con **SigninLogs en KQL** y el análisis del dispositivo, se identificó que el login desde Países Bajos correspondía a una **aplicación de Recursos Humanos** hosteada en infraestructura de Microsoft Azure, y que el patrón se repite en múltiples usuarios de la organización.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260519-004 |
| **Incidente Sentinel / ITSM** | #8728 / Defence Center #116470 |
| **Severidad** | **MEDIA** |
| **Categoría** | Initial Access / Unfamiliar Sign-in |
| **Detección Inicial** | Microsoft Sentinel — Unfamiliar sign-in properties |
| **Usuario afectado** | [username]@[org-domain] |
| **IPs detectadas** | 20.4.234.160 (Azure NL) / 5.91.112.213 (Italia) |

---

## 🛠 Proceso de Investigación

### 1. Análisis de la Alerta

La alerta indicó que **todas** las propiedades del sign-in eran simultáneamente desconocidas para el usuario: ASN, Browser, Device, IP, Location, EASId y TenantIPsubnet.

- **Evaluación inicial:** Cuando la totalidad de las propiedades son inusuales al mismo tiempo, puede indicar acceso desde un dispositivo/ubicación completamente nuevo o el uso de una cuenta comprometida.
- **IP 20.4.234.160:** Pertenece a rangos de Microsoft Azure (Países Bajos), lo que sugería un servicio cloud.

### 2. Análisis de SigninLogs con KQL

Se ejecutó una consulta sobre la tabla `SigninLogs` filtrando por el `UserPrincipalName` del usuario afectado para obtener el historial completo de inicios de sesión.

**Resultados obtenidos:**

| Timestamp | IP | Ubicación |
|---|---|---|
| 09:58 AM | 5.91.112.213 | 🇮🇹 IT, Cosenza, Tivolille |
| 09:59 AM | 5.91.112.213 | 🇮🇹 IT, Cosenza, Tivolille |
| 11:51 AM | 20.4.234.160 | 🇳🇱 NL, Noord-Holland, Amsterdam |
| 11:58 AM | 20.4.234.160 | 🇳🇱 NL, Noord-Holland, Amsterdam |

- **Cada IP tiene ubicación consistente.** No hay inconsistencia geográfica por IP individual.

### 3. Análisis del Dispositivo

Se expandió el registro de sign-in para inspeccionar los detalles del dispositivo desde el que se originó el acceso desde Azure NL.

**Hallazgos críticos del dispositivo:**

| Campo | Valor |
|---|---|
| **deviceId** | `""` (vacío) |
| **isCompliant** | `false` |
| **isManaged** | `false` |
| **OS** | Windows 10 |
| **Browser** | Chrome 140.0.0 |
| **ThroughGlobalSecureAccess** | `false` |
| **TokenType** | saml11 (legacy) |

- **Evaluación:** Dispositivo sin ID registrado, sin gestión corporativa y sin pasar por VPN corporativa desde Países Bajos. En ausencia de contexto, esto apunta a un dispositivo no corporativo realizando un acceso.

### 4. Confirmación con el Usuario y Análisis de Causa Raíz

Se contactó al usuario para confirmar si reconocía el acceso.

- **Resultado:** El usuario confirmó que estaba utilizando una **aplicación de Recursos Humanos**.
- **Causa raíz identificada:** La aplicación de RRHH está hosteada en infraestructura de **Microsoft Azure en Amsterdam**, por lo que genera sign-ins con `deviceId` vacío, `isManaged: false` y ubicación en Países Bajos — propiedades que Sentinel identifica como "unfamiliar".
- **Alcance:** Se verificó que el mismo patrón se reproduce en **múltiples usuarios** de la organización que utilizan la misma aplicación.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Consulta KQL sobre SigninLogs | ✅ Completado | Análisis del historial completo de sign-ins del usuario. |
| Inspección del registro del dispositivo | ✅ Completado | Identificación de dispositivo no gestionado sin deviceId. |
| Confirmación con el usuario | ✅ Completado | Usuario confirmó uso de app de RRHH hosteada en Azure. |
| Verificación de alcance | ✅ Completado | Patrón confirmado en múltiples usuarios de la organización. |
| Cierre del incidente | ✅ Completado | True Positive — Benign. App legítima genera propiedades "unfamiliar". |
| Recomendación estructural | ⏳ Pendiente | Crear exclusión en la regla analítica para la IP de la app de RRHH. |

---

## 💡 Lecciones Aprendidas

1. **Las aplicaciones cloud generan sign-ins "unfamiliar":** Servicios de RRHH, ERP u otras aplicaciones hosteadas en Azure pueden generar inicios de sesión con deviceId vacío, isManaged false y ubicaciones de datacenters. Documentar estas apps en un inventario evita falsas alarmas recurrentes.
2. **Alcance múltiple = patrón sistémico:** Cuando el mismo comportamiento se repite en varios usuarios, es señal de una causa raíz técnica, no de una amenaza individual.
3. **Propuesta de mejora a la regla:** Este caso justifica una exclusión en la regla de Sentinel para la IP de la aplicación, reduciendo el volumen de falsos positivos recurrentes y permitiendo que los analistas focalicen en amenazas reales.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección de la alerta y gestión del incidente.
- **Log Analytics (KQL)** — Consulta sobre `SigninLogs`.
- **Azure Active Directory (Entra ID)** — Análisis del perfil del usuario.
- **Portal ITSM (Relatech)** — Gestión y seguimiento del ticket.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Initial Access | Valid Accounts | T1078 |
| Defense Evasion | Use Alternate Authentication Material | T1550 |
