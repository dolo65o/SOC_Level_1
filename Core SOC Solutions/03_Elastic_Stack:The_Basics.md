## Elastic Stack (ELK) — Overview

### What is ELK?
Open-source SIEM-like platform for collecting, storing, searching, and visualizing data in real-time.
Widely adopted by SOC teams for log analysis and investigations.

---

### 4 Core Components

#### 1. Elasticsearch
- Full-text search and analytics engine
- Stores, analyzes, and correlates JSON-formatted data
- Supports RESTful API for interaction
- Acts as the **database** of the stack

#### 2. Logstash
Data processing pipeline — **Input → Filter → Output**

| Section | Purpose |
|---------|---------|
| **Input** | Define data source (files, beats, ports) |
| **Filter** | Normalize/parse raw logs into field-value pairs |
| **Output** | Send processed data to Elasticsearch, Kibana, file, or port |

Supports many plugins for each section.

#### 3. Beats
Host-based agents (data-shippers) installed on endpoints.
Each Beat is single-purpose — sends specific data type to Elasticsearch.

| Beat | Data it collects |
|------|----------------|
| Winlogbeat | Windows Event Logs |
| Packetbeat | Network traffic flows |
| Filebeat | Log files |
| Metricbeat | System metrics (CPU, memory) |
| Auditbeat | Audit framework data |

#### 4. Kibana
Web-based visualization and investigation interface.
- Searches and analyzes data stored in Elasticsearch
- Creates dashboards, charts, timelines, infographics
- Primary interface for SOC analysts using ELK

---

### How They Work Together

```
Endpoints
    ↓ Beats collect specific data
Logstash
    → Receives from Beats / ports / files
    → Parses + normalizes into field-value pairs
    → Sends to Elasticsearch
Elasticsearch
    → Stores + indexes all data
    → Powers search and correlation
Kibana
    → Queries Elasticsearch
    → Displays data as dashboards, charts, timelines
    → Where SOC analysts investigate
```

---

### ELK vs Splunk (Quick Reference)

| | ELK | Splunk |
|-|-----|--------|
| **Cost** | Open-source (free tier) | Commercial (expensive) |
| **Query language** | KQL / Lucene | SPL |
| **Data shipper** | Beats | Forwarder |
| **Processing** | Logstash | Indexer |
| **Visualization** | Kibana | Search Head + dashboards |
| **SOC use** | Very common | Industry standard |

> As a SOC analyst, your job is to use Kibana for investigation.
> You don't need to manage Logstash pipelines or Elasticsearch clusters —
> that's the SOC engineer's responsibility.

---

## Kibana — Discover Tab Interface

### Overview
Discover tab = where SOC analysts spend most of their investigation time.
Shows ingested logs, search bar, normalized fields, time filters, and event timeline.

---

### Interface Components

<img width="868" height="398" alt="1" src="https://github.com/user-attachments/assets/b1f55315-757c-498b-9b40-015414963705" />

| # | Component | Purpose |
|---|-----------|---------|
| 1 | **Logs** | Each row = one event with all its fields and values |
| 2 | **Fields Pane** (left panel) | List of all normalized fields parsed from logs |
| 3 | **Index Pattern** | Selects which log source to query (e.g. `vpn_connections`) |
| 4 | **Search Bar** | Write KQL queries and apply filters |
| 5 | **Time Filter** | Narrow results to specific time range |
| 6 | **Time Interval Chart** | Visual event count over time — spot spikes |
| 7 | **Top Bar** | Save/load/share searches |
| 8 | **Discover Tab** | Main workspace for raw log exploration |
| 9 | **Add Filter** | Apply field-level filters without writing queries |

---

### Index Pattern
- Tells Kibana which Elasticsearch data to explore
- Each log source has its own index pattern (different log structure)
- One index pattern can point to multiple indices
- Lab index: `vpn_connections`

---

### Fields Pane — How to Use

<img width="363" height="251" alt="fields pane" src="https://github.com/user-attachments/assets/e4ba1df4-90b7-4f0d-a8ca-381984247079" />

```
Click any field in left panel
    → Shows top 5 values + % occurrence
    → Click (+) → add filter: show logs WITH this value
    → Click (-) → add filter: show logs WITHOUT this value
```

---

### Time Filter Options

<img width="367" height="264" alt="time filter" src="https://github.com/user-attachments/assets/e50894d4-6132-4c33-ac32-8b325022840e" />

- Relative: Last 15 min, last 24h, last 7 days, etc.
- Absolute: Custom date/time range
- Quick select: Today, This week, This month

---

### Timeline Chart

<img width="758" height="121" alt="timeline" src="https://github.com/user-attachments/assets/17b84d68-54f4-49c1-ac5f-da71dfbfd282" />

- Bars = event count per time interval
- Click a bar → filter logs to that time period only
- Count (top left) = total events in current filter

---

### Creating a Table View

<img width="1914" height="985" alt="create table" src="https://github.com/user-attachments/assets/d4af9777-f35e-4c05-9eda-5d6292ac0d15" />

Raw logs are noisy. Build a cleaner table:
```
Click any log entry
    → Select important fields
    → Fields added as columns
    → Table view shows only selected fields
    → Reduces noise, easier to spot patterns
```

**Useful fields for VPN log table:**
`UserName` `Source_ip` `Source_Country` `Timestamp` `Action`

---

### Investigation Workflow in Discover
```
1. Select correct index pattern
2. Set time filter to relevant period
3. Use Fields Pane to understand available fields
4. Write search query or apply filters
5. Create table view with relevant fields
6. Use timeline to identify activity spikes
7. Drill into individual events for full detail
```
