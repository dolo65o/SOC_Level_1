## Social Engineering & Human Targeting

### Why Humans Are the Weakest Link
Technical defenses (firewalls, IDS, encryption) are hard to bypass.
Humans are not. One phishing email to the right person = full network access.

> Attacking the fortress walls = days of work, high chance of failure.
> Asking the gatekeeper to open the door = one email, instant access.

---

### Why Humans Are Targeted
Humans provide **access** — to systems, mailboxes, databases, and secrets.
Attackers either hunt specific high-value targets or breach at scale and decide later.

### Attack Examples & Attacker Goals

| Target | What Attacker Does Next |
|--------|------------------------|
| HR manager's Google account | Steal + sell entire employee database |
| Wealthy individual tricked into malware | Hijack web banking session |
| IT admin's VPN credentials | Access core corporate network |
| Government worker tricked into sharing secrets | Simplify future targeted attacks |

---

### The Core Insight
```
Most breaches don't start with a zero-day exploit.
They start with:
  → A phishing email
  → A vishing call
  → A pretexting scenario
  → A USB drop in a parking lot

Technical skill gets you into systems.
Social engineering gets you the keys.
```

> Firewalls don't stop humans from clicking links.
> Antivirus doesn't stop humans from sharing passwords.
> The most hardened network in the world has a help desk that will reset a password if you sound convincing enough.

---

## Social Engineering Attack Types

### Core Requirements for Success
- **Trustworthy** — attacker appears legitimate
- **Emotional** — triggers urgency, fear, or curiosity

---

### Attack Types

#### Phishing
Most common — [3.4 billion](https://aag-it.com/the-latest-phishing-statistics/#:~:text=with%20an%20estimated%203.4%20billion%20spam%20emails%20sent%20every%20day) malicious emails sent daily.
```
Fake email: "Your account has been compromised. Click here!"
→ Victim clicks → fake login page → credentials sent to attacker
```

**Variants:**
| Type | Target |
|------|--------|
| Phishing | Mass email campaign |
| Spear phishing | Specific individual (researched) |
| Whaling | C-suite executives |
| Smishing | SMS-based phishing |
| Vishing | Voice call phishing |

---

#### Malware Downloads
Victim installs [malware](https://www.esentire.com/blog/fake-deepseek-site-infects-mac-users-with-atomic-stealer#:~:text=The%20infection%20process%20begins%20when%20the%20user%20is%20redirected) thinking it's legitimate software.

**Delivery techniques:**
- Fake CAPTCHAs that execute malicious scripts
- Malicious QR codes
- SEO poisoning — fake software sites rank above real ones in search results

---

#### Deepfakes
AI-generated video/audio impersonating trusted people.

**[Real case](https://edition.cnn.com/2024/02/04/asia/deepfake-cfo-scam-hong-kong-intl-hnk):**

```
Hong Kong 2024:
Finance worker received deepfake video call
"Boss" requested urgent wire transfer
Result: $25 million lost
```

---

#### Impersonation
No deepfake needed — just confidence and a phone call.

**Common scenario:**
```
Attacker calls: "Hi, this is IT support. We need to take
over your account for a quick system repair."
Victim: complies
Result: ransomware deployed on corporate network
```
> Black Basta [ransomware](https://thehackernews.com/2024/12/black-basta-ransomware-evolves-with.html#:~:text=make%20initial%20contact%20with%20prospective%20targets) group used exactly this technique at scale.

---

### Other Attack Vectors (SOC Analyst Must Know)

| Attack | Description |
|--------|-------------|
| **USB drop** | Malicious USB left in parking lot — curious employee plugs it in |
| **Physical intrusion** | Tailgating into secure areas |
| **Insider threats** | Employee intentionally or unknowingly assists attacker |
| **Fake job offers** | Malicious attachments in "offer letters" — targets high-value employees |

---

### SOC Relevance
> Social engineering is your most common initial access vector. When you see a phishing alert, an unusual login, or a help desk ticket for an "urgent password reset" — social engineering is your first hypothesis. Always ask: was a human manipulated to get here?

--- 

## Defending Against Human-Targeted Attacks

### Two-Layer Defense Strategy

```
Layer 1: Mitigation  → prevent/reduce attacks reaching humans
Layer 2: Detection   → catch attacks that bypass mitigation (SOC role)
```

> No mitigation is perfect. Some attacks always get through.
> That's why both layers are non-negotiable.

---

### Mitigation Measures

| Control | What it does |
|---------|-------------|
| **Anti-phishing solution** | Blocks malicious emails before users see them — reduces SOC alert volume |
| **Antivirus / EDR** | Prevents malware execution on corporate endpoints |
| **"Trust but Verify"** | Train employees to detect deepfakes and verify suspicious requests (CEO fraud, IT impersonation) |
| **Security awareness training** | Teach phishing recognition + reinforce with simulated phishing campaigns |

---

### SOC Analyst's Role in the Chain

```
Attack arrives
    → Mitigation blocks it?  → Done
    → Mitigation bypassed?   → SOC detects + investigates + stops it
```

**Why mitigation knowledge matters for SOC analysts:**
- You see which attacks bypass controls → you identify gaps
- You can propose new rules/tools to prevent repeat attacks
- Approved mitigations reduce your alert queue over time
- Better mitigation = SOC focuses on sophisticated threats, not noise

---

### Practical Mindset
> Don't just react to alerts — think about why the alert exists. If you're seeing 50 phishing alerts a day from the same domain, the answer isn't to triage all 50 manually. The answer is a block rule + a mitigation proposal to management. Good analysts make themselves less busy by fixing root causes.
