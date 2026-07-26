## Pyramid of Pain – Hash-Based Detection (Round 1)

<img width="1916" height="850" alt="Screenshot 2026-07-26 210359" src="https://github.com/user-attachments/assets/bb21801b-b634-42d3-8452-d2ef62c739df" />

The scenario kicks off with an email from Sphinx, the pentester, explaining the setup: he'll try executing malware samples on a simulated compromised workstation, and my job is to configure PicoSecure's tools to catch them. It's framed as an iterative back-and-forth — every time I successfully block a sample, he escalates to something harder to detect. First sample up: `sample1.exe`.

---

<img width="1917" height="752" alt="Screenshot 2026-07-26 210415" src="https://github.com/user-attachments/assets/1f1e5195-3224-40b4-bce7-62e707eaf908" />

Rather than guessing what the file does, the right move is to run it through PicoSecure's **Malware Sandbox** first. This tool executes the suspicious file inside an isolated Windows 10 (v1803) VM and returns a full behavioral report instead of just a static file scan.

---
<img width="1911" height="858" alt="Screenshot 2026-07-26 210426" src="https://github.com/user-attachments/assets/21c6a1a7-e7b6-4351-9686-ac2aac0a4f45" />

**What the sandbox report showed for `sample1.exe`:**
- File type: PE32+ executable (GUI), x86-64, for MS Windows
- Automatically tagged: `Trojan.Metasploit.A`
- Malicious behavior flagged: **METASPLOIT was detected**
- Suspicious behavior flagged: connects out to an unusual port
- Info-level behavior: reads machine GUID from registry, checks LSA protection, reads computer name, checks supported languages
- Hashes provided: MD5, SHA1, and SHA256

---
<img width="1892" height="838" alt="Screenshot 2026-07-26 210454" src="https://github.com/user-attachments/assets/e1cfd587-b9aa-4b07-8bf5-3f28676f8428" />

Since the sandbox confirmed this was malicious and handed over unique hash values, the natural next step is to go to **IOC Management → Manage Hashes** and manually add one of these hashes (MD5 in this case) to the **Hash Blocklist**. Submitting it updates PicoSecure's EDR signatures automatically.

---
<img width="1906" height="768" alt="Screenshot 2026-07-26 210508" src="https://github.com/user-attachments/assets/f3fca2ae-0295-44cb-bc7e-4619911ec4c1" />

**Result:** the next time `sample1.exe` tried to run, PicoSecure blocked it purely based on that hash match — no need to analyze behavior again, just a signature lookup.

**Why hashes are the weakest indicator (Pyramid of Pain logic):**
- Hashes sit at the very bottom of the pyramid — cheap to generate, but trivially easy for an attacker to invalidate.
- A hash is tied to the *exact* byte content of a file — change even a single bit, and the hash changes completely.
- This showed up almost immediately: Sphinx replied that he simply **recompiled the malware** into `sample2.exe`, generating a brand-new hash, and it executed without issue.
- Takeaway: hash-based detection is useful as a first line of defense and for known, unmodified threats, but it doesn't hold up against even minimal effort from the attacker to evade it — climbing the pyramid (to IPs, domains, tool artifacts, or TTPs) is what actually raises their cost of operating.

---

## Pyramid of Pain – IP Address Detection (Round 2)

<img width="1908" height="722" alt="Screenshot 2026-07-26 211336" src="https://github.com/user-attachments/assets/145b8d55-9d7a-4024-a1c0-42401b9b7f95" />

Since hashes were a dead end, the next round of `sample2.exe` needed a different indicator. Running it through the Malware Sandbox again pulled up its **Network Activity** rather than static file properties this time:

<img width="1225" height="612" alt="Screenshot 2026-07-26 211409" src="https://github.com/user-attachments/assets/953e61c6-c525-455b-a2ec-de50a2c75ad7" />

- **1 HTTP request** — a GET to `154.35.10.113:4444`, hitting a suspicious-looking randomized URL path
- **3 TCP/UDP connections**, including:
  - `154.35.10.113:4444` — attributed to **Intrabuzz Hosting Limited** (no domain, just a raw IP — a hosting provider, not a legitimate cloud/CDN)
  - Two connections to `40.97.128.3:443` and `40.97.128.4:443` — attributed to **Microsoft Corporation** (almost certainly legitimate background traffic, not part of the malicious behavior)

The standout here is the connection to Intrabuzz Hosting on port `4444` — a classic port associated with Metasploit/reverse-shell C2 traffic, and not tied to any real domain. That's the malware's actual command-and-control address.

---
<img width="1902" height="790" alt="Screenshot 2026-07-26 211558" src="https://github.com/user-attachments/assets/2e499464-708e-48c4-8b30-5e46768c7728" />

**Remediation step:** rather than blocking a hash, this called for the **Firewall Rule Manager** under IOC Management:
- Type: **Egress**
- Source IP: `Any`
- Destination IP: `154.35.10.113` (the C2 server)
- Action: **Deny**

Saving this rule blocked outbound traffic to that IP, which stopped `sample2.exe` from being able to reach its C2 server and complete its callback.

<img width="1872" height="808" alt="Screenshot 2026-07-26 211740" src="https://github.com/user-attachments/assets/3800465f-9c61-4b54-b437-c049cdfde3f7" />

**Why this is one step up the pyramid:**
- IP addresses are a bit more durable an indicator than a hash — the attacker's malware itself didn't need to change, just its network destination.
- But IPs are still relatively cheap for an attacker to burn through. As Sphinx pointed out in his follow-up email, this defense isn't "bulletproof" — a motivated adversary can simply spin up a new server on a new public IP (e.g. by signing up with a different cloud provider) and be back in business.
- Sure enough, that's exactly what happened: Sphinx moved `sample3.exe` to a **new IP address** with "plenty more backups to failover" in case those get blocked too — meaning the next detection method needs to target something less disposable than a single IP (like a domain, a certificate, or a behavioral pattern).

---

## Pyramid of Pain – Domain Detection (Round 3)
<img width="1868" height="732" alt="Screenshot 2026-07-26 212602" src="https://github.com/user-attachments/assets/27cc7572-cf26-407f-8ea0-66a1d72195c5" />


With `sample3.exe`, the sandbox network activity showed the malware had moved past a raw IP and started resolving an actual domain:

<img width="1235" height="781" alt="Screenshot 2026-07-26 212620" src="https://github.com/user-attachments/assets/da8343f2-9c98-4637-a57e-01c75fcc68fa" />

- **2 HTTP requests**, both hitting `emudyn.bresonicz.info`:
  - `GET http://emudyn.bresonicz.info:1337/kzn293la` — looks like a C2 check-in/beacon
  - `GET http://emudyn.bresonicz.info/backdoor.exe` — a second-stage payload download, literally named `backdoor.exe`
- **4 TCP/UDP connections**, three of which go to `62.123.140.9` (the resolved IP for `emudyn.bresonicz.info`, hosted on **Xplorita Cloud Services**) — including one from a *second* process, `backdoor.exe` (PID 2712), confirming that sample3 actually drops and runs a follow-up binary
- **2 DNS requests** resolving both `services.microsoft.com` (legitimate) and `emudyn.bresonicz.info` (the malicious one)

Since the IP itself could get swapped out easily but the domain name was consistent across the beacon and the payload download, the right move here was to escalate from the Firewall Rule Manager to the **DNS Rule Manager**:
<img width="1858" height="837" alt="Screenshot 2026-07-26 212910" src="https://github.com/user-attachments/assets/11f6839a-a33b-417e-b91c-d8ef8089f23d" />

- Rule Name: `backdoor`
- Category: **Malware**
- Domain Name: `emudyn.bresonicz.info`
- Action: **Deny**

Denying the domain automatically covers all its subdomains too, which is a nice built-in benefit over IP blocking. This stopped `sample3.exe` from resolving or reaching its C2 domain at all.

**Why domains rank higher on the pyramid than IPs:** a domain is a bit more expensive for an attacker to cycle through than a raw IP — burning a domain means buying a new one, registering it, and updating DNS records, rather than just spinning up a new server. Sphinx's own follow-up email confirmed this: he admitted it now takes actual effort ("purchase and register some new domain names and modify DNS records") to bypass, and noted this is exactly the kind of friction that causes less-motivated attackers to give up and move to an easier target.

<img width="1233" height="630" alt="Screenshot 2026-07-26 212918" src="https://github.com/user-attachments/assets/9a8df743-d299-4137-85bc-b0ff475c1530" />

However, he also flagged that for the next sample, **hashes, IPs, and domains won't work anymore** — pushing the detection challenge up to the next tier of the pyramid: **host-based artifacts**, meaning whatever changes or traces the malware leaves behind on the compromised system itself (files dropped, registry changes, processes created, etc.) rather than anything network-based.

---
## Pyramid of Pain – Host Artifact Detection via Sigma Rule (Round 4)
<img width="1895" height="712" alt="Screenshot 2026-07-26 213613" src="https://github.com/user-attachments/assets/b8a1c1f4-88ad-4c57-a39e-3647cb923758" />

With hashes, IPs, and domains all off the table, `sample4.exe` needed a detection method based on what the malware actually *does* to the host system — a host-based artifact rather than a network one.

<img width="1207" height="727" alt="Screenshot 2026-07-26 213641" src="https://github.com/user-attachments/assets/47804aea-1715-4fe8-8541-b8263babca62" />

**Network activity for sample4.exe** (for context, since it still shows the domain-cycling pattern):
- 2 HTTP GETs to `cranes0ft.iniware.xyz` (a brand-new domain from `sample3.exe`'s domain, confirming domains keep rotating): one beacon-style request to `:1337/ab9z83ja`, one payload download `/backdoor.exe`
- Connections to `102.23.20.118`, hosted on **Xplorita Cloud Services** — the same hosting provider as sample3, just a new domain/IP pointing at it
- A second process, `backdoor.exe`, again dropped and run after the initial download — consistent with the earlier samples

**Registry Activity** is where the real detection opportunity showed up:
- `sample4.exe` (PID 3806) performed a **write** to `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`, setting `DisableRealtimeMonitoring = 1`
- This is a very deliberate, malicious action — the malware disabling Windows Defender's real-time protection so the rest of its activity goes unnoticed
- The other two registry events in the log (explorer.exe writing an `EnableBalloonTips` value, notepad.exe reading a file association) are unrelated, everyday system noise — useful to rule out as false leads
<img width="1216" height="540" alt="Screenshot 2026-07-26 213645" src="https://github.com/user-attachments/assets/06035565-80d0-4009-9b1d-1ab54c73d66f" />

---
<img width="761" height="365" alt="Screenshot 2026-07-26 214303" src="https://github.com/user-attachments/assets/65e59bed-47d3-4fce-b8c9-e58084a9bcad" />
<img width="772" height="800" alt="Screenshot 2026-07-26 214337" src="https://github.com/user-attachments/assets/0d10c211-3df0-450f-ac62-f80d433d7a63" />
<img width="1057" height="550" alt="Screenshot 2026-07-26 214342" src="https://github.com/user-attachments/assets/d1b79a03-07ec-416d-ab72-93d4e1a153e6" />

**Detection built via PicoSecure's Sigma Rule Builder:**
- Target: **Registry Modifications**
- Registry Key: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`
- Registry Name: `DisableRealtimeMonitoring`
- Value: `1`
- Mapped ATT&CK ID: **Defense Evasion (TA0005)** — PicoSecure requires every Sigma rule to map to a MITRE ATT&CK tactic so the SOC has context for triage
- The generated Sigma rule watches for **Sysmon Event ID 4663** (object access) on that specific registry key/value combination, and flags the false-positive scenario as legitimate admin changes to Defender settings

This got validated and deployed straight to PicoSecure's SIEM, successfully catching `sample4.exe` based on its behavior rather than any network or file-based indicator.

**Why this is a meaningful jump up the pyramid:** this is now a **host artifact / tool artifact** level detection — it's tied to something the malware's *technique* needs to do (disable Defender) rather than something disposable like a hash, IP, or domain. Sphinx's reaction confirmed the cost: he had to spend real time and budget getting his team to rework the tool's methodology just to get around it, and admitted plenty of less-funded threat actors would've given up and moved to an easier target by this point.

<img width="1242" height="693" alt="Screenshot 2026-07-26 214401" src="https://github.com/user-attachments/assets/a3cf2318-3e4a-4d19-a09b-913cc397832d" />

For the next round (`sample5.exe`), Sphinx changed strategy entirely — moving all the "heavy lifting" and instructions to his own back-end server so he can freely swap protocols and artifacts on the fly. He handed over only a 12-hour outgoing connection log from the victim machine, meaning the next detection has to come from spotting an abnormal *pattern* of behavior (like beacon timing or connection frequency) — the top tier of the pyramid: **TTPs (Tactics, Techniques, and Procedures)**.

---

## Pyramid of Pain – Behavioral Beacon Detection via Sysmon (Round 5)

<img width="1850" height="841" alt="Screenshot 2026-07-26 214942" src="https://github.com/user-attachments/assets/fcd3c459-66d3-4b1a-ac67-68c9fd0d0289" />

`sample5.exe` was Sphinx's attempt to get around domain/artifact detection by moving all the "heavy lifting" to his back-end server, so he could freely swap protocols and change what gets left on the host. That meant this round couldn't lean on any single fixed indicator — the actual clue was a *behavioral pattern* buried in the outgoing connection log he handed over.


Scanning through `outgoing_connections.log`, one destination stood out immediately: `51.102.10.19:443`. It showed up again and again, almost every 30 minutes, always with the exact same payload size — **97 bytes**. Everything else in the log (various other IPs, wildly different byte sizes, no consistent timing) looked like normal, organic traffic. A connection that repeats on a fixed interval with an identical, tiny payload size is a textbook beacon signature — the malware "checking in" with its C2 server rather than transferring real data.
<img width="1872" height="617" alt="Screenshot 2026-07-26 215006" src="https://github.com/user-attachments/assets/319e74f3-651d-46d4-8988-fc1135919a20" />

Running `sample5.exe` through the sandbox confirmed the theory. Its network activity showed:
<img width="1231" height="782" alt="Screenshot 2026-07-26 215027" src="https://github.com/user-attachments/assets/f1100b2e-b7ea-4d79-8ad7-4dbd2417a353" />

- An initial GET to `bababa10la.cn` that pulls down `beacon.bat`
- That script then fires off **hundreds of POST requests** (402 HTTP requests total) to `https://bababa10la.cn/keep-alive?hostname=WK102`, all going to the same IP `51.102.10.19` on port 443
- Domain and IP both differ from every previous sample — exactly as Sphinx warned, since this sample regenerates its infrastructure freely

**Detection built via the Sigma Rule Builder**, this time targeting **Network Connections** instead of registry or process events:
<img width="771" height="791" alt="Screenshot 2026-07-26 215553" src="https://github.com/user-attachments/assets/bb1a78a3-376e-421a-ab65-4e823595e304" />

- Remote IP: `Any` (deliberately not tied to a specific IP, since that keeps changing)
- Remote Port: `Any`
- Size: `97` bytes
- Frequency: `1800` seconds (30 minutes)
- Mapped ATT&CK ID: **Command and Control (TA0011)**

---
<img width="1076" height="580" alt="Screenshot 2026-07-26 215604" src="https://github.com/user-attachments/assets/0134699d-a049-48be-8e29-4f7bf307a16e" />

The generated Sigma rule watches Sysmon **Event ID 3** (network connection) for any connection matching that specific size-and-frequency fingerprint, regardless of the destination IP or port. This is what finally caught `sample5.exe`.

**Why this is a real jump up the pyramid:** this detection isn't tied to any single artifact the attacker can casually swap out — it's tied to the *shape* of the malware's communication behavior. Sphinx's reaction made the cost obvious: catching this meant he had to scrap his tool entirely and build something new from scratch, plus retrain himself on it. He was explicit that this kind of repeated investment isn't sustainable — most attackers would cut their losses and pick an easier target with weaker detection long before reaching this point.

<img width="1232" height="531" alt="Screenshot 2026-07-26 215615" src="https://github.com/user-attachments/assets/90f7a1f6-859b-4011-8ea5-e6662c101764" />

For his last attempt, `sample6.exe`, Sphinx said even tool and artifact-level detection won't be enough — this next one requires catching something at the very top of the pyramid: his actual **TTPs (Tactics, Techniques, and Procedures)** — the underlying playbook of actions he tends to run once he has remote access, which is much harder for an attacker to change since it's closer to habit/muscle memory than to swappable tooling. He attached a `commands.log` of his past sessions to use as a baseline for spotting that pattern.

---

## Pyramid of Pain – TTP Detection (Final Round)
<img width="1877" height="662" alt="Screenshot 2026-07-26 220021" src="https://github.com/user-attachments/assets/0c12ebc4-1de9-4042-bd46-54f5d2c29acc" />

For the last sample, Sphinx handed over `commands.log` — a record of the actual commands he tends to run once he gets remote access on a victim. Looking through it, every single command followed the exact same pattern:
```
dir c:\ >> %temp%\exfiltr8.log
dir "c:\Documents and Settings" >> %temp%\exfiltr8.log
dir "c:\Program Files" >> %temp%\exfiltr8.log
dir d:\ >> %temp%\exfiltr8.log
net localgroup administrator >> %temp%\exfiltr8.log
ver >> %temp%\exfiltr8.log
systeminfo >> %temp%\exfiltr8.log
ipconfig /all >> %temp%\exfiltr8.log
netstat -ano >> %temp%\exfiltr8.log
net start >> %temp%\exfiltr8.log
```
The individual commands themselves (`dir`, `systeminfo`, `netstat`, etc.) are completely mundane on their own — any admin runs these all the time. But what's consistent across *every* session is that all of the output gets redirected into the same file: `%temp%\exfiltr8.log`. That's the actual procedure — this is his personal "recon and stage the loot" habit, and it's the one thing he can't easily shake even if he swaps out the malware, the domain, or the whole toolset.


Running `sample6.exe` through the sandbox confirmed the pattern held:
<img width="1893" height="420" alt="Screenshot 2026-07-26 220042" src="https://github.com/user-attachments/assets/f121083f-1086-480d-b628-311b27fc2638" />

- Process tree: `sample6.exe` spawns three separate `cmd.exe` children
  <img width="1215" height="625" alt="Screenshot 2026-07-26 220103" src="https://github.com/user-attachments/assets/03ffe84b-dac4-4dcd-8c39-216174a7e0a8" />

- One of those `cmd.exe` processes drops a file: `%temp%\exfiltr8.log` — exactly matching the staging filename from the command log


**Detection built via the Sigma Rule Builder**, targeting **File Creation and Modification** rather than registry or network events:
<img width="801" height="810" alt="Screenshot 2026-07-26 220142" src="https://github.com/user-attachments/assets/c376c82a-e35a-4065-b10a-75e9529e1586" />

- File Path: `%temp%`
- File Name: `exfiltr8.log`
- Mapped ATT&CK ID: **Exfiltration (TA0010)**

This caught `sample6.exe` based purely on his consistent staging behavior — the filename and location he habitually dumps recon output into, regardless of what malware or infrastructure he's using that day.
<img width="1252" height="395" alt="Screenshot 2026-07-26 220239" src="https://github.com/user-attachments/assets/6febf995-8a1d-4a4c-9c74-231de1cf9f2d" />

**Why this is the top of the pyramid:** TTPs are the hardest thing for an attacker to change because they're closer to muscle memory/tooling-independent habits than to any single artifact. Sphinx's final email confirmed the engagement was over at this point — he'd been chased from hash detection, all the way up through IPs, domains, host artifacts, tools, and finally his own behavioral technique, and said retraining himself entirely wasn't worth it anymore. That's the whole point of the Pyramid of Pain: the higher up the indicator, the more it costs the attacker to work around it, until eventually it's cheaper for them to just give up and find a softer target.
