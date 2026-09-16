# Linux Threat Detection 2

**Path:** SOC Level 1 - Linux Security Monitoring
**Date:** 2026-09-16
**Difficulty:** Medium

---

## What This Room Covers

This room picks up right after Initial Access and asks: what does an attacker actually *do* in the first minutes on a freshly compromised Linux box? The answer is almost always **Discovery** — a burst of built-in recon commands — followed by a decision about *why* the box was worth breaking into in the first place. The room frames most Linux compromises as **opportunistic, automated "hack and forget" attacks** rather than hands-on-keyboard targeted intrusions, and walks a real, still-active botnet (**Dota3**) end to end as the case study.

Room flow: Discovery → Detecting Discovery → Motivation (Hack and Forget) → Case Study — Dota3 Cryptominer.

---

## Discovery — Initial Recon

Attackers reach for **built-in Linux tools** (living-off-the-land) rather than custom malware, because it blends into normal admin activity and needs nothing downloaded.

| Goal | Example Commands |
|---|---|
| OS / filesystem | `pwd`, `ls /`, `env`, `uname -a`, `lsb_release -a`, `hostname` |
| User/group info | `id`, `whoami`, `w`, `last`, `cat /etc/passwd`, `cat /etc/sudoers` |
| Process/network | `ps aux`, `top`, `ip a`, `ip r`, `arp -a`, `ss -tnlp`, `netstat -tnlp` |
| Cloud/sandbox check | `systemd-detect-virt`, `lsmod`, `uptime`, `pgrep <edr-name>` |

**Key detection insight:** `whoami` is almost never run by a legitimate application or service — treat basically any execution of it as worth alerting on.

**Goal-specific discovery** narrows recon toward a particular payoff:

- Credential theft → `history | grep pass`, `find / -name .env`, `find /home -name id_rsa`
- Cryptomining feasibility → `cat /proc/cpuinfo`, `lscpu`, `free -m`, `top`/`htop`
- Network/botnet scanning → `ping`, loop scans with `nc`

---

## Detecting Discovery

```bash
# audit rule shape used to log a discovery binary
-a always,exit -S execve -F exe=/usr/bin/whoami ...
```

- Log execution of key discovery binaries with **auditd**, then hunt with `ausearch` or a SIEM.
- The hard part is never *finding* the commands — it's **context**: is this an attacker, an IT admin doing legitimate work, or a monitoring agent?
- Build a **process tree** to resolve that context: trace `ppid`/`pid` chains with `ausearch --pid <n>` / `--ppid <n>` to find the parent script or process that triggered the suspicious command.

---

## Motivation — "Hack and Forget" Attacks

Most Linux compromises are **large-scale, automated, and opportunistic** rather than targeted. Three common end goals:

| Goal | Description |
|---|---|
| **Cryptominer** | Uses victim CPU/GPU cycles for the attacker's profit |
| **Botnet enrollment** | e.g. Mirai-style — victim becomes DDoS infrastructure |
| **Proxy use** | Victim relays phishing traffic, malware, or other attack traffic |

### Ingress Tool Transfer (MITRE)

How the payload actually lands on the box — almost always via a preinstalled tool:

- `wget` — direct file download
- `curl` — download/upload via HTTP request
- `scp` / `sftp` over SSH

**Detection nuance worth remembering:** if the *attacker* initiates `scp`/`sftp` from their own machine to pull or push files, the transfer command itself **never appears in the victim's auditd log** — the only trace is a **new SSH login** in `/var/log/auth.log`. If the *victim* (or a script running as the victim) initiates the transfer, it shows up in auditd as normal.

Other angles for spotting Ingress Tool Transfer:

- **Network:** download source is a known-bad IP/domain, or a legitimate-but-abused host (e.g. GitHub) serving the attack tool
- **File events:** new files dropped in `/tmp` or `/var/tmp`, suspicious filenames
- **EDR/AV:** alerts on the dropped file itself

---

## Case Study — Dota3 Cryptominer

A real, still-active botnet malware family, walked as a full attack chain:

1. **Initial Access** — mass SSH brute-force against the `root` user using top weak passwords, launched from a large distributed botnet.
2. **Discovery** — rapid-fire recon focused specifically on CPU/RAM (`/proc/cpuinfo`, `free -m`, `lscpu`) — a strong early signal of a miner infection specifically, versus generic recon.
3. **Persistence** — resets the compromised user's password (locking out *competing* botnets) and overwrites `~/.ssh/authorized_keys` with its own key (locking out the *legitimate* owner too). Uses a distinctive comment string in the key as an IOC.
4. **Payload delivery** — transfers a malware archive via `scp` using the stolen SSH access, then unpacks it into a **hidden directory under `/tmp`** deliberately named to look like legitimate system files.
5. **Execution** — runs two tools under `nohup` (so they keep running after the SSH session closes): a network scanner that probes internal ranges for more SSH targets, and an XMRig-based cryptominer.

### Detection Takeaways

- Successful SSH logins from untrusted/external IPs: `cat /var/log/auth.log | grep Accepted`
- A spike of discovery commands immediately after login
- Hidden or oddly-named directories and files under `/tmp`
- `nohup` paired with an unfamiliar binary
- Outbound SSH port-scanning traffic toward internal ranges
- Known cryptominer binaries (e.g. **XMRig**) — flagged by most EDR/AV out of the box

---

## Key Terms

| Term | Meaning |
|---|---|
| **Living-off-the-land** | Using preinstalled OS tools instead of custom malware, to blend in |
| **Discovery** | MITRE tactic covering an attacker mapping out the compromised environment |
| **Ingress Tool Transfer** | MITRE technique for getting a payload onto the victim machine |
| **Hack and Forget** | Automated, opportunistic attack with no specific target — contrast with a targeted intrusion |
| **Dota3** | Real-world SSH-brute-force botnet that deploys a cryptominer and network scanner |
| **nohup** | Linux command that lets a process keep running after its parent shell/session closes |
| **XMRig** | Widely-abused open-source Monero mining software, common cryptojacking payload |

---

## My Understanding in Plain English

The single most useful reframe from this room is that most Linux compromises aren't a human attacker sitting there deciding what to do next — they're a script running a fixed playbook (recon → lock out competitors → drop payload → mine or scan) as fast and quietly as possible. That's *why* the discovery commands matter so much as a detection point: a person with legitimate access rarely needs to run `whoami` or `cat /proc/cpuinfo` back-to-back, but a bot executing a hardcoded script does it every single time, in a recognizable burst.

The `scp`/`sftp` direction detail is the one worth remembering for real investigations: it explains a gap that would otherwise look like a missing log. If auditd shows no transfer command but `auth.log` shows a fresh SSH login right before new files appear, that's not a logging failure — it's the attacker pulling files *from their own box*, which never touches the victim's audit trail as a "transfer" event at all.

---

## Related Notes

- [[Linux-Threat-Detection-1]]
- [[Linux-Threat-Detection-3]]
- [[Linux-Logging-for-SOC]]
- [[MITRE]]
- [[Cyber-Kill-Chain]]
