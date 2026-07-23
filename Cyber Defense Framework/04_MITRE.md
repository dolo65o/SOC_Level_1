## MITRE — Cybersecurity Frameworks Overview

### What is MITRE?
Not-for-profit organization conducting R&D across multiple domains.
Mission: **"To solve problems for a safer world."**

---

### MITRE Cybersecurity Resources

| Resource | Purpose |
|----------|---------|
| **MITRE ATT&CK®** | Knowledge base of adversary tactics, techniques, and procedures |
| **CAR Knowledge Base** | Cyber Analytics Repository — detection analytics for ATT&CK techniques |
| **D3FEND** | Defensive countermeasures mapped to ATT&CK |
| **MITRE Engage™** | Framework for adversary engagement and deception operations |

---

### Why MITRE Matters for SOC

**Red Team:** Understand what attacks look like → simulate them accurately

**Blue Team:** 
- Map detections to known adversary behavior
- Identify gaps in coverage
- Build better detection rules
- Understand context of alerts (what tactic/technique triggered it)

---

### Key Framework — MITRE ATT&CK®

**ATT&CK = Adversarial Tactics, Techniques & Common Knowledge**

Structure:
```
Tactic (WHY — attacker's goal)
    └── Technique (HOW — method used)
             └── Sub-technique (specific variation)
                      └── Procedure (exact implementation)
```

**Example:**
```
Tactic:        Credential Access
Technique:     OS Credential Dumping [T1003]
Sub-technique: LSASS Memory [T1003.001]
Procedure:     Attacker uses Mimikatz sekurlsa::logonpasswords
```

---

## MITRE ATT&CK® Framework

### What is ATT&CK?
Globally-accessible knowledge base of adversary tactics and techniques based on **real-world observations**.
Started in 2013 to document TTPs used by APT groups.

🔗 https://attack.mitre.org

---

### TTP Breakdown

| Component | The question it answers | Example |
|-----------|------------------------|---------|
| **Tactic** | WHY — attacker's goal | Reconnaissance |
| **Technique** | HOW — method used | Active Scanning |
| **Sub-technique** | Specific variation | Vulnerability Scanning |
| **Procedure** | Exact implementation | Using Nmap with specific flags |

---

### ATT&CK Matrices

| Matrix | Covers |
|--------|--------|
| **Enterprise** | Windows, macOS, Linux, Cloud, Network, Containers |
| **Mobile** | Android, iOS |
| **ICS** | Industrial Control Systems |

🔗 Enterprise Matrix: https://attack.mitre.org/matrices/enterprise/
🔗 Mobile Matrix: https://attack.mitre.org/matrices/mobile/
🔗 ICS Matrix: https://attack.mitre.org/matrices/ics/

---

### ATT&CK Matrix Structure

<img width="1861" height="732" alt="ATT CK Matrix" src="https://github.com/user-attachments/assets/197d8713-83fd-4221-8cf2-6f5ab285afad" />

```
Tactic 1    | Tactic 2    | Tactic 3    | ... | Tactic 14
------------|-------------|-------------|-----|----------
Technique A | Technique D | Technique G | ... |
Technique B | Technique E | Technique H | ... |
  ↳ Sub-B1  | Technique F | Technique I | ... |
  ↳ Sub-B2  |             |             | ... |
Technique C |             |             | ... |
```

Tactics run across the top — each column contains nested techniques + sub-techniques.

---

### Enterprise Tactics (14 total)

| # | Tactic | Goal |
|---|--------|------|
| 1 | Reconnaissance | Gather target information |
| 2 | Resource Development | Build attack infrastructure |
| 3 | Initial Access | Get into the target |
| 4 | Execution | Run malicious code |
| 5 | Persistence | Maintain foothold |
| 6 | Privilege Escalation | Gain higher permissions |
| 7 | Defense Evasion | Avoid detection |
| 8 | Credential Access | Steal credentials |
| 9 | Discovery | Map the environment |
| 10 | Lateral Movement | Move through the network |
| 11 | Collection | Gather data of interest |
| 12 | Command & Control | Communicate with compromised systems |
| 13 | Exfiltration | Steal data out |
| 14 | Impact | Disrupt/destroy/manipulate |

---

### Example — Drill Down

```
Tactic:        Reconnaissance
    ↓
Technique:     Active Scanning [T1595]
               🔗 https://attack.mitre.org/techniques/T1595/
    ↓
Sub-techniques:
    T1595.001 — Scanning IP Blocks
    T1595.002 — Vulnerability Scanning
    T1595.003 — Wordlist Scanning
```

---

### Each Technique Page Contains

<img width="1723" height="778" alt="Active scanning" src="https://github.com/user-attachments/assets/7ad20766-1678-4d97-859c-237b48bff158" />

- **Description** — what it is and how it works
- **Technique ID** — e.g. T1595
- **Procedure examples** — real APT groups + software that used it
- **Mitigations** — how to prevent it
- **Detections** — how to detect it
- **References** — source material

---

### ATT&CK Navigator
Visual tool for annotating and exploring the ATT&CK matrix.
Use cases: map detections, plan red team coverage, identify gaps.

🔗 https://mitre-attack.github.io/attack-navigator/

---

### Key Links

| Resource | URL |
|----------|-----|
| ATT&CK Home | https://attack.mitre.org |
| Enterprise Tactics | https://attack.mitre.org/tactics/enterprise/ |
| Enterprise Techniques | https://attack.mitre.org/techniques/enterprise/ |
| ATT&CK Navigator | https://mitre-attack.github.io/attack-navigator/ |
| Active Scanning | https://attack.mitre.org/techniques/T1595/ |

---

## MITRE ATT&CK — Practical Applications

### Why ATT&CK Matters
- Provides **standard language** for describing adversary behavior
- Eliminates confusion when same technique has multiple names
- Unique IDs (T1595, T1053, etc.) enable precise communication across teams
- Bridges gap between threat intelligence and actionable detection

---

### Who Uses ATT&CK & How

| Role | Goal | How They Use ATT&CK |
|------|------|---------------------|
| **CTI Teams** | Analyze threats, improve security posture | Map threat actor behavior to TTPs → create actionable profiles |
| **SOC Analysts** | Investigate and triage alerts | Link activity to tactics/techniques → context + prioritization |
| **Detection Engineers** | Design detection systems | Map SIEM/EDR rules to ATT&CK → identify coverage gaps |
| **Incident Responders** | Respond to and investigate incidents | Map incident timeline to tactics → visualize full attack |
| **Red/Purple Teams** | Test and improve defenses | Build emulation plans aligned to known group TTPs |

---

### Threat Intelligence → Detection Pipeline

```
Threat report published (e.g. Mustang Panda uses phishing for initial access)
    ↓
Map to ATT&CK technique: Spearphishing Attachment [T1566.001]
    ↓
Build SIEM detection rule for email attachments with malicious macros
    ↓
Create playbook for analysts when rule fires
    ↓
Intelligence becomes operational defense
```

---

### Case Study — Mustang Panda (G0129)

**Target profile:** Government entities, non-profits, NGOs

**ATT&CK Mapping:**

| Tactic | Technique | What Mustang Panda does |
|--------|-----------|------------------------|
| Initial Access | Phishing [T1566] | Spearphishing emails to gain entry |
| Persistence | Scheduled Task [T1053] | Maintains foothold via scheduled tasks |
| Defense Evasion | Obfuscated Files [T1027] | Obfuscates malware to evade detection |
| C2 | Ingress Tool Transfer [T1105] | Transfers tools into compromised environment |

🔗 Mustang Panda ATT&CK page: https://attack.mitre.org/groups/G0129/
🔗 ATT&CK Navigator: https://mitre-attack.github.io/attack-navigator/

---

### Post-Incident ATT&CK Mapping Value

```
Attack happens
    ↓
Map each stage to ATT&CK tactic/technique
    ↓
Visualize full attack chain in Navigator
    ↓
Identify which techniques had no detection
    ↓
Build rules to cover gaps
    ↓
Better prepared for future campaigns from same/similar group
```

---

### Key Links

| Resource | URL |
|----------|-----|
| ATT&CK Groups | https://attack.mitre.org/groups/ |
| Mustang Panda | https://attack.mitre.org/groups/G0129/ |

---

## MITRE CAR — Cyber Analytics Repository

### What is CAR?
Knowledge base of **ready-made detection analytics** built around ATT&CK.
Each analytic describes how to detect a specific adversary behavior with real queries.

🔗 https://car.mitre.org

---

### CAR vs ATT&CK

| | ATT&CK | CAR |
|-|--------|-----|
| **Purpose** | Documents what attackers do | Shows how to detect what attackers do |
| **Output** | TTP descriptions | Detection queries + pseudocode |
| **Audience** | All security roles | Detection engineers, SOC analysts |
| **Tool-specific?** | No | Yes — Splunk, EQL, LogPoint examples |

---

### CAR Analytic Structure

Each analytic contains:

| Section | Content |
|---------|---------|
| **Description** | What behavior is being detected and why |
| **ATT&CK References** | Linked tactics and techniques |
| **Implementations** | Pseudocode + tool-specific queries |
| **Unit Tests** | (where available) validate the analytic works |

---

### Example — CAR-2020-09-001: Scheduled Task File Access

🔗 https://car.mitre.org/analytics/CAR-2020-09-001/

**ATT&CK Mapping:**
- Tactic: Persistence / Privilege Escalation
- Technique: Scheduled Task/Job [T1053]

**Pseudocode:**
```
files WHERE
  file_path LIKE '%System32\Tasks%'
  AND action IN ('CREATE', 'MODIFY')
```

**Splunk Implementation:**
```splunk
index=* sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
EventCode=11
TargetFilename="*\\System32\\Tasks\\*"
| table _time, ComputerName, TargetFilename, Image
```

**LogPoint:**
```
norm_id=WindowsSysmon event_id=11
path="*\System32\Tasks\*"
```

---

### Why CAR is Valuable for SOC Analysts

```
ATT&CK says: "Attackers use Scheduled Tasks for persistence"
CAR says:    "Here's the exact Splunk query to detect it"
    ↓
No need to build detection from scratch
    ↓
Copy → paste → adapt to your environment → detection live
```

---

### Key Links

| Resource | URL |
|----------|-----|
| CAR Home | https://car.mitre.org |
| CAR Analytics List | https://car.mitre.org/analytics/ |
| CAR-2020-09-001 | https://car.mitre.org/analytics/CAR-2020-09-001/ |
| CAR Data Model | https://car.mitre.org/data_model/ |

---

## MITRE D3FEND

### What is D3FEND?
**Detection, Denial, and Disruption Framework Empowering Network Defense**
Maps out **defensive techniques** with a standard language for how security controls work.

🔗 https://d3fend.mitre.org

---

### ATT&CK vs D3FEND

| | ATT&CK | D3FEND |
|-|--------|--------|
| **Focus** | How attacks happen | How to stop attacks |
| **Perspective** | Offensive | Defensive |
| **Output** | TTPs | Defensive countermeasures |
| **Matrix** | 14 tactics (attacker goals) | 7 tactics (defender actions) |

---

### D3FEND Matrix — 7 Defensive Tactics

| Tactic | Purpose |
|--------|---------|
| **Model** | Understand and map the environment |
| **Harden** | Reduce attack surface |
| **Detect** | Identify malicious activity |
| **Isolate** | Contain threats |
| **Deceive** | Mislead attackers |
| **Evict** | Remove attacker presence |
| **Restore** | Recover from attack |

---

### D3FEND Technique Page — What It Contains

- **Description** — how the defensive technique works
- **Implementation considerations** — what to think about when deploying
- **Digital artifacts** — what systems/data the defense relates to
- **ATT&CK mapping** — which offensive techniques this defense counters

---

### Example — Credential Rotation [D3-CRO]

🔗 https://d3fend.mitre.org/technique/d3f:CredentialRotation/

```
Defense:    Regularly rotate passwords + credentials
Why:        Prevents attackers from reusing stolen credentials
ATT&CK:     Counters credential-based attacks (T1078, T1550, etc.)
Artifacts:  User accounts, service accounts, API keys
```

---

### How ATT&CK + D3FEND Work Together

```
ATT&CK: Attacker uses Pass-the-Hash [T1550.002]
    ↓
D3FEND: Find defensive countermeasures for this technique
    → Credential Rotation [D3-CRO]
    → Multi-factor Authentication
    → Privileged Account Management
    ↓
Implement defenses → close the gap
```

---

### Key Links

| Resource | URL |
|----------|-----|
| D3FEND Home | https://d3fend.mitre.org |
| D3FEND Matrix | https://d3fend.mitre.org/matrix |
| Credential Rotation | https://d3fend.mitre.org/technique/d3f:CredentialRotation/ |

---

## MITRE — Additional Tools & Frameworks

### Adversary Emulation Library
Free collection of step-by-step adversary emulation plans.
Maintained by **CTID (Center for Threat Informed Defense)**.

| Resource | URL |
|----------|-----|
| CTID Home | https://ctid.mitre.org |
| Emulation Library | https://ctid.mitre.org/resources/adversary-emulation-library/ |
| GitHub Library | https://github.com/center-for-threat-informed-defense/adversary_emulation_library |

**What emulation plans contain:**
- Step-by-step guide mimicking a specific threat group
- ATT&CK-mapped techniques used by that group
- Tools and commands to replicate the attack behavior
- Used by red/purple teams to test real defenses against real TTPs

---

### Caldera — Automated Adversary Emulation

🔗 https://caldera.mitre.org

- Automated tool for simulating real-world attacker behavior
- Built on ATT&CK framework
- Tests detection methods + incident response in controlled environment
- Supports both **red team** (attack simulation) and **blue team** (detection validation)

```
Red team use:  Automate attack chains based on ATT&CK techniques
Blue team use: Validate detections fire correctly against simulated attacks
Purple team:   Both simultaneously — identify gaps in real-time
```

---

### New & Emerging MITRE Frameworks

#### AADAPT
**Adversarial Actions in Digital Asset Payment Technologies**
🔗 https://aadapt.mitre.org

- Covers threats targeting **digital asset management systems**
- Includes its own ATT&CK-style matrix
- Focus areas: blockchain networks, smart contracts, digital wallets
- Helps defenders understand and mitigate crypto/fintech-specific threats

#### ATLAS
**Adversarial Threat Landscape for Artificial-Intelligence Systems**
🔗 https://atlas.mitre.org
🔗 Matrix: https://atlas.mitre.org/matrices/ATLAS

- Knowledge base for threats targeting **AI/ML systems**
- Documents real-world attack techniques against AI
- Covers: model poisoning, prompt injection, model theft, evasion attacks
- Provides mitigations specific to AI/ML technology

---

### MITRE Ecosystem — Full Summary

| Tool | Purpose |
|------|---------|
| **ATT&CK** | Document adversary TTPs |
| **CAR** | Ready-made detection analytics |
| **D3FEND** | Defensive countermeasures |
| **Engage** | Adversary engagement + deception |
| **Emulation Library** | Step-by-step threat group emulation plans |
| **Caldera** | Automated adversary emulation platform |
| **AADAPT** | Digital asset / crypto threat framework |
| **ATLAS** | AI/ML threat framework |

---

### Key Links

| Resource | URL |
|----------|-----|
| MITRE ATT&CK | https://attack.mitre.org |
| MITRE CAR | https://car.mitre.org |
| MITRE D3FEND | https://d3fend.mitre.org |
| MITRE Caldera | https://caldera.mitre.org |
| AADAPT | https://aadapt.mitre.org |
| ATLAS | https://atlas.mitre.org |
| Emulation Library | https://github.com/center-for-threat-informed-defense/adversary_emulation_library |
