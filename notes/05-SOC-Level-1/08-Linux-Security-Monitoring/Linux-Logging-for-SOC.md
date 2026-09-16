# Linux Logging for SOC

**Path:** SOC Level 1 - Linux Security Monitoring
**Date:** 2026-09-15
**Difficulty:** Easy

---

## Why Linux Logging Matters

Linux runs most of the server estate a SOC defends — web servers, databases, containers, build infrastructure — and unlike Windows there is no single unified event log to pull from. Evidence is scattered across plain-text files in `/var/log`, each written by a different service in a different format, and a lot of what an analyst actually wants (process creation, file changes, network connections) **isn't logged at all by default**.

This room is the foundation for the rest of the module: knowing where the logs are, how to filter them fast, and how to turn on the runtime visibility that Linux doesn't give you for free.

---

## Where Linux Logs Live

| Log File | What It Holds | RHEL/CentOS Equivalent |
|---|---|---|
| `/var/log/syslog` | Aggregated system events — kernel messages, service starts, package installs | `/var/log/messages` |
| `/var/log/auth.log` | Authentication, sudo, SSH, user management | `/var/log/secure` |
| `/var/log/kern.log` | Kernel messages and errors | — |
| `/var/log/dpkg.log` | Debian package manager activity | `/var/log/dnf.log`, `/var/log/yum.log` |
| `/var/log/audit/audit.log` | auditd runtime events | same |
| `~/.bash_history` | Per-user interactive command history | same |
| `/var/log/nginx/access.log` | Web server request log (app-specific) | same |

**Distribution matters.** Paths and formats differ between Debian/Ubuntu and RHEL/CentOS, and application logs vary per service. Never assume a path — verify it on the host you're triaging.

---

## Finding Logs You Didn't Know Existed

When you land on an unfamiliar host, don't guess filenames — search `/var/log` for the concept you care about:

```bash
grep -R -E "auth|login|session" /var/log
```

This recursively sweeps every log file for authentication-related keywords and shows you which files are actually carrying that data on this particular box.

---

## Working With Logs

```bash
cat /var/log/syslog | grep CRON
cat /var/log/auth.log | grep -E 'session opened|session closed'
```

The workflow is almost always the same: pick the right file, then filter hard with `grep` (use `-E` for alternation, `-R` to recurse). Logs are high-volume and mostly noise — the skill is knowing which keyword narrows it to the handful of lines that matter.

---

## Authentication Logs

`/var/log/auth.log` is the highest-value file for triage. It covers every way a user can reach the system.

### Session Events (PAM)

| Log Pattern | Meaning |
|---|---|
| `pam_unix(login:session): session opened` | Local or SSH interactive login |
| `pam_unix(cron:session): session opened` | A scheduled cron job started a session |
| `sudo: pam_unix(sudo:session): session opened` | Privilege escalation via sudo |

Sessions come in pairs — `session opened` and `session closed` — so you can bound how long an account was active.

### SSH Entries

SSH lines follow the shape `<result> <method> for <user> from <ip>`:

```
Accepted publickey for bob from 10.19.92.18 port 55050 ssh2
```

The three things to read every time: **result** (Accepted vs Failed), **method** (publickey vs password), and **source IP**.

### User Management and Sudo

Grep for the account-management binaries — `useradd`, `userdel`, `usermod`, `passwd` — to catch an attacker creating or modifying accounts. For sudo activity, the `COMMAND=` field records exactly what was run with elevated privileges.

---

## Generic System Logs

`/var/log/syslog` is the catch-all stream: kernel messages, services starting and stopping, cron execution, package installations. It's less targeted than `auth.log` but it's where you confirm *context* — did a service crash right before the suspicious activity, was a package installed that shouldn't have been.

Package manager logs (`dpkg.log`, `dnf.log`) deserve their own look during an investigation: software appearing on a host that nobody requested is a strong signal.

---

## Bash History and Why You Can't Trust It

`~/.bash_history` records commands a user typed, which sounds ideal — but it's trivially avoided and easy to miss things with:

| Evasion / Gap | Effect |
|---|---|
| Leading space before a command | Command is not written to history |
| Writing and running a script (`nano legit.sh && ./legit.sh`) | Only the script invocation is recorded, not its contents |
| Switching shell (`sh`) | Doesn't persist history the way Bash does |
| Non-interactive execution | Cron jobs, web server child processes and OS-initiated processes never appear at all |

Treat bash history as a **bonus artifact, never as proof of absence**. "Nothing in `.bash_history`" does not mean nothing ran.

---

## Runtime Monitoring and System Calls

Everything a program does that touches the operating system goes through a **system call** — opening a file, creating a process, making a network connection, accessing hardware. Linux has over 300 of them; the one that matters most for detection is `execve`, which executes a program.

This is the layer EDR and runtime-monitoring tools hook into, and it's how you see **runtime events** — process creation, file modification, network connections — that the default text logs simply don't record.

---

## The Audit Daemon (auditd)

`auditd` is the built-in Linux answer to runtime visibility. It hooks system calls and writes structured records to `/var/log/audit/audit.log`, which you query with `ausearch`.

```bash
ausearch -i -k proc_wget
ausearch -i | grep <search_term>
```

The `-i` flag is important — it *interprets* raw numeric IDs into human-readable names (UIDs into usernames, syscall numbers into syscall names).

### Record Types

| Type | What It Carries |
|---|---|
| `PROCTITLE` | The full command line as it was invoked |
| `SYSCALL` | The system call itself plus process metadata |
| `EXECVE` | Execution details with individual arguments |
| `CWD` | Current working directory at execution time |
| `PATH` | The file(s) the operation touched |

### Key Fields

| Field | Meaning |
|---|---|
| `pid` | Process ID of the executed command |
| `ppid` | Parent Process ID — the field that lets you trace origin |
| `auid` | Audit user — the account used at login, survives `su`/`sudo` |
| `uid` | The user the command actually ran as |
| `tty` | Session identifier |
| `exe` | Absolute path to the binary |
| `key` | Custom tag set in the audit rule, for cheap filtering |

The `auid` vs `uid` distinction is the useful one: `auid` tells you *who logged in*, `uid` tells you *who the command ran as*. An attacker who escalates changes `uid` but drags `auid` along behind them.

---

## Key Terms

| Term | Meaning |
|---|---|
| **System Call** | The interface a program uses to ask the kernel for anything — file, process, network, hardware |
| **execve** | The syscall that executes a program; the primary process-creation detection point |
| **auditd** | Linux audit daemon — hooks syscalls and writes structured runtime records |
| **ausearch** | Query tool for auditd records; `-i` interprets IDs into names |
| **PAM** | Pluggable Authentication Modules — the subsystem that writes `session opened/closed` lines |
| **auid** | Audit user ID — the original login account, preserved across privilege changes |
| **Runtime Event** | Process creation, file modification or network connection — not captured by default Linux logs |

---

## My Understanding in Plain English

The mental model here is that Linux gives you two very different classes of evidence. The first is the plain-text logs in `/var/log`, which are free, always on, and tell you about *sessions and services* — who logged in, from where, what sudo ran, what package got installed. The second is runtime visibility from auditd, which is not on by default but is the only thing that tells you about *processes* — what actually executed, as whom, and spawned by what.

Triage almost always starts in the first class and finishes in the second: `auth.log` tells you an account logged in from a strange IP at 3am, and auditd tells you what that session then went and ran. Bash history sits awkwardly between them — nice when it's there, but so easy to defeat that finding it empty proves nothing. The one habit worth building early is not memorising paths but reaching for `grep -R` across `/var/log` first, because the file layout changes between distributions and applications and the logs that matter are often ones nobody told you about.

---

## Related Notes

- [[Linux-Threat-Detection-1]]
- [[Windows-Logging-for-SOC]]
- [[Introduction-to-SIEM]]
- [[Introduction-to-EDR]]
