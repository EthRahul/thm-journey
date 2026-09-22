<div align="center">

# 🔐 THM Journey — Rahul Sunouri

**Offensive foundations. Defensive depth.**

*Documenting my cybersecurity learning path through TryHackMe — from exploitation*
*fundamentals and web app pentesting into SOC analysis and detection engineering.*

<br>

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Ethrahul-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/p/Ethrahul)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Rahul_Sunouri-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/rahul-sunouri)

![Rooms documented](https://img.shields.io/badge/Rooms_documented-95-2ea44f?style=flat-square)
![Paths](https://img.shields.io/badge/Paths-6-blue?style=flat-square)
![Writeups](https://img.shields.io/badge/Writeups-6-orange?style=flat-square)
![Focus](https://img.shields.io/badge/Current_focus-SOC_Level_1-8957e5?style=flat-square)

</div>

---

## 📌 About This Repository

A working record of hands-on cybersecurity training: **room notes**, **full walkthroughs**, and the
methodology I've built around both. It started on the offensive side — Metasploit, Burp, Nmap,
privilege escalation — and has since moved into blue team work: SIEM queries, packet analysis,
IDS rules and host telemetry.

The two halves reinforce each other. Knowing exactly how an attack is executed is what makes the
detection for it obvious.

| | |
|:---|:---|
| 🎯 **Currently** | SOC Level 1 — Linux & Windows threat detection |
| ✅ **Completed** | Cyber Security 101 · Jr Penetration Tester |
| 🔄 **Alongside** | Web App Pentesting · AI Security · CCNA networking |
| 📓 **Notes** | 95 rooms documented — [browse the index](notes/README.md) |

---

## 📂 Repository Structure

```text
thm-journey/
├── notes/        → room-by-room knowledge base, grouped by path and module
│   ├── 01-CyberSec-101/            16 rooms  ✅
│   ├── 02-Jr-Pentesting-Path/      26 rooms  ✅
│   ├── 03-Web-App-Pentesting/       6 rooms  🔄
│   ├── 04-AI-Security/              2 rooms  🔄
│   ├── 05-SOC-Level-1/             31 rooms  🔄  ← active
│   └── NetworkChuck-CCNA/          14 modules 🔄
└── writeups/     → full room walkthroughs with screenshots and command output
```

> **Legend** — ✅ Complete · 🔄 In progress · ⏳ Planned

---

## 🛡️ Current Path — SOC Level 1

The path I'm working through now: detection, triage and investigation from the defender's chair.
**8 modules · 31 rooms documented · [full index →](notes/README.md#soc)**

| # | Module | Rooms | Covers | Status |
|:---:|:---|:---:|:---|:---:|
| 01 | [Core SOC Solutions](notes/README.md#soc-core) | 5 | SIEM, EDR, SOAR, Splunk, Elastic Stack | ✅ |
| 02 | [Cyber Defence Frameworks](notes/README.md#soc-frameworks) | 4 | Pyramid of Pain, Kill Chains, MITRE ATT&CK | ✅ |
| 03 | [Network Traffic Analysis](notes/README.md#soc-nta) | 5 | Wireshark, NetworkMiner, pcap forensics | ✅ |
| 04 | [Network Security Monitoring](notes/README.md#soc-nsm) | 5 | Recon / MITM / exfil detection, Snort | ✅ |
| 05 | [Web Security Monitoring](notes/README.md#soc-web) | 3 | Web attack & web shell detection, WAFs | ✅ |
| 06 | [Windows Security Monitoring](notes/README.md#soc-windows) | 4 | Event Logs, Sysmon, full intrusion chain | ✅ |
| 07 | [Malware Concepts for SOC](notes/README.md#soc-malware) | 1 | Classification and response mapping | ✅ |
| 08 | [Linux Security Monitoring](notes/README.md#soc-linux) | 4 | auditd, process trees, Dota3 botnet case study | ✅ |

---

## ✅ Completed Paths

<details>
<summary><b>Cyber Security 101</b> — completed March 2026 &nbsp;<code>16 rooms documented</code></summary>

<br>

| Module | Covers | Status |
|:---|:---|:---:|
| Linux Fundamentals | Shell, filesystem, permissions | ✅ |
| Networking | OSI, protocols, ports | ✅ |
| Cryptography | Hashing, encryption basics | ✅ |
| [Exploitation Basics](notes/README.md#cs101-exploit) | Metasploit, Meterpreter, Blue, Moniker Link | ✅ |
| [Web Hacking](notes/README.md#cs101-web) | Web app basics, JS, SQL, Burp Suite | ✅ |
| [Offensive Security Tooling](notes/README.md#cs101-tooling) | Hydra, Gobuster, SQLMap, shells | ✅ |
| [OWASP Top 10 (2025)](notes/README.md#cs101-owasp) | IAAA failures, design flaws, data handling | ✅ |

</details>

<details>
<summary><b>Jr Penetration Tester</b> — completed April 2026 &nbsp;<code>26 rooms documented</code></summary>

<br>

| Module | Covers | Status |
|:---|:---|:---:|
| [Introduction to Web Hacking](notes/README.md#jrpt-web) | IDOR, LFI, SSRF, XSS, SQLi, race conditions | ✅ |
| [Burp Suite](notes/README.md#jrpt-burp) | Repeater, Intruder, Decoder, Comparer | ✅ |
| [Network Security](notes/README.md#jrpt-netsec) | Passive/active recon, full Nmap series | ✅ |
| [Vulnerability Research](notes/README.md#jrpt-vuln) | CVE research, exploit adaptation | ✅ |
| [Privilege Escalation](notes/README.md#jrpt-privesc) | Linux & Windows privesc, shell handling | ✅ |

</details>

---

## 🔄 Also In Progress

| Path | Focus | Progress |
|:---|:---|:---|
| [Web Application Pentesting](notes/README.md#webapp) | Auth logic, sessions, OAuth, JWT, MFA, uploads | 6 rooms |
| [AI Security](notes/README.md#aisec) | LLM & ML attack surface, training data risks | 2 rooms |
| [Network Fundamentals — CCNA](notes/README.md#ccna) | Infrastructure through an offensive lens | 14 modules |

---

## 📝 Writeups

Full walkthroughs — enumeration, exploitation, flags, and what I'd do differently.

| Room | Path | Difficulty | Walkthrough |
|:---|:---|:---:|:---|
| Blue | Cyber Security 101 | 🟢 Easy | [Read](writeups/01-CyberSec-101/Blue/README.md) |
| Metasploit Introduction | Cyber Security 101 | 🟢 Easy | [Read](writeups/01-CyberSec-101/Metasploit-Introduction/README.md) |
| Net Sec Challenge | Jr Penetration Tester | 🟡 Medium | [Read](writeups/02-Jr-Pentesting-Path/Net-Sec-Challenge/README.md) |
| Vulnerability Capstone | Jr Penetration Tester | 🟡 Medium | [Read](writeups/02-Jr-Pentesting-Path/Vulnerability-Capstone/README.md) |
| Linux Privilege Escalation Capstone | Jr Penetration Tester | 🟡 Medium | [Read](writeups/02-Jr-Pentesting-Path/Linux-Privilege-Escalation-Capstone/README.md) |
| Pickle Rick | Web App Pentesting | 🟢 Easy | [Read](writeups/03-Web-App-Pentesting/Pickle-Rick/README.md) |

---

## 🛠️ Tools Covered

<table>
<tr><td valign="top" width="50%">

**🗡️ Offensive**

| Tool | Purpose |
|:---|:---|
| Nmap | Port scanning, service & OS detection |
| Metasploit | Exploitation framework |
| Burp Suite | Web proxy, Repeater, Intruder |
| Gobuster | Directory & subdomain enumeration |
| SQLMap | Automated SQL injection |
| Hydra | Credential brute forcing |
| Netcat | Listeners and reverse shells |
| Msfvenom | Payload generation |

</td><td valign="top" width="50%">

**🛡️ Defensive**

| Tool | Purpose |
|:---|:---|
| Splunk | SIEM search, dashboards, alerting |
| Elastic Stack | Log ingestion, KQL, Kibana |
| Wireshark | Packet analysis & traffic forensics |
| NetworkMiner | Artifact extraction from pcap |
| Snort | IDS/IPS rule writing and tuning |
| Sysmon | Windows process & network telemetry |
| Event Viewer | Windows log triage |
| auditd | Linux process tree reconstruction |

</td></tr>
</table>

---

## 🎯 Goals

```text
✅  Complete TryHackMe Cyber Security 101
✅  Complete Jr Penetration Tester path
🔄  Complete SOC Level 1 path
🔄  Web Application Penetration Testing
⏳  Earn PTSv2 (PT1) certification
⏳  Earn HTB CPTS certification
⏳  First bug bounty submission on HackerOne
⏳  Security analyst / offensive security role
```

---

## 📜 Certifications

| Certification | Issuer | Year |
|:---|:---|:---:|
| Jr Penetration Tester | TryHackMe | 2026 |
| Cyber Security 101 | TryHackMe | 2026 |
| Ethical Hacking Essentials (EHE) | EC-Council | 2025 |
| Google Cybersecurity Professional | Google | 2025 |

---

<div align="center">

**[📓 Browse the notes index →](notes/README.md)** · **[📝 Read the writeups →](writeups)**

<br>

*"The quieter you become, the more you are able to hear."* — Kali Linux motto

</div>
