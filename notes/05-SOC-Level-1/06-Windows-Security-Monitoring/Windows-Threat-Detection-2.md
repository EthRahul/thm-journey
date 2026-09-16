# Windows Threat Detection 2

**Path:** SOC Level 1 - Windows Security Monitoring
**Date:** 2026-09-05
**Difficulty:** Easy

---

## Overview

Where *Windows Threat Detection 1* covers how an attacker gets in, this room covers what they do **immediately after** — the post-compromise stages: **Discovery**, **Collection**, **Credential Access**, **Exfiltration**, and **Ingress Tool Transfer**. All of it is reconstructed from Sysmon logs and Windows Event Viewer alone.

---

## Discovery

- After landing on a host, an attacker rarely knows anything about the environment — the first priority is figuring out **where they are, what privileges they have, and what security controls are watching**
- Example command: `net user Administrator` — reveals which privileged group(s) the account belongs to (e.g. confirms membership in **Administrators**)
- Every command run shows up in Sysmon **Event ID 1** (Process Creation) — e.g. running `net user` produces an event with **Image**: `C:\Windows\System32\net.exe`

| MITRE Technique | What It Looks Like |
|---|---|
| **T1033 / System Owner/User Discovery** | `whoami`, `net user` |
| **T1069 / Permission Groups Discovery** | `net user Administrator`, `net localgroup administrators` |
| **T1518.001 / Security Software Discovery** | Checking for EDR/AV before proceeding, e.g. `tasklist /v \| findstr MsSense.exe` |

### Real Example from the Room

- A phishing attachment (`invoice.pdf.exe`) was executed; Sysmon logs reconstructed its discovery behavior:
  1. First command: `whoami` — establishes the security context it's running under
  2. Second command: `cmd /c "tasklist /v | findstr MsSense.exe || echo No MS Defender EDR"` — checks whether Microsoft Defender EDR (`MsSense.exe`) is present before doing anything riskier
  3. The gathered discovery info was sent out to an attacker-controlled domain (`exfil.beecz.cafe`)

## Detecting Discovery

- **Detection approach:** filter Sysmon Event ID 1 for native Windows discovery binaries (`net.exe`, `whoami.exe`, `systeminfo.exe`, `tasklist.exe`, `query.exe`) launched by processes/users that don't normally run them
- Reconstruct the **process tree** (parent → child, via ProcessId/ParentProcessId, same technique as Windows Threat Detection 1) to trace discovery commands back to the original malicious process (e.g. the phishing executable)
- Outbound connections immediately following a burst of discovery commands are a strong signal that reconnaissance results are being phoned home

---

## Collection

Once the attacker understands the environment, they pivot to **collecting valuable data** already sitting on the host.

| MITRE Technique | Example in the Room |
|---|---|
| **T1555.003 / Credentials from Web Browsers** | Facebook password recovered directly from Chrome's Password Manager |
| **T1552.004 / Private Keys** | An SSH key (`thm-access-database.key`) found on disk — valuable because it can grant direct access without needing a password |
| **T1213 / Data from Information Repositories** | A sensitive internal document (a network-diagram PDF) — useful to an attacker for planning lateral movement |

**Why these matter:** browser-stored credentials and SSH keys are high-value because they skip the need to crack or guess anything; internal documents like network diagrams accelerate the *next* stage of an attack (lateral movement, privilege escalation).

## Detecting Collection

Simulated via a data-stealer executable; Sysmon logs revealed its full collection behavior:

1. **Staging** (T1074 / Data Staged) — creates a working directory to temporarily hold gathered files before packaging, e.g. `staging_58f1`
2. **Targeted file search** — searches specifically for high-value document types: **.docx, .pdf, .xlsx** (financial data, business records, confidential files)
3. **Clipboard capture** (T1115 / Clipboard Data) — uses the PowerShell cmdlet **`Get-Clipboard`** to steal whatever the user last copied (passwords, tokens, crypto wallet addresses are common clipboard contents)
4. **Compression + Exfiltration** (T1560 / Archive Collected Data, T1567.002 / Exfiltration to Cloud Storage) — compresses the staged files and uploads the archive to a cloud storage bucket (e.g. an `.s3.amazonaws.com` address) — blending malicious traffic in with ordinary cloud/business traffic

---

## Ingress Tool Transfer (T1105)

Once attackers need more tools than what they arrived with, they pull additional payloads onto the compromised host using **legitimate, built-in Windows utilities** — which is exactly what makes this technique hard to catch by tool name alone.

| Method | Notes |
|---|---|
| **Web Browser** | Simplest method — just download the file directly |
| **Curl** | Command-line HTTP client, commonly present or easily invoked |
| **Certutil** | Legitimate certificate utility abused for its `-urlcache`/download functionality — a classic "living off the land" technique |
| **PowerShell `Invoke-WebRequest`** | Native cmdlet for fetching remote content, no extra tooling required |

- **Detection approach:** since the *tool* used is legitimate, the differentiator is context — an unusual parent process invoking `certutil.exe` or `powershell.exe` with a URL argument, followed by a new file being written to disk (Sysmon Event ID 11) and a new process launching from it (Event ID 1)
- Following the **DNS query trail** in Event Viewer (Sysmon Event ID 22) is one of the most reliable ways to catch this — every one of these methods still has to resolve the domain it's downloading from

---

## Key Terms

| Term | Meaning |
|---|---|
| **Discovery** | MITRE tactic covering an attacker learning about their new environment |
| **Collection** | MITRE tactic covering gathering data of interest before exfiltration |
| **Credential Access** | MITRE tactic covering theft of account credentials/keys |
| **Data Staging** | Temporarily consolidating collected files in one place before packaging |
| **Exfiltration to Cloud Storage** | Using a legitimate cloud service (S3, etc.) to disguise data theft as normal traffic |
| **Ingress Tool Transfer (T1105)** | Pulling additional attacker tools onto a compromised host, often via built-in OS utilities |
| **Living off the Land** | Abusing legitimate, pre-installed tools (certutil, PowerShell, curl) instead of custom malware, to blend in |

---

## Event ID / Detection Reference

| Log Source | Event ID | Use Here |
|---|---|---|
| Sysmon | **1** | Process Creation — catches every discovery command, the stealer's launch, and any tool-transfer process |
| Sysmon | **3** | Network Connection — outbound calls to exfil/C2 domains |
| Sysmon | **11** | File Create — the staging directory and any downloaded tool landing on disk |
| Sysmon | **22** | DNS Query — the domain trail behind exfiltration and tool downloads |

---

## What's Essential in an Investigation Flow

```
1. Spot the discovery burst     → native commands (net, whoami, tasklist) run in quick succession
2. Trace the process tree       → find the parent process that spawned them (the initial payload)
3. Watch for staging            → a newly-created directory collecting specific file types
4. Watch the clipboard/creds    → Get-Clipboard, browser password store access
5. Watch outbound traffic       → compressed archive leaving via an unusual (often cloud) domain
6. Watch for tool transfer      → certutil/curl/PowerShell fetching a URL, followed by a new file + new process
```

Nearly every step in this chain reduces to the same two Sysmon events: **Event ID 1** (what ran) and **Event ID 3/22** (where it talked to) — chained together via ProcessId, exactly as in Windows Threat Detection 1.

---

## My Understanding in Plain English

This room is really about watching what an attacker does in the minutes right after getting a foothold — and the pattern is remarkably consistent: figure out where you are and whether anyone's watching (discovery), grab whatever's easy and valuable (saved passwords, SSH keys, sensitive docs), package it up somewhere temporary, and ship it out disguised as normal traffic (a cloud bucket instead of some obviously-malicious IP). The part that stands out is Ingress Tool Transfer — attackers don't need to bring exotic malware, because Windows already ships with everything they need to download more of it (certutil, curl, PowerShell). That's exactly why this stage is hard to catch on tool name alone; the signal isn't "certutil.exe ran," it's "certutil.exe ran with a URL, followed immediately by a brand-new file and a brand-new process" — the sequence is the tell, not any single event.

---

## Related Notes

- [[Windows-Threat-Detection-1]]
- [[Windows-Logging-for-SOC]]
- [[Detecting-Web-Shells]]
- [[MITRE]]
