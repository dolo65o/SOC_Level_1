## IDS vs IPS & Snort Overview

### IDS — Intrusion Detection System
**Passive** — detects and alerts, does NOT stop threats.

| Type | Scope |
|------|-------|
| **NIDS** | Monitors entire network subnet |
| **HIDS** | Monitors single endpoint device |

---

### IPS — Intrusion Prevention System
**Active** — detects AND terminates threats automatically.

| Type | Scope |
|------|-------|
| **NIPS** | Protects entire network subnet |
| **NBA** (Behaviour-based) | Network-wide + requires baselining/training period |
| **WIPS** | Wireless network traffic |
| **HIPS** | Single endpoint device |

**NBA Note:**
- Requires training period to learn "normal" traffic
- More effective against new/unknown threats
- Security breach during training = corrupted baseline = bad results

---

### Detection/Prevention Techniques

| Technique | How it works | Best for |
|-----------|-------------|---------|
| **Signature-based** | Matches known malicious patterns/rules | Known threats |
| **Behaviour-based** | Compares normal vs abnormal behavior | Unknown/new threats |
| **Policy-based** | Compares activity against security policies | Policy violations |

---

### IDS vs IPS — Key Difference

```
IDS → detects threat → creates alert → human must act
IPS → detects threat → terminates connection → less human involvement
```

---

### Snort — NIDS/NIPS

Open-source rule-based Network IDS/IPS.
Developed by Martin Roesch, maintained by Cisco Talos team.

🔗 https://www.snort.org

**Capabilities:**
- Live traffic analysis
- Attack and probe detection
- Packet logging
- Protocol analysis
- Real-time alerting
- Modules, plugins, pre-processors
- Cross-platform (Linux + Windows)

### Snort Modes

| Mode | What it does |
|------|-------------|
| **Sniffer** | Reads and displays IP packets in console (like tcpdump) |
| **Packet Logger** | Logs all inbound/outbound IP packets for later analysis |
| **NIDS/NIPS** | Logs or drops packets matching user-defined rules |

---

## Snort — First Interaction & Basic Parameters

### Verify Installation
```bash
snort -V
```
Shows version, build info, library versions (libpcap, PCRE, ZLIB).

---

### Validate Configuration
```bash
sudo snort -c /etc/snort/snort.conf -T
```
- `-c` → specify config file path
- `-T` → test/validate the configuration
- Success message: `Snort successfully validated the configuration!`

> Configuration file = Snort's all-in-one management file.
> Contains: rules, plugins, detection mechanisms, default actions, output settings.
> Only ONE config file can be used at runtime.
> Multiple config files can exist for different purposes.

---

### Essential Parameters

| Parameter | Description |
|-----------|-------------|
| `-V` / `--version` | Show Snort version info |
| `-c <file>` | Specify configuration file |
| `-T` | Self-test — validate configuration |
| `-q` | Quiet mode — suppress banner and startup info |

---
## Snort — Sniffer Mode

### Sniffer Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-v` | Verbose — display TCP/IP output in console (like tcpdump) |
| `-d` | Display packet data (payload) in hex + ASCII |
| `-e` | Display link-layer headers (MAC addresses, frame type) |
| `-X` | Display full packet details in HEX |
| `-i <interface>` | Specify network interface to sniff (e.g. eth0) |

---

### Commands & What They Show

```bash
# Basic verbose — IP/TCP/UDP headers only
sudo snort -v

# Specific interface
sudo snort -v -i eth0

# Payload data (includes verbose)
sudo snort -d

# Link-layer + payload (MAC addresses visible)
sudo snort -de

# Full packet in HEX (most detail)
sudo snort -X
```

---

### Output Examples

**`-v` output:**
```
12/01-20:10:13.846653 192.168.175.129:34316 -> 192.168.175.2:53
UDP TTL:64 TOS:0x0 ID:23826 IpLen:20 DgmLen:64 DF
```

**`-d` output (adds payload):**
```
99 A5 01 00 00 01 00 00 00 00 00 00 06 67 6F 6F  .............goo
67 6C 65 03 63 6F 6D 00 00 1C 00 01              gle.com.....
```

**`-de` output (adds MAC addresses):**
```
00:0C:29:A5:B7:A2 -> 00:50:56:E1:9B:9D type:0x800 len:0x46
192.168.175.129:47395 -> 192.168.175.2:53 UDP TTL:64
```

**`-X` output (full HEX dump):**
```
0x0000: 01 00 5E 7F FF FA 00 50 56 C0 00 08 08 00 45 00
0x0010: 00 C4 BE DD 00 00 01 11 9A A7 C0 A8 AF 01 EF FF
```

---

### Parameter Combinations
```bash
snort -v          # headers only
snort -vd         # headers + payload
snort -de         # link-layer + payload
snort -v -d -e    # same as -de (space separated)
snort -X          # full hex dump (most verbose)
```

> Stop sniffing: `CTRL+C`
> Snort summarizes total packets captured on exit.

---

## Snort — Packet Logger Mode

### Logger Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-l <dir>` | Log to specified directory (default: `/var/log/snort`) |
| `-K ASCII` | Log in ASCII format (human-readable, categorized by IP) |
| `-r <file>` | Read and analyze a previously logged file |
| `-n <count>` | Process only specified number of packets then stop |

---

### Logging Commands

```bash
# Log to current directory (binary/tcpdump format)
sudo snort -dev -l .

# Log in ASCII format to current directory
sudo snort -dev -K ASCII -l .

# Read binary log file
sudo snort -r snort.log.1638459842

# Read with filter (BPF)
sudo snort -r logname.log icmp
sudo snort -r logname.log tcp
sudo snort -r logname.log 'udp and port 53'

# Read only first 10 packets
sudo snort -dvr logname.log -n 10

# Full hex dump from log file
sudo snort -r logname.log -X
```

---

### Binary vs ASCII Log Format

| | Binary (default) | ASCII (-K ASCII) |
|-|-----------------|-----------------|
| **Format** | tcpdump/pcap format | Human-readable text |
| **Structure** | Single `.log` file | Folders named by IP address |
| **Readable without Snort?** | No | Yes (text editor) |
| **Compatible with Snort -r?** | Yes | No |
| **Compatible with tcpdump?** | Yes | No |
| **Compatible with Wireshark?** | Yes | No |

---

### ASCII Log Structure
```
./
├── 142.250.187.110/
├── 192.168.175.129/
│   ├── ICMP_ECHO
│   ├── UDP:36648-53
│   ├── UDP:40757-53
│   └── UDP:50624-123
└── snort.log.1638459842
```
Logs categorized by IP and protocol — readable directly as text files.

---

### Reading Logs with Other Tools

```bash
# Read with tcpdump (first 10 packets, no hostname resolution)
sudo tcpdump -r snort.log.1638459842 -ntc 10

# Open in Wireshark
wireshark snort.log.1638459842
```

---

### File Ownership Note
```bash
# Snort runs as root → log files owned by root
# Fix ownership to read as your user:
sudo chown username snort.log.1638459842
sudo chown username -R ./log_directory/   # recursive
```

---

### BPF Filter Resources
- https://en.wikipedia.org/wiki/Berkeley_Packet_Filter
- https://biot.com/capstats/bpf.html
- https://www.tcpdump.org/manpages/tcpdump.1.html

---
## Snort — IDS/IPS Mode

### IDS/IPS Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-c <file>` | Define configuration file |
| `-T` | Test configuration file |
| `-N` | Disable logging |
| `-D` | Background (daemon) mode |
| `-A <mode>` | Set alert mode |

---

### Alert Modes (-A)

| Mode | Console Output | Log Output | Detail Level |
|------|---------------|-----------|-------------|
| `console` | ✓ | ✓ | Fast style — rule + IPs + ports |
| `cmg` | ✓ | ✓ | Full headers + payload in hex/text |
| `fast` | ✗ | ✓ | Alert message + timestamp + IPs + ports |
| `full` | ✗ | ✓ | All possible information |
| `none` | ✗ | ✗ (no alert file) | Only log file created |

---

### Commands

```bash
# Test configuration
sudo snort -c /etc/snort/snort.conf -T

# IDS with console alerts (see alerts in terminal)
sudo snort -c /etc/snort/snort.conf -A console

# IDS with CMG alerts (headers + hex payload in terminal)
sudo snort -c /etc/snort/snort.conf -A cmg

# IDS with fast alerts (log file only)
sudo snort -c /etc/snort/snort.conf -A fast

# IDS with full alerts (log file only)
sudo snort -c /etc/snort/snort.conf -A full

# IDS with no alerts (log only, no alert file)
sudo snort -c /etc/snort/snort.conf -A none

# Disable logging
sudo snort -c /etc/snort/snort.conf -N

# Background/daemon mode
sudo snort -c /etc/snort/snort.conf -D

# Run with rules only (no config — testing mode)
sudo snort -c /etc/snort/rules/local.rules -A console
```

---

### Background Mode Management
```bash
# Check if Snort is running
ps -ef | grep snort

# Kill Snort daemon
sudo kill -9 <PID>
```

> Daemon mode is for automation/scripts only.
> Don't use unless you have stable config and working knowledge of Snort.

---

### IPS Mode — Drop Packets
Requires at least **two interfaces**.

```bash
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A console
```

**IPS alert output shows `[Drop]` instead of `[**]`:**
```
[Drop] [**] [1:1000001:0] ICMP Packet found [**] {ICMP} 192.168.175.131 -> 192.168.175.2
```

| Mode | Action on detection |
|------|-------------------|
| IDS | Alert only — packet passes |
| IPS | Alert + DROP packet |

---

### Console vs CMG Output Comparison

**Console (`-A console`):**
```
12/12-02:08:27 [**] [1:366:7] ICMP PING *NIX [**] [Priority: 3] {ICMP} 192.168.175.129 -> 142.250.187.110
```

**CMG (`-A cmg`):**
```
12/12-02:23:56 [**] [1:366:7] ICMP PING *NIX [**] [Priority: 3] {ICMP} 192.168.175.129 -> 142.250.187.110
00:0C:29:A5:B7:A2 -> 00:50:56:E1:9B:9D type:0x800 len:0x62
192.168.175.129 -> 142.250.187.110 ICMP TTL:64 TOS:0x0
BC CD B5 61 00 00 00 00 CE 68 0E 00 ...
```

---

## Snort — PCAP Investigation Mode

### PCAP Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-r` / `--pcap-single=` | Read a single PCAP file |
| `--pcap-list=""` | Read multiple PCAPs (space separated) |
| `--pcap-show` | Show which PCAP file is being processed in console |

---

### Commands

```bash
# Basic PCAP read (stats only — no rules)
snort -r icmp-test.pcap

# Single PCAP with config + console alerts (first 10 packets)
sudo snort -c /etc/snort/snort.conf -q -r icmp-test.pcap -A console -n 10

# Multiple PCAPs
sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console

# Multiple PCAPs — show which file each alert comes from
sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console --pcap-show
```

---

### Why --pcap-show Matters

**Without `--pcap-show`:**
```
[**] ICMP Packet found [**] 192.168.175.129 -> 142.250.187.110
[**] ICMP Packet found [**] 142.250.187.110 -> 192.168.175.129
# Can't tell which PCAP generated which alert
```

**With `--pcap-show`:**
```
Reading network traffic from "icmp-test.pcap" with snaplen = 1514
[**] ICMP Packet found [**] 192.168.175.129 -> 142.250.187.110

Reading network traffic from "http2.pcap" with snaplen = 1514
[**] ICMP Packet found [**] 142.250.187.110 -> 192.168.175.129
# Now you know exactly which PCAP each alert came from
```

---

### PCAP Investigation Workflow

```
Have a suspicious PCAP from incident
    ↓
Run with config + rules + console alerts
    ↓
Snort applies all detection rules against historical traffic
    ↓
Rules fire on matching patterns → alerts generated
    ↓
Much faster than manual packet-by-packet analysis
```

---

### Full Command Reference — Combined

```bash
# Single PCAP — full investigation
sudo snort -c /etc/snort/snort.conf -q -r suspicious.pcap -A console

# Single PCAP — limit packets
sudo snort -c /etc/snort/snort.conf -q -r suspicious.pcap -A console -n 100

# Multiple PCAPs — identify alert source
sudo snort -c /etc/snort/snort.conf -q \
  --pcap-list="file1.pcap file2.pcap file3.pcap" \
  -A console --pcap-show

# Read PCAP with BPF filter
sudo snort -r suspicious.pcap 'tcp and port 80'
```
---

## Snort — Rule Writing

### Rule Structure
```
action protocol src_ip src_port direction dst_ip dst_port (options)
```

<img width="1140" height="784" alt="snort rule" src="https://github.com/user-attachments/assets/4ed10e4e-5d9c-451a-882b-f3719e46f74e" />

---

### Actions

| Action | Behavior |
|--------|---------|
| `alert` | Generate alert + log packet (IDS) |
| `log` | Log packet only |
| `drop` | Block + log packet (IPS) |
| `reject` | Block + log + terminate session (IPS) |

---

### Protocols
Snort 2 supports only: `ip` `tcp` `udp` `icmp`
> No FTP/HTTP keywords — use port numbers instead (FTP = TCP port 21)

---

### Direction Operators
| Operator | Meaning |
|----------|---------|
| `->` | Source to destination only |
| `<>` | Bidirectional |
> No `<-` operator in Snort

---

### IP Filtering Examples

```
# Single IP
alert icmp 192.168.1.56 any <> any any (msg:"From specific IP"; sid:100001; rev:1;)

# Subnet
alert icmp 192.168.1.0/24 any <> any any (msg:"From subnet"; sid:100001; rev:1;)

# Multiple subnets
alert icmp [192.168.1.0/24,10.1.1.0/24] any <> any any (msg:"Multi subnet"; sid:100001; rev:1;)

# Exclude IP/range (! = negation)
alert icmp !192.168.1.0/24 any <> any any (msg:"NOT from subnet"; sid:100001; rev:1;)
```

---

### Port Filtering Examples

```
# Specific port
alert tcp any any <> any 21 (msg:"FTP traffic"; sid:100001; rev:1;)

# Exclude port
alert tcp any any <> any !21 (msg:"Not FTP"; sid:100001; rev:1;)

# Port range (1 to 1024)
alert tcp any any <> any 1:1024 (msg:"System ports"; sid:100001; rev:1;)

# Up to port 1024
alert tcp any any <> any :1024 (msg:"Up to 1024"; sid:100001; rev:1;)

# Port 1025 and above
alert tcp any any <> any 1025: (msg:"Non-system ports"; sid:100001; rev:1;)

# Multiple specific ports
alert tcp any any <> any [21,23] (msg:"FTP or Telnet"; sid:100001; rev:1;)
```

---

### Rule Options

#### General Options

| Option | Purpose | Example |
|--------|---------|---------|
| `msg` | Alert message shown on trigger | `msg:"ICMP Found";` |
| `sid` | Unique rule ID (user rules: ≥1,000,000) | `sid:1000001;` |
| `rev` | Revision number | `rev:1;` |
| `reference` | Link to CVE or external info | `reference:cve,CVE-2021-1234;` |

**SID Ranges:**
```
< 100          → Reserved
100–999,999    → Built-in rules
≥ 1,000,000    → User-created rules
```

#### Payload Detection Options

| Option | Purpose | Example |
|--------|---------|---------|
| `content` | Match ASCII or HEX in payload | `content:"GET";` |
| `nocase` | Case-insensitive content match | `content:"get"; nocase;` |
| `fast_pattern` | Prioritize this content for initial match | `content:"GET"; fast_pattern;` |

```bash
# ASCII content match
alert tcp any any <> any 80 (msg:"GET Request"; content:"GET"; sid:1000001; rev:1;)

# HEX content match
alert tcp any any <> any 80 (msg:"GET Request"; content:"|47 45 54|"; sid:1000001; rev:1;)

# Multiple content with fast_pattern
alert tcp any any <> any 80 (msg:"GET www"; content:"GET"; fast_pattern; content:"www"; sid:1000001; rev:1;)
```

#### Non-Payload Detection Options

| Option | Purpose | Example |
|--------|---------|---------|
| `id` | Filter by IP ID field | `id:123456;` |
| `flags` | Filter TCP flags (F S R P A U) | `flags:S;` (SYN only) |
| `dsize` | Filter by payload size | `dsize:100<>300;` |
| `sameip` | Alert if src IP = dst IP | `sameip;` |

```bash
# SYN packets only
alert tcp any any <> any any (msg:"SYN Flag"; flags:S; sid:1000001; rev:1;)

# Payload size filter
alert ip any any <> any any (msg:"Size 100-300"; dsize:100<>300; sid:1000001; rev:1;)

# Same source and destination IP
alert ip any any <> any any (msg:"Same IP"; sameip; sid:1000001; rev:1;)
```

---

### Local Rules File
```bash
# Edit local rules
sudo gedit /etc/snort/rules/local.rules
# or
sudo nano /etc/snort/rules/local.rules
```
Location: `/etc/snort/rules/local.rules`

---

### Full Rule Example
```
alert icmp any any <> any any (msg:"ICMP Packet Found"; reference:cve,CVE-XXXX; sid:1000001; rev:1;)
```
---

## Snort — Architecture & Configuration

### Main Components

| Component | Role |
|-----------|------|
| **Packet Decoder** | Collects and prepares packets for pre-processing |
| **Pre-processors** | Arranges and modifies packets for detection engine |
| **Detection Engine** | Processes, dissects, and analyzes packets against rules |
| **Logging & Alerting** | Generates logs and alerts |
| **Outputs & Plugins** | Output integration (syslog, MySQL) + additional plugins |

---

### Rule Types

| Type | Cost | Notes |
|------|------|-------|
| **Community Rules** | Free | GPLv2, no registration needed |
| **Registered Rules** | Free | Requires registration — 30-day delay on subscriber rules |
| **Subscriber Rules** | Paid | Main ruleset, updated Tuesdays + Thursdays |

---

### Key Files

| File | Purpose |
|------|---------|
| `/etc/snort/snort.conf` | Main configuration file |
| `/etc/snort/rules/local.rules` | User-created rules |
| `/etc/snort/rules/` | Rule directory |

> Never replace snort.conf — always edit manually or use update tools.

---

### snort.conf — Key Sections

#### Step 1 — Network Variables
```bash
sudo gedit /etc/snort/snort.conf
```

| Variable | Purpose | Example |
|----------|---------|---------|
| `HOME_NET` | Network you are protecting | `192.168.1.0/24` or `any` |
| `EXTERNAL_NET` | External/untrusted network | `any` or `!$HOME_NET` |
| `RULE_PATH` | Path to rules directory | `/etc/snort/rules` |
| `SO_RULE_PATH` | Subscriber/registered rule path | `$RULE_PATH/so_rules` |
| `PREPROC_RULE_PATH` | Preprocessor rule path | `$RULE_PATH/plugin_rules` |

#### Step 2 — Configure Decoder (IPS Mode)

| Setting | Purpose | Value |
|---------|---------|-------|
| `config daq` | Select DAQ module | `afpacket` (for IPS) |
| `config daq_mode` | Activate inline mode | `inline` |
| `config logdir` | Default log path | `/var/logs/snort` |

**DAQ Modules:**

| Module | Mode | Use |
|--------|------|-----|
| `pcap` | Default/Sniffer | Passive monitoring |
| `afpacket` | Inline/IPS | Active packet dropping |
| `ipq` | Inline (Linux/Netfilter) | IPS via Netfilter |
| `nfq` | Inline (Linux) | Alternative IPS |
| `ipfw` | Inline (BSD) | OpenBSD/FreeBSD IPS |
| `dump` | Testing | Test inline + normalization |

#### Step 6 — Output Plugins
Configure logging and alerting format.
Default: console output.

#### Step 7 — Customize Ruleset
```bash
# Activate local rules (remove # to uncomment)
include $RULE_PATH/local.rules

# Activate downloaded rules
include $RULE_PATH/community.rules
```

> `#` = comment operator — uncomment a line to activate it.

---

### Quick Config Summary
```bash
# Open main config
sudo gedit /etc/snort/snort.conf

# Set your network
HOME_NET 192.168.1.0/24
EXTERNAL_NET !$HOME_NET

# Enable IPS mode
config daq: afpacket
config daq_mode: inline

# Activate rules
include $RULE_PATH/local.rules
```
---

> [Snort cheatsheet](https://assets.tryhackme.com/cheatsheets/Snort%20Cheatsheet%20-%20TryHackMe.pdf)
