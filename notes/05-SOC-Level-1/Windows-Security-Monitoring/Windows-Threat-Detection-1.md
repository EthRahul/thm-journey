# Windows Threat Detection 1

**Path:** SOC Level 1 - Windows Security Monitoring
**Date:** 2026-09-05
**Difficulty:** Easy

---

## What Is Initial Access?

**Initial Access** is the first stage of an attack — the moment a threat actor gets through the "front door" of a target system. Every later stage (privilege escalation, lateral movement, exfiltration) depends on this first foothold. The methods split into two broad groups:

| Category | Idea |
|---|---|
| **Exposed Services** | The system itself has an internet-facing door left open (RDP, a vulnerable web app) |
| **User-Driven** | A human is tricked into opening the door themselves (phishing, a planted USB) |

---

## Exposed Services

Running a Windows server directly on the internet is routine (websites need HTTP, mail needs SMTP, admins need RDP) — but every exposed service is also scanned by bots within minutes, looking for weak passwords or unpatched flaws.

| MITRE Technique | Description |
|---|---|
| **T1133 — External Remote Services** | Threat actors hunt for exposed RDP/VNC/SSH protected only by a weak password |
| **T1190 — Exploit Public-Facing Application** | Attackers exploit a vulnerable or misconfigured web app/service (e.g. an unpatched mail server) |

---

## Initial Access via RDP

**Typical attack flow:** network scan → RDP brute force → successful login → further malicious action.

### Indicators

- Large volume of **failed logons** (Event **4625**)
- **Logon Type 10** events (RemoteInteractive / RDP)
- Repeated authentication attempts concentrated on a predictable account — almost always **Administrator**

### Investigation Workbook

1. Filter Security log for **4625** — the account most repeatedly targeted reveals which user botnets are brute-forcing
2. Filter for a **successful** logon (**4624**) with **Logon Type 10** from the same attacking IP — this is the breach
3. To recover the attacker's **real Workstation Name**, check the field on that same successful-logon event (or the preceding Logon Type 3 event, if NLA is in play — same technique as the Security Log: Authentication task in Windows Logging for SOC)

*In this room's scenario: an IT admin temporarily opened RDP for weekend remote work; bots immediately found it and brute-forced their way in.*

---

## Initial Access via Phishing

| MITRE Technique | Description |
|---|---|
| **T1566 — Phishing** | Users are tricked into opening a malicious attachment or link, launching the malware themselves |

### Common Phishing Techniques

| Technique | How It Fools the User |
|---|---|
| **Misleading file extensions** | A file named to look harmless, e.g. `www.skype.com.exe` — the `.com` reads as a website, but the real extension is `.exe` |
| **Malicious LNK files** | `.lnk` = a Windows "link file" / shortcut, normally used to point to a frequently-used file or folder; weaponized, it silently downloads and runs next-stage malware from an external URL when opened |
| **Double-extension executables** | A file like `best-cat.jpg.exe` — looks like an image at a glance, but the real (final) extension is `.exe` |

---

## Continuing Phishing Topic — Malware Behavior Chain

Once the malicious file runs, a fairly consistent chain plays out:

1. **Download** — victim downloads an archive via the browser (e.g. a `.zip` landing in `Downloads`)
2. **Extraction** — the archive gets unpacked, often into an unrelated folder (e.g. `Pictures`) to blend in
3. **Execution** — the malware process launches; find it via Sysmon **Event ID 1** (Process Create) and note its **Process ID**
4. **Command & Control** — the malware connects out to a malicious domain (Sysmon **Event ID 3** Network Connection / **22** DNS Query)

**Investigation tip used throughout this room:** set the Event Viewer date/time range to span from the start to the end of the incident window — events line up in the order they occurred, making the chain easy to walk step by step.

---

## Initial Access via USB

| MITRE Technique | Description |
|---|---|
| **T1091 — Removable Media** | Malware infects a USB device, hoping a user plugs it into another machine (classic "dropped in a parking lot" social engineering) |

### Observed Behavior

- Malware can be disguised with a trustworthy-sounding filename to bait the user into launching it (e.g. `Open Sandisk 4GB USB.exe`)
- Once executed, it **drops a payload to disk** — often in a shared, low-scrutiny location like `C:\Users\Public\Documents\`
- It can then **propagate to other removable media** — copying itself onto a second USB drive letter to spread further when that drive is used elsewhere

---

## Key Terms

| Term | Meaning |
|---|---|
| **Initial Access** | The first stage of an attack — the point where a threat actor gets into the target system |
| **T1133** | External Remote Services — brute-forcing exposed RDP/VNC/SSH |
| **T1190** | Exploit Public-Facing Application |
| **T1566** | Phishing |
| **T1091** | Removable Media |
| **LNK file** | Windows shortcut file; weaponized versions silently fetch/run malware |
| **Double Extension** | A filename with two extensions where the real (final) one is hidden in plain sight, e.g. `.jpg.exe` |
| **Logon Type 10** | RemoteInteractive — an RDP session |

---

## Event ID Reference for Initial Access

| Log Source | Event ID | Use Here |
|---|---|---|
| Security | **4625** | Failed logon — spot brute-force targeting |
| Security | **4624** (Type 10) | Successful RDP logon — confirms the breach |
| Sysmon | **1** | Process Creation — catches the phishing payload / USB executable launching |
| Sysmon | **3** | Network Connection — malware calling out to C2 |
| Sysmon | **11** | File Create — the dropped payload on disk (USB/phishing malware) |
| Sysmon | **22** | DNS Query — resolves the malicious domain the malware contacts |

---

## What's Essential in an Investigation Flow

```
1. Identify the vector          → exposed service (RDP/web) or user-driven (phishing/USB)?
2. Anchor the time window        → set logs to span incident start → end; events then read in order
3. Find the entry point          → 4625/4624 for RDP, or the first suspicious process (Sysmon 1) for phishing/USB
4. Follow execution              → Process ID from Sysmon 1 links to everything that process does next
5. Follow outbound activity      → Sysmon 3/22 reveals the C2 domain/IP
6. Check for spread               → dropped files (Sysmon 11), propagation to other media/accounts
```

The throughline across every vector in this room is the same: find the first anomalous event, then walk forward in time using the Process ID (Sysmon) or Logon ID (Security) as the thread connecting every subsequent action.

---

## My Understanding in Plain English

Nearly every attack starts the same way conceptually, even though the door being used is different — an exposed RDP port and a phishing email are just two different ways for an attacker to get one process running on the box. Once that first process exists, the investigation becomes the same exercise every time: find it, note its Process ID (or Logon ID for a login-based entry), and then chase that ID forward through the logs to see what it touched — files written, domains contacted, other media infected. Phishing and USB attacks are really just social engineering wrapped around the same technical payoff as a brute-forced RDP session: getting one piece of attacker-controlled code to execute. The double-extension trick and the fake USB filename exist for the exact same reason a weak RDP password does — they're both just the easiest way through a door a human left unlocked.

---

## Related Notes

- [[Windows-Logging-for-SOC]]
- [[Detecting-Web-Shells]]
- [[Detecting-Web-Attacks]]
- [[MITRE]]
