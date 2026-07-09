## Security Hierarchy & Departments

### Organizational Pyramid

<img width="2072" height="882" alt="678ecc92c80aa206339f0f23-1756329066210" src="https://github.com/user-attachments/assets/77ed894e-fe14-4897-9cc7-92da58c00768" />

---

### Level Breakdown

| Level | Roles | Focus |
|-------|-------|-------|
| **Executive** | CEO, CFO, Company Owner | Global business objectives |
| **Security Leadership** | CISO, CTO, CIO | Company-wide IT/security program |
| **Security Managers** | SOC Manager, Red Team Lead | Manage single team/department |
| **Technical** | SOC Analyst, SOC Engineer, GRC Specialist, Pentester | Hands-on technical work |

---

### Security Departments (Large Companies)

| Team | Type | Responsibility |
|------|------|---------------|
| **Red Team** | Offensive | Pentesters, ethical hackers — find security issues |
| **Blue Team** | Defensive | SOC analysts, engineers, IR — detect and respond |
| **GRC Team** | Compliance | Policies, regulations (PCI DSS, ISO 27001, etc.) |

---

### Security Priorities Vary by Industry

| Industry | Primary CIA Focus |
|----------|-----------------|
| Law firms | Confidentiality — legal document privacy |
| Factories | Availability — production line uptime |
| Hospitals | Integrity + Availability — patient safety |

> No universal security structure. CISO's job is to match the security program to the business's actual risk profile.

---

## Blue Team — Departments & Roles

### SOC (Security Operations Center)
First line of defense — most likely entry point for your career.

<img width="1432" height="592" alt="SOC TEAM" src="https://github.com/user-attachments/assets/d40335da-82c3-4ffc-9bd9-4a18b71fa45d" />

| Role | Responsibility |
|------|---------------|
| **L1 Analyst** | Triage alerts, pass complex cases to L2 |
| **L2 Analyst** | Investigate advanced attacks |
| **SOC Engineer** | Configure/maintain security tools (EDR, SIEM) |
| **SOC Manager** | Manage the entire SOC team |

---

### CIRT / CSIRT / CERT
Called in when SOC can't handle the incident alone — the "firefighters".
Must handle breaches without relying on tools like EDR/SIEM.

<img width="1397" height="575" alt="CIRT" src="https://github.com/user-attachments/assets/8ba89468-a1e9-4236-b50d-24f353536cc5" />

**CIRT Team Roles:**

| Role | Focus |
|------|-------|
| CSIRT Manager | Oversee incident response operations |
| Forensics Lead | Digital evidence collection and analysis |
| Threat Intel Expert | Track threat actors and emerging TTPs |
| Threat Hunting Expert | Proactively hunt for hidden threats |
| Malware Analyst | Reverse engineer malicious code |

**Real-world CIRT examples:**
- **JPCERT** — Japan's national CERT
- **Mandiant** — Private global IR team
- **AWS CIRT** — Responds to AWS customer incidents

---

### Specialized Blue Team Roles
Required by large companies, tech startups, and government agencies.
Need deep expertise — usually built on SOC/IT experience first.

| Role | What they do |
|------|-------------|
| **DevSecOps** | Embed security into development pipeline |
| **Penetration Tester** | Simulate attacks to find weaknesses |
| **GRC Auditor** | Policies, compliance (PCI DSS, ISO 27001) |
| **AppSec Engineer** | Secure software development lifecycle |
| **Threat Intelligence Analyst** | Gather data on emerging threat groups |
| **Digital Forensics Analyst** | Uncover hidden threats in disk and memory |
| **AI Researcher** | Study AI threats and defensive countermeasures |

---

### Career Path Reality
```
Entry: SOC L1 Analyst
  → SOC L2 → SOC Engineer / Threat Hunter
    → CIRT / Forensics / Threat Intel / AppSec
      → Specialized roles (AI Security, Red Team Lead, CISO track)
```

> SOC is the foundation. Every specialized role benefits from time spent in the alert queue — you learn how attacks actually look in the wild, not just how they look in textbooks.

---

## Internal SOC vs MSSP

| | Internal SOC | MSSP (Managed Security Service Provider) |
|-|-------------|------------------------------------------|
| **Example** | Bank SOC protecting bank's own systems | Global MSSP protecting 60 customers across Europe |
| **Working Pace** | Calmer shifts, less time pressure | Starts every shift with a queue of urgent alerts |
| **Security Tools** | Few tools — but must know them deeply | 60+ diverse tools and platforms across customers |
| **Incident Exposure** | ~2 major attacks per year | Weekly attacks and breaches — rapid learning curve |

---

### Which is Better?

| Goal | Choose |
|------|--------|
| Deep expertise in one environment | Internal SOC |
| Broad exposure, faster skill growth | MSSP |
| Less stress, stable pace | Internal SOC |
| More incidents = more learning | MSSP |

---

### Career Paths After L1

```
SOC L1
  ├── SOC L2 Analyst          (most common next step)
  ├── SOC Engineer            (if tooling/configuration appeals to you)
  ├── CIRT / IR Specialist    (if incident response excites you)
  └── Management track        (→ SOC Manager → CISO)
```

> Your first 1–2 years = get real work experience.
> Don't overthink the path yet. Work the queue, learn from every alert, and the direction that excites you most will become obvious.
