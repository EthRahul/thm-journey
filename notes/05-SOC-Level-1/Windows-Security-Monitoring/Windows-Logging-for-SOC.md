# Windows Logging for SOC

**Path:** SOC Level 1 - Windows Security Monitoring
**Date:** 2026-09-05
**Difficulty:** Easy

---

## What Is Logged

- Whenever a program starts, a file is created, or a user logs in, the OS can record it as a **log** — an entry with timestamp, action details, and the responsible user
- Logging supports three core SOC activities: **Incident Response** (reconstruct when/how an attack happened), **Threat Hunting** (search for signs of malicious activity), **Alerting/Triage** (the raw material behind every detection rule)

### Anatomy of a Log Entry

- Windows event logs are stored in binary `.evtx` format under `C:\Windows\System32\winevt\Logs\`
- Each `.evtx` file corresponds to a log category — e.g. **Application** (IIS, SQL, other user-mode apps), **Security** (logons, process activity, user management)
- Read them with **Event Viewer** (`Win+R` → `eventvwr`):
  - **Log Sources** — each `.evtx` file = one item in the left panel
  - **Log List** — each row is one event: Keywords (success/failure), Date/Time (system time, not UTC), **Event ID** (unique per event type, e.g. failed login is always `4625`)
  - **Log Details** — full plaintext/XML content of the event
  - **Filters** — "Filter Current Log" / "Find" to narrow down by Event ID etc.
- There are 500+ distinct Event IDs in the Security log alone — this room focuses on the handful that carry the most day-to-day SOC value

---

## Security Log: Authentication

The two highest-value Security events for any SOC: **4624 (Successful Logon)** and **4625 (Failed Logon)**.

### Key Fields (4624 / 4625)

| Field | Why It Matters |
|---|---|
| **Logon Type** | Tells you *how* the logon happened (see table below) |
| **Account Name** | Which account attempted/succeeded |
| **Source Network Address** | Attacking/originating IP |
| **Workstation Name** | Hostname the logon claims to come from |
| **Logon ID** | Unique session identifier — the thread that ties this logon to every later action taken in that session |

### Common Logon Types

| Type | Meaning |
|---|---|
| **2** | Interactive (local console logon) |
| **3** | Network (e.g. accessing a share; also default RDP with NLA enabled) |
| **4** | Batch (scheduled task) |
| **5** | Service (service start) |
| **7** | Unlock (workstation unlock) |
| **10** | RemoteInteractive (RDP, when NLA is not used/misconfigured) |
| **11** | CachedInteractive (logon using cached credentials) |

### Detect RDP Brute Force (Workbook)

1. Filter Security log for **4625**
2. Look at Logon Types **3** and **10** (most modern systems show 3, since NLA is on by default; older/misconfigured systems show 10)
3. Red flags:
   - Many different usernames tried (`admin`, `helpdesk`, `cctv`) → password spraying
   - Many failures on one account, usually **Administrator** → brute force
   - Workstation Name doesn't match corporate naming (`kali` instead of `THM-PC-06`)
   - Unexpected source IP (e.g. a printer trying to log into a server)

### Analyse RDP Logons (Workbook)

1. Filter Security log for **4624**, Logon Type **10**
2. If NLA is enabled, every real RDP logon is preceded by a **4624** with Logon Type **3** — check that earlier event to get the true Workstation Name
3. Red flags: a preceding brute force, or a suspicious source IP/hostname
4. Note the **Logon ID** from the successful login — this is the thread you pivot on for everything that follows

---

## Security Log: User Management

Every user-management event splits into three parts: **Subject** (who did it — includes a Logon ID you can correlate back to the 4624 that started their session), **Object** (the target account/group), **Details** (exactly what changed).

### Key Event IDs

| Event ID | Meaning |
|---|---|
| **4720** | A user account was created |
| **4722** | A user account was enabled |
| **4724** | An attempt was made to reset an account's password |
| **4728** | A member was added to a security-enabled global group |
| **4732** | A member was added to a security-enabled local group |
| **4733** | A member was removed from a security-enabled local group |
| **4756** | A member was added to a security-enabled universal group |

### Hunt for Backdoored Users (Workbook)

1. Filter Security log for **4720** / **4732**
2. Red flags:
   - No one in IT can confirm the action
   - Change made outside working hours / on a weekend
   - Subject account name is unfamiliar (e.g. `adm.old.2008` creating new users)
   - Target account name breaks naming convention (`backup` instead of `thm_svc_backup`)
3. If confirmed malicious: copy the **Logon ID** from the `4720`/`4732` event and find the matching `4624` logon for full session context
- **Real-world pattern:** ransomware actors have reset all account passwords en masse to slow recovery; others created new admin accounts purely for persistence

---

## Sysmon: Process Monitoring

Authentication logs tell you *who* got in — Sysmon tells you *what they did on the endpoint*.

- **Sysmon** = free Microsoft Sysinternals tool, the de facto standard for endpoint visibility beyond the default logs (found under `Applications & Services > Microsoft > Sysmon > Operational` once installed)
- Preferable to the native, noisy **4688** (Process Creation) event — Sysmon gives richer, more configurable fields

### Sysmon Event ID 1 — Process Creation, Field Groups

| Group | Contains |
|---|---|
| **Process Info** | PID, image path, command line of the launched process |
| **Parent Info** | Same details for the parent process — builds the process tree / attack chain |
| **Binary Info** | Process hash, digital signature, PE metadata |
| **User Context** | The running user, and critically the **Logon ID** — same field as in Security logs, the link between the two log sources |

### Analyse Process Launch (Workbook)

1. Filter Sysmon for Event ID **1**
2. Red flags in Process/Binary Info: image in an uncommon directory (`C:\Temp`, `C:\Users\Public`), suspiciously-named binary (`aa.exe`, `jqyvpqldou.exe`), hash matches known malware on VirusTotal
3. Red flags in Parent Info: parent itself looks suspicious, or the parent is unexpected (e.g. Notepad spawning `cmd.exe`)
4. If still unsure, walk up the process tree: find the prior event where `ProcessId == ParentProcessId` of your event, repeat the analysis
5. Finally, pivot across **all** Security + Sysmon events sharing the same **Logon ID** to trace the full attack chain

---

## Sysmon: Files and Network

Beyond process creation, Sysmon can log file/registry changes, network connections, and DNS queries — fully configurable (this room uses a popular open-source config, "Florian's config").

### Additional Event IDs

| Event ID | Meaning |
|---|---|
| **1** | Process Creation (covered above) |
| **3** | Network Connection |
| **11** | File Create |
| **22** | DNS Query |

- These events largely omit Logon ID / parent-process context — instead, pivot using the **ProcessId** field to jump back to the matching Event ID 1 for full context

### Analyse Process Activities (Workbook)

1. Copy the **ProcessId** from the Event ID 1 you're investigating
2. Search other Sysmon events sharing that same ProcessId
3. Network connection red flags: connections to external IPs on port **80** or unusual ports (**4444**), connections to known-bad IPs (check VirusTotal), DNS queries to suspicious domains (`*.top`, `*.click`, or random-looking strings)
4. File/registry red flags: files dropped into staging directories (`C:\Temp`, `C:\Users\Public`), dropped scripts (`.bat`, `.ps1`) or executables (`.exe`, `.com`), files/keys used for persistence (e.g. a `.url` file placed in the Startup folder)

---

## PowerShell: Logging Commands

- PowerShell is trusted and powerful — attackers use it for malware download, discovery, exfiltration, even process injection
- **The problem:** once `powershell.exe` launches, it generates a *single* Sysmon Event ID 1 (or 4688) — every command run *inside* that session is invisible to process-creation logging, since no new process is spawned per command

### PowerShell History File

```
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

- Plain-text file, auto-created by PowerShell, updated the instant a command is submitted (Enter pressed)
- One history file **per user** — five active users could mean five separate history files
- Survives reboots; retains commands indefinitely unless manually deleted
- **Limitations:** does not log command *output*, and does not show the contents of an executed script (e.g. running `powershell .\script.ps1` only shows that line, not what's inside `script.ps1`)

---

## Event ID Quick Reference

| Log Source | Event ID | Meaning |
|---|---|---|
| Security | **4624** | Successful logon |
| Security | **4625** | Failed logon |
| Security | **4688** | Process creation (native, noisier alternative to Sysmon 1) |
| Security | **4720** | User account created |
| Security | **4722** | User account enabled |
| Security | **4724** | Password reset attempted |
| Security | **4728** | Member added to a global security group |
| Security | **4732** | Member added to a local security group |
| Security | **4733** | Member removed from a local security group |
| Security | **4756** | Member added to a universal security group |
| Sysmon | **1** | Process Creation |
| Sysmon | **3** | Network Connection |
| Sysmon | **11** | File Create |
| Sysmon | **22** | DNS Query |

---

## Key Terms

| Term | Meaning |
|---|---|
| **EVTX** | Binary Windows event log file format (`C:\Windows\System32\winevt\Logs`) |
| **Logon ID** | Unique per-session identifier — the golden thread linking a 4624 logon to every later Security/Sysmon event in that session |
| **Logon Type** | Field on 4624/4625 describing *how* the logon happened (interactive, network, RDP, etc.) |
| **NLA** | Network Level Authentication — when enabled, RDP logons show as Logon Type 3 before the 4624 with Type 10 |
| **Sysmon** | Sysinternals tool providing detailed, configurable endpoint logging beyond default Windows logs |
| **ProcessId (Sysmon)** | Shared key used to pivot from a file/network/DNS event back to its originating Event ID 1 |
| **PSReadline History** | Per-user plaintext file recording every PowerShell command typed, with no output or script contents |

---

## What's Essential in an Investigation Flow

The whole room builds toward one habit: **pivot on the Logon ID**.

```
1. Find the suspicious logon        → Security 4624/4625, note Logon Type + Logon ID
2. Check what that session did      → Sysmon Event ID 1, filtered by the same Logon ID
3. Check what that process touched  → Sysmon 3 / 11 / 22, filtered by matching ProcessId
4. Check for persistence/account abuse → Security 4720/4728/4732, same Logon ID
5. Check interactive commands       → PowerShell history file for that user
```

Every log source only tells part of the story — Security shows *who logged in and what accounts changed*, Sysmon shows *what ran and what it touched on disk/network*, and PowerShell history shows *what was typed inside a session Sysmon can't see into*. The Logon ID (and ProcessId within Sysmon) is what stitches all of it into one attack chain.

---

## My Understanding in Plain English

Windows logging looks intimidating mostly because of volume — hundreds of Event IDs, most of them useless for day-to-day triage. In practice, a handful of IDs do almost all the work: 4624/4625 tell you who's getting in and how, 4720/4732 tell you if someone's quietly creating backdoor accounts, Sysmon Event ID 1 tells you what actually ran on the box, and Sysmon 3/11/22 tell you what that process touched over the network and on disk. The one thing that makes all of this usable together is the Logon ID — it's the thread you pull to go from "this login looks weird" to "here's the account they created, here's the malware they ran, here's the C2 server it called home to." PowerShell is the deliberate blind spot in all of this: it hides everything inside one process, which is exactly why attackers love it, and why the plain PS history file — as limited as it is — is often the only record of what was actually typed.

---

## Related Notes

- [[Detecting-Web-Attacks]]
- [[Detecting-Web-Shells]]
- [[Core-SOC-Solutions]]
- [[Sysmon]]
