# Splunk: The Basics

**Path:** SOC Level 1 - Core SOC Solutions
**Date:** 2026-05-21
**Difficulty:** Easy

---

## Splunk Components

Splunk has three core components that work together:

| Component | Role |
|---|---|
| **Splunk Forwarder** | Lightweight agent installed on endpoints/servers — collects and forwards logs to indexer |
| **Splunk Indexer** | Receives, parses, normalises, and stores log data — makes it searchable |
| **Search Head** | The UI analysts use to search, visualise, and create dashboards from indexed data |

### Data Flow
```
Log Source → Forwarder → Indexer (parse + store) → Search Head (query + visualise)
```

- **Universal Forwarder** → sends raw data, minimal processing
- **Heavy Forwarder** → can parse/filter data before sending
- In small deployments, all three components run on one machine
- In enterprise, each component is separate for scalability

---

## Navigating Splunk

### Main Menu Areas

| Section | Purpose |
|---|---|
| **Search & Reporting** | Primary interface for writing SPL queries and investigating |
| **Dashboards** | Pre-built and custom visual panels |
| **Alerts** | Manage and view triggered alerts |
| **Reports** | Saved searches run on schedule |
| **Datasets** | Browse indexed data sources |
| **Settings** | Manage inputs, indexes, users, forwarders |

### Search & Reporting Interface
- **Search Bar** → where you write SPL queries
- **Time Picker** → filter events by time range (last 15 min, 24hrs, custom)
- **Events Tab** → raw log results
- **Statistics Tab** → tabular results from transforming commands
- **Visualisation Tab** → charts and graphs from search results
- **Field Sidebar** → shows all fields extracted from results (left side)

---

## Adding Data

### Data Input Methods
- **Upload** → manually upload a log file (CSV, JSON, syslog) — good for testing
- **Monitor** → watch a file/directory on the Splunk server continuously
- **Forward** → receive data from Splunk Forwarders (production use)

### Source Types
- Splunk uses **source types** to identify the format of incoming data
- Common source types: `WinEventLog`, `syslog`, `access_combined` (Apache), `json`
- Correct source type ensures proper field extraction

### Key Fields in Every Event
| Field | Description |
|---|---|
| `_time` | Timestamp of the event |
| `host` | The device that generated the log |
| `source` | The file or input the log came from |
| `sourcetype` | The format/type of the log |
| `index` | Which Splunk index stores this data |

---

## SPL — Splunk Search Processing Language

### Basic Search Syntax
```spl
index=<indexname> key=value key2=value2
```

### Essential Commands

| Command | Use | Example |
|---|---|---|
| `search` | Filter events | `index=windows EventCode=4625` |
| `stats` | Aggregate/count | `stats count by user` |
| `table` | Show specific fields | `table _time, user, src_ip` |
| `sort` | Sort results | `sort -count` (descending) |
| `dedup` | Remove duplicates | `dedup user` |
| `rex` | Extract fields with regex | `rex field=_raw "user=(?P<username>\w+)"` |
| `eval` | Create new field | `eval risk=if(count>10,"high","low")` |
| `where` | Filter after stats | `where count > 5` |
| `top` | Most common values | `top limit=10 src_ip` |
| `rare` | Least common values | `rare user` |
| `timechart` | Time-based chart | `timechart count by EventCode` |
| `rename` | Rename a field | `rename src_ip as "Source IP"` |
| `fields` | Include/exclude fields | `fields user, src_ip, _time` |
| `head` | First N results | `head 20` |
| `tail` | Last N results | `tail 20` |

### Search Operators
```spl
AND   → index=windows user=admin AND EventCode=4625
OR    → index=windows EventCode=4625 OR EventCode=4624
NOT   → index=windows NOT user=system
*     → wildcard: user=adm*
""    → exact phrase: "failed login"
```

---

## SOC Analyst Workflow in Splunk

### Standard Investigation Workflow
```
1. RECEIVE ALERT
   → Alert fires with basic details (time, rule name, affected host)

2. OPEN SEARCH & REPORTING
   → Set time range to cover the alert window

3. INITIAL SEARCH
   → Search for the specific event that triggered the alert
   index=windows host="affected-host" EventCode=4625

4. EXPAND SCOPE
   → Look at all activity from the same source/user/IP
   index=* src_ip="x.x.x.x" | stats count by sourcetype, EventCode

5. BUILD TIMELINE
   → Order events chronologically to understand the attack chain
   index=* src_ip="x.x.x.x" | sort _time | table _time, EventCode, user, host

6. PIVOT & CORRELATE
   → Follow the attacker's footsteps across different log sources
   → Did the brute force succeed? Did they log in? What did they do after?

7. DETERMINE SCOPE
   → How many systems affected? How many accounts compromised?

8. DOCUMENT & ESCALATE
   → Write up findings with timeline, IOCs, affected systems
   → Escalate if True Positive
```

---

## Common SOC Searches in Splunk

### Authentication & Brute Force
```spl
# Failed logins (Windows)
index=windows EventCode=4625
| stats count by user, src_ip
| sort -count

# Successful login after multiple failures (brute force success)
index=windows (EventCode=4625 OR EventCode=4624)
| stats count(eval(EventCode=4625)) as failures,
        count(eval(EventCode=4624)) as successes by user
| where failures > 5 AND successes > 0

# Login outside business hours
index=windows EventCode=4624
| eval hour=strftime(_time, "%H")
| where hour < 8 OR hour > 18
```

### Network & Reconnaissance
```spl
# Top talkers by source IP
index=firewall
| stats count by src_ip
| sort -count
| head 20

# Connections to suspicious port
index=firewall dest_port=4444
| table _time, src_ip, dest_ip, dest_port

# DNS queries to new/rare domains
index=dns
| rare limit=20 query
```

### Malware & Process Activity
```spl
# Suspicious parent-child process (Word spawning PowerShell)
index=windows EventCode=4688
| where ParentProcessName="winword.exe" AND NewProcessName="powershell.exe"

# Encoded PowerShell (common malware indicator)
index=windows EventCode=4688 CommandLine="*-EncodedCommand*"
| table _time, host, user, CommandLine

# Process running from temp directory
index=windows EventCode=4688 NewProcessName="*\\Temp\\*"
```

---

## Useful Splunk Tips

- Use **time picker** first — always narrow down time range before searching
- `index=*` searches all indexes — slow; specify index when possible
- Use **field sidebar** to quickly identify useful fields in your results
- `| head 1` to check what fields exist in your data before writing complex queries
- Save frequent searches as **reports** to run on schedule
- Use **transaction** command to group related events into a single session:
  ```spl
  index=windows EventCode=4625
  | transaction user maxspan=5m
  | where eventcount > 5
  ```

---

## My Understanding in Plain English

Splunk is where a SOC analyst spends most of their working day. Everything that happens on the network eventually ends up here as a log entry, and SPL is the language you use to ask questions about those logs. The key mental shift is thinking about logs as a database — you're not reading them line by line, you're querying them. A brute force attack that generates 10,000 failed logins becomes one line when you run `stats count by user` — and the one account that then successfully logs in stands out immediately. The workflow is always the same: start with the alert, search for the triggering event, expand outward to build the full picture, then decide if it's real and how bad it is.

---

## Related Notes

- [[Introduction-to-SIEM]]
- [[Introduction-to-EDR]]
- [[Protocols-and-Servers-Combined]]
- [[Enumeration-and-Brute-Force]]
