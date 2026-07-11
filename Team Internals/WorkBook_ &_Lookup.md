## Identity & Asset Inventory

### Why They Matter
Alerts without context are guesses.
Inventory turns "suspicious activity" into "expected or unexpected" — fast.

**Scenario:**
```
Alert: G.Baker logged into HQ-FINFS-02 and shared financial data with R.Lund

Without inventory: Is this normal? No idea.
With inventory:    G.Baker = Finance Manager, HQ-FINFS-02 = Finance file server
                   R.Lund = External contractor with no finance access → ESCALATE
```

---

### Identity Inventory
Catalogue of **who** exists in the organization — users, service accounts, their roles and privileges.

**Key questions it answers:**
- Who is this user? What's their role and department?
- What are their normal working hours?
- Do they have legitimate access to this resource?

**Sources:**

| Solution | Examples | Notes |
|----------|---------|-------|
| **Active Directory** | On-prem AD, Entra ID | Most common — identity database + access control |
| **SSO Providers** | Okta, Google Workspace | Cloud alternative to AD |
| **HR Systems** | BambooHR, SAP, HiBob | Full employee data — limited to humans only |
| **Custom Solution** | CSV, Excel sheets | Common in smaller orgs |

---

### Asset Inventory
Catalogue of **what** exists — servers, workstations, and other computing resources.

**Key questions it answers:**
- What is this server/machine used for?
- Who is authorized to access it?
- Where is it located (HQ, branch, cloud)?

**Sources:**

| Solution | Examples | Notes |
|----------|---------|-------|
| **Active Directory** | On-prem AD, Entra ID | Solid asset DB in addition to identity |
| **SIEM or EDR** | Elastic, CrowdStrike | Agents collect host info automatically |
| **MDM Solution** | MS Intune, Jamf | Dedicated asset management platform |
| **Custom Solution** | CSV, Excel sheets | Common fallback in smaller orgs |

---

### Triage Using Both Inventories

```
Alert fires
    ↓
Identity lookup: Who is the user? Role? Normal hours? Expected access?
    ↓
Asset lookup:    What is the affected system? Sensitivity? Who should access it?
    ↓
Cross-reference: Does this user + this system + this time make sense?
    ↓
Yes → likely False Positive (document why)
No  → likely True Positive (escalate with context)
```

> Identity + Asset inventory = the difference between
> a 5-minute triage and a 2-hour investigation.
> Know where your org's inventories live before your first shift.

---

## Network Diagrams & Network Context

### Why Network Context Matters
IPs alone mean nothing without knowing what they represent.
Network diagrams turn raw IPs into an attack narrative.

---

### Scenario — Reconstructing an Attack from Logs

```
08:00  103.61.240.174 → repeated connections to firewall TCP/10443
08:23  103.61.240.174 → translated to internal IP 10.10.0.53 (VPN assigned)
08:25  10.10.0.53 → scanning 172.16.15.0/24 (no open ports found)
08:32  10.10.0.53 → scanning 172.16.23.0/24 (ongoing)
```

**Without network diagram:** a list of IPs and ports — meaningless.
**With network diagram:** a clear attack path.

---

### Attack Path Reconstruction

```
Step 1: VPN Brute Force
    103.61.240.174 → hammering TCP/10443 (VPN port)
    Target: vpn.tryhatme.thm

Step 2: Successful VPN Login
    Attacker authenticated → assigned IP from VPN Subnet (10.10.0.53)
    Now inside the network perimeter

Step 3: Database Subnet Scan (172.16.15.0/24)
    Attacker scans for databases → firewall blocks → no open ports found

Step 4: Office Subnet Scan (172.16.23.0/24)
    Attacker pivots → scanning office machines → ongoing at time of alert
```

---

### What Network Diagrams Show

| Element | Investigative Value |
|---------|-------------------|
| **Subnets** | Identify what segment a suspicious IP belongs to |
| **Services per subnet** | Understand what an attacker is trying to reach |
| **Firewall rules between segments** | Explain why some scans fail and others succeed |
| **VPN/DMZ zones** | Trace how external attacker became internal |
| **Named hosts/services** | Map IPs to actual systems (DB server, web server, etc.) |

---

### SOC Usage of Network Diagrams

```
Suspicious IP appears in alert
    ↓
Check network diagram
    ↓
What subnet is it in?      → tells you what zone the attacker reached
What services are nearby?  → tells you what they're targeting
What firewall rules exist? → tells you what they can/can't reach
    ↓
Reconstruct full attack path → write into report
```

> Network diagrams aren't optional reference material.
> They're a core investigation tool.
> Know your org's network layout before your first shift —
> during an active attack is the wrong time to learn it.

---

## SOC Workbooks

### What is a Workbook?
A structured document defining **exact steps** to investigate and remediate a specific threat type.
Also called: playbook, runbook, workflow.

**Why they exist:**
- L1 analysts are junior — can't be expected to perfectly triage every scenario
- Ensures consistent, high-quality triage across all analysts
- Eliminates verdicts made without sufficient evidence
- Senior analysts build them to support L1

---

### Workbook Structure — 3 Logical Phases

<img width="1457" height="747" alt="Unusual Login Location Workbook" src="https://github.com/user-attachments/assets/64bbce55-be18-49a2-ac6d-99dcb5fdd6fb" />

#### Phase 1: Enrichment
Gather context before forming any opinion.
```
→ Check identity inventory (who is the affected user?)
→ Run Threat Intelligence lookups (is the IP/domain/hash known malicious?)
→ Check asset inventory (what system is involved?)
→ Review user's normal behavior baseline
```

#### Phase 2: Investigation
Use gathered data + SIEM logs to reach a verdict.
```
→ Is this login/action expected for this user?
→ Does the time, location, and device make sense?
→ Are there surrounding suspicious events?
→ Does TI confirm or deny malicious nature?
→ Verdict: True Positive or False Positive?
```

#### Phase 3: Escalation
Act on your verdict appropriately.
```
True Positive  → escalate to L2 with full report
Requires user validation → contact user via alternative channel
False Positive → document reasoning → close
```

---

### Example — Atypical Login Workbook

```
Alert: Unusual login detected for user J.Smith

ENRICHMENT:
  → Identity lookup: J.Smith = IT Engineer, standard hours 9-5
  → TI lookup: Login from IP 185.220.101.x → known Tor exit node
  → Asset: Login to CORP-ADMIN-01 (admin server)

INVESTIGATION:
  → 3AM login outside business hours ✗
  → Tor exit node = anonymized source ✗
  → Admin server access by IT engineer = possible but suspicious ✗
  → No prior logins from this IP ✗
  → Verdict: TRUE POSITIVE

ESCALATION:
  → Write report with all 5 Ws
  → Escalate to L2 immediately
  → Do NOT contact user via corporate chat (may be compromised)
  → Phone call to verify if user initiated this login
```

---

### Key Rule
> Follow the workbook in order — every time.
> Skipping enrichment to jump straight to verdict
> is how analysts miss attacks that don't look obvious at first glance.
> The workbook exists because experience showed what gets missed without it.
