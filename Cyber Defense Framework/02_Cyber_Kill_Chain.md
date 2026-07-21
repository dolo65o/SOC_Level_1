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

---

## Cyber Kill Chain – Weaponization Phase

After reconnaissance, the attacker turns raw info into an actionable attack tool by crafting **malware/exploits into a payload**.
- Often uses automated tools or buys malware on the **Dark Web**.
- Sophisticated actors / **APTs** (Advanced Persistent Threat groups) write **custom malware** to stay unique and evade detection.

### Key Terms
- **Malware** – software designed to damage, disrupt, or gain unauthorized access to a system.
- **Exploit** – code that takes advantage of a vulnerability/flaw.
- **Payload** – the malicious code the attacker actually runs on the target.

### Common Weaponization Tactics
- Infected MS Office docs with malicious **macros/VBA scripts**.
- Malicious payload/worm implanted on **USB drives**, distributed publicly.
- Setting up **Command and Control (C2)** infrastructure to execute commands / deliver more payloads.
- Infecting host with a **backdoor** to bypass security and maintain access.
- Tailoring **phishing templates** or **OAuth-consent apps** to appear legitimate.
---

## Cyber Kill Chain – Delivery Phase

Delivery is the stage where the attacker chooses how to transmit the payload or malware into the target environment. Common methods include:

| Method | How it Works |
|---|---|
| **Phishing email** | After recon, attacker crafts a malicious email targeting one person (**spear phishing**) or many. Contains a malicious link/attachment that compromises the victim on interaction. |
| **USB drops** | Physical delivery — infected USBs dropped in public places (coffee shops, car parks). Sophisticated version: branded with the company logo and mailed in as a fake "gift." |
| **Watering hole attack** | Compromises a website a target group frequently visits, redirecting them to a malicious site — leading to a **drive-by download** (e.g. a fake pop-up prompting a "browser extension" install). |

---

## Cyber Kill Chain – Exploitation Phase

Exploitation is the moment the attacker's code actually executes on the target, taking advantage of a known vulnerability. Megatron can use several key techniques to gain access at this stage:

- **Malicious macro execution** — often delivered via a phishing email, running ransomware the moment the victim opens the document.
- **Zero-day exploits** — target unknown, unpatched flaws in a system. Since the vulnerability isn't publicly known yet, there's little chance of early detection.
- **Known CVEs** — attacker exploits publicly disclosed vulnerabilities that haven't been patched in the target environment.

Once access is gained, the attacker can further exploit software, system, or server-based vulnerabilities to **escalate privileges** or **move laterally** across the network.

**Signs of exploitation to watch for:**

| Indicator | Description |
|---|---|
| Unexpected process spawns | New or unusual processes starting without a clear trigger |
| Registry changes / new services | Modifications to the registry or creation of unfamiliar services |
| Suspicious command-line arguments | Unusual or malicious-looking commands appearing in system logs |

---

## Cyber Kill Chain – Installation Phase

As covered in Weaponization, a **backdoor** (also known as an *access point*) lets an attacker bypass security measures and hide their access. Once initial access is gained, the attacker wants a way back in if the connection drops, they're detected and removed, or the system gets patched — this is achieved by installing a [persistent backdoor](https://www.offensive-security.com/metasploit-unleashed/persistent-backdoors/), letting them return to a previously compromised system. (See the [Windows Persistence Room](https://tryhackme.com/room/windowslocalpersistence) for a hands-on look at Windows persistence techniques.)

**Ways persistence can be achieved:**

- **Web shell on a webserver** — a malicious script (`.php`, `.asp`, `.aspx`, `.jsp`, etc.) used to maintain access. Their simplicity and common file formats make them hard to detect and easy to mistake for benign files. (See [Microsoft's write-up on web shell attacks](https://www.microsoft.com/security/blog/2021/02/11/web-shell-attacks-continue-to-rise/).)

- **Backdoor on the victim's machine** — e.g. using [Meterpreter](https://www.offensive-security.com/metasploit-unleashed/meterpreter-backdoor/), a Metasploit Framework payload that gives the attacker an interactive remote shell.

- **Creating/modifying Windows services** — MITRE ATT&CK technique [T1543.003](https://attack.mitre.org/techniques/T1543/003/). Attackers use tools like `sc.exe` (create/start/stop/query/delete services) and [Reg](https://attack.mitre.org/software/S0075/) to configure malicious services, often **masquerading** them under names resembling legitimate OS/software services.

- **Registry Run Keys / Startup Folder entries** — the payload runs every time a user logs in. Windows has both per-user startup locations and a system-wide startup folder checked regardless of which account logs in. (See [MITRE ATT&CK T1547.001](https://attack.mitre.org/techniques/T1547/001/).)

Attackers may also use the [Timestomping](https://attack.mitre.org/techniques/T1070/006/) technique — modifying a file's timestamps (modify, access, create, change) — to evade forensic detection and make malware appear as part of a legitimate program.

---

## Cyber Kill Chain – Command & Control (C2) Phase

After achieving persistence and running the malware, the attacker opens a **C2 (Command and Control)** channel to remotely control and manipulate the victim's machine. This is also referred to as **C&C** or **C2 Beaconing** — a type of malicious communication between a C2 server and the malware on the infected host. The term *beaconing* comes from the fact that the infected host consistently "checks in" with the C2 server.

Once the compromised endpoint establishes this connection to the attacker's external server, the attacker gains full control of the victim's machine. Traditionally, **IRC (Internet Relay Chat)** was the go-to C2 channel, but modern security solutions now easily detect malicious IRC traffic, so it has fallen out of favor.

**Common C2 channels used today:**

| Channel | Notes |
|---|---|
| **HTTP (port 80) / HTTPS (port 443)** | Blends malicious traffic in with legitimate web traffic, helping evade firewalls. |
| **DNS** | Infected machine makes constant DNS requests to an attacker-controlled DNS server — known as **DNS Tunneling**. |

Note: the C2 infrastructure isn't necessarily owned directly by the adversary — it can also be another compromised host acting on the attacker's behalf.

---

## Cyber Kill Chain – Actions on Objectives Phase

After moving through the first six phases, Megatron finally reaches the last stage — **taking action on his original objectives**. With hands-on-keyboard access to the environment, the attacker can now pursue goals such as:

- **Credential collection** — harvesting user credentials for further access.
- **Privilege escalation** — gaining elevated access (e.g. jumping from a workstation to domain administrator) by exploiting misconfigurations.
- **Internal reconnaissance** — exploring internal software and systems to uncover further vulnerabilities.
- **Lateral movement** — spreading through the company's environment to reach additional systems.
- **Data collection and exfiltration** — gathering sensitive data and extracting it out of the network.
- **Deleting backups and shadow copies** — removing recovery options by targeting **Shadow Copy**, a Microsoft technology that creates backup snapshots of files/volumes.
- **Overwriting or corrupting data** — destroying data integrity as part of the final attack impact.
