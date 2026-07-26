## APT Investigation – Mapping Attacker Behavior to MITRE ATT&CK (Eviction Room)

> [This link](https://static-labs.tryhackme.cloud/sites/eviction/) to check out the MITRE ATT&CK Navigator layer for the APT group

This scenario follows an analyst, Sunny, tracing an APT's intrusion into E-corp step by step, matching each stage of the attack to a specific MITRE ATT&CK technique. It's basically a walkthrough of the attack lifecycle from recon all the way to attempted exfiltration.

**1. Recon + Initial Access — Spearphishing Link**
The APT used the same technique to do double duty here: **spearphishing links**. Instead of just scraping public info, a phishing link can silently gather intel on the target (what browser/OS they use, whether the link was clicked, etc.) while *also* being the actual entry point if the victim clicks through to a malicious page. It's efficient for the attacker — one action, two ATT&CK phases covered.

**2. Resource Development — Compromising Email Accounts**
Before pushing further, the APT works on building out its "kit" — and one common resource to acquire is **compromised email accounts**. Owning a real, trusted email account makes future phishing far more convincing (it's coming from someone the victim actually knows) and gives the attacker a legitimate-looking base for follow-on social engineering.

**3. Execution — Malicious File and Malicious Link**
Once inside, the APT relies on tricking a user into doing something on its behalf — this falls under "User Execution." The two flavors here are:
- **Malicious file** — e.g. an infected attachment or document the victim is convinced to open
- **Malicious link** — a link that, when clicked, runs code or redirects to something that does

**Execution (deeper) — PowerShell and Windows Command Shell**
Once a file or link triggers execution, the actual code usually runs through a **scripting interpreter** rather than as a standalone binary — this is stealthier since these interpreters are legitimate, built-in Windows tools. The two most common ones attackers abuse are:
- **PowerShell** — a scripting language/shell built into Windows, extremely powerful for system administration (and just as powerful for attackers, since it's trusted and rarely blocked outright)
- **Windows Command Shell (cmd.exe)** — the classic Windows command-line interpreter

**Persistence — Registry Run Keys**
Sunny found obfuscated scripts modifying the registry to survive reboots — this points to **Registry Run Keys**. These are specific registry locations (e.g. `HKCU\...\Run` or `HKLM\...\Run`) where any program listed automatically launches whenever the user logs in. It's one of the simplest and most common persistence mechanisms because it doesn't require installing an actual service — just adding a line to a config-like key.

**Defense Evasion — Rundll32 (Proxy Execution)**
The APT was found executing **Rundll32**, a legitimate Windows system binary normally used to run functions stored inside DLL files. Attackers abuse it for **proxy execution** — running their malicious code *through* a trusted, signed Windows binary so it doesn't look suspicious to security tools that might flag an unknown or unsigned executable running directly.

**Discovery — Network Sniffing (via tcpdump)**
Finding **tcpdump** on a compromised host is a strong indicator of **network sniffing** — passively capturing network traffic to learn about the internal network: what hosts exist, what protocols are in use, and potentially catching credentials sent in cleartext. It's a discovery technique because the attacker isn't attacking yet, just quietly mapping the environment.

**Lateral Movement — SMB / Windows Admin Shares**
To move from one compromised machine to others, the APT exploited **SMB (Server Message Block)** and **Windows Admin Shares**. These are built-in Windows file-sharing features (things like `\\hostname\C$` or `\\hostname\ADMIN$`) that, if an attacker has valid credentials, let them access and execute things on other machines on the network without needing any new exploit — just legitimate admin functionality turned against the organization.

**Collection — SharePoint**
The APT's real target for stealing intellectual property was **SharePoint**, a Microsoft platform organizations use to store and collaborate on internal documents. Since it's a centralized hub for exactly the kind of sensitive files (designs, contracts, source code, etc.) an APT would want, it's a natural collection target once they have enough access to browse or query it.

**Exfiltration — External Proxy and Multi-hop Proxy**
Finally, when trying to sneak the stolen data out to their C2 server, the APT's likely approach was routing through:
- **External proxy** — bouncing traffic through a third-party server not owned by the victim, making the traffic look like it's going somewhere unrelated to the actual C2
- **Multi-hop proxy** — chaining multiple proxy servers together, so tracing the traffic back to its true origin (the attacker's real C2) becomes much harder for defenders

**Overall takeaway:** this exercise is really a tour through the ATT&CK kill chain end-to-end — recon → resource development → initial access → execution → persistence → defense evasion → discovery → lateral movement → collection → exfiltration — where each stage has one or two "signature" techniques to watch for, and spotting them early enough (as Sunny did) is what let E-corp stop the APT before the data actually left the network.
