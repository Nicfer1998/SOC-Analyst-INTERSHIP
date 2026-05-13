# INC-20260421-004 — Unfamiliar sign-in location — **False Positive (VPN)**

# 📝 Executive summary
On April 21, 2026, a security alert was triggered due to a sign-in from an unusual geographic location (USA). After analyzing the logs in **Microsoft Entra ID** and checking the network infrastructure, it was confirmed that the traffic originated from the company's **VPN**. The incident was closed as a false positive.

---

## 🔍 Incident details
| Field | Value |
|---|---|
| **Incident ID** | 20260421-004 |
| **Severity** | **low** |
| **Category** | unusual access / geographic anomaly |
| **Detection** | login detected from an external data center ip |

---

## 🛠 Investigation process
1. **Ip analysis:** an ip lookup was performed on the suspicious address. It was identified that the provider (asn) belongs to a cloud infrastructure service, which is typical for vpn exit nodes.
2. **User correlation:** by filtering the logs for the suspicious ip, it was observed that multiple employees from the organization had successful logins from the same address during the same timeframe.
3. **Device consistency:** the user agent and device id matched the user's registered corporate equipment, maintaining consistency with their usual activity.

---

## 🛡 Response and remediation
* **Network validation:** it was confirmed with the it/infrastructure team that the ip belongs to an authorized segment for remote work.
* **Policy adjustment:** a recommendation was made to mark this ip range as "trusted locations" (named locations) in conditional access to reduce noise in future monitoring.

---

## 💡 Lessons learned
* **Infrastructure identification:** differentiating between a residential ip and a data center ip is the first step in dismissing false positives caused by vpn usage.
* **Organizational context:** as an analyst, knowing the connectivity tools used by the company (vpn, proxies, etc.) is vital to avoid unnecessary escalations.
