# Pyramid of Pain

**Path:** SOC Level 1 - Cyber Defence Frameworks
**Date:** 2026-05-21
**Difficulty:** Easy

---

## What is the Pyramid of Pain?

- Created by **David Bianco** — a model that ranks Indicators of Compromise (IOCs) by how difficult they are for an attacker to change
- Higher up the pyramid = more pain for the attacker if defenders detect and block it
- Helps SOC teams prioritise **which IOCs are worth hunting** and which are trivial
- The pyramid is read **bottom to top** — trivial at the bottom, devastating at the top

```
        /\
       /  \          TTPs          ← Tough (most painful)
      /----\
     /      \        Tools         ← Challenging
    /--------\
   /          \      Network Artifacts  ← Annoying
  /------------\
 /              \    Host Artifacts    ← Annoying
/----------------\
/                 \  Domain Names      ← Simple
\-----------------/
 \               /   IP Addresses      ← Easy
  \-------------/
   \           /     Hash Values       ← Trivial (least painful)
    \---------/
```

---

## Level 1 — Hash Values (Trivial)

- **What it is:** MD5, SHA1, SHA256 hash of a malicious file
- **Pain for attacker:** Trivial — change one byte of the file → completely different hash
- **How defenders use it:** Block known malicious hashes in AV/EDR, search SIEM for hash matches

### Tools for Hash Lookup
- **VirusTotal** → paste hash → see detection ratio across 70+ AV engines
- **MalwareBazaar** → malware sample database
- **Hybrid Analysis** → sandbox + hash lookup

### Why It's Trivial
- Attacker just recompiles the malware → new hash instantly
- "Fuzzy hashing" (ssdeep) helps — finds similar files even after minor changes
- Hash blocking is still worth doing — stops lazy/script kiddie attackers

---

## Level 2 — IP Addresses (Easy)

- **What it is:** IP address used by attacker (C2 server, phishing host, scanner)
- **Pain for attacker:** Easy — spin up new VPS, use proxy/VPN, rotate IPs
- **How defenders use it:** Block on firewall, search SIEM for connections to malicious IP

### Fast Flux
- Attackers use **fast flux DNS** — domain resolves to different IPs constantly
- Makes IP blocking ineffective — block one IP, attacker uses another within minutes
- Why IP blocking alone is not enough

### Tools for IP Lookup
- **AbuseIPDB** → community-reported abuse
- **Shodan** → what services are running on that IP
- **VirusTotal** → IP reputation
- **Cisco Talos** → threat intelligence on IPs

---

## Level 3 — Domain Names (Simple)

- **What it is:** Domain used for C2, phishing, malware delivery
- **Pain for attacker:** Simple but slightly more effort — register new domain, update DNS
- **How defenders use it:** Block on DNS/proxy, hunt for domain in logs

### Punycode Attacks
- Attackers register lookalike domains using Unicode characters
- Example: `adıdas.com` looks like `adidas.com` but is different
- Browsers show the punycode version: `xn--addas-o4a.com`

### Detection Tips
- Look for newly registered domains (< 30 days old)
- Check for typosquatting (missing/swapped letters)
- Analyse DNS query logs for unusual domains
- Tools: **URLScan.io**, **VirusTotal**, **Cisco Umbrella**

---

## Level 4 — Host Artifacts (Annoying)

- **What it is:** Traces left on the compromised system
  - Registry keys created/modified
  - Suspicious files dropped (temp directory, AppData)
  - Scheduled tasks created
  - Services installed
  - Prefetch files
  - Event log entries
- **Pain for attacker:** Annoying — changing these requires significant malware rewrite
- **How defenders use it:** Hunt for artifacts in EDR, run YARA rules

### Common Host Artifacts to Hunt
```
Registry: HKCU\Software\Microsoft\Windows\CurrentVersion\Run  (persistence)
Files:    C:\Users\*\AppData\Roaming\*.exe
          C:\Windows\Temp\*.exe
          C:\ProgramData\*.dll
Tasks:    Unusual scheduled tasks in Task Scheduler
Services: Newly installed services with suspicious names
```

---

## Level 5 — Network Artifacts (Annoying)

- **What it is:** Observable patterns in network traffic
  - Unusual User-Agent strings
  - C2 beacon patterns (regular intervals, specific URIs)
  - Specific URI patterns in HTTP traffic
  - DNS query patterns
  - Unusual protocols/ports
- **Pain for attacker:** Annoying — requires retooling C2 infrastructure
- **How defenders use it:** Write SIEM/IDS rules for specific patterns

### Examples
```
Unusual User-Agent: "python-requests/2.25.1"  (not a browser)
C2 beacon: HTTP GET to /update.php every 60 seconds exactly
DNS: Querying randomly generated domains (DGA — Domain Generation Algorithm)
```

---

## Level 6 — Tools (Challenging)

- **What it is:** Software and utilities used by the attacker
  - Custom malware
  - Offensive tools (Mimikatz, Cobalt Strike, Metasploit)
  - Scripts (PowerShell, Python)
  - Living-off-the-land binaries (LOLBins — certutil, wscript, mshta)
- **Pain for attacker:** Challenging — forces attacker to develop or buy new tools, retrain
- **How defenders use it:** YARA rules, AV signatures, behaviour-based detection

### YARA Rules
- Pattern-matching language for identifying malware
- Can match on strings, byte patterns, file headers
```yara
rule detect_mimikatz {
    strings:
        $s1 = "mimikatz" nocase
        $s2 = "sekurlsa" nocase
    condition:
        any of them
}
```

### Detecting Tool Usage
- Cobalt Strike → look for default beacon URIs, specific SSL certificates
- Mimikatz → LSASS access, specific event IDs (4656, 4663)
- PowerShell Empire → encoded commands, specific HTTP patterns

---

## Level 7 — TTPs (Tough)

- **What it is:** Tactics, Techniques, and Procedures — **how** the attacker operates
  - Their methodology, approach, attack chain
  - Mapped to **MITRE ATT&CK framework**
- **Pain for attacker:** Devastating — forces complete change of attack methodology, retraining, new infrastructure
- **How defenders use it:** Behaviour-based detection, threat hunting, purple team exercises

### MITRE ATT&CK Connection
- TTPs map directly to MITRE ATT&CK techniques
- Example TTP: "Attacker uses spearphishing → macro-enabled Office doc → PowerShell → Cobalt Strike"
- Blocking at TTP level = blocking the entire attack chain, not just one indicator

### Why TTPs Are Most Valuable
- Hash changes → trivial
- IP changes → spin up new VPS in 5 minutes
- Domain changes → register new domain in 10 minutes
- TTPs change → months of development, retraining, cost

---

## Pyramid Summary

| Level | IOC Type | Attacker Effort to Change | Defender Value |
|---|---|---|---|
| 1 | Hash Values | Trivial (seconds) | Low |
| 2 | IP Addresses | Easy (minutes) | Low |
| 3 | Domain Names | Simple (hours) | Medium |
| 4 | Host Artifacts | Annoying (days) | Medium-High |
| 5 | Network Artifacts | Annoying (days) | Medium-High |
| 6 | Tools | Challenging (weeks) | High |
| 7 | TTPs | Tough (months) | Very High |

---

## SOC Application

- **Reactive defence (bottom of pyramid):** Block hashes and IPs as they appear — fast but easily bypassed
- **Proactive defence (top of pyramid):** Hunt for TTPs and tool usage — harder to evade
- Best SOC teams combine both:
  - Automate hash/IP/domain blocking via SOAR
  - Use threat hunting to find TTPs before alerts fire

---

## My Understanding in Plain English

The Pyramid of Pain is a mindset shift for defenders. Blocking a malicious IP feels productive but costs the attacker nothing — they switch to a new one immediately. The real value is in detecting how attackers operate — their techniques, their tools, their patterns. If you can detect that someone is using Mimikatz to dump credentials, it doesn't matter what hash or IP they're using. The attacker now has to completely change their approach, which takes weeks or months and costs money. Build detections that target the top of the pyramid and you're playing defence that actually hurts adversaries.

---

## Related Notes

- [[Introduction-to-SIEM]]
- [[Introduction-to-SOAR]]
- [[Introduction-to-EDR]]
- [[Splunk-The-Basics]]
