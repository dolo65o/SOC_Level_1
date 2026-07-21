## Cyber Kill Chain – Introduction & Threat Modelling

Originating from the military, the term **"Kill Chain"** describes the various stages of an attack. In cybersecurity, it refers to the methodology or path that attackers — hackers, APTs, etc. — follow to approach and intrude on a target.

For example, an attacker scanning a system, exploiting a web vulnerability, and then escalating privileges would collectively form a "Kill Chain."

The goal of understanding an attacker's Kill Chain is to put defensive measures in place — either to pre-emptively protect a system or to disrupt the attacker mid-attempt.

### Threat Modelling

Threat modelling is a series of steps aimed at improving a system's security by identifying risk. It generally breaks down into four stages:

1. **Identify** what systems/applications need securing and what role they play (e.g. is it critical to operations, or does it hold sensitive data like payment info or addresses?).
2. **Assess** what vulnerabilities and weaknesses these systems have and how they could be exploited.
3. **Plan** a course of action to secure the systems against the vulnerabilities found.
4. **Implement policies** to prevent those vulnerabilities from recurring (e.g. adopting a secure SDLC, or running phishing-awareness training for employees).

Threat modelling gives a high-level overview of an organisation's IT assets (any piece of software or hardware) and the procedures needed to resolve vulnerabilities in them. The **Cyber Kill Chain (CKC)** framework itself encourages threat modelling by helping identify potential attack surfaces and how they might be exploited.

Other frameworks used specifically for threat modelling include **STRIDE**, **DREAD**, and **CVSS**.

---

## Unified Kill Chain (UKC)

Paul Pols' [Unified Kill Chain](https://www.unifiedkillchain.com/assets/The-Unified-Kill-Chain.pdf), published in 2017, was designed to **complement** — not compete with — other cybersecurity kill chain frameworks like Lockheed Martin's and MITRE's ATT&CK.

The UKC defines **18 phases** of an attack, covering everything from reconnaissance all the way through to data exfiltration and understanding the attacker's motive. 

<img width="1230" height="794" alt="18 phases" src="https://github.com/user-attachments/assets/5aa1576f-7827-46bd-9bbe-c3f764c759bf" />

### UKC vs. Other Frameworks

| Benefit of the UKC | How Other Frameworks Compare |
|---|---|
| Modern — released in 2017, updated in 2022 | Some frameworks, like MITRE's, date back to 2013, when the threat landscape looked very different |
| Extremely detailed — 18 official phases | Other frameworks typically cover only a small handful of phases |
| Covers the entire attack lifecycle: recon, exploitation, post-exploitation, and attacker motivation | Other frameworks tend to cover only a limited slice of the attack |
| Reflects realistic attack behavior — stages often repeat (e.g. after exploiting a machine, an attacker returns to recon to pivot to another system) | Other frameworks don't account for attackers moving back and forth between phases |

---

## Unified Kill Chain – Initial Foothold Phases

This section of the UKC covers the phases an attacker moves through to gain and secure access to a network. What makes the UKC useful here is that it doesn't treat these as a strict one-way sequence — attackers loop back into recon and weaponization repeatedly, and often layer several techniques together rather than relying on just one.

### Gathering Intel Before Touching Anything (Reconnaissance – [TA0043](https://attack.mitre.org/tactics/TA0043/))
Everything the attacker does later depends on what they learn here. This phase isn't just "scan the target" — it's building a profile that feeds every later stage:
- Which systems/services are exposed → shapes what to weaponize and exploit
- Employee names/contact lists → material for phishing or social engineering
- Any exposed credentials → useful later for pivoting or initial access
- Network layout → tells the attacker what else might be reachable once they're in

### Getting the Tools Ready (Weaponization – [TA0001](https://attack.mitre.org/tactics/TA0001/))
This is the setup work — standing up infrastructure like a C2 server or something that can catch reverse shells and push payloads. Think of it as building the delivery pipeline before actually using it.

### Manipulating People Instead of Systems (Social Engineering – [TA0001](https://attack.mitre.org/tactics/TA0001/))
Same MITRE tactic ID as Weaponization, but the target here is humans, not machines:
- Tricking someone into opening a malicious attachment
- Fake login pages to harvest credentials
- Impersonation — calling in to request a password reset, or posing as a utility worker to get physical access

### Actually Breaking In (Exploitation – [TA0002](https://attack.mitre.org/tactics/TA0002/))
The UKC is specific here: exploitation means abusing a vulnerability to get **code execution**, not just finding a flaw. Examples: uploading a reverse shell to a vulnerable web app, hijacking an automated script, or triggering code execution through a web vulnerability.

### Making Sure You Can Come Back (Persistence – [TA0003](https://attack.mitre.org/tactics/TA0003/))
Once in, the goal shifts to not losing that access. Common approaches:
- Installing a service that re-establishes access later
- Registering the compromised system with a C2 server for on-demand control
- Planting a backdoor that triggers on a specific event (e.g. fires a reverse shell whenever an admin logs in)

### Staying Under the Radar (Defence Evasion – [TA0005](https://attack.mitre.org/tactics/TA0005/))
Arguably the most valuable phase to study from a defender's perspective — it's specifically about how attackers slip past firewalls, AV, and IDS. Understanding this phase is less about the attack itself and more about what it reveals for improving detection later.

### Taking the Wheel Remotely (Command & Control – [TA0011](https://attack.mitre.org/tactics/TA0011/))
This is where the Weaponization groundwork pays off — the C2 channel goes live and the attacker can run commands, exfiltrate data/credentials, or use the compromised host as a launchpad to reach further into the network.

### Reaching What Isn't Directly Exposed (Pivoting – [TA0008](https://attack.mitre.org/tactics/TA0008/))
Not every valuable system sits on the internet — a lot of the good stuff is tucked away internally, often with weaker security. Pivoting is how an attacker uses one compromised, internet-facing box (like a web server) as a stepping stone to reach those internal systems.

---

## Unified Kill Chain – "Through" Phase (Network Propagation)

Once an attacker has a working foothold, the "Through" stage kicks in — this is the phase where they stop knocking on the door and start moving around inside the house. If they can't reach their end goal directly from that first compromised system, they'll turn it into a base of operations and start expanding outward. Here's how that expansion typically plays out:

**Pivoting ([TA0008](https://attack.mitre.org/tactics/TA0008/))** — The first compromised system becomes more than just an entry point; it turns into a staging ground. It acts as a tunnel connecting the attacker's command infrastructure to the internal network, and later on, it doubles as the launch point for pushing out further malware and backdoors.

**Discovery ([TA0007](https://attack.mitre.org/tactics/TA0007/))** — Before moving further, the attacker needs a map. This is where they quietly build out a picture of the environment: what user accounts exist, what permissions those accounts have, which apps and software are running, browsing activity, shared folders and files, and general system configuration. Essentially, they're learning the terrain before deciding where to go next.

**Privilege Escalation ([TA0004](https://attack.mitre.org/tactics/TA0004/))** — Armed with what Discovery uncovered, the attacker looks for a way to gain stronger permissions on the pivot system — usually by chaining together misconfigurations or account weaknesses. The end goal is landing one of these access levels:

| Target Access Level | What It Means |
|---|---|
| SYSTEM / ROOT | Full control of the machine |
| Local Administrator | Admin rights on that specific system |
| Admin-like user account | Not technically admin, but functions like one |
| Specific-access account | Limited but useful access to a particular function |

**Execution ([TA0002](https://attack.mitre.org/tactics/TA0002/))** — With better access secured, the attacker starts actually running their malicious code on the pivot system — things like remote trojans, C2 scripts, malicious links, or scheduled tasks. The purpose here isn't a one-off action; it's about keeping a recurring presence alive on the system.

**Credential Access ([TA0006](https://attack.mitre.org/tactics/TA0006/))** — This runs in parallel with privilege escalation. The attacker tries to grab real account credentials — through keylogging, credential dumping, or similar techniques. Stolen legitimate credentials are valuable because they let the attacker blend in as a "normal" user instead of tripping alarms as an obvious intruder.

**Lateral Movement ([TA0008](https://attack.mitre.org/tactics/TA0008/))** — Now with elevated access and real credentials in hand, the attacker starts hopping from system to system across the network, working toward their actual objective. The quieter and more low-profile the technique, the longer they stay undetected.

---

## Unified Kill Chain – "Out" Phase (Achieving the Objective)

This is the payoff stage — the attacker now has access to critical assets and is positioned to actually accomplish what they set out to do. Everything here revolves around breaking one or more pillars of the **CIA triad** (Confidentiality, Integrity, Availability).

**Collection ([TA0009](https://attack.mitre.org/tactics/TA0009/))** — Before anything leaves the network, the attacker rounds up the data that matters: files on drives, browser data, audio, video, email. Just gathering this already breaks confidentiality, and it sets up the next move — getting it out.

**Exfiltration ([TA0010](https://attack.mitre.org/tactics/TA0010/))** — The collected data gets pushed out of the network, usually compressed and encrypted first to slip past detection tools. This is also where all that earlier C2 infrastructure finally pays off — the tunnel built during Command & Control becomes the exit route for the stolen data.

**Impact ([TA0040](https://attack.mitre.org/tactics/TA0040/))** — If the goal instead is to hit integrity or availability rather than steal quietly, the attacker shifts to disruption: wiping disks, locking accounts, deploying ransomware, defacing systems, or launching denial-of-service attacks. The point isn't stealth anymore — it's causing operational damage.

**Objectives** — Ultimately, all of this serves whatever the attacker's underlying motive was. A financially driven attacker might deploy ransomware and demand payment. Someone after reputational damage might instead leak sensitive company data publicly rather than encrypt it. The tactics used in Impact and Exfiltration are just the means — the objective is the real "why" behind the entire attack chain.
