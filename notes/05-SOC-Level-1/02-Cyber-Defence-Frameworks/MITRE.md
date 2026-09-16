# MITRE

**Path:** SOC Level 1 - Cyber Defence Frameworks
**Date:** 2026-05-21
**Difficulty:** Easy

---

## What is MITRE?

- **MITRE** → non-profit organisation that works with US government agencies
- Maintains several free, publicly available cybersecurity knowledge bases and frameworks
- Core mission: advance security for defenders worldwide
- Their resources are used by every major SOC, threat intel team, and red team globally

---

## ATT&CK Framework

- **ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge)** → knowledge base of real-world attacker behaviour
- Built from analysis of actual threat actor campaigns and malware
- URL: **https://attack.mitre.org**
- Constantly updated as new techniques are discovered

### ATT&CK Structure

```
Tactics → Techniques → Sub-techniques → Procedures
```

| Component | Description | Example |
|---|---|---|
| **Tactic** | The "why" — attacker's goal at this phase | Credential Access |
| **Technique** | The "how" — method used to achieve the tactic | OS Credential Dumping (T1003) |
| **Sub-technique** | More specific variation of a technique | LSASS Memory (T1003.001) |
| **Procedure** | Specific real-world implementation by a threat actor | APT29 using Mimikatz |

### ATT&CK Tactics (14 total)

| ID | Tactic | Goal |
|---|---|---|
| TA0043 | Reconnaissance | Gather info before attack |
| TA0042 | Resource Development | Build infrastructure/tools |
| TA0001 | Initial Access | Get into the target |
| TA0002 | Execution | Run malicious code |
| TA0003 | Persistence | Maintain access |
| TA0004 | Privilege Escalation | Gain higher permissions |
| TA0005 | Defence Evasion | Avoid detection |
| TA0006 | Credential Access | Steal credentials |
| TA0007 | Discovery | Map the environment |
| TA0008 | Lateral Movement | Move through network |
| TA0009 | Collection | Gather target data |
| TA0011 | Command & Control | Communicate with compromised hosts |
| TA0010 | Exfiltration | Move data out |
| TA0040 | Impact | Destroy/disrupt/encrypt |

### Technique ID Format
- Format: `TXXXX` for technique, `TXXXX.XXX` for sub-technique
- Example: `T1059` = Command and Scripting Interpreter
- Example: `T1059.001` = PowerShell (sub-technique)

---

## ATT&CK in Operation

### ATT&CK Navigator
- Free web tool for visualising ATT&CK coverage
- URL: **https://mitre-attack.github.io/attack-navigator/**
- Use cases:
  - Map your detection coverage (which techniques do you detect?)
  - Visualise a threat actor's known techniques
  - Compare two threat actor profiles
  - Plan red team exercises

### How SOC Teams Use ATT&CK

**1. Detection Engineering**
- Map SIEM rules to ATT&CK techniques
- Identify gaps — which techniques have no detection?
- Build new rules to cover gaps

**2. Threat Hunting**
- Hypothesis: "APT29 uses T1003.001 — are there signs of LSASS dumping in our environment?"
- Search SIEM/EDR for indicators of specific techniques

**3. Incident Response**
- Map attacker behaviour during an incident to ATT&CK techniques
- Understand what the attacker did and what they might do next

**4. Purple Team**
- Red team simulates specific ATT&CK techniques
- Blue team validates detection capability
- Gap = write new detection rule

---

## ATT&CK for Threat Intelligence

### Threat Actor Profiles
- ATT&CK documents known **APT (Advanced Persistent Threat) groups**
- Each group has a profile showing:
  - Known techniques used
  - Malware/tools associated with the group
  - Target sectors and countries
  - Example campaigns

### Key Threat Actor Examples

| Group | Origin | Known For |
|---|---|---|
| **APT29 (Cozy Bear)** | Russia (SVR) | SolarWinds supply chain, spearphishing |
| **APT28 (Fancy Bear)** | Russia (GRU) | Election interference, credential theft |
| **Lazarus Group** | North Korea | Financial theft, ransomware, WannaCry |
| **APT41** | China | Espionage + financially motivated |
| **FIN7** | Criminal | Payment card theft, spearphishing |

### Using ATT&CK for Threat Intel
```
1. Identify relevant threat actors for your sector
2. Go to attack.mitre.org → Groups → select actor
3. View all known techniques → map to your detections
4. Build hunting hypotheses based on their TTPs
5. Prioritise patching for vulnerabilities they exploit
```

---

## Cyber Analytics Repository (CAR)

- **CAR** → MITRE's knowledge base of analytics for detecting ATT&CK techniques
- URL: **https://car.mitre.org**
- Each analytic maps to one or more ATT&CK techniques
- Provides **pseudocode and specific queries** for Splunk, EQL, etc.

### How CAR Works
- Search for a technique (e.g. T1059.001 PowerShell)
- CAR provides ready-made detection logic:
  ```
  CAR-2014-11-004: Remote PowerShell Sessions
  Pseudocode: processes where parent_exe = "wsmprovhost.exe"
  Splunk: index=windows EventCode=4688 ParentProcessName=wsmprovhost.exe
  ```
- Faster detection development — analytics are community validated

### CAR vs ATT&CK
- ATT&CK → describes what attackers do
- CAR → tells you how to detect it

---

## MITRE D3FEND Framework

- **D3FEND** → countermeasure knowledge graph — the defensive counterpart to ATT&CK
- URL: **https://d3fend.mitre.org**
- Maps **defensive techniques** to the ATT&CK offensive techniques they counter

### D3FEND Structure
| Category | Examples |
|---|---|
| **Harden** | Application hardening, credential hardening |
| **Detect** | File analysis, process analysis, network traffic analysis |
| **Isolate** | Network isolation, execution isolation |
| **Deceive** | Decoy files, honeypots |
| **Evict** | Credential eviction, process eviction |

### Practical Use
- Start with an ATT&CK technique (e.g. T1003 — Credential Dumping)
- Go to D3FEND → find countermeasures
- Implement the recommended defensive techniques
- Links attack techniques directly to the controls that stop them

---

## Other MITRE Projects

### MITRE ATLAS
- **ATLAS (Adversarial Threat Landscape for Artificial-Intelligence Systems)**
- ATT&CK for AI/ML systems
- Documents attacks against AI models (model inversion, data poisoning, adversarial examples)
- URL: **https://atlas.mitre.org**
- Relevant to your AI Security interest

### MITRE CVE
- **CVE (Common Vulnerabilities and Exposures)** → managed by MITRE
- Unique identifier for every publicly known vulnerability
- Format: `CVE-YEAR-NUMBER` e.g. `CVE-2017-0144` (EternalBlue)
- MITRE assigns CVE IDs, NVD (NIST) scores them with CVSS

### MITRE ENGAGE
- Framework for cyber deception and adversary engagement
- Guides defenders on using honeypots, deception techniques
- URL: **https://engage.mitre.org**

### MITRE CALDERA
- Automated adversary emulation platform
- Simulates ATT&CK techniques automatically
- Used for purple teaming and detection validation
- URL: **https://caldera.mitre.org**

---

## Quick Reference — MITRE Resources

| Resource | URL | Purpose |
|---|---|---|
| ATT&CK | attack.mitre.org | Attacker techniques knowledge base |
| ATT&CK Navigator | mitre-attack.github.io/attack-navigator | Visualise coverage |
| CAR | car.mitre.org | Ready-made detection analytics |
| D3FEND | d3fend.mitre.org | Defensive countermeasures |
| ATLAS | atlas.mitre.org | AI/ML attack techniques |
| ENGAGE | engage.mitre.org | Deception and adversary engagement |
| CALDERA | caldera.mitre.org | Adversary emulation platform |

---

## My Understanding in Plain English

MITRE is the backbone of modern threat intelligence — every serious SOC uses ATT&CK as a common language to describe what attackers do. The power is in the standardisation: when an analyst says "the attacker used T1059.001," every SOC engineer in any country knows exactly what that means. ATT&CK Navigator is what you use to see your detection gaps visually — a heatmap where red means no coverage. CAR saves time by giving you the actual query instead of writing detection logic from scratch. D3FEND is ATT&CK in reverse — instead of "here's what attackers do," it's "here's what defenders should do to stop each technique." Together these tools give a SOC both the intelligence and the tools to build systematic, comprehensive defence.

---

## Related Notes

- [[Cyber-Kill-Chain]]
- [[Unified-Kill-Chain]]
- [[Pyramid-of-Pain]]
- [[Introduction-to-SIEM]]
- [[Introduction-to-SOAR]]
