# INC-20260526-031 — New Access Credential Added to Application or Service Principal — **Verdadero Positivo Benigno**

# 📝 Resumen

El 26 de mayo de 2026, Microsoft Sentinel generó un incidente de severidad media al detectar que se agregó una nueva credencial a una aplicación registrada en Azure AD. La táctica identificada fue **Defense Evasion**, dado que agregar credenciales a aplicaciones es una técnica usada por atacantes para obtener persistencia silenciosa. Tras investigar los AuditLogs y verificar el perfil del usuario iniciador, se determinó que el Global Administrator de la organización agregó intencionalmente una credencial de **Keycloak** a la aplicación de IAM corporativa para configurar una integración SSO. El incidente fue cerrado como Verdadero Positivo Benigno.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260526-031 |
| **Incidente Sentinel / ITSM** | #41622 / Defence Center #122694 |
| **Severidad** | **MEDIA** |
| **Categoría** | Defense Evasion / Persistence |
| **Detección Inicial** | Microsoft Sentinel — New access credential added to Application or Service Principal |
| **Usuario iniciador** | adm.[username]@[org-domain] |
| **IP** | [known-admin-ip] |
| **Aplicación afectada** | [iam-app-name] |
| **Credencial agregada** | Keycloak (Password) |

---

## 🛠 Proceso de Investigación

### ¿Por qué es sensible agregar credenciales a una aplicación?

Una aplicación registrada en Azure AD puede autenticarse de forma autónoma usando **secrets o certificados** — sin necesitar las credenciales de ningún usuario. Si un atacante agrega una nueva credencial a una aplicación con permisos elevados, puede autenticarse como esa aplicación silenciosamente, sin que aparezca en los logs de sign-in de usuarios. Por eso la táctica es **Defense Evasion**.

### 1. Análisis de las Entidades

| Kind | Valor |
|---|---|
| **Account** | adm.[username]@[org-domain] |
| **Account** | [object-id] |
| **IP** | [known-admin-ip] |

- `adm.[username]` → convención de cuenta administrativa dedicada. Buena práctica de separación de roles.
- [object-id] → Object ID de la aplicación afectada en Azure AD.

### 2. Investigación con KQL sobre AuditLogs

Se ejecutó una consulta sobre `AuditLogs` filtrando por operaciones de gestión de credenciales:

**Hallazgos:**

| Campo | Valor |
|---|---|
| **OperationName** | Update application — Certificates and secrets management |
| **InitiatingUserPrincipalName** | adm.[username]@[org-domain] |
| **InitiatingIpAddress** | [known-admin-ip] |
| **UserAgent** | Mozilla/5.0 Windows NT 10.0 Win64 — Chrome |
| **targetDisplayName** | [iam-app-name] |
| **keyDisplayName** | Keycloak |
| **keyType** | Password |
| **keyUsage** | Verify |

### 3. Verificación del usuario y la IP

- **IP [known-admin-ip]** → IP conocida y habitual del administrador. Verificada en historial de sign-ins.
- **adm.[username]** → Global Administrator de la organización. Cuenta administrativa dedicada con convención `adm.` — indica separación correcta de roles.
- **UserAgent real** → Chrome en Windows 10 → sesión interactiva de un humano, no proceso automatizado.

### 4. Contexto de la aplicación y la credencial

**[iam-app-name]** → aplicación de Identity and Access Management corporativa.

**Keycloak** → herramienta open-source de gestión de identidades (SSO). El administrador agregó una credencial de Keycloak a la aplicación IAM para configurar una integración de Single Sign-On entre Keycloak y Azure AD — permitiendo que los empleados accedan a múltiples sistemas con una sola autenticación.

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Análisis de entidades | ✅ Completado | IP conocida del administrador confirmada. |
| Consulta KQL sobre AuditLogs | ✅ Completado | Identificación de aplicación, credencial y actor. |
| Verificación del rol del usuario | ✅ Completado | Global Administrator con IP habitual y UserAgent real. |
| Cierre del incidente | ✅ Completado | True Positive — Benign. Integración SSO legítima. |

---

## 💡 Lecciones Aprendidas

1. **Agregar credenciales a apps = siempre investigar:** Es una técnica legítima para integraciones SSO pero idéntica a la usada en ataques de App Consent Abuse. El contexto — quién lo hizo, desde dónde y para qué aplicación — determina si es benigno o crítico.
2. **UserAgent real vs vacío:** En este caso el UserAgent era de Chrome en Windows → sesión humana real. En un ataque automatizado el UserAgent suele estar vacío o ser genérico. Esa diferencia es un indicador rápido de legitimidad.
3. **Convención `adm.` como buena práctica:** El uso de cuentas administrativas dedicadas con prefijos como `adm.` indica que la organización sigue el principio de separación de roles — reduce el riesgo de que una cuenta comprometida en el trabajo diario tenga privilegios de administrador.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Detección del incidente.
- **Log Analytics (KQL)** — Consulta sobre `AuditLogs`.
- **Azure Active Directory (Entra ID)** — Verificación del perfil del usuario y la aplicación.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Defense Evasion | Use Alternate Authentication Material | T1550 |
| Persistence | Account Manipulation — Additional Cloud Credentials | T1098.001 |
