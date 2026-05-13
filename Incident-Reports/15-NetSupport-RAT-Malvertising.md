# INC-20260513-015 — NetSupport RAT via Malvertising (Fake Dynamics 365) — 🔴 TRUE POSITIVE

# 📝 Resumen

El 13 de mayo de 2026, Microsoft Defender for Endpoint generó una alerta de severidad alta por actividad multietapa que involucró ejecución de código malicioso, instalación de herramienta de acceso remoto (RAT) y establecimiento de canal de Command & Control (C2). La investigación confirmó que un usuario descargó un instalador falso de Microsoft Dynamics 365 al hacer clic en un anuncio publicitario malicioso de Google (malvertising). El equipo afectado fue aislado de forma preventiva y las sesiones activas del usuario fueron revocadas.

---

## 🔍 Detalles del incidente

| Campo | Valor |
| --- | --- |
| **ID del incidente** | INC-20260513-015 |
| **Severidad** | Alta |
| **Categoría** | TRUE POSITIVE — Infección por RAT vía Malvertising |
| **Alert Product** | Microsoft Defender for Endpoint |
| **Tácticas MITRE** | Initial Access, Execution, Persistence, Command and Control |
| **Técnicas MITRE** | T1566 Phishing, T1059 Scripting, T1053 Scheduled Task, T1219 Remote Access Tools |
| **Malware identificado** | NetSupport Manager RAT |
| **Vector de entrada** | Anuncio malicioso de Google (Malvertising) |
| **Clasificación final** | True Positive — Contención ejecutada |

---

## 🛠 Proceso de investigación

1. **Análisis del timeline de alertas:** Se identificaron 7 alertas activas correlacionadas por Defender for Endpoint en un intervalo de 5 minutos, abarcando 4 categorías distintas: ejecución sospechosa, software de gestión remota en ubicación inusual, tarea programada sospechosa y actividad vinculada a actor de amenaza emergente.

2. **Reconstrucción del process tree:**
   ```
   userinit.exe (arranque de sesión Windows)
     └── explorer.exe (escritorio del usuario)
           └── chrome.exe (navegador)
                 └── Dynamics365-[versión]-latest-win-x64.exe ← instalador falso descargado
                       └── pythonw.exe -x "[AppData\Local\Temp\...\sdk\marionettes.txt]"
                             ├── vc_redist.x86.exe (dependencia legítima usada como distracción)
                             ├── [pythonw.exe] → crea client32.exe ← componente NetSupport RAT
                             ├── [pythonw.exe] → crea remcmdstub.exe ← componente NetSupport RAT
                             ├── [pythonw.exe] → crea tarea programada mHuTS9or0TcRg50Z
                             └── [pythonw.exe] → conexión saliente a IP externa:443
   ```

3. **Identificación del malware:** Los archivos `client32.exe` y `remcmdstub.exe` son componentes reconocidos de **NetSupport Manager**, una herramienta de acceso remoto legítima frecuentemente abusada por atacantes como RAT. Microsoft Defender generó alertas específicas: `An active 'Netsupport' unwanted software was detected` (Informational) y `'Netsupport' unwanted software was detected and was active` (Low - Prevented).

4. **Análisis de persistencia:** El script Python creó una tarea programada con nombre aleatorio (`mHuTS9or0TcRg50Z`) configurada para lanzar `client32.exe`, asegurando que el RAT se reinstalara tras cada reinicio del equipo. Esta técnica de nombre generado aleatoriamente es una firma característica de malware automatizado.

5. **Análisis del canal C2:** Se registró una conexión saliente desde el equipo comprometido hacia una IP externa en el puerto 443 (HTTPS), utilizado para camuflar el tráfico del canal de comando y control como tráfico web legítimo.

6. **Determinación del vector de entrada:** El proceso padre del instalador malicioso fue `chrome.exe`, confirmando la descarga desde el navegador. La investigación con el usuario afectado reveló que intentó acceder al sitio oficial de Dynamics 365 y realizó clic en un anuncio de Google que redirigió a un sitio falso con un instalador troyanizado. Esta técnica se conoce como **Malvertising**.

---

## 🛡 Respuesta y remediación

- **Aislamiento del equipo:** El dispositivo afectado fue aislado de la red de forma inmediata desde la consola de Defender for Endpoint para cortar el canal C2 y prevenir movimiento lateral.
- **Revocación de sesiones:** Todas las sesiones activas del usuario afectado fueron revocadas en Azure AD / Microsoft Entra ID para invalidar tokens de acceso potencialmente comprometidos.
- **Bloqueo parcial por Defender:** Microsoft Defender bloqueó parcialmente la actividad maliciosa, previniendo la ejecución continua de NetSupport, aunque no evitó la instalación inicial.
- **Recomendaciones adicionales ejecutadas:**
  - Reformateo del equipo y reinstalación del sistema operativo desde imagen limpia.
  - Reset completo de credenciales del usuario afectado.
  - Verificación de otros equipos que pudieran haber contactado la IP del C2.

---

## 💡 Análisis técnico: Cadena de ataque completa

### Fase 1 — Initial Access (T1566 / Malvertising)
El usuario buscó el instalador oficial de Microsoft Dynamics 365. Un anuncio malicioso de Google posicionado sobre los resultados orgánicos redirigió al usuario a un sitio falso que ofrecía un instalador con el mismo nombre y apariencia que el oficial.

### Fase 2 — Execution (T1059 — Scripting)
El instalador falso ejecutó `pythonw.exe` en modo silencioso (`-x`, sin ventana visible) con un script alojado en la carpeta temporal del usuario (`AppData\Local\Temp`). El nombre del script (`marionettes.txt`) fue diseñado para no levantar sospechas al usar extensión `.txt` en lugar de `.py`.

### Fase 3 — Installation (T1219 — Remote Access Tools)
El script Python desplegó los componentes de NetSupport Manager (`client32.exe` y `remcmdstub.exe`) en ubicaciones del sistema. NetSupport es una herramienta de administración remota legítima, lo que permite al atacante operar bajo el radar de controles que bloquean malware conocido.

### Fase 4 — Persistence (T1053 — Scheduled Task)
Se creó una tarea programada con nombre aleatorio para garantizar la supervivencia del RAT tras reinicios del sistema. Este mecanismo asegura que el acceso remoto se restaure automáticamente incluso si el proceso es terminado manualmente.

### Fase 5 — Command & Control (T1071 — Application Layer Protocol)
Se estableció una conexión saliente persistente hacia la infraestructura del atacante usando el puerto 443 (HTTPS), protocolo permitido por la mayoría de los firewalls corporativos, camuflando el tráfico malicioso como navegación web normal.

---

## 🔑 Indicadores de Compromiso (IoCs)

| Tipo | Valor |
| --- | --- |
| **Proceso malicioso** | `pythonw.exe` ejecutado desde `AppData\Local\Temp` |
| **Archivos RAT** | `client32.exe`, `remcmdstub.exe` |
| **Tarea programada** | `mHuTS9or0TcRg50Z` |
| **Técnica de persistencia** | Scheduled Task (T1053) |
| **Puerto C2** | 443 (HTTPS) |

---

## 💡 Lección aprendida: ¿Qué es Malvertising?

**Malvertising** (Malicious Advertising) es una técnica de ataque que utiliza la red de publicidad legítima de Google u otras plataformas para distribuir malware. Los atacantes pagan por anuncios que imitan el aspecto de sitios oficiales y los posicionan sobre los resultados de búsqueda orgánicos.

Es especialmente efectiva porque:
- El usuario ve un anuncio en Google, plataforma de confianza.
- La URL del anuncio puede parecerse mucho al sitio legítimo.
- No requiere que el usuario tenga el equipo previamente comprometido.
- Es difícil de distinguir del sitio real sin verificar el certificado SSL del dominio.

**Recomendación de concienciación:** Siempre navegar directamente a la URL oficial del fabricante escribiéndola en el navegador, sin hacer clic en anuncios de búsqueda para descargar software empresarial.
