## Traditional SOCs – Overview

A **SOC (Security Operations Center)** is a centralized hub for monitoring and protecting an org's digital assets, combining people, processes, and technology.

### Key Capabilities
- **Monitoring & Detection** – continuous scanning for suspicious activity (mainly via SIEM).
  _e.g. multiple failed logins, login from unknown location_
- **Recovery & Remediation** – first responders to incidents; isolate endpoints, remove malware, stop malicious processes (via EDR, firewall, IAM).
  _e.g. isolate endpoint via EDR, block IP on firewall, disable user in IAM_
- **Threat Intelligence** – continuous feed of IOCs (IPs, hashes, domains) to stay current.
  _e.g. blocking a malicious domain from a TI feed_
- **Communication** – coordinates with IT/management to report and resolve incidents.
  _e.g. ticket to IT team to verify a patch deployment_

### Challenges Faced by SOCs
- **Alert Fatigue** – too many alerts, many false positives → analysts overwhelmed.
- **Too Many Disconnected Tools** – lack of integration (e.g. firewall logs vs endpoint logs handled separately).
- **Manual Processes** – undocumented, tribal-knowledge-based investigations → slower response.
- **Talent Shortage** – hard to hire/scale, worsens alert overload → longer incident response times.

---

## SOAR (Security Orchestration, Automation, and Response)

**SOAR** unifies all SOC security tools (SIEM, EDR, Firewall, etc.) into a single interface, plus provides ticketing/case management.

### Core Capabilities

1. **Orchestration**
   - Connects tools from different vendors into one interface.
   - Defines **Playbooks** – predefined, dynamic workflows for investigating alerts (branches based on step results).
   - _Example (VPN brute force):_ alert from SIEM → check if user normally uses IP → check TI reputation → check for successful logins → escalate to containment.

2. **Automation**
   - SOAR executes playbook steps automatically — no manual clicks.
   - _Example:_ auto-query SIEM → auto-check TI → auto-disable user in IAM if malicious → auto-open ticket.

3. **Response**
   - Takes action across tools from one interface (e.g. block IP on firewall, disable user in IAM, open ticket) — automated per playbook.

### Do SOAR tools replace SOC analysts?
- **No.** SOAR automates repetitive tasks, but analysts are still needed for:
  - Complex investigations & judgment calls
  - Business-context understanding
  - Creating/maintaining the playbooks themselves
- SOAR eases SOC burden; it doesn't remove the need for analysts.

---

## SOAR Playbooks – Examples

Playbooks = predefined "if this, do this; else, do that" workflows for recurring alert types, built by SOC analysts.

### Phishing Playbook

<img width="1434" height="1140" alt="phising playbook" src="https://github.com/user-attachments/assets/05a04bd3-8977-414e-a0eb-8e91411263f8" />

- Trigger: alert **"Suspicious email received"**
- Steps:
  1. Create a ticket
  2. Check if email contains a **URL or attachment**
     - If **no** → notify users, close out
     - If **yes** → branch further based on whether it's a URL or an attachment (analyze URL / analyze attachment via TI, sandboxing, etc.)
- Goal: automate the time-consuming manual analysis (URLs, attachments, TI lookups) and remediate if confirmed malicious.

### CVE Patching Playbook

<img width="3000" height="962" alt="CVE patching playbook" src="https://github.com/user-attachments/assets/e7d9ec90-8c1f-4b74-9396-47f222bf1290" />

- **CVE** = publicly disclosed vulnerability with an assigned CVE number.
- Problem: frequent CVE releases → manual tracking/patching overwhelms SOC, causes backlog.
- Playbook steps:
  1. Analyze CVE details
  2. Assess risk threshold
  3. Create a patching ticket
  4. Test the patch
  5. Push to production
