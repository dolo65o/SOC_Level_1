## File Hashes & Hash-Based IOCs

### What is a Hash?
A fixed-length numeric value that uniquely identifies data.
Same input → always same hash. Any change → completely different hash.

---

### Common Hashing Algorithms

| Algorithm | Hash Length | Status | Notes |
|-----------|-------------|--------|-------|
| **MD5** | 128-bit (32 hex chars) | ❌ Not cryptographically secure | Collision attacks possible — RFC 6151 (2011) |
| **SHA-1** | 160-bit (40 hex chars) | ❌ Deprecated | NIST deprecated 2011, banned for digital signatures 2013 |
| **SHA-256** | 256-bit (64 hex chars) | ✅ Recommended | Part of SHA-2 family — current standard |

> A hash algorithm is broken if two different files produce the same hash (collision).

---

### Security Use of Hashes
- Uniquely identify malware samples
- Reference malicious artifacts in threat reports
- IOC (Indicator of Compromise) in threat hunting
- Lookup on VirusTotal / Metadefender OPSWAT

---

### The Hash Limitation — One Bit Change = New Hash

```powershell
# Original file hash
Get-FileHash .\OpenVPN_2.5.1_I601_amd64.msi -Algorithm MD5
# MD5: D1A008E3A606F24590A02B853E955CF7

# Append a single string to the file
echo "AppendTheHash" >> .\OpenVPN_2.5.1_I601_amd64.msi

# Hash after modification
Get-FileHash .\OpenVPN_2.5.1_I601_amd64.msi -Algorithm MD5
# MD5: 9D52B46F5DE41B73418F8E0DACEC5E9F
```

**Completely different hash — from one trivial change.**

---

### Why Hash-Based Hunting is Insufficient

```
Attacker modifies malware by 1 bit (add a space, change a comment)
    ↓
New hash generated
    ↓
Your hash-based detection misses it entirely
```

> With thousands of malware variants, hash-based IOCs alone
> won't catch anything but the exact known sample.
> Use hashes as one layer — not the only layer.
> Behavioral detection is more resilient than hash matching.

---

### Practical Commands

```powershell
# PowerShell — get file hash
Get-FileHash .\filename -Algorithm MD5
Get-FileHash .\filename -Algorithm SHA256
Get-FileHash .\filename -Algorithm SHA1
```

```bash
# Linux — get file hash
md5sum filename
sha256sum filename
sha1sum filename
```

### Hash Lookup Resources
- https://www.virustotal.com
- https://metadefender.opswat.com
- https://thedfirreport.com (real-world malware reports with IOCs)

---

## IP Addresses as IOCs — Pyramid of Pain

### Position in Pyramid
**Color: Green** = Easy for defenders to detect, but also easy for attackers to change.

---

### Defensive Use of IP IOCs
- Block/drop/deny inbound requests from known malicious IPs
- Applied at perimeter firewall or external firewall
- **Limitation:** Trivial for attacker to switch to a new public IP

---

### Fast Flux — Evading IP Blocking

**What it is:**
DNS technique used by botnets to hide malicious infrastructure behind constantly rotating IP addresses.

**How it works:**
```
Malware → resolves domain (e.g. evil.com)
    ↓
DNS returns IP #1 → analyst blocks it
    ↓
Next DNS query → returns IP #2 (different compromised host)
    ↓
Block IP #2 → DNS returns IP #3
    ↓
(continues indefinitely)
```

**Key concept:** Multiple IPs associated with one domain, rotating rapidly
Each IP = a compromised host acting as proxy

**Purpose:**
- Hide C2 server communication
- Resist takedowns and IP blocking
- Support phishing, malware delivery, web proxying

---

### IP IOC Limitations Summary

| Tactic | Attacker Bypass |
|--------|----------------|
| Block specific IP | Switch to new IP (minutes of effort) |
| Block IP range | Use IPs from different ASN/provider |
| Track C2 by IP | Fast Flux — IP rotates constantly |

---

### Useful Resources
- Sandbox analysis: https://app.any.run
- Fast Flux research: Palo Alto Unit42 "Fast Flux 101"
- Akamai Fast Flux analysis

---

## Domain Names as IOCs — Pyramid of Pain

### Position in Pyramid
**Color: Teal** = Slightly harder for attacker than IPs, but still relatively easy to change.

**Why harder than IPs:**
- Must purchase domain
- Must register it
- Must modify DNS records
- **But:** Many DNS providers have loose standards + APIs → still easy to rotate

---

### Punycode Attacks
Trick users into visiting malicious domains that look legitimate.

**How it works:**
```
Legitimate: adidas.de
Malicious:  adıdas.de  (Punycode: http://xn--addas-o4a.de/)
```
The `ı` (dotless i) looks identical to `i` in most fonts.
User sees a legitimate-looking domain → clicks → lands on malware/phishing page.

**Detection:** Modern browsers (Chrome, Edge, Safari, IE) now display full Punycode for suspicious domains.

**Detect via:** Proxy logs, web server logs

---

### URL Shorteners — Hiding Malicious Domains

Attackers use URL shorteners to obscure the real destination.

**Common services abused:**
`bit.ly` `goo.gl` `ow.ly` `s.id` `tinyurl.com` `tiny.pl` `x.co` `smarturl.it`

**Reveal actual destination:**
```
Append "+" to the shortened URL in browser address bar
Example: https://bit.ly/examplelink+
→ Shows the real redirect destination before visiting
```

---

### Analyzing Domain IOCs in Any.run Sandbox

Navigate to **Networking tab** after detonating sample:

| Tab | Shows |
|-----|-------|
| **HTTP Requests** | Resources retrieved from web servers (droppers, callbacks) |
| **Connections** | All communications — C2 traffic, FTP uploads/downloads |
| **DNS Requests** | Domains resolved — C2 domains, connectivity checks |

> Malware often makes DNS requests to check internet connectivity.
> If it can't reach home → may go dormant (sandbox evasion).

> ⚠️ Never visit IPs or URLs from malware sandbox reports directly.

---

### Domain IOC Detection Methods
- **Proxy logs** — log all outbound domain requests
- **DNS logs** — capture every domain resolution attempt
- **Web server logs** — inbound requests to your domains
- **Sandbox analysis** — detonate sample, review networking tab
- **Punycode awareness** — check for homograph characters in URLs

---

## Host Artifacts — Pyramid of Pain

### Position in Pyramid
**Color: Yellow** = Attacker must change tools and methodologies — time-consuming and resource-intensive.

---

### What are Host Artifacts?
Traces left by attackers on compromised systems.

| Type | Examples |
|------|---------|
| **Registry values** | Persistence keys, config modifications |
| **Suspicious processes** | Unusual parent-child relationships |
| **Attack patterns** | Specific command sequences, scripts |
| **Dropped files** | Malware payloads, temp files, batch scripts |
| **IOCs** | Hashes, mutexes, named pipes specific to the threat |

---

### Example — Suspicious Process from Word
```
winword.exe
    └── cmd.exe
         └── powershell.exe -encodedCommand ...
              └── malicious_payload.exe
```
This parent-child chain is a host artifact.
Legitimate Word never spawns PowerShell → immediately suspicious.

---

### Why Yellow = More Pain for Attacker

| Level | Attacker effort to bypass |
|-------|--------------------------|
| IP block | Change IP — minutes |
| Domain block | Register new domain — minutes/hours |
| **Host artifact detection** | Rewrite tools, change attack patterns, rebuild infrastructure — hours/days |

---

### Where to Find Host Artifacts

| Tool | What it shows |
|------|--------------|
| **Windows Event Viewer** | Process creation (4688), registry changes |
| **Sysmon** | Detailed process, network, file, registry events |
| **EDR** | Full process tree, file drops, registry mods |
| **Autoruns** | Persistence mechanisms in registry/startup |
| **Procmon** | Real-time file/registry/process activity |

---

## Network Artifacts — Pyramid of Pain

### Position in Pyramid
**Color: Yellow** = Same as host artifacts — attacker must modify tools and tactics to bypass detection.

---

### What are Network Artifacts?
Observable patterns in network traffic left by attacker tools and C2 communication.

| Type | Examples |
|------|---------|
| **User-Agent strings** | Custom/unusual HTTP User-Agent headers |
| **C2 information** | C2 server addresses, beacon intervals, protocols |
| **URI patterns** | Specific URL paths used in HTTP POST requests |
| **HTTP headers** | Unusual or hardcoded header values |

---

### User-Agent as a Network Artifact

User-Agent = HTTP request header identifying the client making the request.
Malware often uses hardcoded or unusual User-Agent strings.

**Detection approach:**
```
Normal environment User-Agents:
→ Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/...
→ Mozilla/5.0 (Macintosh...) Safari/...

Suspicious:
→ Never-before-seen User-Agent string
→ Outdated browser version (IE 6 in 2024)
→ Emotet-specific hardcoded string
```

---

### Detection Tools

#### TShark (CLI Wireshark)
```bash
# Extract host + user-agent from PCAP
tshark --Y http.request -T fields -e http.host -e http.user_agent -r analysis_file.pcap
```

#### Wireshark (GUI)
```
Filter: http.request
Follow HTTP stream → inspect headers manually
```

#### Snort / IDS
- Create rules matching specific User-Agent strings
- Alert on known malicious URI patterns

---

### Example — Emotet Network Artifacts
Emotet trojan uses specific hardcoded User-Agent strings in its HTTP C2 traffic.
Detecting these strings → block all Emotet C2 communication network-wide.

---

### Why Yellow = Painful for Attacker
```
You detect User-Agent "EvilMalware/1.0"
    → Block it at proxy/firewall
    → Attacker must recompile malware with new User-Agent
    → Rebuild C2 infrastructure
    → Re-deploy to victims
    → Hours to days of rework
```

---

### Network Artifact Investigation Workflow
```
Suspicious HTTP POST requests in logs
    ↓
Extract User-Agent strings (TShark/Wireshark)
    ↓
Compare against known environment baseline
    ↓
Identify anomalous/hardcoded strings
    ↓
Search threat intel for User-Agent → attribute to malware family
    ↓
Block User-Agent at proxy → create IDS rule
    ↓
Hunt for other hosts using the same User-Agent
```

---

## Tools as IOCs — Pyramid of Pain

### Position in Pyramid
**Color: Orange** = Highly painful for attacker — must build, buy, or learn new tools entirely.

---

### What are Tool-Based IOCs?
The actual utilities and executables attackers use to carry out their attack.

| Tool Type | Examples |
|-----------|---------|
| **Maldocs** | Malicious macro Word/Excel files for spearphishing |
| **Backdoors** | Custom C2 implants, RATs |
| **Custom executables** | .EXE, .DLL payloads |
| **Password crackers** | Mimikatz, hashcat configs |
| **Droppers** | Files that download/execute second-stage payloads |

**Example:** Trojan drops `Stealer.exe` in `C:\Users\[user]\AppData\Local\Temp\`

---

### Why Orange = Maximum Tool-Level Pain

```
You detect + block their tool
    ↓
Attacker must:
    → Build an entirely new tool (costly + time-consuming)
    → OR find a tool with same capability
    → OR learn to use a different tool proficiently
    → OR buy a new tool (if financially capable)
= Game over for many attackers
```

---

### Detection Methods at This Level

#### Antivirus Signatures
Match known malicious file signatures — catches known tools.

#### YARA Rules
Pattern-matching rules that identify malware based on strings, byte sequences, or structural properties.

#### Detection Rules
Community-shared rules for SIEM/EDR platforms.
Resource: **SOC Prime Threat Detection Marketplace** — https://tdm.socprime.com

---

### Fuzzy Hashing (SSDeep)
Standard hashing breaks on 1-bit change. Fuzzy hashing finds **similar** files.

```
Standard hash:
File A (original):  abc123...
File A (1 bit changed): xyz789...  ← completely different → evades detection

Fuzzy hash (SSDeep):
File A (original):  3:AXGBicGIA:A7G  
File A (modified):  3:AXGBicGIa:A7G  ← similarity score: 97% → still detected
```

**Use case:** Attacker recompiles malware with minor changes to evade signature detection.
SSDeep catches it via similarity analysis.

---

### Tool-Based IOC Resources

| Resource | Purpose |
|----------|---------|
| **MalwareBazaar** (bazaar.abuse.ch) | Malware samples, feeds, YARA results |
| **Malshare** (malshare.com) | Malware samples for threat hunting/IR |
| **SOC Prime TDM** (tdm.socprime.com) | Community detection rules for SIEM/EDR |
| **SSDeep** (ssdeep-project.github.io) | Fuzzy hashing for similarity analysis |
| **VirusTotal** | Hash lookup + SSDeep fuzzy hash display |

---

## TTPs — Pyramid of Pain (Apex)

### Position in Pyramid
**Color: Red** = Highest pain level for adversary — forces complete rethink of attack strategy.

**TTPs = Tactics, Techniques & Procedures**
The complete playbook of how an attacker operates from start to finish.

---

### What TTPs Cover

| Component | Definition | Example |
|-----------|-----------|---------|
| **Tactics** | The "why" — attacker's goal at each stage | Persistence, Lateral Movement, Exfiltration |
| **Techniques** | The "how" — method used to achieve the tactic | Pass-the-Hash, Scheduled Task, Spearphishing |
| **Procedures** | The specific implementation | Exact commands, scripts, tool configs used |

**Full reference:** MITRE ATT&CK Matrix → https://attack.mitre.org

---

### Why TTPs = Maximum Pain

```
Detect and respond to TTPs
    ↓
Attacker's options:
    1. Go back → research + training + reconfigure everything
    2. Give up → find easier target

Option 2 is almost always chosen.
```

At every lower level (IP → domain → host → tool), the attacker tweaks one thing.
At the TTP level, they must **change how they think and operate entirely**.

---

### Example — Pass-the-Hash Detection

```
Attack: Attacker steals NTLM hash → uses it to authenticate without knowing password
Detection: Windows Event Log monitoring → Event ID 4624 (Logon Type 3 with NTLM)
Response: Identify compromised host → isolate → reset credentials → stop lateral movement
Result: Attacker's technique is burned → must find entirely new approach
```

---

### Pyramid of Pain — Full Summary

| Level | IOC Type | Attacker Pain | Bypass Effort |
|-------|----------|--------------|---------------|
|  Hash values | File hashes | Trivial | Change 1 bit |
|  IP addresses | Malicious IPs | Easy | New IP in minutes |
|  Domain names | C2 domains | Easy-Medium | Register new domain |
|  Host artifacts | Registry, dropped files | Medium | Rewrite tooling |
|  Network artifacts | User-Agent, URI patterns | Medium | Recompile, reconfigure |
|  Tools | Malware, exploits | Hard | Build/buy new tool |
|  **TTPs** | **Full attack behavior** | **Devastating** | **Must retrain + rebuild everything** |

---

### Key Takeaway
> The higher up the Pyramid you detect, the more you hurt the attacker.
> Hash detection = slap on the wrist.
> TTP detection = force the attacker to become a different attacker.
> SOC maturity = moving detection from the bottom of the pyramid to the top.
### Key Takeaway
> At the IP/domain level, attackers pivot in minutes.
> At the tool level, they must rebuild from scratch.
> Every tool detection + rule you write forces the attacker
> to invest real time, money, and skill to continue.
> This is where defenders start winning.

