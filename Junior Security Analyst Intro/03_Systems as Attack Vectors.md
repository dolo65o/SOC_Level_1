## System-Targeted Attacks

### Why Systems Are High-Value Targets
Breaching one user via phishing = one compromised account.
Breaching the server = thousands of accounts compromised simultaneously.

> A trained gatekeeper is useless if the gate lock is cheap.
> Attackers bypass humans entirely by targeting vulnerable systems directly.

---

### System Value to Attackers

| Breached System | Attack Value |
|----------------|-------------|
| School student's laptop | Steal gaming accounts, add to botnet |
| Bank IT admin's laptop | Access internal banking infrastructure |
| Criminal law firm mail server | Dump all mailboxes → blackmail victims |
| Industrial network server | Encrypt entire network with ransomware |
| Government website panel | Defacement / hacktivism |

---

### Key Insight: Scale of Impact

```
Phishing one user    → 1 mailbox compromised
Breaching mail server → thousands of mailboxes compromised

Malware on one laptop  → 1 machine encrypted
RCE on network server  → entire network encrypted
```

> The higher the system's value and connectivity,
> the higher it sits on an attacker's target list.
> Admin accounts, servers, and cloud management panels
> are always priority targets — protect them accordingly.

---

# How Systems Are Attacked

## Three Attack Entry Points

### 1. Human-Led Attacks
Users inadvertently start the attack themselves.

**Common causes:**
- Inserting malicious USB found on street
- Downloading malware from pirated software sites
- Reusing weak passwords across multiple services

> **[81%](https://deepstrike.io/blog/password-statistics-2025) of breaches involve stolen or breached passwords.**
> Check your passwords: [HaveiBeenPawned](https://haveibeenpwned.com/Passwords)

---

### 2. Vulnerabilities
Software flaws exploited directly — no human interaction needed.

**[2024 stats](https://cyberpress.org/over-40000-cves-published-in-2024/):**

- 40,000+ CVEs published
- [300+](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) actively exploited in major attacks


**Admin mistakes that amplify risk:**
- Weak passwords on internet-facing systems
- Unrestricted access (no least privilege)
- Unpatched software left exposed

---

### 3. Supply Chain Attacks
Attacker compromises a trusted software vendor → pushes malicious update to all users.

```
Breach one library/app
    → Push malicious update
    → All users auto-install it
    → Thousands of orgs compromised simultaneously
```

**Famous examples:**
| Attack | Impact |
|--------|--------|
| SolarWinds | Thousands of companies + US government agencies |
| 3CX | VoIP software update compromised enterprise customers |
| Lottie Player | Affected TryHackMe (animation library) |

**Why it's hard to defend:**
- You can't fully control every library in every app
- Updates from trusted vendors are auto-installed
- Malicious code arrives via legitimate, signed channels

---

### SOC Relevance
```
Human-led   → Watch for credential alerts, unusual logins, USB events
Vulnerability → Monitor CVE feeds, patch management, exploit attempts
Supply chain → Behavioral monitoring — trusted software doing unusual things
```

> Supply chain is the hardest to catch.
> A legitimate, signed process doing malicious things
> won't trigger signature-based detection.
> Behavioral anomalies are your only signal.

---

## Software Vulnerabilities & CVE Response

### Key Concepts

| Term | Definition |
|------|-----------|
| **Vulnerability** | A flaw in software that can be exploited |
| **Zero-day** | Vulnerability discovered by attackers before anyone else — no patch exists |
| **CVE** | Public identifier assigned once vulnerability is disclosed |
| **Patch** | Vendor-supplied fix — the only permanent answer to a CVE |

> [Shellshock](https://www.invicti.com/blog/web-security/cve-2014-6271-shellshock-bash-vulnerability-scan/) existed in Linux **since 1992** — discovered in **2014**.
> 22 years of exposure. Zero-days can hide for decades.

---

### Notable CVE Timeline

<img width="1345" height="536" alt="CVE 2024" src="https://github.com/user-attachments/assets/f11feaaa-4aef-442b-9666-3884fef54cac" />

---

### Vulnerability Race — Once a CVE Goes Public
```
CVE published
    ↓
Attackers: develop exploits
Defenders: rush to patch all systems
    ↓
Whoever moves faster wins
```

---

### Zero-Day Response (No Patch Available)

While waiting for the vendor patch:

| Temporary Measure | Purpose |
|------------------|---------|
| Restrict access to trusted IPs only | Reduce exposure surface |
| Apply vendor-provided workarounds | Partial mitigation |
| Block known attack patterns on IPS/WAF | Stop known exploit signatures |
| Monitor intensively for exploitation traces | SOC detects active exploitation early |

---

### SOC Analyst's Role
> You can't patch zero-days. You can detect exploitation attempts.
> Your logs, behavioral alerts, and anomaly detection are the only
> defense between disclosure and the vendor releasing a patch.
> This is when SOC work matters most.

---

## Misconfigurations — System Setup Failures

### What It Is
Not a software bug — a **human setup mistake**.
Usually introduced to save time or simplify administration.

---

### Real-World Examples

| Misconfiguration | Impact |
|-----------------|--------|
| [Password "123456"](https://www.bleepingcomputer.com/news/security/123456-password-exposed-chats-for-64-million-mcdonalds-job-chatbot-applications/) on McDonald's chatbot | 64 million job applications exposed |
| [Misconfigured AWS WAF (Capital One)](https://www.bleepingcomputer.com/news/security/capital-one-data-breach-affects-106-million-people-suspect-arrested/#:~:text=intrusion%20occurred%20through%20a%20misconfigured%20web%20application%20firewall) | 106 million bank customers breached |
| Improperly configured [smart fridges](https://www.sectigo.com/resource-library/when-refrigerators-attack-how-cyber-criminals-infect-appliances-and-how-manufacturers-can-stop-them) | Silently recruited into botnets |

---

### Misconfiguration Chain — Database Example

<img width="1900" height="715" alt="misconfiguration" src="https://github.com/user-attachments/assets/82ae02b8-7e27-4911-9318-5dc3612ee6d0" />


> Latest software + two bad config decisions = full breach.
> The software had zero vulnerabilities. The admin introduced all the risk.

---

### SOC Response to Misconfigurations

Typically detected **after exploitation** — but proactive options exist:

| Approach | What it does |
|----------|-------------|
| **Penetration Testing** | Ethical hackers simulate attacks, report discovered flaws |
| **Vulnerability Scans** | Automated tools detect default passwords, outdated software |
| **Configuration Audits** | Manual review against standards like CIS Benchmarks |

> CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks
> Industry-standard hardening guides for OS, databases, cloud services.

---

### Key Difference: Vulnerability vs Misconfiguration

| | Vulnerability | Misconfiguration |
|-|--------------|-----------------|
| **Cause** | Software bug | Admin setup mistake |
| **Fix** | Vendor patch | Reconfigure the system |
| **Requires update?** | Yes | No |
| **Who introduces it?** | Developer | IT/Admin team |

---

## Defending Systems — Mitigation Measures

### Core Principle
```
Humans  → train them to recognize attacks
Systems → can't be trained → configure them correctly + patch them fast
```

Attackers don't separate human and system attacks.
Neither should your defense strategy.

---

### System Mitigation Measures

| Control | What it does |
|---------|-------------|
| **Patch Management** | Track + apply patches promptly — closes CVE windows before attackers exploit them |
| **IT Training** | Teach admins the risks of misconfigurations — informed teams make fewer setup mistakes |
| **Network Protection** | Restrict system access to trusted IPs/users — reduces attack surface dramatically |
| **Antivirus / EDR** | Stops or detects many attack types at the endpoint level |

---

### Defense Layers — Combined View

```
Attack arrives
    ↓
Patch applied?        → Exploit fails (vulnerability closed)
Antivirus active?     → Malware blocked/detected
Network restricted?   → Attacker can't reach the system
IT trained?           → No misconfiguration to exploit
    ↓
All bypassed?         → SOC detects + investigates + stops it
```

---

