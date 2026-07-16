## EDR — Endpoint Detection and Response

### What is EDR?
Security solution providing **deep-level protection for endpoints** regardless of network location.
Critical for remote work environments where endpoints operate outside perimeter protection.

---

### Popular EDR Solutions
`CrowdStrike Falcon` `SentinelOne ActiveEDR` `Microsoft Defender for Endpoint` `OpenEDR` `Symantec EDR`

---

### 3 Pillars of EDR

#### 1. Visibility
Collects detailed endpoint data and presents it in structured format with full context.

**Data collected:**
| Type | Examples |
|------|---------|
| Process modifications | New processes spawned, parent-child relationships |
| Registry modifications | Keys created, modified, deleted |
| File/folder modifications | Files created, renamed, deleted |
| User actions | Logins, privilege changes, command execution |

**Key capability:** Full **process tree** with complete activity timeline
- Each node = a process
- Lines = parent-child relationships
- Click any process → see all network connections, registry changes, file changes

#### 2. Detection
Wins over traditional antivirus on every dimension.

| Capability | Description |
|-----------|-------------|
| Signature-based | Matches known malware signatures |
| Behavior-based | Detects unexpected user/process activity |
| ML baseline deviation | Flags anything deviating from normal behavior |
| Fileless malware | Detects threats living only in memory |
| Custom IOCs | Feed your own indicators for targeted detection |
| MITRE ATT&CK mapping | Each detection mapped to Tactic + Technique |

#### 3. Response
Take action directly from central EDR console — no need to physically access endpoints.

**Available actions:**
- Isolate entire endpoint from network
- Terminate malicious process
- Quarantine suspicious files
- Remote shell — execute commands on host directly
- And more depending on the EDR solution

---

### Important Limitation
> EDR is a **host-only** solution.
> It does not detect network-level threats.
> Combine with SIEM + NDR/IDS for full coverage.

---

## AV vs EDR — Detailed Comparison

### The Analogy
```
AV  = Airport immigration check
      → Matches passports against known criminal database
      → Misses unknown threats with clean history

EDR = Security officers inside the airport
      → Constantly monitor CCTV + motion sensors
      → Watch behavior: restricted areas? unattended bags? suspicious movement?
      → Can act AND alert management with full incident details
```

---

### Scenario: Phishing → Macro → PowerShell → Payload → RCE

| Step | What Happens | AV Response | EDR Response |
|------|-------------|-------------|--------------|
| **1** | Phishing email with malicious Word macro downloaded | Does nothing — no prior signature | Logs file download, starts monitoring |
| **2** | User opens the document | Does nothing — winword.exe is legitimate | Records winword.exe execution, keeps monitoring |
| **3** | Macro silently spawns PowerShell | Does nothing — no prior macro signature | **Flags** unusual parent-child: winword.exe → PowerShell.exe |
| **4** | Obfuscated PowerShell downloads second-stage payload | Typically misses obfuscated scripts | **Flags** obfuscated script execution |
| **5** | Payload injected into legitimate svchost.exe | No memory injection monitoring | **Detects** process injection into svchost.exe |
| **6** | Attacker gains remote access | No network-level visibility | **Flags** svchost.exe making unexpected outbound connection |
| **Final** | — | May mark as clean — breach undetected | Alert generated with **full attack chain** + response options available |

---

### Key EDR Advantages Over AV

| Capability | AV | EDR |
|-----------|-----|-----|
| Known malware signatures | ✓ | ✓ |
| Unknown/new malware | ✗ | ✓ behavioral detection |
| Process relationship monitoring | ✗ | ✓ parent-child tree |
| Obfuscated script detection | ✗ | ✓ |
| Memory injection detection | ✗ | ✓ |
| Network behavior of processes | ✗ | ✓ |
| Full attack chain visibility | ✗ | ✓ |
| Org-wide file/hash search | ✗ | ✓ |
| Response actions from console | ✗ | ✓ |

---

### Core Difference in One Line
> AV asks: "Have I seen this before?"
> EDR asks: "Is this behavior normal?"

---

## How EDR Works — Architecture

### EDR Components

```
Endpoints (laptops, servers, workstations)
    ↓ (EDR Agent installed on each)
Agent collects + sends data in real-time
    ↓
EDR Central Console
    → Correlates data
    → Applies ML + threat intelligence
    → Generates alerts
    ↓
SOC Analyst investigates + responds
```

---

### EDR Agents (Sensors)
- Installed directly on each endpoint
- "Eyes and ears" of the EDR
- Monitor all endpoint activity in real-time
- Send detailed data to central console
- Can perform basic signature + behavior detection locally
- Enable remote response actions from console

---

### EDR Console (The Brain)
- Receives data from all agents simultaneously
- Correlates events using complex logic + ML algorithms
- Matches against threat intelligence feeds
- Connects the dots → generates alerts with full context
- Dashboard gives holistic view across all endpoints

**Alert details available on click:**
- Files executed
- Processes spawned (process tree)
- Network connection attempts
- Registry modifications
- And more

---

### Post-Detection Workflow

```
Alert generated with severity (Critical/High/Medium/Low/Info)
    ↓
SOC Analyst acknowledges + prioritizes by severity
    ↓
Click alert → review full context
    ↓
False Positive → document → close
True Positive  → respond from within EDR console
                 (isolate / terminate / quarantine / remote shell)
```

---

### EDR in the Larger Security Ecosystem

| Solution | Protects |
|----------|---------|
| Firewall | Network perimeter |
| EDR | Endpoints |
| DLP | Data exfiltration |
| Email Security Gateway | Phishing / malicious attachments |
| IAM | Identity + access |
| **SIEM** | **Central integration point — aggregates all above** |

```
All security solutions → feed into SIEM → single investigation console for analysts
```

> EDR alone is powerful. EDR + SIEM = full security ecosystem.
> A standalone EDR only sees endpoint activity.
> SIEM correlates endpoint + network + identity + email data together.

---

## EDR Telemetry

### What is Telemetry?
All data collected by EDR agents from endpoints.
**The black box of an endpoint** — everything needed for detection and investigation.

---

### Telemetry Types Collected

| Type | What it captures | Why it matters |
|------|-----------------|---------------|
| **Process Executions/Terminations** | All running/idle processes + parent-child relationships | Spots suspicious child processes (e.g. Word → PowerShell) |
| **Network Connections** | All inbound/outbound connections, ports used | Detects C2 beaconing, data exfiltration, lateral movement |
| **Command Line Activity** | All CMD/PowerShell commands executed | Catches obfuscated scripts, malicious commands missed by AV |
| **File/Folder Modifications** | Files created, renamed, deleted, moved | Detects ransomware encryption, data staging, malware drops |
| **Registry Modifications** | All registry key changes | Identifies persistence mechanisms, config tampering |

---

### Why Detailed Telemetry Matters

```
Advanced threats use legitimate utilities (LOLBins)
→ Each individual action looks harmless
→ Individually: powershell.exe running = normal
→ In context: winword.exe → powershell.exe → encoded command
              → svchost.exe outbound connection = attack chain
```

**Telemetry enables:**
- Detect advanced threats through behavioral patterns
- Understand full chain of events
- Identify root cause of breach
- Reconstruct complete attack timeline

---

### Telemetry vs Raw Logs

| | Raw Logs | EDR Telemetry |
|-|----------|--------------|
| Context | Minimal | Rich — full process tree, relationships |
| Volume | High | Structured + filtered |
| Correlation | Manual | Automated via ML |
| Timeline | Fragmented | Continuous, chronological |
| Investigation use | Difficult | Ready for analyst consumption |

> Telemetry is why EDR analysts can reconstruct a full attack in minutes
> instead of the hours it takes to manually correlate raw logs.

---

## EDR — Detection & Response Capabilities

### Detection Techniques

| Technique | How it works | Example |
|-----------|-------------|---------|
| **Behavioral Detection** | Observes complete behavior, not just signatures | `winword.exe` spawning `PowerShell.exe` → flagged (unusual parent-child) |
| **Anomaly Detection** | Learns baseline, flags deviations | Process modifying auto-start registry key — never seen before on this endpoint |
| **IOC Matching** | Matches against threat intelligence feeds | Downloaded executable hash matches known malware in TI feed → instant flag |
| **MITRE ATT&CK Mapping** | Maps detection to tactic + technique | Scheduled task created → Persistence: Scheduled Task/Job [T1053] |
| **Machine Learning** | Models trained on normal vs malicious behavior datasets | Fileless attacks, multi-stage intrusions detected via pattern recognition |

---

### Response Capabilities

#### Isolate Host
```
Purpose:  Cut infected endpoint from network
When:     Active breach spreading laterally
Effect:   Stops lateral movement to other endpoints
Risk:     Business disruption if critical system
```

#### Terminate Process
```
Purpose:  Kill specific malicious process
When:     Malicious activity contained to one process
          Host isolation would cause more damage than the threat
Risk:     Terminating legitimate process disrupts endpoint
```

#### Quarantine File
```
Purpose:  Move malicious file to isolated location
          Cannot be executed from quarantine
When:     Malicious file identified on endpoint
After:    Analyst reviews → restore or permanently delete
```

#### Remote Access (RTR — Real Time Response)
```
Purpose:  Shell access to any endpoint from EDR console
When:     Built-in response options aren't enough
          Custom investigation or actions needed
Allows:   Run commands/scripts, collect custom data,
          deeper manual investigation
```

#### Artefact Collection
Extract forensic evidence remotely — no physical access needed.

| Artefact | Use |
|----------|-----|
| Memory dump | Volatile data, injected code, credentials in RAM |
| Event logs | Windows security/system/application logs |
| Specific folder contents | Malware staging directories |
| Registry hives | Persistence keys, configuration data |

---

### Response Decision Framework
```
Malicious activity detected
    ↓
Spreading laterally?      → Isolate host
Single process?           → Terminate process
Malicious file on disk?   → Quarantine
Need deeper investigation → Remote access (RTR)
Need forensic evidence    → Artefact collection
```
