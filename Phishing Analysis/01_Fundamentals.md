## Email Analysis — Phishing Investigation

### Why It Matters
Phishing = most common initial access vector.
One click → attacker foothold → full network compromise.

### Defender's Role
- Analyze email components to determine malicious or benign
- Identify true origin of suspicious messages
- Gather intel to harden defenses against future attacks

### Email Components to Inspect

| Component | What to look for |
|-----------|-----------------|
| **Headers** | True sender IP, spoofed From address, reply-to mismatches |
| **From/Reply-To** | Domain mismatch — display name vs actual address |
| **Links/URLs** | Hover to reveal real URL, shortened URLs, lookalike domains |
| **Attachments** | Malicious macros, executables disguised as docs |
| **Body** | Urgency, threats, requests for credentials |
| **Timestamps** | Sent time inconsistencies |

### Key Header Fields for Investigation
```
Received:        → shows actual mail server path (read bottom to top)
X-Originating-IP → true sender IP
Reply-To:        → if different from From → suspicious
Return-Path:     → where bounces go → often reveals real sender
Message-ID:      → unique ID — check domain matches sender
```

### Quick Verdict Process
```
Check From vs Reply-To → mismatch? → suspicious
Check links → not matching displayed text? → suspicious
Check attachments → macros, executables? → malicious
Check headers → originated from expected server? → verify
```
---
## Email Address Anatomy

### Structure
```
username @ domain
  david  @ tryhackme.com
```

| Part | Purpose | Example |
|------|---------|---------|
| **Username** | Identifies specific mailbox on the server | `david` |
| **@** | Separator — routes email to correct server | `@` |
| **Domain** | Mail server responsible for receiving | `tryhackme.com` |

### Analogy
```
Domain   = apartment building (street address)
Username = specific person/mailbox inside
@ symbol = the postal routing instruction
```

> History: Email popularized in 1970s on ARPANET by Ray Tomlinson — introduced the `@` symbol.

### Why It Matters for Phishing Analysis
```
Attackers spoof or imitate legitimate addresses:
  legitimate: david@tryhackme.com
  spoofed:    david@tryhackme.com.evil.com  ← domain is evil.com
  lookalike:  david@tryhackm3.com           ← typosquat
  display:    "TryHackMe Support" <attacker@evil.com>
```
Always check the **actual email address**, not just the display name.

---

## Email Protocols

### Three Core Protocols

| Protocol | Port | Role |
|----------|------|------|
| **SMTP** | 25 / 587 | Sends emails (client → server, server → server) |
| **POP3** | 110 / 995 | Downloads emails to single device |
| **IMAP** | 143 / 993 | Syncs emails across multiple devices |

---

### POP3 vs IMAP

| | POP3 | IMAP |
|-|------|------|
| Storage | Local device | Server |
| Multi-device | ✗ | ✓ |
| Deleted from server? | Yes (after download) | No (unless manually deleted) |
| Sent messages | Local only | Server (synced) |
| Best for | Single device users | Multiple device users |

---

### Email Journey (Sender → Recipient)

```
1. User sends email
   └── Email client → SMTP → sender's mail server

2. Sender's server queries DNS
   └── "What's the mail server for recipient's domain?"

3. DNS responds
   └── Returns recipient's mail server address (MX record)

4. Email delivered
   └── Sender's server → Internet → recipient's mail server

5. Recipient checks mailbox
   └── Email client connects to their mail server

6. Email retrieved
   └── POP3 → downloaded to device (removed from server)
   └── IMAP → synced to device (stays on server)
```

---

### SOC Relevance
```
SMTP logs → trace where phishing email originated
POP3/IMAP → understand how attacker accessed mailbox after compromise
MX records → identify legitimate mail servers (spoofing check)
```

---

## Viewing Raw Email Source

### How to View in Thunderbird
```
View → Message Source (or Ctrl+U)
```
Shows full raw email including all headers + body (plain text or HTML).

---

### Key Fields in Raw Email Source

```
Return-Path:     → actual sender address (often reveals spoofing)
Received:        → mail server hops — read BOTTOM to TOP for true origin
                   last "Received" at bottom = originating server/IP
X-Originating-IP → true sender IP address
From:            → display name + email (can be spoofed)
Reply-To:        → where replies go (mismatch with From = suspicious)
To:              → recipient
Subject:         → email subject
Date:            → timestamp
Message-ID:      → unique ID — domain should match sender domain
MIME-Version:    → email format version
Content-Type:    → plain text or HTML, attachments
```

---

### Reading Received Headers (Tracing Origin)

```
# Read from BOTTOM to TOP — oldest hop at bottom

Received: from mail.evil.com (1.2.3.4)   ← ORIGIN (read this first)
Received: from mx.middleserver.com        ← hop 2
Received: from mail.victim.com            ← delivered here (read last)
```

---

### What Raw Source Reveals (vs Normal View)

| Normal View | Raw Source |
|-------------|-----------|
| Display name only | Actual email address behind display name |
| No server info | Full server hop chain |
| No IPs | Originating IP address |
| Rendered HTML | Raw HTML/links (reveals hidden URLs) |
| No auth results | SPF/DKIM/DMARC pass/fail results |

---

### Quick Phishing Checklist from Raw Source
```
✓ Does Return-Path match From domain?
✓ Does Reply-To match From?
✓ Does originating IP match claimed sending domain?
✓ Does Message-ID domain match sender domain?
✓ Any SPF/DKIM/DMARC failures?
✓ Any suspicious links in HTML body?
```
---

## Email Body & Attachment Analysis

### Email Body Formats
- **Plain text** — no formatting, what you see is what you get
- **HTML** — supports images, links, styling — rendered by email client

---

### Viewing HTML Source
```
View → Message Source → scroll to body section
```
Reveals raw HTML behind the rendered email — critical for finding:
- Hidden/disguised links (`<a href="evil.com">Click here</a>`)
- Embedded images loading from external servers (tracking pixels)
- Obfuscated content not visible in rendered view

---

### Rendered vs Source — What Differs

| Rendered View | HTML Source |
|---------------|-------------|
| Clickable "Login Here" text | Actual URL the link points to |
| Displayed image | External URL image is loaded from |
| Blocked images | `<img src="http://tracker.evil.com/pixel.png">` |
| Clean formatted text | Raw HTML tags + embedded scripts |

---

### Attachment Analysis

Key headers in raw source for attachments:

```
Content-Type: application/pdf        ← file type
Content-Disposition: attachment;
    filename="invoice.pdf"           ← filename
Content-Transfer-Encoding: base64   ← encoding method

[base64 encoded data follows...]
```

---

### Reconstructing Attachments from Base64

```
1. Copy base64 encoded data from raw source
2. Decode using:
   → CyberChef: From Base64 operation
     https://gchq.github.io/CyberChef/#recipe=From_Base64(...)
   → Online converter: https://www.apivoid.com/tools/base64-to-pdf/
3. Save as original file type
4. Analyze the reconstructed file
```

---

### Common Attachment Content-Types

| Content-Type | File |
|-------------|------|
| `application/pdf` | PDF document |
| `application/msword` | .doc Word file |
| `application/vnd.openxmlformats-officedocument` | .docx/.xlsx |
| `application/zip` | ZIP archive |
| `application/octet-stream` | Generic binary (often executable) |
| `image/jpeg` / `image/png` | Image files |

---

### Phishing Indicators in HTML Body
```
✗ Displayed link text ≠ actual href URL
✗ External image URLs (tracking pixels)
✗ Urgency language + malicious link
✗ Form action pointing to attacker domain
✗ Attachment with macros or executables
✗ Base64 encoded scripts embedded in body
```
---

## Malicious Email Types & Phishing Analysis

### Email Threat Types

| Type | Description | Target |
|------|-------------|--------|
| **Spam** | Unsolicited bulk emails (malspam = malicious spam) | Mass |
| **Phishing** | Impersonates trusted entity to steal info | Mass |
| **Spear Phishing** | Targeted phishing using personalized info | Specific individual/org |
| **Whaling** | Spear phishing targeting executives (CEO/CFO) | C-suite |
| **Smishing** | Phishing via SMS/text message | Mobile users |
| **Vishing** | Phishing via voice call | Phone users |

---

### Phishing Email Indicators

| Indicator | Example |
|-----------|---------|
| **Spoofed From address** | `noreply@microsof.com` (typo) |
| **Urgency** | "Your account will be locked in 24 hours" |
| **Brand impersonation** | Fake logos, colors matching real company |
| **Grammar/spelling issues** | Awkward phrasing, unnatural wording |
| **Generic greeting** | "Dear Customer" instead of your name |
| **Hidden/shortened links** | `bit.ly/secure-login` hides real destination |
| **Malicious attachments** | `invoice.pdf.exe` — double extension trick |

---

### Safe Analysis — Defanging

**Never click URLs or open attachments directly during analysis.**
Defang URLs and IPs to make them unclickable.

**URL Defanging:**
```
Original:  http://www.suspiciousdomain.com
Defanged:  hxxp[://]www[.]suspiciousdomain[.]com
```

**Email Defanging:**
```
Original:  attacker@evil.com
Defanged:  attacker[@]evil[.]com
```

**IP Defanging:**
```
Original:  192.168.1.1
Defanged:  192[.]168[.]1[.]1
```

**CyberChef Defang Tools:**
- URL: https://gchq.github.io/CyberChef/#recipe=Defang_URL(...)
- IP: https://gchq.github.io/CyberChef/#recipe=Defang_IP_Addresses()

---

### Double Extension Trick
```
invoice.pdf.exe
         ↑
Real extension — it's an executable, not a PDF
Windows hides known extensions → user sees "invoice.pdf"
```

---

### Quick Phishing Identification Checklist
```
□ From address — typosquat or domain mismatch?
□ Reply-To — different from From?
□ Subject — urgency or fear tactics?
□ Greeting — generic ("Dear Customer")?
□ Links — hover → does URL match displayed text?
□ Attachments — unexpected? Double extension?
□ Grammar — errors or unnatural phrasing?
□ Branding — logos present but domain wrong?
```
