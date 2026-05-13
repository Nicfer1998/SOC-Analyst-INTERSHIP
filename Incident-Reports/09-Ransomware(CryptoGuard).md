# INC-20260422-009 — Detección de ransomware (CryptoGuard) en proceso de actualización — Falso positivo

# 📝 Resumen
El 22 de abril de 2026, la solución EDR (Sophos) generó una alerta crítica de ransomware (CryptoGuard) asociada a un proceso nativo del sistema operativo Windows. La investigación inicial, basada en el análisis de la traza de procesos y la validación del identificador de detección en los portales del proveedor, confirmó que se trata de un falso positivo provocado por el comportamiento legítimo de un paquete de actualización.

---

## 🔍 Detalles del incidente
| Campo | Valor |
|---|---|
| **ID del incidente** | 20260422-009 |
| **Severidad** | **crítica** (mitigada a baja) |
| **Categoría** | Falso positivo / Comportamiento anómalo de EDR |
| **Detección** | Alerta de CryptoGuard sobre el cliente de Windows Update (`wuaucltcore.exe`) |

---

## 🛠 Proceso de investigación
1. **Análisis de la cadena de procesos:** Se revisó el rastreo de procesos en la consola de Sophos. Se validó que la ejecución provenía de un flujo legítimo de Windows: `wininit.exe` > `services.exe` > `svchost.exe -k netsvcs` (servicio wuauserv) > `wuaucltcore.exe`.
2. **Evaluación de la ruta y arquitectura:** Se identificó el ejecutable en `C:\Windows\UUS\Packages\Preview\amd64\wuaucltcore.exe`. Se determinó que la carpeta `amd64` refiere a la arquitectura de 64 bits de Windows y `UUS` al motor de actualizaciones (Update Undocked Stack), descartando software de terceros o drivers.
3. **Recolección de identificadores:** Se extrajo el identificador de detección (Detection ID) reportado por la plataforma para realizar un cruce de datos con la base de conocimientos oficial de Sophos.

---

## 🛡 Respuesta y remediación
* **Clasificación del evento:** El incidente fue cerrado como un falso positivo operativo.
* **Remediación en consola:** Se reconoció la alerta en Sophos Central y se aplicó una exclusión sobre el identificador de detección para permitir que el sistema operativo finalice su ciclo de actualización sin bloqueos.

---

## 💡 Análisis técnico: ¿Qué sucedió y por qué se generó la alerta?
El motor CryptoGuard de Sophos no funciona buscando firmas de virus conocidos, sino que analiza **comportamientos** en tiempo real. 

Durante este incidente, el motor de actualizaciones de Windows (`wuaucltcore.exe`) comenzó a descargar y preparar un paquete nuevo ("Preview"). Para hacerlo, el proceso realizó operaciones masivas y a altísima velocidad de lectura, escritura y sobreescritura en el disco, soltando múltiples archivos con nombres de códigos largos (GUIDs) en una carpeta oculta del sistema (`C:\$WinREAgent\Scratch\`).

Para el algoritmo de comportamiento de Sophos, un proceso que repentinamente empieza a sobrescribir archivos del sistema rápidamente y a crear archivos con nombres alfanuméricos incomprensibles imita de manera exacta el comportamiento destructivo de un **ransomware cifrando el disco**. Al tratarse de un paquete de actualización muy reciente, su comportamiento exacto no estaba contemplado aún en la lista de confianza (whitelist) del EDR, lo que provocó que el sistema cortara el proceso como medida de precaución para proteger el equipo.
