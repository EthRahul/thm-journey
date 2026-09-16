# Introduction to SIEM

**Path:** SOC Level 1 - Core SOC Solutions
**Date:** 2026-05-20
**Difficulty:** Easy

---

## Logs Everywhere, Answers Nowhere

- Every device on a network generates logs — firewalls, servers, endpoints, applications
- Problem: logs are scattered across hundreds of different systems in different formats
- A SOC analyst can't manually check every log source for every incident
- **Without centralisation:** slow detection, missed attacks, no correlation between events

### Types of Logs Generated

| Source | What it logs |
|---|---|
| **Windows Event Logs** | Logins, process creation, registry changes, errors |
| **Linux Syslogs** | Auth events, service activity, kernel messages |
| **Firewall logs** | Allowed/denied connections, IPs, ports, protocols |
| **Web server logs** | HTTP requests, URLs, response codes, user agents |
| **DNS logs** | Queries, responses, domain lookups |
| **Proxy logs** | Web traffic, URLs visited, file downloads |
| **IDS/IPS logs** | Signature matches, blocked attacks |
| **EDR logs** | Process events, file events, network events |
| **Application logs** | Login attempts, errors, transactions |

---

## Why SIEM?

- **SIEM (Security Information and Event Management)** → centralised platform that collects, aggregates, correlates, and analyses logs from all sources in real time
- Gives SOC analysts a **single pane of glass** to monitor the entire environment
- Enables **correlation** — connecting events across different sources to detect attacks that span multiple systems

### What SIEM Does

| Capability | Description |
|---|---|
| **Log Collection** | Ingests logs from all sources via agents or syslog |
| **Normalisation** | Converts different log formats into a standard schema |
| **Correlation** | Links related events across sources to detect attack patterns |
| **Alerting** | Fires alerts when correlation rules match suspicious activity |
| **Dashboards** | Visual overview of security posture in real time |
| **Threat Hunting** | Analysts search historical data for hidden threats |
| **Compliance** | Stores logs for audit and regulatory requirements |

### SIEM vs EDR
- **EDR** → deep visibility into a single endpoint
- **SIEM** → broad visibility across the entire environment
- They complement each other — EDR feeds telemetry into SIEM

---

## Log Sources and Ingestion

### How Logs Get Into SIEM

**Method 1 — Agent-based**
- Lightweight agent installed on each endpoint/server
- Agent collects and forwards logs to SIEM in real time
- More control, better filtering at source
- Examples: Beats (Elastic), Splunk Universal Forwarder

**Method 2 — Agentless (Syslog)**
- Device sends logs directly to SIEM via Syslog protocol (UDP/TCP port 514)
- Used for network devices (routers, switches, firewalls) that can't run agents
- Less overhead but less control

**Method 3 — API Integration**
- SIEM pulls logs from cloud services via API
- Used for cloud platforms (AWS, Azure, Office 365, Okta)

### Log Ingestion Pipeline
```
Log Source → Collection (Agent/Syslog/API) → Parsing → Normalisation → Indexing → Storage → Query/Alert
```

### Common SIEM Platforms
| Platform | Type |
|---|---|
| **Splunk** | Commercial — industry standard |
| **Microsoft Sentinel** | Cloud-native (Azure) |
| **IBM QRadar** | Enterprise commercial |
| **Elastic SIEM** | Open source based |
| **Wazuh** | Open source |
| **AlienVault OSSIM** | Open source |

---

## Alerting Process and Analysis

### How Alerts Work
- SOC team writes **correlation rules** — logic that triggers an alert when specific conditions are met across log sources
- Example rule: "Alert if same user fails login 5 times in 60 seconds then succeeds"

### Types of Detection Rules
- **Threshold-based** → alert after N events in X time (brute force detection)
- **Blacklist-based** → alert if known malicious IP/domain/hash seen
- **Correlation-based** → alert when multiple events from different sources match a pattern
- **Anomaly-based** → alert when behaviour deviates from baseline

### Alert Triage Process
```
1. Alert fires in SIEM
2. Analyst reviews alert details:
   - Source IP / Destination IP
   - User account involved
   - Time and frequency
   - Raw log entries
3. Pivot to related events → build timeline
4. Determine: True Positive or False Positive?
5. TP → escalate / respond / contain
6. FP → tune the rule to reduce noise
```

### Alert Severity Levels
| Level | Meaning |
|---|---|
| **Critical** | Active compromise, immediate response needed |
| **High** | Strong indicators of attack, urgent investigation |
| **Medium** | Suspicious activity, investigate within hours |
| **Low** | Informational, review when possible |

### Key Metrics SOC Tracks
- **MTTD** (Mean Time to Detect) → how long to spot an attack
- **MTTR** (Mean Time to Respond) → how long to contain/remediate
- **False Positive Rate** → % of alerts that are not real threats
- **Alert Volume** → total alerts generated per day

---

## Key SIEM Concepts for SOC Analysts

### Important Terms
- **IOC (Indicator of Compromise)** → evidence of attack (malicious IP, hash, domain)
- **TTPs (Tactics, Techniques, Procedures)** → how attackers operate (MITRE ATT&CK)
- **Use Case** → a specific detection scenario built as a SIEM rule
- **Playbook** → step-by-step response procedure for a specific alert type
- **Normalisation** → converting different log formats to a common field structure
- **Parsing** → extracting structured fields from raw log text

### Common SIEM Searches (Splunk syntax as example)
```spl
# Find all failed logins
index=windows EventCode=4625

# Find logins from a specific IP
index=* src_ip="10.10.10.5"

# Count events by user
index=windows EventCode=4625 | stats count by user

# Find DNS queries to suspicious domain
index=dns query="*.evil.com"

# Detect brute force (5+ failures in 60s)
index=windows EventCode=4625 | bucket span=60s _time | stats count by user _time | where count > 5
```

---

## My Understanding in Plain English

SIEM is the nerve centre of a SOC — it's where all the noise from hundreds of log sources gets turned into actionable alerts. The challenge isn't collecting logs, it's making sense of them. A single brute force attack might show up as failed logins in Windows Event Logs, firewall blocks, and authentication errors in the application logs all at the same time — SIEM correlates all three and fires one alert instead of three separate ones from different tools. As a SOC analyst, your job is to take that alert, trace it back through the raw logs, build a timeline of what actually happened, and decide whether it's a real attack or a false alarm. The faster and more accurately you do that, the lower your MTTD and MTTR — the two numbers that define how effective a SOC actually is.

---

## Related Notes

- [[Introduction-to-EDR]]
- [[Passive-Reconnaissance]]
- [[Active-Reconnaissance]]
- [[Protocols-and-Servers-Combined]]
