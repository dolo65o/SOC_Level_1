## From Events to Alerts — SOC Alert Management

### Event → Alert Pipeline
```
Event occurs (login, process launch, file download)
    ↓
System logs it (OS, firewall, cloud provider)
    ↓
Logs shipped to SIEM/EDR (millions/day)
    ↓
Security solution applies detection rules
    ↓
Alert generated for suspicious/anomalous events
    ↓
SOC analyst triages alert (dozens/day, not millions)
```

> Alerts exist so analysts review dozens of suspicious events
> instead of manually scanning millions of raw logs.

---

### Alert Management Platforms

| Platform | Examples | Use Case |
|----------|---------|---------|
| **SIEM** | Splunk ES, Elastic | Primary alert management — best for most SOC teams |
| **EDR/NDR** | MS Defender, CrowdStrike | Own dashboards but prefer feeding into SIEM/SOAR |
| **SOAR** | Splunk SOAR, Cortex SOAR | Aggregates alerts from multiple solutions — large SOC teams |
| **ITSM** | Jira, TheHive | Ticket-based alert/case management |

---

### L1 Role in Alert Triage

| Role | Responsibility |
|------|---------------|
| **SOC L1** | Review alerts, distinguish real from false positive, escalate to L2 if real threat |
| **SOC L2** | Receive escalated alerts, deeper analysis + remediation |
| **SOC Engineer** | Ensure alerts contain enough context for efficient triage |
| **SOC Manager** | Track triage speed + quality — ensure real attacks aren't missed |

---

### Alert Priority Example
```
"Email Marked as Phishing"      → common, triage quickly
"Unusual Gmail Login Location"  → investigate — possible account compromise
"Unapproved Mimikatz Usage"   → critical — credential dumping in progress escalate to L2 immediately
```

> Not all alerts are equal. Severity + context determine response speed.
> A Mimikatz alert during business hours from an IT machine looks very different from the same alert at 3AM on an HR laptop.

---

## Alert Properties — Anatomy of a SOC Alert

| # | Property | Description | Examples |
|---|----------|-------------|---------|
| 1 | **Alert Time** | When alert was created — usually a few minutes after the actual event | Alert: 15:35 / Event: 15:32 |
| 2 | **Alert Name** | Summary of what happened — based on detection rule name | Unusual Login Location, Mimikatz Usage, RDP Bruteforce |
| 3 | **Alert Severity** | Urgency level — set by detection engineers, adjustable by analysts |  Low /  Medium /  High /  Critical |
| 4 | **Alert Status** | Current triage state |  New /  In Progress /  Closed |
| 5 | **Alert Verdict** | Is it a real threat or noise? |  True Positive /  False Positive |
| 6 | **Alert Assignee** | Analyst responsible for the alert — takes ownership | Also called alert owner |
| 7 | **Alert Description** | Explains the alert in 3 parts | Rule logic / Why it indicates attack / How to triage |
| 8 | **Alert Fields** | Specific values that triggered the rule | Affected hostname, commandline entered, source IP |

---

### Alert Description — 3 Sections

```
1. Rule Logic      → what conditions triggered this alert
2. Attack Context  → why this activity could indicate an attack
3. Triage Guide    → (optional) steps to investigate this specific alert
```

---

### Alert Severity — When to Act

| Severity | Response |
|----------|---------|
|  Low/Informational | Document, monitor — no immediate action |
|  Medium | Investigate within shift |
|  High | Prioritize — investigate promptly |
|  Critical | Drop everything — investigate immediately, escalate |

---

### Key Notes
- **Alert Time ≠ Event Time** — factor in the delay when building attack timelines
- **Severity is adjustable** — if context reveals higher/lower risk, analysts can change it
- **Assignee = accountability** — once you take an alert, you own the outcome
- **Verdict matters** — False Positive feedback helps engineers tune detection rules

---

## Alert Prioritization

### Why It Matters
Hundreds of alerts in the queue. You can only work one at a time.
Wrong prioritization = real attack progresses while you triage noise.

---

### 3-Step Prioritization Process

#### Step 1 — Filter
```
Only take: New / Unassigned alerts
Skip:      Already assigned, In Progress, or Closed alerts
```
Don't duplicate work. Check assignee before picking up any alert.

#### Step 2 — Sort by Severity
```
 Critical  → first
 High      → second
 Medium    → third
 Low       → last
```
Detection engineers design critical rules for high-confidence, high-impact threats.
Critical alerts are most likely real and most damaging if ignored.

#### Step 3 — Sort by Time (within same severity)
```
Oldest first → Newest last
```

**Why oldest first?**
```
Alert A (2 hours ago): attacker already inside, dumping data
Alert B (5 mins ago):  attacker just started reconnaissance

If you take B first → A's attacker has even more time to cause damage
Always work the oldest breach first
```

---

### Priority Decision Flow
```
Queue of alerts
    ↓
Filter: only New/Unassigned
    ↓
Sort: Critical → High → Medium → Low
    ↓
Within same severity: Oldest → Newest
    ↓
Pick up alert → assign to yourself → begin triage
```

---

### Key Insight
> Prioritization isn't just about severity — it's about **time in the network**.
> An attacker from 3 hours ago has had 3 hours to move laterally, escalate privileges, and establish persistence. A fresh alert can wait 10 minutes. The old one cannot.

---

## Alert Triage Process

<img width="1555" height="528" alt="reference flow for alert triage" src="https://github.com/user-attachments/assets/8065eb1f-2d61-484d-bfd3-efffedcc0bba" />

### Also called:
Alert handling / Alert processing / Alert investigation / Alert analysis

---

### Phase 1 — Initial Actions
```
1. Assign alert to yourself        ← take ownership
2. Move status to "In Progress"    ← prevent duplicate work
3. Read alert details              ← name, description, key indicators
```

---

### Phase 2 — Investigation
Most complex step — requires technical knowledge + experience.

**If no workbook available, follow this sequence:**

```
1. WHO is affected?
   → User, hostname, cloud resource, network segment, website

2. WHAT happened?
   → Suspicious login? Malware execution? Phishing? Data exfiltration?

3. CONTEXT — review surrounding events
   → What happened shortly BEFORE the alert? (initial access, recon)
   → What happened shortly AFTER? (lateral movement, persistence)

4. VERIFY using threat intelligence
   → Check IPs, hashes, domains against VirusTotal, MISP, etc.
   → Cross-reference with known TTPs
```

> Surrounding events are critical.
> The alert is rarely the full story — it's the moment the attacker was caught.
> What happened before tells you how they got in.
> What happened after tells you how far they've gone.

---

### Phase 3 — Final Actions

```
Determine verdict:
     True Positive  → real threat → escalate to L2 + comment
     False Positive → no threat  → document why + close

Write detailed comment:
    → What you investigated
    → What you found
    → Why you reached your verdict

Close the alert:
    → Move status to Closed/Resolved
    → Set verdict (TP/FP)
```

---

### Full Triage Flow
```
Pick alert from queue
    ↓
Assign to self → mark In Progress
    ↓
Read name, description, indicators
    ↓
Investigate: Who / What / Context / Intel
    ↓
Verdict: True Positive or False Positive
    ↓
Comment: document your full reasoning
    ↓
Close alert
```

---

### Key Rule
> Your comment is your work product.
> If you can't explain your verdict in writing,
> you haven't finished the investigation.
> Other analysts and L2 will rely on your comment
> to understand what you found and why.
