# INC-20260513-013 — User and Group Membership Reconnaissance (SAMR) desde ADCS — Falso Positivo

# 📝 Resumen

El 13 de mayo de 2026, Microsoft Defender for Identity generó una alerta de severidad media indicando actividad de reconocimiento de usuarios y grupos mediante el protocolo SAMR (Security Account Manager Remote) con destino al Domain Controller del entorno. La investigación confirmó que el origen del tráfico es el servidor de Active Directory Certificate Services (ADCS) de la organización, cuya comunicación con el DC es un comportamiento técnico esperado y necesario para su operación normal.

---

## 🔍 Detalles del incidente

| Campo | Valor |
| --- | --- |
| **ID del incidente** | INC-20260513-013 |
| **Severidad** | Media |
| **Categoría** | Falso positivo / Tráfico legítimo de infraestructura |
| **Alert Product** | Microsoft Defender for Identity |
| **Táctica MITRE** | Discovery |
| **Protocolo** | SAMR sobre SMB (puerto 445) |
| **Clasificación final** | False Positive |

---

## 🛠 Proceso de investigación

1. **Revisión del incidente:** La alerta identificó a una cuenta de servicio como origen de las consultas SAMR dirigidas al Domain Controller del dominio. Se registraron 11 entidades involucradas, incluyendo múltiples IPs internas.
2. **Análisis del tráfico de red:** Se revisaron los detalles del evento en Defender for Identity. El tráfico correspondía a conexiones TCP sobre el puerto 445 (SMB/CIFS) desde un equipo origen hacia el Domain Controller. La autenticación utilizada fue Kerberos con la opción `MutualRequired`, indicando autenticación mutua entre ambos extremos.
3. **Identificación del equipo origen:** El nombre del equipo origen fue identificado en la documentación de infraestructura interna de la organización como el servidor de **Active Directory Certificate Services (ADCS)**. Esto fue confirmado cruzando el hostname con el inventario de activos documentado.
4. **Análisis de legitimidad:** ADCS requiere consultar el Domain Controller de manera regular para verificar identidades antes de emitir certificados digitales, validar membresías de grupos y consultar atributos de directorio. Este comportamiento es técnicamente obligatorio para el funcionamiento del servicio de certificados.
5. **Revisión de incidentes similares:** El único caso previo relacionado (#30632 Medium) fue cerrado como `Closed - Undetermined`, sin evidencia de compromiso.
6. **Conclusión:** El tráfico SAMR corresponde a la operación normal del servidor ADCS consultando el DC. No existe actividad maliciosa.

---

## 🛡 Respuesta y remediación

- **Clasificación del evento:** False Positive — comunicación legítima entre ADCS y Domain Controller.
- **Acción tomada:** Cierre del incidente con evidencia de la documentación de infraestructura y análisis técnico del tráfico.
- **Recomendación:** Crear una exclusión en Microsoft Defender for Identity para suprimir alertas SAMR entre el servidor ADCS y el Domain Controller, evitando ruido recurrente en el SIEM.

---

## 💡 Análisis técnico: ¿Qué es SAMR y por qué es sensible?

**SAMR (Security Account Manager Remote Protocol)** es un protocolo de Windows que permite consultar remotamente información sobre usuarios, grupos y políticas de seguridad del dominio. Es utilizado legítimamente por múltiples componentes del sistema, pero también es una herramienta habitual en la fase de reconocimiento de un ataque, donde el atacante busca mapear usuarios privilegiados antes de intentar una escalada.

En este caso, la combinación de SAMR + puerto 445 + consultas al Domain Controller activó la detección de Defender for Identity. Sin embargo, el origen del tráfico era el servidor ADCS, cuya función principal requiere exactamente este tipo de consultas al directorio para validar la identidad de los solicitantes de certificados y verificar la estructura de grupos del dominio.

La autenticación Kerberos con `MutualRequired` refuerza la legitimidad, ya que implica que ambos servidores se autenticaron mutuamente antes de intercambiar información, lo cual es inconsistente con el comportamiento de un atacante que típicamente usa credenciales robadas sin control sobre ambos extremos de la comunicación.
