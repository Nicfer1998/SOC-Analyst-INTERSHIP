# INC-20260418-001 — Phishing Attempt & Lateral Movement — **False Positive**

# 📝 Executive Summary

On April 18, 2026, a security alert was triggered, initially categorized as a suspicious login and potential lateral movement. Following a detailed technical investigation using **Microsoft Defender for Office 365** and **Exchange Online**, the incident was reclassified as a Phishing attempt with a **False Positive** for lateral movement, caused by a pre-existing automated forwarding rule.

---

## 🔍 Incident Details

| Field | Value |
|---|---|
| **Incident ID** | 20260418-001 |
| **Severity** | **CRITICAL** |
| **Category** | Phishing / Unauthorized Access Attempt |
| **Initial Detection** | Phishing URL click and internal email forwarding anomaly |

---

## 🛠 Investigation Process

### 1. Phishing Analysis

The investigation began when **Microsoft Defender** alerted that a user clicked on the URL: `zedsms.com/`.

* **Analysis:** The URL was identified as a **Credential Harvesting** site designed to mimic a Microsoft login page.
* **Findings:** Defender confirmed the click. However, sign-in logs showed no successful logins from unusual locations or non-compliant devices immediately following the event.
* **Hypothesis:** The user reached the malicious site but likely did not submit their credentials.

### 2. Internal Mail Flow Anomaly

A second alert was triggered for "Internal Sending" (suspicious activity) toward a Finance account.

* **Investigation:** Upon reviewing the user's role in the Tenant, it was confirmed they belong to the **Finance department**. This justifies communication with that mailbox, but not the volume or frequency of the detected emails.

### 3. Root Cause Analysis (RCA)

To determine if this was **Lateral Movement** (an attacker moving from one account to another), the mailbox configuration in **Exchange Online (EXO)** was inspected.

* **Action:** Reviewed mail flow and forwarding rules via the **Exchange Admin Center**.
* **Result:** An active **Forwarding Rule** was discovered. All emails received in the user's personal account were automatically redirected to the Finance shared mailbox.
* **Conclusion:** The lateral movement alert was a **False Positive** triggered by this automated redirection. No manual attacker activity was found.

---

## 🛡 Response and Remediation

| Action | Status | Description |
|---|---|---|
| **Password Reset** | ✅ Completed | Forced password change to invalidate any potentially leaked credentials. |
| **Session Revocation & MFA** | ✅ Completed | Terminated all active sessions and required a new MFA challenge. |
| **URL Blocking** | ✅ Completed | Added `zedsms.com` to the Tenant’s block list. |
| **User Communication** | ⏳ In Progress | Interviewing the user to confirm if credentials were entered on the spoofed site. |
| **Rule Review** | ✅ Completed | Validated that the forwarding rule is part of the department's business process. |

---

## 💡 Lessons Learned

1.  **Context is King:** Knowing the user's department allowed for quickly dismissing a real intrusion and understanding the traffic as operational.
2.  **Event Correlation:** Automated forwarding rules often mimic **Business Email Compromise (BEC)** behavior. It is vital to verify Exchange rules before confirming lateral movement.
3.  **Proactive Defense:** Even without a confirmed successful login, the mere click on a phishing site mandates a **Zero Trust** approach: resetting credentials and sessions.

---

## 📊 Tools Used

* **Microsoft 365 Defender** — Incident timeline and click tracking.
* **Entra ID (Azure AD)** — Sign-in log analysis.
* **Exchange Online Admin Center** — Mail flow rules.

---

## 🏗️ Structural Solution (Best Practices)

The tenant should maintain documentation for all forwarding rules created for operational reasons. Each rule should include its justification, creation date, and the responsible party. 

When this type of alert occurs, the analyst has an immediate reference point to determine if the rule is known or new. If documented, it is a false positive; if not in the inventory, it remains a high-priority red flag.
