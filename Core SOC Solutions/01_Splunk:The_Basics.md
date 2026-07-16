## Splunk — SIEM Fundamentals

### What is Splunk?
Leading SIEM solution for collecting, analyzing, and correlating network and machine logs in real-time.
Provides visibility of network activity and speeds up threat detection.

---

### 3 Core Components

```
Endpoints/Log Sources
    ↓
Forwarder (collect + send)
    ↓
Indexer (parse + normalize + store)
    ↓
Search Head (query + analyze)
```

---

### Component Breakdown

#### 1. Splunk Forwarder
Lightweight agent installed on monitored endpoints.
- Collects data from log sources
- Sends to Splunk Indexer
- Minimal resource usage — doesn't impact endpoint performance

**Common data sources:**
| Source | Data collected |
|--------|---------------|
| Web server | Web traffic logs |
| Windows machine | Event logs, PowerShell, Sysmon |
| Linux host | Host-centric logs |
| Database | Connection requests, responses, errors |

#### 2. Splunk Indexer
The processing engine.
- Receives raw data from forwarders
- **Parses** → breaks into fields
- **Normalizes** → converts to field-value pairs
- **Categorizes** → organizes by source type
- **Stores** → saves as searchable events

#### 3. Search Head
The analyst interface — Search & Reporting App.
- Users write queries using **SPL (Search Processing Language)**
- Sends search requests to Indexer
- Returns matching events as field-value pairs
- Where SOC analysts spend most of their investigation time

---

### Data Flow Summary
```
Log source generates data
    ↓ Forwarder collects + ships
Indexer parses + normalizes + stores as events
    ↓ Analyst queries via SPL
Search Head returns matching field-value pairs
    ↓
Analyst investigates → triage → verdict
```

> SPL is the core skill for Splunk-based SOC work.
> Every investigation starts with a search query.
> The better your SPL, the faster your triage.

---

## Splunk — Adding Data & VPN Log Analysis

### Adding Data to Splunk — 5 Steps

<img width="1905" height="904" alt="add data" src="https://github.com/user-attachments/assets/cc7ca961-0cfc-4647-9680-99ef46986288" />

```
Add Data → Upload
    ↓
1. Select Source      → choose VPN_logs.json file
2. Select Source Type → JSON (auto-detected)
3. Input Settings     → create/select index: VPN_Logs
                        set hostname
4. Review             → verify all settings
5. Done               → upload complete, ready to search
```

> After upload: open Search & Reporting → set time picker to **All time**
> If field names don't appear → add `| spath` after base search

---

### Useful SPL Queries — VPN Log Analysis

#### Verify data loaded
```splunk
index=VPN_Logs
| stats count
```

#### Search by username
```splunk
index=VPN_Logs
| spath
| search UserName="Maleena"
| stats count
```

#### Find users from a specific IP
```splunk
index=VPN_Logs
| spath
| search Source_ip="107.14.182.38"
| stats values(UserName) as UserName count
```

#### Find logins NOT from a specific country
```splunk
index=VPN_Logs
| spath
| search Source_Country!="France"
| stats count
```

#### Count events from specific IP
```splunk
index=VPN_Logs
| spath
| search Source_ip="107.3.206.58"
| stats count
```

---

### SPL Basics Used Here

| Command | Purpose |
|---------|---------|
| `index=VPN_Logs` | Scope search to VPN_Logs index |
| `\| spath` | Parse JSON fields into searchable key-value pairs |
| `\| search field="value"` | Filter results by field value |
| `\| stats count` | Count matching events |
| `\| stats values(X) as X` | List unique values of field X |
| `field!="value"` | Exclude specific field value |

---

### VPN Log Fields (from JSON)
```
UserName        → who logged in
Source_ip       → origin IP address
Source_Country  → origin country
(+ other fields parsed by spath)
```
