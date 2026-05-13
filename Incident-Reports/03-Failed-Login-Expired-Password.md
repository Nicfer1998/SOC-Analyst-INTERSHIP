# INC-20260420-003 — Mass Failed Login Attempts — **False Positive**

# 📝 Summary

On April 20, 2026, a security alert was triggered due to multiple failed login attempts on a user's account. After investigating the logs in **Microsoft Entra ID**, it was determined that the attempts originated from the user's legitimate devices trying to authenticate with an expired password. The incident was closed as a **False Positive** of operational origin.

---

## 🔍 Incident Details

| Field | Value |
|---|---|
| **Incident ID** | 20260420-003 |
| **Severity** | **MEDIUM** |
| **Category** | Failed access / Suspicious brute force |
| **Initial detection** | Automated alert for multiple authentication errors within a short timeframe |

---

## 🛠 Investigation Process

### 1. Sign-in logs analysis
We accessed the user's profile in **Microsoft Entra ID** to review the authentication history:
* **Error code:** Error **50126** (Error validating credentials due to invalid username or password) was repeatedly identified.
* **Device verification:** Verified that the attempts originated from registered and compliant organization devices (the user's corporate laptop and mobile device).
* **Geolocation:** The IP address location matched the user's habitual workspace, ruling out unauthorized access from suspicious regions.

### 2. Credential Policy Review
The account status was verified:
* **Last Password Change:** It was detected that the last password change occurred exactly 91 days ago.
* **Root Cause:** The company's 90-day rotation policy had expired the password. The user's mobile device continued to attempt automatic synchronization with the old credential, triggering the security alert.

---

## 🛡 Response and remediation

| Action | Status | Description |
| :--- | :--- | :--- |
| **Identity Validation** | ✅ Completed | Contacted the user to confirm if they were experiencing access issues. |
| **Password Reset** | ✅ Completed | Assisted the user in resetting their password through the self-service portal. |
| **Device Synchronization** | ✅ Completed | Advised the user to update the new credentials on all desktop and mobile applications to stop automated failed logs. |

---

## 💡 Lessons learned

1.  **Technical differentiation:** It is crucial to distinguish between a "Password Spraying" or "Brute Force" attack and a credential de-synchronization. The key markers are the **Source (IP/Device)** and the specific error code.
2.  **Expiration patterns:** Keeping track of corporate policy durations (30/60/90 days) allows analysts to quickly hypothesize the cause of mass failures occurring on specific days of the month.
3.  **Application hygiene:** Users should be instructed on the importance of updating credentials on mobile devices immediately after a system-wide password change to prevent automated account lockouts.

---

## 📊 Tools used

* **Microsoft Entra ID (Azure AD)** — Analysis of Sign-in logs and device compliance status.
* **Microsoft 365 defender** — Identity alert management.
