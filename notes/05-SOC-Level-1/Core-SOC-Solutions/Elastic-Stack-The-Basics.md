# Elastic Stack: The Basics

**Path:** SOC Level 1 - Core SOC Solutions
**Date:** 2026-05-21
**Difficulty:** Easy

---

## Elastic Stack Overview

- **Elastic Stack (ELK Stack)** → open-source suite of tools for collecting, storing, searching, and visualising log data
- ELK = **E**lasticsearch + **L**ogstash + **K**ibana
- Beats added later → sometimes called **BELK** or just **Elastic Stack**
- Widely used in SOC environments as a SIEM alternative/complement

### Core Components

| Component | Role |
|---|---|
| **Elasticsearch** | Search and analytics engine — stores and indexes all data |
| **Logstash** | Data processing pipeline — ingests, transforms, and forwards logs |
| **Kibana** | Web UI — visualise and query data stored in Elasticsearch |
| **Beats** | Lightweight data shippers installed on endpoints |

### Data Flow
```
Log Source → Beats (collect) → Logstash (parse + transform) → Elasticsearch (index + store) → Kibana (search + visualise)
```

---

## Component Deep Dive

### Elasticsearch
- Stores data as **JSON documents** in **indexes**
- Provides powerful full-text search via REST API
- Horizontally scalable — add more nodes as data grows
- Data organised into: **Index → Document → Fields**
- Think of it as a database optimised for search

### Logstash
- Three stages: **Input → Filter → Output**
- **Input** → receive data (from Beats, syslog, files, APIs)
- **Filter** → parse, transform, enrich data (grok patterns, mutate, geoip)
- **Output** → send to Elasticsearch, another system, or file

```
# Logstash pipeline example
input { beats { port => 5044 } }
filter {
  grok { match => { "message" => "%{COMBINEDAPACHELOG}" } }
  date { match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"] }
}
output { elasticsearch { hosts => ["localhost:9200"] } }
```

### Beats (Data Shippers)

| Beat | What it collects |
|---|---|
| **Filebeat** | Log files (Apache, Nginx, system logs) |
| **Metricbeat** | System metrics (CPU, memory, disk, network) |
| **Packetbeat** | Network traffic analysis |
| **Winlogbeat** | Windows Event Logs |
| **Auditbeat** | Linux audit framework data |
| **Heartbeat** | Uptime monitoring |

### Kibana
- Web interface running on port **5601** by default
- Connects to Elasticsearch to query and visualise data
- Main areas: Discover, Visualise, Dashboard, Alerts, Maps

---

## Discover Tab

- Primary area for **raw log investigation** (equivalent to Splunk's Search & Reporting)
- Search bar uses **KQL (Kibana Query Language)**
- **Time filter** → top right — always set this first
- **Field list** → left sidebar showing all available fields
- **Document view** → expandable raw log entries
- **Histogram** → shows event volume over time

### Discover Interface Layout
```
┌─────────────────────────────────────────────────┐
│  Search Bar (KQL)          │  Time Picker       │
├──────────┬──────────────────────────────────────┤
│          │  Histogram (event count over time)   │
│  Field   ├──────────────────────────────────────┤
│  List    │  Log results (expandable documents)  │
│          │                                      │
└──────────┴──────────────────────────────────────┘
```

---

## KQL — Kibana Query Language

### Basic Syntax
```kql
field: value
field: "exact phrase"
field: value AND field2: value2
field: value OR field2: value2
NOT field: value
field: *          (field exists)
```

### KQL Examples

```kql
# Find all events from a specific IP
source.ip: "10.10.10.5"

# Failed logins (HTTP 401)
http.response.status_code: 401

# Multiple status codes
http.response.status_code: 401 OR http.response.status_code: 403

# Events from specific host
host.name: "DESKTOP-ABC123"

# Wildcard search
user.name: admin*

# Find events with a specific field
event.action: *

# Combine conditions
source.ip: "10.10.10.5" AND http.response.status_code: 200

# Exclude specific values
NOT event.action: "heartbeat"

# Range query
http.response.bytes > 1000

# Find specific user activity
user.name: "john" AND event.outcome: "failure"
```

### KQL vs Lucene
- KQL is simpler, designed for Kibana — use this by default
- Lucene is more powerful, supports regex — toggle in search bar if needed
- Lucene regex example: `source.ip: /10\.10\.[0-9]+\.[0-9]+/`

---

## Creating Visualisations

### Visualisation Types

| Type | Best For |
|---|---|
| **Bar chart** | Comparing counts across categories |
| **Line chart** | Trends over time |
| **Pie chart** | Proportional breakdown |
| **Data table** | Tabular summary of aggregated data |
| **Metric** | Single number (total events, unique IPs) |
| **Heat map** | Activity patterns across two dimensions |
| **Maps** | Geographic distribution of events |

### How to Create a Visualisation
```
1. Go to Kibana → Visualise Library → Create New
2. Choose visualisation type
3. Select data source (index pattern)
4. Configure:
   - Metrics (Y-axis) → what to measure (count, sum, avg)
   - Buckets (X-axis) → what to group by (field, time interval)
5. Apply filters if needed
6. Save with a descriptive name
```

### Common SOC Visualisations
- **Top 10 source IPs** (bar chart → terms aggregation on `source.ip`)
- **Failed logins over time** (line chart → date histogram filtered on failure)
- **Event count by host** (data table → terms on `host.name`)
- **Geographic origin of traffic** (map → geo_point field)

---

## Creating Dashboards

- Dashboards = collections of saved visualisations on one screen
- Give SOC analysts real-time situational awareness at a glance

### Building a Dashboard
```
1. Kibana → Dashboards → Create New Dashboard
2. Add → Add from Library (pick saved visualisations)
3. Resize and arrange panels by dragging
4. Add filters at dashboard level (applies to all panels)
5. Set time range
6. Save dashboard
```

### Good SOC Dashboard Should Show
- Event volume over time (baseline + spikes)
- Top source IPs
- Top destination IPs/ports
- Failed authentication attempts
- Alerts triggered
- Geographic map of connections

---

## SOC Analyst Workflow in Kibana

```
1. RECEIVE ALERT
   → Open Kibana → Discover tab

2. SET TIME RANGE
   → Cover the alert window + buffer (e.g. 1 hour before/after)

3. INITIAL SEARCH
   → KQL search for the alerting host/IP/user
   host.name: "affected-host" AND event.outcome: "failure"

4. EXPAND SCOPE
   → Search all activity from the same source
   source.ip: "x.x.x.x"
   → Sort by _time ascending to build timeline

5. INVESTIGATE FIELDS
   → Click on event → expand document → examine all fields
   → Look for: process names, parent processes, file paths, DNS queries

6. PIVOT
   → If brute force → did they succeed? Search for success after failures
   → If malware → what processes were spawned? What connections made?

7. CORRELATE ACROSS INDEXES
   → Check multiple index patterns (windows, network, dns)

8. DOCUMENT
   → Screenshot relevant events
   → Note IOCs (IPs, hashes, domains, usernames)
   → Write timeline of events
```

---

## Elastic Stack vs Splunk Comparison

| Feature | Elastic Stack | Splunk |
|---|---|---|
| Cost | Open source (free core) | Commercial (expensive) |
| Query language | KQL / Lucene | SPL |
| Ease of setup | More complex | Easier |
| Scalability | Excellent | Excellent |
| Community | Large open source | Large enterprise |
| SOC usage | Very common | Industry standard |

---

## My Understanding in Plain English

Elastic Stack is Splunk's open-source rival — same concept, different implementation. Elasticsearch is the database that makes searching millions of log events fast, Logstash is the pipeline that cleans and routes logs, Beats are the lightweight collectors sitting on every endpoint, and Kibana is the window you actually look through. KQL is simpler than SPL but covers 95% of what you need for day-to-day SOC work. The real power comes from combining Discover for raw investigation, Visualise for spotting patterns, and Dashboards for maintaining situational awareness across the entire environment. In a real SOC you'll work with both Splunk and Elastic — the concepts are identical, only the syntax changes.

---

## Related Notes

- [[Introduction-to-SIEM]]
- [[Splunk-The-Basics]]
- [[Introduction-to-EDR]]
