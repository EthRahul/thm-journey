# Introduction to EDR

**Path:** SOC Level 1 - Core SOC Solutions
**Date:** 2026-05-20
**Difficulty:** Easy

---

## What is an EDR?

- **EDR (Endpoint Detection and Response)** → security solution that monitors endpoints (laptops, servers, workstations) for malicious activity
- Endpoints = any device connecting to a network
- EDR goes beyond traditional antivirus — it detects, investigates, and responds to threats in real time
- Works by collecting and analysing telemetry data from endpoints continuously

---

## Beyond the Antivirus

### Antivirus vs EDR

| Feature | Antivirus | EDR |
|---|---|---|
| Detection method | Signature-based | Behaviour + signature + AI |
| Unknown threats | ❌ Misses zero-days | ✅ Detects anomalous behaviour |
| Visibility | File scanning only | Full endpoint telemetry |
| Response | Quarantine/delete | Isolate, kill process, remediate |
| Forensics | ❌ None | ✅ Full attack timeline |
| Real-time monitoring | Limited | Continuous |

- Antivirus = **reactive** (known signatures)
- EDR = **proactive** (behaviour analysis, even for unknown threats)
- Modern threats use fileless malware, living-off-the-land techniques — antivirus misses these, EDR catches them

---

## How an EDR Works

### Core Process
```
1. Agent installed on endpoint (lightweight software)
2. Agent continuously collects telemetry data
3. Data sent to central EDR console/cloud
4. Analysis engine processes data (rules + ML + behaviour)
5. Alert generated if suspicious activity detected
6. SOC analyst investigates alert
7. Response action taken (isolate, kill, remediate)
```

### Key Components
- **EDR Agent** → installed on every endpoint, collects data locally
- **Central Console** → where analysts monitor all endpoints from one place
- **Detection Engine** → analyses telemetry using rules, signatures, and ML
- **Response Module** → allows remote actions on endpoints
- **Threat Intelligence Feed** → continuously updated IOCs (Indicators of Compromise)

---

## EDR Telemetry

Telemetry = all the data the EDR agent collects from the endpoint

### Types of Telemetry Collected

| Telemetry Type | What it captures |
|---|---|
| **Process events** | Every process created, parent-child relationships |
| **File events** | Files created, modified, deleted, renamed |
| **Network events** | Connections made, DNS queries, IPs contacted |
| **Registry events** | Registry keys read/written/deleted (Windows) |
| **User logon events** | Login/logout, failed attempts, privilege escalation |
| **Memory events** | Code injection, suspicious memory allocation |
| **Script execution** | PowerShell, Python, Bash commands run |

- This telemetry is what makes EDR so powerful — it creates a **complete timeline** of everything that happened on an endpoint
- Attackers can't hide their tracks easily because every action generates telemetry

---

## Detection and Response Capabilities

### Detection Methods
- **Signature-based** → matches known malware hashes/patterns
- **Behaviour-based** → flags unusual process behaviour (e.g. Word spawning PowerShell)
- **Anomaly-based** → baseline normal behaviour → alert on deviations
- **Threat Intelligence** → matches IOCs from global threat feeds
- **MITRE ATT&CK mapping** → maps detections to known attacker techniques

### Response Actions (what SOC analyst can do remotely)
- **Isolate endpoint** → cut it off from network while keeping EDR connection
- **Kill process** → terminate malicious process immediately
- **Quarantine file** → move malicious file to safe container
- **Delete file** → remove malicious artifact
- **Run script** → deploy remediation script remotely
- **Collect forensic data** → pull memory dump, logs for investigation

### Common EDR Tools
| Tool | Vendor |
|---|---|
| CrowdStrike Falcon | CrowdStrike |
| Microsoft Defender for Endpoint | Microsoft |
| SentinelOne | SentinelOne |
| Carbon Black | VMware |
| Cybereason | Cybereason |
| Elastic EDR | Elastic |

---

## Investigate an Alert on EDR

### Alert Investigation Workflow
```
1. Alert fires → SOC analyst receives notification
2. Check alert details:
   - Which endpoint?
   - Which process triggered it?
   - What parent process spawned it?
   - What network connections were made?
   - What files were created/modified?
3. Correlate with other events on the same timeline
4. Determine: True Positive or False Positive?
5. If TP → contain (isolate endpoint) → investigate scope → remediate
6. If FP → tune detection rule to reduce noise
```

### Key Things to Look For
- **Unusual parent-child process relationships**
  - e.g. `winword.exe` → `powershell.exe` = suspicious
  - e.g. `explorer.exe` → `cmd.exe` = could be normal
- **Process running from unusual location**
  - e.g. `svchost.exe` running from `C:\Users\Public\` = suspicious
- **Outbound connections to unknown IPs**
- **Encoded PowerShell commands** (`-EncodedCommand`)
- **Processes accessing LSASS** (credential dumping)

---

## My Understanding in Plain English

EDR is essentially a CCTV system for your endpoints — it records everything that happens and alerts you when something looks wrong. Unlike antivirus which only catches known threats by matching signatures, EDR watches behaviour. If a Word document suddenly opens a command prompt and starts downloading files, EDR sees that chain of events even if the malware itself has never been seen before. As a SOC analyst, EDR is your primary tool for understanding what an attacker actually did on a system — the telemetry gives you the full story from initial execution through every file, process, and network connection the attacker touched.

---

## Related Notes

- [[Passive-Reconnaissance]]
- [[Active-Reconnaissance]]
- [[Nmap-Basic-Port-Scans]]
