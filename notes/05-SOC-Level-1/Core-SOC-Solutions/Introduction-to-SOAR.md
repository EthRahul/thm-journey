# Introduction to SOAR

**Path:** SOC Level 1 - Core SOC Solutions
**Date:** 2026-05-21
**Difficulty:** Easy

---

## Traditional SOC and Challenges

### How a Traditional SOC Operates
- Analysts manually monitor alerts from SIEM, EDR, firewalls, threat intel feeds
- Each alert is triaged, investigated, and responded to manually
- Playbooks exist as documents — analysts follow them step by step by hand

### Core Problems with Traditional SOC

| Challenge | Impact |
|---|---|
| **Alert fatigue** | SOCs receive thousands of alerts daily — analysts can't keep up |
| **Manual processes** | Repetitive tasks (IP lookup, hash check, user lookup) waste analyst time |
| **Slow response** | Manual triage and response takes hours — attackers move in minutes |
| **Human error** | Manual steps get missed, especially under pressure |
| **Tool sprawl** | Analysts jump between 10+ tools — no single workflow |
| **Inconsistency** | Different analysts handle same alert type differently |
| **Burnout** | High alert volume + repetitive work = analyst turnover |

### The Numbers Problem
- A mid-sized SOC can receive **10,000+ alerts per day**
- Analysts can realistically investigate **~50 alerts per shift**
- Without automation → most alerts go uninvestigated → missed breaches

---

## Overcoming SOC Challenges with SOAR

- **SOAR (Security Orchestration, Automation, and Response)** → platform that automates repetitive SOC tasks and orchestrates tools to work together
- SOAR doesn't replace analysts — it frees them to focus on complex investigations
- Three pillars:

| Pillar | Meaning |
|---|---|
| **Orchestration** | Connects and coordinates all security tools into unified workflows |
| **Automation** | Executes repetitive tasks automatically without human input |
| **Response** | Takes response actions (block IP, isolate host, reset password) automatically or with one-click approval |

### What SOAR Automates

**Enrichment (most common):**
- IP reputation lookup (VirusTotal, AbuseIPDB, Shodan)
- Domain/URL reputation check
- File hash lookup (VirusTotal, MalwareBazaar)
- User account lookup (Active Directory, HR system)
- Geolocation of source IP
- Threat intel feed correlation

**Response Actions:**
- Block IP on firewall automatically
- Isolate endpoint via EDR
- Disable compromised user account
- Reset user password
- Create ticket in ticketing system (Jira, ServiceNow)
- Send notification to analyst (email, Slack, Teams)

### SOAR vs SIEM

| Feature | SIEM | SOAR |
|---|---|---|
| Primary function | Detect + alert | Respond + automate |
| Data storage | Yes | No (pulls from SIEM) |
| Manual effort | High | Low |
| Tool integration | Limited | Extensive |
| Response capability | Basic | Full automation |

- They work **together** — SIEM generates the alert, SOAR acts on it

### Popular SOAR Platforms
| Platform | Notes |
|---|---|
| **Splunk SOAR (Phantom)** | Most widely used, integrates with Splunk SIEM |
| **Palo Alto XSOAR** | Enterprise grade, extensive integrations |
| **Microsoft Sentinel** | Built-in SOAR (Logic Apps) for Azure environments |
| **TheHive + Cortex** | Open source — popular in smaller SOCs |
| **Shuffle** | Open source SOAR, community driven |

---

## Building SOAR Playbooks

- **Playbook** → automated workflow that defines what actions to take when a specific alert type fires
- Visual, drag-and-drop in most SOAR platforms
- Replaces a human following a Word document checklist

### Playbook Structure
```
TRIGGER (alert type) 
    ↓
ENRICHMENT (gather context automatically)
    ↓
DECISION (is it malicious? based on enrichment results)
    ↓
RESPONSE (automated action OR analyst approval required)
    ↓
NOTIFICATION (update ticket, alert analyst, close case)
```

### Example Playbook — Phishing Email Alert

```
TRIGGER: Email flagged as phishing by email gateway

ENRICHMENT:
→ Extract sender domain → check reputation (VirusTotal)
→ Extract URLs in email → check each URL (VirusTotal, URLScan)
→ Extract attachments → check file hash (VirusTotal)
→ Look up recipient user (Active Directory)

DECISION:
→ If domain/URL/hash is malicious → HIGH confidence phishing
→ If all clean → LOW confidence → send to analyst for review

RESPONSE (HIGH confidence):
→ Delete email from all mailboxes automatically
→ Block sender domain on email gateway
→ Block malicious URLs on proxy/firewall
→ Notify affected user
→ Create incident ticket

NOTIFICATION:
→ Slack message to SOC channel with summary
→ Ticket created in Jira with all enrichment data attached
```

### Example Playbook — Brute Force Alert

```
TRIGGER: SIEM alert — 10+ failed logins for same account in 5 minutes

ENRICHMENT:
→ Look up source IP → reputation check (AbuseIPDB)
→ Look up target user → is account sensitive? (AD lookup)
→ Check if source IP is known VPN/proxy (threat intel)
→ Check if login eventually succeeded (SIEM query)

DECISION:
→ Login succeeded + IP is malicious → HIGH → auto-respond
→ Login failed + IP is cloud provider → likely FP → close
→ Login succeeded + IP unknown → MEDIUM → analyst review

RESPONSE (HIGH):
→ Disable compromised account (Active Directory)
→ Block source IP on firewall
→ Force password reset
→ Notify user + manager

NOTIFICATION:
→ Create P1 incident ticket
→ Page on-call analyst
```

---

## Threat Intel Workflow Practical

### Threat Intelligence in SOAR Context
- SOAR integrates with **Threat Intelligence Platforms (TIPs)** to enrich alerts automatically
- Common TI sources integrated into SOAR:
  - **VirusTotal** → file/URL/IP/domain reputation
  - **AbuseIPDB** → IP abuse reports
  - **MalwareBazaar** → malware sample database
  - **Shodan** → internet-facing device info
  - **MISP** → open source threat intel sharing platform
  - **AlienVault OTX** → community threat intel

### IOC Enrichment Workflow
```
1. Alert arrives with raw IOC (IP, hash, domain, URL)
2. SOAR automatically queries TI sources
3. Results aggregated into case:
   - Detection ratio (X/90 engines flagged)
   - First/last seen
   - Malware family (if known)
   - Associated campaigns/threat actors
4. Score assigned (benign/suspicious/malicious)
5. Decision made based on score
6. Response triggered automatically or escalated
```

---

## Key SOAR Concepts Summary

| Term | Definition |
|---|---|
| **Playbook** | Automated response workflow for a specific alert type |
| **Action** | Single automated task (lookup IP, block user, send email) |
| **Integration** | Connection between SOAR and a security tool (EDR, firewall, SIEM) |
| **Case** | Container for all data related to one incident |
| **Enrichment** | Automatically gathering context about an IOC or event |
| **MTTD** | Mean Time to Detect — how long to spot an attack |
| **MTTR** | Mean Time to Respond — how long to contain/fix |

---

## My Understanding in Plain English

SOAR is what happens when a SOC gets tired of analysts spending 80% of their time doing repetitive lookups. Every time an alert fires, someone was manually copying an IP into VirusTotal, then AbuseIPDB, then checking Active Directory for the user, then writing up the results in a ticket — the same steps, hundreds of times a day. SOAR automates all of that. The analyst now sees a pre-enriched alert with all the context already gathered, a confidence score, and a one-click response button. The result: analysts investigate more alerts, respond faster, make fewer mistakes, and don't burn out. MTTD and MTTR drop dramatically. The playbook is the core concept — build it once, run it thousands of times consistently.

---

## Related Notes

- [[Introduction-to-SIEM]]
- [[Introduction-to-EDR]]
- [[Splunk-The-Basics]]
- [[Elastic-Stack-The-Basics]]
