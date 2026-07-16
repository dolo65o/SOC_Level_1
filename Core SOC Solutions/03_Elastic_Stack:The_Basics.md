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

---

## Kibana Query Language (KQL)

### Two Search Types
```
1. Free text search  → search across ALL fields
2. Field-based search → search specific field: value
```

---

### Free Text Search

**Basic:**
```kql
United States
```
Returns all logs containing exact phrase "United States"

**Wildcard (`*`):**
```kql
United*
```
Returns logs containing "United", "United States", "United Nations", etc.

> KQL searches for whole terms — no wildcard = exact match only

---

### Logical Operators

#### AND — both conditions must match
```kql
"United States" AND "Virginia"
```

#### OR — either condition matches
```kql
"United States" OR "England"
```

#### NOT — exclude a term
```kql
"United States" AND NOT ("Florida")
```

#### Combined
```kql
"United States" AND NOT ("Florida" OR "Texas")
```

---

### Field-Based Search

<img width="1903" height="911" alt="field base search" src="https://github.com/user-attachments/assets/1cde5c72-b5cc-4beb-ae32-359b6db1fbe4" />

Syntax: `FieldName : Value`

**Single field:**
```kql
Source_ip : 238.163.231.224
```

**Multiple fields (AND):**
```kql
Source_ip : 238.163.231.224 AND UserName : Suleman
```

**Field with wildcard:**
```kql
UserName : Su*
```

**Exclude field value:**
```kql
Source_Country : "United States" AND NOT Source_Country : "France"
```

---

### KQL Quick Reference

| Syntax | Example | Meaning |
|--------|---------|---------|
| `term` | `security` | Free text search |
| `"phrase"` | `"United States"` | Exact phrase match |
| `term*` | `United*` | Wildcard — anything after |
| `field : value` | `UserName : Bob` | Field-based search |
| `AND` | `A AND B` | Both must match |
| `OR` | `A OR B` | Either matches |
| `NOT` | `NOT A` | Exclude matches |

---

### VPN Log Investigation Examples

```kql
# Find all logins from specific country
Source_Country : "United States"

# Find specific user's login activity
UserName : "Maleena"

# Find logins from suspicious IP
Source_ip : 107.14.182.38

# Find logins NOT from expected country
NOT Source_Country : "France"

# Combine — specific user from unexpected location
UserName : "Admin" AND NOT Source_Country : "United States"
```
## Kibana – Visualization Tab

<img width="1914" height="1043" alt="create viuslaization" src="https://github.com/user-attachments/assets/91164c4b-249a-4836-8106-567fcaddc5d8" />

- Used to visualize data as **tables, pie charts, bar charts**, etc.
- **Access:** Discover tab → click a field → select "Visualize", or go directly to the Visualize tab.
- **Correlation:** drag a second field (e.g. `Source_Country`) into the visualization to correlate it with the first (e.g. `Source_IP`).

### Saving a Visualization
1. Create the visualization.
2. Click **Save** (top right).
3. Add a title + description.
4. **Save and add to library**.

### Example: Failed VPN Connection Attempts (Table)

<img width="1914" height="966" alt="failed connection attempts visualization" src="https://github.com/user-attachments/assets/999f1f68-d32d-41cd-83b5-45b8fb9de1cf" />

- Data view: `vpn_connections`
- Time range: January 2022
- Filter: `action: failed` (show only failed attempts, don't exclude them)
- Fields: `UserName`, `Source_ip`

---

## Kibana – Dashboards

- **Dashboards** combine saved `Searches` + `Visualizations` into one view for a specific need (e.g. VPN log visibility).
- Multiple dashboards can be created for different purposes.

### Creating a Custom Dashboard

<img width="1914" height="1034" alt="createdashboard-ezgif com-optimize" src="https://github.com/user-attachments/assets/4b8220a8-cef0-4c94-b705-a9cf9c29580d" />


1. Go to **Dashboard tab** → click **Create dashboard**.
2. Click **Add from Library**.
3. Select the saved visualizations/searches to add them to the dashboard.
4. Resize/arrange the panels as needed.
5. **Save** the dashboard.

> Prerequisite: have Searches (Discover tab) and Visualizations already created & saved before building the dashboard.
