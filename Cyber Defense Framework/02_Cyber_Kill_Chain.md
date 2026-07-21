## Cyber Kill Chain — Phase 1: Reconnaissance

### What is the Cyber Kill Chain?
Framework developed by **Lockheed Martin (2011)** based on military kill chain concept.
Defines the 7 phases adversaries must complete to succeed in a cyberattack.

**Key principle:** Disrupt ANY phase = attack fails.
**SOC value:** Identify which phase you're seeing → predict next steps → stop the attack.

---

### Why It Matters
- Understand and defend against ransomware, APTs, security breaches
- Identify missing security controls in your infrastructure
- Recognize intrusion attempts early
- Understand attacker goals at each stage

---

### Phase 1 — Reconnaissance

**Goal:** Gather intelligence about the target before launching the attack.
**Detection:** Often passive and undetected — hardest phase to catch.

> Poor recon = sloppy attacks.
> Good recon = highly targeted, believable payloads with higher success rate.

---

### Recon Types

| Type | Interaction | Examples |
|------|-------------|---------|
| **Passive** | No direct contact with target | WHOIS lookups, social media scraping, breach data review |
| **Active** | Direct contact with target | Port scanning, banner grabbing, social engineering, probing open services |

---

### OSINT — Open Source Intelligence

Information gathered from **publicly available sources**.

| Source | What attackers find |
|--------|-------------------|
| Search engines | Technologies used, subdomains, exposed files |
| Social media | Employee names, roles, org structure |
| Online forums/blogs | Tech stack, internal processes |
| Print/online media | Business relationships, key personnel |
| WHOIS / technical data | Domain registrant info, IP ranges, DNS records |
| Public record databases | Company filings, contracts |

---

### Email Harvesting
Collecting email addresses for phishing campaigns.

| Tool | Capability |
|------|-----------|
| **theHarvester** | Emails, names, subdomains, IPs, URLs — from multiple public sources |
| **Hunter.io** | Contact info associated with a domain |
| **OSINT Framework** | Collection of categorized OSINT tools |

**Links:**
- https://github.com/laramies/theHarvester
- https://hunter.io
- https://osintframework.com

---

### Defender Takeaway
```
Minimize your OSINT footprint:
→ Limit public employee info on LinkedIn
→ Keep WHOIS data private (registrar privacy)
→ Don't expose tech stack in job postings
→ Monitor for your domain in breach databases
→ Treat your own org as a recon target — find what attackers find first
```
