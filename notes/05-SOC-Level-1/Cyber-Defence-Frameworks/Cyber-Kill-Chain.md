# Cyber Kill Chain

**Path:** SOC Level 1 - Cyber Defence Frameworks
**Date:** 2026-05-21
**Difficulty:** Easy

---

## What is the Cyber Kill Chain?

- Developed by **Lockheed Martin** in 2011, adapted from a military concept
- Framework that describes the **7 stages of a cyberattack** from initial reconnaissance to final objective
- Used by defenders to **identify, detect, and prevent** attacks at each stage
- Breaking any single link in the chain = attack fails
- Helps SOC teams understand **where in the attack lifecycle** a threat actor currently is

```
1. Reconnaissance
2. Weaponization
3. Delivery
4. Exploitation
5. Installation
6. Command & Control (C2)
7. Actions on Objectives
```

---

## Stage 1 — Reconnaissance

- **Goal:** Gather as much information about the target as possible before attacking
- Entirely passive from the target's perspective — hard to detect

### OSINT Techniques Used by Attackers
- **Email harvesting** → find employee emails (theHarvester, Hunter.io)
- **LinkedIn** → enumerate employees, roles, technologies used
- **Shodan** → find exposed services, open ports, technologies
- **Google Dorking** → find exposed files, login pages, sensitive data
- **WHOIS** → domain registration info, nameservers
- **DNS enumeration** → subdomains, MX records, SPF records
- **Job postings** → reveal internal tech stack ("experience with AWS and Kubernetes required")
- **GitHub** → leaked credentials, API keys, internal code

### Defender Actions at This Stage
- Monitor for unusual DNS queries
- Reduce public OSINT footprint
- Implement email security (SPF, DKIM, DMARC)
- Remove sensitive info from job postings

---

## Stage 2 — Weaponization

- **Goal:** Create a weapon (malware/exploit) tailored to the target
- Attacker combines exploit + payload into a deliverable weapon
- No interaction with target yet — entirely attacker-side

### Common Weapons Created
- **Malicious documents** → Word/Excel/PDF with macros or exploits
- **Payload-embedded files** → executables, scripts, ISO files
- **Custom malware** → RATs, backdoors, ransomware
- **Exploit kits** → automated exploitation of browser/plugin vulnerabilities
- **Weaponized USB drives** → physical delivery

### Key Terms
- **Malware** → malicious software designed to damage, disrupt, or gain unauthorised access
- **Exploit** → code that takes advantage of a vulnerability
- **Payload** → the actual malicious code that executes after exploit succeeds
- **RAT (Remote Access Trojan)** → gives attacker remote control of victim machine

---

## Stage 3 — Delivery

- **Goal:** Transmit the weapon to the target
- The first stage where the attacker directly interacts with the target

### Delivery Methods
| Method | Example |
|---|---|
| **Phishing email** | Malicious attachment or link sent to employee |
| **Spearphishing** | Targeted phishing using personal info from recon |
| **Watering hole** | Compromise a site the target frequently visits |
| **USB drop** | Leave infected USB in target's parking lot |
| **Supply chain** | Compromise a trusted vendor/software update |
| **Drive-by download** | Victim visits compromised website → auto download |

### Defender Actions
- Email gateway filtering (block malicious attachments)
- User awareness training
- Web proxy filtering
- Disable AutoRun for USB devices

---

## Stage 4 — Exploitation

- **Goal:** Execute the weapon on the target system to gain access
- The exploit triggers a vulnerability to run the attacker's code

### Common Exploitation Techniques
- **Software vulnerabilities** → unpatched CVEs (EternalBlue, Log4Shell)
- **Macro execution** → user opens malicious Office doc → enables macros
- **Zero-day exploits** → vulnerabilities unknown to vendor — no patch exists
- **Browser exploits** → drive-by downloads targeting outdated browsers
- **Credential abuse** → using stolen/guessed credentials to log in

### Defender Actions
- Keep systems patched and updated
- Disable macros by default in Office
- Application whitelisting
- Vulnerability scanning

---

## Stage 5 — Installation

- **Goal:** Install malware/backdoor to maintain persistent access
- Attacker ensures they can return even if initial access is lost

### Persistence Mechanisms (Windows)
```
Registry Run Keys:
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run

Scheduled Tasks → schtasks /create
Services → sc create maliciousservice
Startup folder → C:\Users\user\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
DLL hijacking → replace legitimate DLL with malicious one
```

### Timestomping
- Attacker modifies file timestamps to make malware appear old/legitimate
- Makes forensic timeline analysis harder

### Defender Actions
- Monitor registry run keys for changes
- Audit scheduled tasks
- File integrity monitoring
- Hunt for new services/startup entries

---

## Stage 6 — Command & Control (C2)

- **Goal:** Establish a communication channel between attacker and compromised system
- Allows attacker to remotely issue commands, exfiltrate data, move laterally

### C2 Communication Methods
| Method | How it blends in |
|---|---|
| **HTTP/HTTPS** | Looks like normal web traffic |
| **DNS tunneling** | Hides data in DNS queries |
| **ICMP tunneling** | Hides data in ping packets |
| **Social media** | Twitter/Discord DMs as C2 channel |
| **Slack/Teams** | Legitimate platforms used for C2 |

### C2 Frameworks Used by Attackers
- **Cobalt Strike** → most popular commercial C2
- **Metasploit** → open source
- **Sliver** → open source, growing use
- **Covenant** → .NET C2
- **Havoc** → newer alternative to Cobalt Strike

### Beacon Behaviour
- C2 implant "beacons" home at regular intervals
- **Jitter** → random delay added to beacon timing to avoid detection
- Default beacon = every 60 seconds → SOC rule: alert on regular intervals

### Defender Actions
- Monitor outbound traffic for unusual patterns
- DNS query analysis (high volume, unusual domains)
- Threat intel blocking of known C2 IPs/domains
- Detect regular beacon intervals in network logs

---

## Stage 7 — Actions on Objectives (Exfiltration)

- **Goal:** Accomplish the mission — the reason the attacker targeted the organisation
- Everything up to this point was preparation

### Common Final Objectives
| Objective | Example |
|---|---|
| **Data exfiltration** | Steal customer data, IP, credentials |
| **Ransomware** | Encrypt files → demand payment |
| **Destruction** | Wipe systems, destroy data |
| **Lateral movement** | Compromise more systems in the network |
| **Credential harvesting** | Dump password hashes (Mimikatz) |
| **Espionage** | Long-term silent monitoring |
| **Cryptomining** | Use victim's compute resources |

### Exfiltration Techniques
- Upload to cloud storage (Dropbox, Google Drive, MEGA)
- FTP/SFTP to attacker server
- DNS exfiltration (encode data in DNS queries)
- Email sensitive files out
- HTTP POST to attacker's web server

### Defender Actions
- DLP (Data Loss Prevention) monitoring
- Egress traffic filtering
- Anomaly detection (large outbound transfers)
- Honeypots and honeytokens

---

## Kill Chain — Defender Perspective

| Stage | Detection Opportunity | Prevention |
|---|---|---|
| Reconnaissance | Honeypots, monitor web logs | Reduce OSINT footprint |
| Weaponization | Threat intel feeds | N/A (attacker-side) |
| Delivery | Email gateway, web proxy | User training, filtering |
| Exploitation | EDR, IDS/IPS, patching | Patch management |
| Installation | EDR, file integrity monitoring | Application whitelisting |
| C2 | Network monitoring, DNS analysis | Firewall egress rules |
| Objectives | DLP, anomaly detection | Segmentation, least privilege |

---

## Kill Chain vs MITRE ATT&CK

| Framework | Focus | Granularity |
|---|---|---|
| **Cyber Kill Chain** | High-level attack phases | 7 stages |
| **MITRE ATT&CK** | Specific techniques per phase | 200+ techniques |

- Kill Chain = the **"what phase"**
- MITRE ATT&CK = the **"exactly how"**
- Both used together for comprehensive detection coverage

---

## My Understanding in Plain English

The Cyber Kill Chain is the attacker's to-do list. Every attack follows the same pattern — gather intel, build the weapon, deliver it, exploit a vulnerability, install persistence, establish C2, then accomplish the mission. The power of the framework for defenders is that you only need to break one link to stop the attack. Detecting at stage 6 (C2) stops exfiltration even if you missed everything before. Detecting at stage 2 (weaponization via threat intel) stops the attack before it even reaches you. The best SOC teams build detection rules at every stage rather than relying on catching it at one point — defence in depth applied to the attack lifecycle.

---

## Related Notes

- [[Pyramid-of-Pain]]
- [[Introduction-to-SIEM]]
- [[Introduction-to-SOAR]]
- [[Introduction-to-EDR]]
- [[Passive-Reconnaissance]]
