# Detecting Web Shells

**Path:** SOC Level 1 - Web Security Monitoring
**Date:** 2026-09-05
**Difficulty:** Easy

---

## What Is a Web Shell?

A web shell is a malicious program uploaded to a target web server that lets an attacker execute commands remotely through the web interface. It typically serves two roles at once:

- **Initial Access** — deployed via a file upload vulnerability
- **Persistence** — a standing backdoor an attacker can return to after the initial compromise

Once running, a web shell becomes a launchpad for the rest of the kill chain: reconnaissance, privilege escalation, lateral movement, and data exfiltration.

- **MITRE ATT&CK mapping:** Persistence sub-technique **T1505.003 — Server Software Component: Web Shell**
- **Deployment requirement:** a file upload vulnerability, a misconfiguration, or existing access that fails to validate file type/extension/content/destination (e.g. a "pet photo upload" feature that doesn't check the file is actually an image, letting an attacker upload `shell.php` or `mydog.aspx` instead)

### Real-World Examples

| Group | Technique |
|---|---|
| **Hafnium (ProxyLogon)** | Uploaded `.aspx` web shells to Exchange servers (e.g. `\inetpub\wwwroot\aspnet_client\`), then performed recon, credential dumping, new-account persistence, lateral movement |
| **Conti Ransomware** | Abused the same Exchange vulnerability class to drop `aspnetclient_log.aspx`, then rapidly deployed a backup web shell and mapped the network's computers/domain controllers within minutes |

- **Common file extension for Exchange-targeting web shells:** `.aspx`

---

## Anatomy of a Web Shell

Web shells work by abusing **legitimate** language functions rather than exploiting some exotic mechanism. In PHP, the usual suspects are system-execution functions: `shell_exec()`, `exec()`, `system()`, `passthru()`.

**Minimal PHP web shell logic:**
1. Reads a `cmd` parameter from the URL (`?cmd=whoami`)
2. Stores the attacker-supplied command in a variable
3. Runs it through `shell_exec()`
4. Prints the output back to the page

Web shells range from this kind of one-liner all the way up to fully-featured tools with GUIs, password protection, and built-in file managers.

- Interact with one directly via browser (`?cmd=whoami`) or via `curl`, URL-encoding the command first (`ls -la` → `ls%20-la`) — CyberChef's URL-Encode recipe is handy for this
- Hands-on check: accessing a deployed shell and running `whoami` returned **www-data** (the low-privilege web server account) — confirming code execution, not full system compromise, at this stage

---

## Log-Based Detection

Web shells abuse the web server, so **access logs** are the first place to look.

### HTTP Methods — Normal vs Abused

| Method | Normal Usage | Possible Abuse |
|---|---|---|
| **GET** | Retrieve a resource | Reconnaissance, or interacting with an already-uploaded shell |
| **POST** | Submit data to the server | Uploading or interacting with a shell |
| **PUT** | Upload/replace a file | Directly uploading a shell |
| **DELETE** | Remove a resource | Covering tracks |
| **OPTIONS** | List supported methods | Reconnaissance |
| **HEAD** | Like GET, headers only | Checking if a file exists |

### Key Indicators to Correlate

| Indicator | What to Look For |
|---|---|
| **Request pattern** | Repeated `GET`s (probing for an upload point) followed by a `POST`/`PUT` to that location; repeated hits on the same file afterward (shell interaction) |
| **User-Agent** | Altered (`Mozilla/4.0+(+Windows+NT+5.1)` truncated to `Mozilla/4.0`), outdated (`MSIE 6.0`), or blacklisted (`curl/1.x`, `wget/1.x`) strings |
| **Source IP** | Traffic from outside the network's normal/expected range |
| **Query strings** | Long or suspicious values, especially `cmd=` / `exec=`; also watch for **encoded** payloads (`?query=whoami` → Base64 `?query=d2hvYW1p`) — decode with CyberChef |
| **Referrer** | Missing referrer *can* indicate direct/scripted access — not conclusive alone, since browsers/privacy settings can also strip it |

### Auditd (Linux Audit Trail)

`auditd` tracks OS-level events per custom rules (e.g. "alert on writes to `/uploads/`"). Query matches with `ausearch`:

```bash
ausearch -k web_shell
time->Wed Jul 23 06:20:36 2025
"name = /uploads/webshell.php"
"OGID = www-data"
```

- The syscall that confirms a file was **written to disk** (e.g. after a suspicious `POST /upload.php`) is **`creat`**
- Correlating a suspicious web log `POST` with an `auditd` event containing `creat` or `execve` confirms both the *upload* and the *execution* — a much stronger signal than either log alone

### Why SIEM Helps

- Centralizes multiple log types (web + auditd + more) into one searchable place
- Lets analysts build targeted queries instead of grepping file by file
- Speeds up correlation across sources during an investigation

---

## Beyond Logs

### File System Analysis

A web shell has to live *somewhere* on disk (except on platforms like WordPress/Django that can store malicious code in a database instead — those won't show up in a filesystem search).

| Web Server | Default Web Root |
|---|---|
| **Apache** | `/var/www/html/` |
| **Nginx** | `/usr/share/nginx/html/` |

- Attackers also target guessable/scanned upload paths: `/uploads/`, `/images/`, `/admin/`, or loosely-permissioned temp dirs like `/tmp`
- **Red flags:** random/suspicious filenames, executable extensions (`.php`, `.jsp`) where they don't belong, and double extensions used to disguise a shell (`image.jpg.php`)

**Useful commands:**
```bash
# Find .php files modified in a date window
find /var/www -type f -name "*.php" -newerct "2025-07-01" ! -newerct "2025-08-01"

# Search for a suspicious function across a directory
grep -r "eval(" wp-content
```

### Network Traffic Analysis

Goes beyond logs by exposing full request/response payloads — the same indicator categories from log analysis apply here too, plus a few network-specific ones: encoded payloads, malicious commands in request bodies, unexpected ports/protocols, and web server processes spawning command-line tools.

**Useful Wireshark filters:**
```wireshark
http.request.method == "PUT"          # Find file-upload requests
http.request.uri contains ".php"      # Find requests to suspicious/modified files
http.user_agent                       # Surface unusual/outdated User-Agents
```

A `POST /upload.php` packet capture can show the shell's actual PHP source code sitting in the request payload — direct confirmation of what was uploaded.

---

## Investigation

**Scenario:** suspicious activity reported on a WordPress site; investigate Apache access logs (`/var/log/apache2/access.log`) to reconstruct the compromise.

**Reconstructed attack sequence:**
1. Attacker IP `203.0.113.66` probes the site and finds the `/wordpress` directory
2. Uses `upload_form.php` to upload the web shell
3. First command run through the shell: `whoami`
4. Pulls a second-stage tool onto the box: `linpeas.sh` (privilege-escalation enumeration script)
5. The web shell's own source code contained a hidden flag — found by `cat`-ing the file directly

---

## Key Terms

| Term | Meaning |
|---|---|
| **Web Shell** | Malicious script uploaded to a server that lets an attacker run commands via the web interface |
| **T1505.003** | MITRE ATT&CK Persistence sub-technique for web shells |
| **www-data** | Default low-privilege Linux user that web servers (Apache/Nginx) run as |
| **auditd / ausearch** | Linux audit daemon and its query tool for OS-level event logs |
| **creat (syscall)** | Confirms a new file was written to disk |
| **Query String** | The `?key=value` part of a URL — common home for shell commands and encoded payloads |
| **linpeas.sh** | Common post-exploitation privilege-escalation enumeration script |

---

## What's Essential in a Detection/Correlation Flow

No single log source tells the whole story — the value is in correlating them:

```
1. Web access logs   → WHO connected, WHAT was requested, WHEN, response code
2. auditd logs       → WHETHER a file was actually written/executed on disk, and by whom
3. File system check → WHERE the shell physically lives, its name/extension/timestamps
4. Network capture   → WHAT was actually sent — the real payload, the real shell source code
```

A suspicious `POST` in the web log becomes a confirmed compromise once it lines up with a `creat`/`execve` event in `auditd` and a matching file discovered on disk — that convergence across sources is what turns a hunch into an incident.

---

## My Understanding in Plain English

A web shell is deceptively simple — it's often just a few lines of code that take a command from a URL and hand it to the OS. What makes it dangerous is that it hides in plain sight inside a normal file upload feature, and once it's there it becomes the attacker's remote control for everything that follows. Detecting one is a triangulation exercise: the web log shows the suspicious request, but not proof anything actually happened on disk; auditd closes that gap by confirming a file was genuinely written or executed; and the file system or a packet capture lets you actually read the malicious code itself. None of these sources alone is convincing — it's the overlap between them that turns "this request looks odd" into "this server is compromised."

---

## Related Notes

- [[Detecting-Web-Attacks]]
- [[Web-Security-Essentials]]
- [[Intro-to-Log-Analysis]]
- [[MITRE]]
