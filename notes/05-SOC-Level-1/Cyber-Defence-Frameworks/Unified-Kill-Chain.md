# Unified Kill Chain

**Path:** SOC Level 1 - Cyber Defence Frameworks
**Date:** 2026-05-21
**Difficulty:** Easy

---

## What is a Kill Chain?

- A **Kill Chain** is a framework that describes the phases of an attack in sequential order
- Originally a military concept — "kill chain" describes the structure of an attack
- In cybersecurity, breaking any link = attack fails
- Helps defenders understand attacker behaviour and build detection at each phase

---

## What is Threat Modelling?

- **Threat Modelling** → structured process of identifying potential threats to a system and determining how to mitigate them
- Done **before** an attack happens — proactive security
- Answers: Who might attack us? How? What are they after? What's our weakest point?

### Threat Modelling Process
```
1. Identify Assets → what needs protecting?
2. Identify Threat Actors → who might attack?
3. Identify Attack Vectors → how might they attack?
4. Assess Impact → what's the damage if they succeed?
5. Prioritise → focus resources on highest risk
6. Mitigate → implement controls
```

### Common Threat Modelling Frameworks
- **STRIDE** → Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- **PASTA** → Process for Attack Simulation and Threat Analysis
- **DREAD** → Damage, Reproducibility, Exploitability, Affected Users, Discoverability

---

## Introducing the Unified Kill Chain

- Created by **Paul Pols** in 2017 — published in 2022
- Extends and unifies the **Lockheed Martin Cyber Kill Chain** and **MITRE ATT&CK**
- Contains **18 phases** organised into 3 goals
- More comprehensive than the original 7-stage Kill Chain
- Accounts for modern attacks — internal threats, supply chain, living-off-the-land

### Three Goals of UKC
```
GOAL 1: IN    → Initial Foothold    (get inside the target)
GOAL 2: THROUGH → Network Propagation  (move through the target)
GOAL 3: OUT   → Action on Objectives  (accomplish the mission)
```

### UKC vs Cyber Kill Chain vs MITRE ATT&CK

| Framework | Stages | Focus |
|---|---|---|
| Cyber Kill Chain | 7 | High-level attack phases |
| Unified Kill Chain | 18 | Detailed, modern attack lifecycle |
| MITRE ATT&CK | 14 tactics, 200+ techniques | Specific techniques per phase |

- UKC sits between Kill Chain (too high level) and ATT&CK (very granular)
- UKC maps directly to MITRE ATT&CK techniques within each phase

---

## Goal 1 — IN (Initial Foothold)

Getting the attacker's first foothold inside the target environment.

### Phase 1: Reconnaissance
- OSINT, email harvesting, LinkedIn, Shodan, Google Dorking
- Goal: understand the target before any interaction

### Phase 2: Weaponization
- Create malware, exploits, malicious documents
- Combine payload + exploit into deliverable weapon

### Phase 3: Social Engineering
- Phishing, spearphishing, pretexting, vishing (voice phishing)
- Manipulate humans to bypass technical controls
- Often the easiest way in — targets the human vulnerability

### Phase 4: Exploitation
- Execute exploit against a vulnerability
- Software CVEs, zero-days, macro execution, browser exploits

### Phase 5: Persistence
- Maintain access even after reboot/detection
- Registry run keys, scheduled tasks, services, startup folder

### Phase 6: Defence Evasion
- Avoid detection by security tools
- Obfuscation, timestomping, disabling AV, living-off-the-land (LOLBins)
- **LOLBins** → using legitimate Windows tools maliciously (certutil, mshta, wscript, regsvr32)

### Phase 7: Command & Control
- Establish C2 channel back to attacker
- HTTP/HTTPS, DNS tunneling, ICMP — blends in with normal traffic

### Phase 8: Pivoting
- Use compromised system as a launchpad to reach internal network
- SSH tunneling, SOCKS proxy, port forwarding

---

## Goal 2 — THROUGH (Network Propagation)

Moving laterally through the target network to reach high-value assets.

### Phase 9: Discovery
- Map the internal network from inside
- Enumerate hosts, services, users, shares, AD structure
- Tools: net commands, BloodHound, PowerView, Nmap

### Phase 10: Privilege Escalation
- Gain higher privileges (user → admin → SYSTEM/root)
- Exploit misconfigurations, kernel exploits, token impersonation
- **Windows:** SUID abuse, unquoted service paths, DLL hijacking
- **Linux:** SUID binaries, sudo misconfigurations, writable cron jobs

### Phase 11: Execution
- Run malicious code on target systems
- PowerShell, WMI, scheduled tasks, PsExec, LOLBins

### Phase 12: Credential Access
- Steal credentials to authenticate as legitimate users
- Mimikatz (LSASS dump), Kerberoasting, Pass-the-Hash, Pass-the-Ticket
- Keylogging, credential files, browser password stores

### Phase 13: Lateral Movement
- Move from compromised system to other systems in the network
- PsExec, WMI, RDP, SMB, SSH, stolen credentials
- Goal: reach domain controllers, file servers, databases

---

## Goal 3 — OUT (Action on Objectives)

Accomplishing the attacker's final mission.

### Phase 14: Collection
- Gather data of interest before exfiltration
- Screenshots, keylogging, file search, database queries
- Compress and encrypt data to prepare for exfiltration

### Phase 15: Exfiltration
- Move stolen data out of the network to attacker's infrastructure
- HTTP/HTTPS upload, DNS exfiltration, cloud storage (Dropbox, MEGA)
- Scheduled at times of low monitoring (nights, weekends)

### Phase 16: Impact
- Achieve the destructive/disruptive goal
- Ransomware encryption, data destruction, service disruption
- Wiper malware, database deletion, infrastructure sabotage

### Phase 17: Objectives
- Long-term strategic goals achieved
- Espionage (long-term silent access), financial gain, competitive intelligence
- Nation-state actors often stay in networks for months/years undetected

---

## UKC Full Phase Summary

| Goal | Phase | Description |
|---|---|---|
| **IN** | Reconnaissance | OSINT, target research |
| **IN** | Weaponization | Build malware/exploit |
| **IN** | Social Engineering | Phish/manipulate humans |
| **IN** | Exploitation | Trigger vulnerability |
| **IN** | Persistence | Maintain access |
| **IN** | Defence Evasion | Avoid detection |
| **IN** | Command & Control | Establish C2 |
| **IN** | Pivoting | Enter internal network |
| **THROUGH** | Discovery | Map internal network |
| **THROUGH** | Privilege Escalation | Gain higher privileges |
| **THROUGH** | Execution | Run malicious code |
| **THROUGH** | Credential Access | Steal credentials |
| **THROUGH** | Lateral Movement | Move across systems |
| **OUT** | Collection | Gather target data |
| **OUT** | Exfiltration | Move data out |
| **OUT** | Impact | Ransomware/destruction |
| **OUT** | Objectives | Final strategic goal |

---

## Why UKC Matters for SOC Analysts

- When an alert fires, map it to the UKC phase immediately
- Knowing the phase tells you: **how far into the attack are they?**
- Phase 1-3 → early stage, attacker hasn't gained access yet
- Phase 9-13 → attacker is inside, moving laterally — urgent response needed
- Phase 14-17 → attacker at final stage — contain immediately

### Alert Triage Using UKC
```
Alert fires → Identify UKC phase → Assess urgency → Respond accordingly

Phase IN  → Investigate, monitor, may still prevent
Phase THROUGH → Urgent — attacker inside network, isolate affected hosts
Phase OUT → Critical — data may already be stolen, full incident response
```

---

## My Understanding in Plain English

The Unified Kill Chain fixes the main problem with the original Cyber Kill Chain — the original 7 stages are too high level to be actionable for modern attacks. The UKC's 18 phases map exactly to what attackers actually do step by step, and each phase maps to specific MITRE ATT&CK techniques so defenders know exactly what to detect. The three-goal structure (IN, THROUGH, OUT) is the most useful mental model — when you're investigating an alert, the first question is always "which goal is the attacker at?" because that tells you both the urgency and what they're likely to do next. An attacker in the THROUGH phase is already inside and moving laterally — that's a very different response than detecting them in the IN phase during delivery.

---

## Related Notes

- [[Cyber-Kill-Chain]]
- [[Pyramid-of-Pain]]
- [[Introduction-to-SIEM]]
- [[Introduction-to-SOAR]]
- [[Passive-Reconnaissance]]
