## Alert Reporting, Escalation & Communication

### The Alert Path

```
Alerts arrive (SIEM / EDR / Ticket system)
    ↓
L1 Analyst triages
    ↓
False Positive → comment → close
True Positive  → report → escalate to L2
    ↓
L2 Analyst → deeper investigation → remediation
```

---

### Alert Reporting
Before closing or escalating — document your investigation.

**When required:**
- High/Critical severity alerts
- True Positives heading to L2
- When team standards require formal documentation

**What to include:**
- Timeline of events investigated
- Evidence gathered (logs, IPs, hashes, screenshots)
- Analysis steps taken
- Verdict + reasoning

> Your report is L2's starting point.
> A good report saves L2 hours of re-investigation.
> A poor report means L2 starts from scratch — wasting time during an active breach.

---

### Alert Escalation
Escalate True Positives that need deeper investigation or remediation beyond L1 scope.

```
Conditions for escalation:
→ Confirmed True Positive
→ Requires deeper forensic analysis
→ Active breach or critical system affected
→ Remediation beyond L1 authority
```

**Process:**
- Follow team's agreed escalation procedures
- Attach your alert report
- Brief L2 on key findings verbally if urgent

---

### Communication with Other Departments

| Department | When to contact | Example |
|-----------|----------------|---------|
| **IT Team** | Verify legitimate admin actions | "Did you grant admin rights to this user?" |
| **HR** | Clarify user context | "Is this person a new hire or contractor?" |
| **Legal/Compliance** | Data breach implications | Notify if PII may be involved |
| **Management** | Critical/high-impact incidents | Escalate beyond SOC if breach is confirmed |

> A SOC analyst who asks questions gets better answers than one who guesses.
> One call to IT can confirm or rule out a False Positive in 2 minutes.

---

## Alert Report Writing

### Why Write Reports?

| Purpose | Why it matters |
|---------|---------------|
| **Context for escalation** | L2 gets instant context — no re-investigation from scratch |
| **Permanent record** | SIEM logs kept 3-12 months; alerts kept indefinitely — context must live in the alert |
| **Skill development** | If you can't explain it simply, you don't fully understand it |

---

### Report Format — The 5 Ws

| W | Question | What to include |
|---|----------|----------------|
| **Who** | Which user/account was involved? | Username, user role, department |
| **What** | What action or event occurred? | Command run, file downloaded, login attempted |
| **When** | When did the activity happen? | Start time, end time, duration |
| **Where** | Which system was involved? | Hostname, IP address, website, cloud resource |
| **Why** | Why is this your verdict? | Your reasoning — the most critical W |

---

### Good Report Structure

```
WHO:   john.doe (Finance dept) — standard user account
WHAT:  Executed mimikatz.exe via PowerShell with -dumpcreds flag
WHEN:  March 21, 2024 — 03:42 AM to 03:44 AM
WHERE: Hostname: FIN-PC-042 | IP: 192.168.1.55
WHY:   Mimikatz is a known credential dumping tool with no
       legitimate business use. Execution at 3AM outside business
       hours by a non-IT finance user confirms malicious intent.
       Verdict: TRUE POSITIVE — escalate to L2 immediately.
```

---

### Key Rule
> The "Why" is the most important W. Anyone can document what happened.
> Your value as an analyst is explaining **what it means** and why you reached the verdict you did.
> L2 will trust your escalation based on the quality of your "Why".

---

## Alert Escalation to L2

### When to Escalate

| Condition | Reason |
|-----------|--------|
| Major cyberattack indicators | Requires DFIR-level investigation beyond L1 scope |
| Remediation needed | Malware removal, host isolation, password reset |
| External communication required | Customers, partners, management, law enforcement |
| You don't fully understand the alert | Better to ask than to blindly close |

> Escalating when unsure is not a weakness — it's good practice.
> Especially in your first months, clarifying beats guessing every time.

---

### Escalation Steps

```
1. Move alert to In Progress → complete your analysis
2. Write alert report → set your verdict (True Positive)
3. Reassign alert to L2 analyst on shift
4. Ping L2 in corporate chat or in person
5. L2 reads your report → contacts you if questions
```

**Some teams require:** formal written escalation request with required fields.
**Most teams:** reassign + ping is enough.

---

### What L2 Does After Receiving Escalation

```
Read L1's alert report
    ↓
Validate True Positive independently
    ↓
Communicate with other departments if needed
    ↓
For major incidents → start formal Incident Response process
```

---

### Requesting L2 Support (Without Full Escalation)

If you're unsure but not ready to escalate:
```
1. Keep alert In Progress
2. Document what you've found so far
3. Ask L2 for guidance — explain what's confusing
4. Apply the guidance, complete your triage
```

> L2 exists to support L1 — not just to receive escalations.
> Use them as a resource. A 2-minute discussion with L2
> can save you 30 minutes of incorrect investigation.

---

## Crisis Communication in SOC

### Core Rule
> SOC teams should have formal Crisis Communication procedures.
> If they don't — know these scenarios by heart.

---

### Critical Communication Scenarios

| Scenario | What to do |
|----------|-----------|
| **L2 unavailable for 30+ min on urgent alert** | Call L2 → call L3 → call manager. Always know where emergency contacts are. |
| **Compromised account (Slack/Teams) needs user verification** | Never contact user through the breached platform. Use phone call or alternative contact method. |
| **Alert flood — multiple criticals at once** | Prioritize per standard workflow AND immediately inform L2 on shift about the volume. |
| **Realized you misclassified an alert days later** | Contact L2 immediately. Explain your concern. Don't sit on it — attackers can be silent for weeks before striking. |
| **SIEM logs not parsed/searchable** | Don't skip the alert. Investigate what you can. Report the technical issue to L2 or SOC engineer. |

---

### Key Principles

**On compromised communication channels:**
```
Account breached → attacker may be reading messages
Contacting victim via breached platform = tipping off the attacker
Always use out-of-band communication (phone call)
```

**On misclassification:**
```
You closed it as False Positive → but it was real
Attacker has been in the network for days
Every hour matters — escalate the moment you realize
Threat actors average 200+ days in a network before detection
```

**On alert floods:**
```
Prioritize: Critical → High → Medium → Low + oldest first
But don't suffer in silence — L2 needs to know
They may spin up additional resources or trigger incident response
```

---

### Escalation Contact Order
```
L2 on shift → L3 → SOC Manager
```
> Always know these contacts before your shift starts.
> Finding them during a critical incident wastes time you don't have.
