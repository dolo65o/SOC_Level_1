## Email Artifact Collection

### Header Artifacts

| Artifact | What to look for |
|----------|-----------------|
| **Sender email address** | Spoofed domain, typosquat, display name mismatch |
| **Sender IP address** | Reverse lookup — does it match claimed sending domain? |
| **Subject line** | Urgency, threats, call-to-action language |
| **Recipient address** | To / CC / BCC — targeted or mass send? |
| **Reply-To address** | Different from From? → attacker redirecting replies |
| **Date and time** | Off-hours sending, timezone inconsistencies |

---

### Body Artifacts

| Artifact | What to do |
|----------|-----------|
| **URLs/hyperlinks** | Extract all links, expand shortened URLs, defang before documenting |
| **Attachment names** | Check for double extensions, suspicious names |
| **Attachment hash** | Generate MD5/SHA256 → lookup on VirusTotal |

---

### Artifact Collection Workflow

```
1. Open raw email source (View → Message Source)
    ↓
2. Extract header artifacts:
   → Sender address + IP
   → Reply-To
   → Subject
   → Date/time
    ↓
3. Extract body artifacts:
   → Copy all URLs → defang → expand shortened ones
   → Save attachments → generate hash → don't open
    ↓
4. Investigate:
   → IP reputation: AbuseIPDB, VirusTotal
   → URL reputation: VirusTotal, URLScan.io
   → Hash lookup: VirusTotal, MalwareBazaar
   → Domain: WHOIS lookup
```

---

### Artifact Documentation Template

```
=== EMAIL ARTIFACT REPORT ===

HEADER:
  Sender:     attacker[@]evil[.]com
  Sender IP:  192[.]168[.]1[.]1
  Reply-To:   harvest[@]different-evil[.]com
  Subject:    "Urgent: Your account will be suspended"
  Recipient:  victim@company.com
  Date/Time:  2024-03-21 03:42:17 UTC

BODY:
  URL:        hxxp[://]evil[.]com/login
  Attachment: invoice.pdf  (actually invoice.pdf.exe)
  Hash (MD5): d41d8cd98f00b204e9800998ecf8427e

VERDICT:
  Indicators: [list red flags]
  Conclusion: [Phishing / Malspam / Benign]
```

---

## Email Analysis Tools

### Mail Header Analysis

| Tool | Use | Link |
|------|-----|------|
| **Messageheader** | Paste full header → extracts sender IP, routing path, misconfigs | [Google Admin Toolbox](https://toolbox.googleapps.com/apps/messageheader/analyzeheader) |
| **Message Header Analyzer** | Same functionality, alternative interface | [MHA](https://mha.azurewebsites.net/) |

**How to use:**
```
1. Open raw email source (View → Message Source)
2. Copy full header section
3. Paste into tool
4. Review: sender IP, routing hops, SPF/DKIM results
```

---

### IP & URL Reputation Tools

| Tool | What it does | Link |
|------|-------------|------|
| **IPinfo** | IP geolocation, ISP/org info, quick legitimacy check | [ipinfo.io](https://ipinfo.io/) |
| **URLScan.io** | Safely browses URL, records all activity, captures screenshot | [urlscan.io](https://urlscan.io/) |
| **Talos Intelligence** | Cisco threat intel — IP/domain reputation + classification | [talosintelligence.com](https://talosintelligence.com/reputation_center/) |
| **VirusTotal** | Hash, URL, IP, domain multi-engine scan | [virustotal.com](https://www.virustotal.com) |

---

### When to Use Each Tool

```
Got a sender IP?
    → IPinfo.io    (location + org)
    → Talos        (reputation + classification)
    → AbuseIPDB    (abuse history)

Got a suspicious URL?
    → URLScan.io   (safe browse + screenshot)
    → VirusTotal   (reputation check)

Got a file hash?
    → VirusTotal   (multi-engine AV scan)
    → MalwareBazaar (malware family lookup)

Got full email headers?
    → Messageheader or MHA (automated parsing)
```

---

### Full Phishing Investigation Toolkit

| Tool | Link |
|------|------|
| Google Messageheader | [toolbox.googleapps.com](https://toolbox.googleapps.com/apps/messageheader/analyzeheader) |
| Message Header Analyzer | [mha.azurewebsites.net](https://mha.azurewebsites.net/) |
| IPinfo | [ipinfo.io](https://ipinfo.io/) |
| URLScan.io | [urlscan.io](https://urlscan.io/) |
| Talos Intelligence | [talosintelligence.com](https://talosintelligence.com/reputation_center/) |
| VirusTotal | [virustotal.com](https://www.virustotal.com) |
| MalwareBazaar | [bazaar.abuse.ch](https://bazaar.abuse.ch) |
| AbuseIPDB | [abuseipdb.com](https://www.abuseipdb.com) |
| CyberChef | [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef) |
| WHOIS Lookup | [whois.domaintools.com](https://whois.domaintools.com) |
