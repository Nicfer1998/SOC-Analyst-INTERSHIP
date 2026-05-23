# INC-20260522-029 — Malicious Network Traffic Blocked on Firewall — **En Investigación**

# 📝 Resumen

El 22 de mayo de 2026, Microsoft Sentinel generó un incidente al recibir alertas del firewall **Sophos Central**, detectando múltiples intentos de comunicación con un servidor de **Command & Control (C2/Generic-A)** desde una IP interna no identificada. El firewall Sophos bloqueó todos los intentos exitosamente. La investigación identificó que el dispositivo origen no tiene agente Sophos, no está unido al dominio y no tiene registro DNS — lo que sugiere un dispositivo no gestionado, posiblemente un móvil personal infectado conectado a la red WiFi corporativa. El incidente fue escalado al encargado de IT al finalizar el turno y queda pendiente de identificación del dispositivo.

---

## 🔍 Detalles del Incidente

| Campo | Valor |
|---|---|
| **ID del incidente** | INC-20260522-029 |
| **Severidad** | **MEDIA** |
| **Categoría** | Command & Control |
| **Detección Inicial** | Sophos Central — Malicious Network Traffic Blocked |
| **Firewall** | [firewall-id] |
| **IP origen** | [internal-ip] (interna) |
| **Clasificación Sophos** | C2/Generic-A |
| **Cliente** | [org-name] |
| **Estado** | 🔄 En investigación — escalado a IT |

---

## 🛠 Proceso de Investigación

### Conceptos clave

**C2 (Command & Control):** Servidor externo que controla un malware instalado en un dispositivo. El malware intenta conectarse periódicamente para recibir instrucciones — robar datos, moverse lateralmente, descargar más malware.

**C2/Generic-A:** Sophos detectó el patrón de tráfico como comunicación con servidor C2 pero no pudo identificar la familia exacta del malware. Lo clasificó genéricamente basándose en el comportamiento del tráfico.

### 1. Análisis del Timeline

Se generaron **6 alertas** entre las 01:42 y las 06:12 AM, todas del mismo tipo y desde la misma IP. El patrón de intentos periódicos es consistente con el comportamiento de un **beacon** — el malware intentando conectarse a su C2 en intervalos regulares.

| Timestamp | Evento |
|---|---|
| May 22 — 01:42 | Malicious network traffic blocked (x3) |
| May 22 — 06:12 | Malicious network traffic blocked (x3) |

### 2. Investigación del dispositivo origen

Se ejecutaron múltiples técnicas para identificar el dispositivo con la IP interna afectada:

**Sophos Central → Devices:**
- La IP no aparece en la lista de dispositivos gestionados.
- **Conclusión:** El dispositivo no tiene agente Sophos instalado.

**ping -a [internal-ip]:**
```
Respuesta: 4/4 paquetes — 0% pérdida — TTL=56
```
- El dispositivo **está encendido y activo** en la red.
- No devolvió nombre → sin registro DNS.
- **TTL=56** → consistente con dispositivo Android (TTL base 64, menos hops de red).

**nslookup [internal-ip]:**
```
DNS server no è in grado di trovare [internal-ip]
```
- El DNS del dominio no tiene registro PTR para esa IP.
- **Conclusión:** El dispositivo nunca se registró en el dominio.

**arp -a [internal-ip]:**
```
Impossibile trovare voci ARP
```
- El servidor no tiene la MAC en su tabla ARP porque están en subredes distintas. El tráfico entre subredes pasa por el router, no directamente entre hosts.

### 3. Perfil del dispositivo desconocido

Basándose en toda la evidencia recopilada:

| Característica | Evidencia |
|---|---|
| Sin agente Sophos | No aparece en Sophos Central |
| Sin nombre DNS | nslookup sin resultado |
| Fuera del dominio | No registrado en AD |
| IP por DHCP | No tiene registro PTR estático |
| TTL=56 | Consistente con Android |
| Activo de noche | Intentos entre 01:42 y 06:12 AM |

**Hipótesis principal:** Dispositivo móvil personal (Android) de un empleado conectado al WiFi corporativo con malware instalado — posiblemente adware agresivo, PUA o malware disfrazado de aplicación legítima (juego, utilidad, etc.).

### 4. Escalación

Al finalizar el turno se escaló al encargado de IT con toda la evidencia, solicitando:
- Revisar el **DHCP lease history** del scope correspondiente para la IP afectada en el horario del incidente
- Identificar la **MAC address** del dispositivo
- Con la MAC: identificar fabricante y al usuario propietario

---

## 🛡 Respuesta y Remediación

| Acción | Estado | Descripción |
|---|---|---|
| Análisis del timeline | ✅ Completado | 6 alertas — patrón de beacon C2 identificado. |
| Búsqueda en Sophos Central | ✅ Completado | Dispositivo sin agente — no gestionado. |
| ping -a | ✅ Completado | Dispositivo activo, sin nombre DNS, TTL=56. |
| nslookup | ✅ Completado | Sin registro PTR en DNS del dominio. |
| arp -a | ✅ Completado | Sin entrada ARP — subredes distintas. |
| Escalación a IT | ✅ Completado | Solicitud de DHCP lease history y MAC address. |
| Identificación del dispositivo | 🔄 Pendiente | IT debe revisar DHCP y switch ARP table. |
| Aislamiento / remediación | 🔄 Pendiente | Una vez identificado el dispositivo. |

---

## 💡 Lecciones Aprendidas

1. **Un dispositivo bloqueado sigue siendo una amenaza activa:** Sophos bloqueó el tráfico saliente hacia el C2, pero el malware sigue dentro del dispositivo en la red. Puede intentarlo de nuevo, puede escanear otros dispositivos internos o puede usar técnicas alternativas de C2 como DNS tunneling.
2. **Dispositivos sin agente = punto ciego:** Los móviles personales, tablets, impresoras y dispositivos IoT conectados a la red corporativa sin agente de seguridad son invisibles para las herramientas EDR. El firewall es la única capa que los monitorea.
3. **TTL como indicador de tipo de dispositivo:** El valor TTL de la respuesta al ping puede dar pistas sobre el sistema operativo. Windows usa TTL=128, Linux/Android TTL=64, iOS TTL=64. Los hops de red reducen ese valor.
4. **BYOD sin NAC = riesgo sistémico:** En entornos sanitarios, un móvil infectado puede estar en la misma VLAN que dispositivos médicos críticos. Implementar NAC (Network Access Control) para verificar la identidad y postura de seguridad de cualquier dispositivo antes de permitirle acceso a la red es la recomendación estructural de este caso.

---

## 📊 Herramientas Utilizadas

- **Microsoft Sentinel** — Recepción y gestión del incidente.
- **Sophos Central** — Detección y bloqueo del tráfico C2.
- **CMD (ping, nslookup, arp)** — Identificación del dispositivo origen.
- **Portal ITSM** — Gestión y seguimiento del ticket.

---

## 🎯 MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Command & Control | Application Layer Protocol | T1071 |
| Command & Control | Non-Standard Port | T1571 |
| Defense Evasion | Masquerading | T1036 |
