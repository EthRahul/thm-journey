# Windows Threat Detection 3

**Path:** SOC Level 1 - Windows Security Monitoring
**Date:** 2026-09-05
**Difficulty:** Easy

---

## Overview

The final room in the series: how a threat actor **stays** on a breached Windows host. Three tactics: **Command and Control (C2)**, **Persistence**, and **Impact**.

---

## Command and Control (C2)

- **Attacks without C2:** if the breach was via RDP, an attacker can just type commands directly in the RDP session — no C2 needed, but this dies the moment RDP is closed or secured
- **Why C2 exists:** for other entry methods (phishing, USB), the attacker needs a process that phones home and waits for commands 24/7
- **Simplest C2:** the phishing attachment itself becomes that process and opens the channel directly (e.g. Cobalt Strike beacon)
- **More advanced C2:** the attachment doesn't connect back itself — it quietly downloads a *separate* C2 payload, hides it (e.g. in `%APPDATA%` or `C:\Temp`), and runs it as its own stealthy process. This survives the victim deleting the original attachment (seen in real ransomware cases and the APT29 phishing campaign)

**Investigation example from this room (Sysmon logs):**
- Malicious archive downloaded: `URGENT!.zip` — found via Sysmon **Event ID 11**, note the `:Zone.Identifier` **Alternate Data Stream (ADS)** attached to it (an NTFS feature that tags internet-downloaded files — also abusable by attackers to hide extra data behind a normal-looking file)
- C2 payload hidden at: `C:\Users\Administrator\AppData\Roaming\update.exe`
- C2 domain contacted: `route.m365officesync.workers.dev` (deliberately named to look like a legitimate Microsoft 365 sync service)

---

## Persistence Overview

- **Persistence** = maintaining reliable, long-term access to a target that survives reboots and password changes
- Data-stealer infections are often "smash and grab" (breach → collect → exfiltrate → exit, all within minutes) — but most other attacks depend on staying present for days or months
- **Persisting via RDP (exposed-service breaches):** the same weak RDP door stays open until patched, but attackers usually still add a backup method in case it closes:
  - Add a hidden vulnerability/backdoor to the breached service, **or**
  - Create a new user (**T1136**), make it an administrator (**T1098**), and use it for future RDP logins

### Detecting Backdoored Users

- Every account creation = Security **Event ID 4720**
- Don't rely on suspicious-sounding names alone — investigate: who created it, was the source IP/login time expected, what else did that creator's session do?
- A new user alone isn't useful to an attacker until it's privileged — adding it to **Administrators** or **Remote Desktop Users** is tracked by Security **Event ID 4732**
- Alternative to creating a new account: reset the password of an old/unused account instead — tracked by Security **Event ID 4724**

**Investigation example:** 6 failed logon attempts (**4625**) against Administrator, then a successful login, followed by creation of a backdoor account named `support` (**4720**), added to **Administrators** (**4732**).

---

## Persistence: Tasks and Services

Malware persistence (as opposed to a backdoored user) is needed when the entry point wasn't RDP — e.g. phishing or USB — since there's no login session to reuse. The malware itself needs to survive a reboot and keep calling home.

### Services

- View/manage via `services.msc`; creating/modifying one from the command line needs admin rights and `sc.exe`
- Threat actors create their own malicious service to auto-run a program at startup
- **Detect it three ways:**
  1. Launch of `sc.exe create` via Sysmon **Event ID 1**
  2. Service creation via Security **Event ID 4697** or System **Event ID 7045**
  3. A suspicious process with a **services.exe** parent

### Scheduled Tasks

- Manage via `taskschd.msc` (GUI) or `schtasks.exe` (CLI)
- **Easier to configure and hide than services** — the most common persistence method in real attacks (seen in APT28 and APT41)
- **Detect it three ways:**
  1. Launch of `schtasks.exe /create` via Sysmon **Event ID 1**
  2. Scheduled task creation via Security **Event ID 4698**
  3. A suspicious process with a `svchost.exe ... -s Schedule` parent

**Investigation example:** the "Nessie" malware persisted via a service named **Data Protection Service**; the "Troy" malware persisted via a scheduled task named **AmazonSync**, running `troy.exe` with parent `svchost.exe -k netsvcs -p -s Schedule`.

---

## Persistence: Run Keys and Startup

Services and scheduled tasks run at boot and need admin rights. For a program that only needs to run when a *specific user* logs in, Windows offers two lighter, per-user methods.

### Startup Folder

```
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\
All users: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```

- Meant for inexperienced users to drag a program/shortcut in for auto-start on login — legitimate use is rare, so this folder is normally **empty**
- Malware placed here (e.g. seen with Lumma Stealer) is caught via **file creation events in this folder** (Sysmon **Event ID 11**)
- Programs launched this way have an **explorer.exe** parent — the same parent as ordinary user activity, making it easy to overlook

### Registry Run Keys

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
All users: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
```

- Same underlying MITRE technique as the Startup folder (**T1547.001**) — the only difference is *how* the entry gets added: a new registry value pointing to the program, instead of a copied shortcut
- View via `regedit.exe`
- Detect via **registry value-set events** (Sysmon **Event ID 13**) targeting the Run key path

**Investigation example:** the "Odin" malware's parent process was `C:\Windows\explorer.exe` (Startup-folder style persistence); its final output line was `Done doing bad stuff!`.

---

## Impact — Why Persistence Matters

Why not just steal data and leave immediately? Three common reasons attackers stay:

| Reason | Example |
|---|---|
| **Add the host to a botnet for further attacks** | Kraken Botnet combined crypto-mining, data-stealing, and C2 capability |
| **Long-term espionage (state-sponsored)** | Volt Typhoon stayed undetected in the US electric grid for nearly a year |
| **Use it as an entry point into a larger network** | Attackers have spent as long as 29 days breaching a full network from one initial foothold |

- The single biggest threat to most corporate Windows networks: **ransomware** — capable of halting an entire organization (e.g. the McLaren hospital attack affected 743,000 patients)
- **The best point to stop an attack is at Initial Access** — every stage after that (Persistence, Credential Access, Lateral Movement, Impact) gets progressively harder and more damaging to unwind

---

## Key Terms

| Term | Meaning |
|---|---|
| **C2 (Command and Control)** | The channel an attacker uses to send commands to a compromised host after breach |
| **Persistence** | Maintaining long-term access that survives reboots/password changes |
| **Zone.Identifier** | An NTFS Alternate Data Stream marking a file as downloaded from the internet; abusable to hide data |
| **T1136 / T1098** | Create Account / Account Manipulation (used to make a backdoored user privileged) |
| **T1547.001** | Registry Run Keys / Startup Folder persistence |
| **Alternate Data Stream (ADS)** | Hidden data stored "behind" a normal NTFS file |

---

## Event ID Reference

| Log Source | Event ID | Use Here |
|---|---|---|
| Security | **4625** | Failed logon — brute-force attempts before the breach |
| Security | **4720** | New user account created (backdoor user) |
| Security | **4724** | Password reset on an existing account |
| Security | **4732** | Member added to a privileged local group |
| Security | **4697** | Service installed |
| System | **7045** | Service installed (alternate/legacy source) |
| Security | **4698** | Scheduled task created |
| Sysmon | **1** | Process Creation — catches `sc.exe create`, `schtasks.exe /create`, and malware execution |
| Sysmon | **11** | File Create — the C2 archive/payload, or a file dropped in the Startup folder |
| Sysmon | **13** | Registry Value Set — catches new Run key entries |

---

## What's Essential in an Investigation Flow

```
1. Confirm the C2 channel        → Sysmon 1/3/11/22: dropped payload, domain contacted
2. Check for a backdoored user   → 4625 (brute force) → 4624 (breach) → 4720 (new user) → 4732 (privilege)
3. Check for malware persistence → 4697/7045 (service) or 4698 (task), cross-check parent process
4. Check for per-user persistence → Sysmon 11 (Startup folder) or Sysmon 13 (Run key)
5. Ask "why persist?"            → botnet, espionage, or staging for a bigger breach (ransomware)
```

The throughline for the whole Windows Threat Detection series: **the earlier you catch it, the cheaper it is to fix** — a brute-force attempt (4625) is trivial to block; a fully-persisted ransomware foothold is not.

---

## My Understanding in Plain English

This room is the "why don't they just leave?" question, answered. Once an attacker has one working session, the smart move is turning that single session into something durable — a backdoor account, a fake service, a scheduled task, or a quiet registry entry — because sessions die (RDP gets closed, a reboot happens, a password gets reset) but a well-hidden persistence mechanism doesn't. What's striking is how ordinary each mechanism looks in isolation: a new user, a new service, a new scheduled task, a new Run key entry — these are all things IT does constantly for legitimate reasons. The signal isn't the existence of the event, it's the context around it — an unexpected creator, an odd naming pattern, a parent process that doesn't belong, or timing that falls outside business hours. And the reason all of this matters at the SOC level is blunt: ransomware doesn't happen the moment of breach, it happens after days or months of exactly this kind of quiet persistence — which means Initial Access is still, by far, the cheapest place to stop it.

---

## Related Notes

- [[Windows-Threat-Detection-1]]
- [[Windows-Threat-Detection-2]]
- [[Windows-Logging-for-SOC]]
- [[MITRE]]
