# Gobuster: The Basics
**Path:** CyberSec_101 - Offensive Security Tooling
**Date:** 2026-03-16
**Difficulty:** Easy

---

## What is Gobuster

- Directory, file, subdomain and vhost brute forcing tool
- Used in recon and enumeration phase
- Finds hidden content not linked anywhere on the site
- Pre-installed on Kali Linux
- Three main modes: dir, dns, vhost

---

## Commands Used
```bash
# Directory and file enumeration
gobuster dir -u http://target.com \
-w /usr/share/wordlists/dirb/common.txt

# With file extensions
gobuster dir -u http://target.com \
-w /usr/share/wordlists/dirb/common.txt \
-x php,html,txt,bak

# Subdomain enumeration
gobuster dns -d target.com \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Vhost enumeration
gobuster vhost -u http://target.com \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
--append-domain
```

---

## Important Flags

- `-u` → target URL
- `-w` → wordlist path
- `-x` → file extensions to append
- `-t` → threads (default 10, increase for speed)
- `-o` → save output to file
- `-q` → quiet mode, less output
- `-r` → follow redirects
- `--append-domain` → required for vhost mode

---

## Three Modes Explained

**dir** → finds hidden directories and files
```
http://target.com/admin       ← found
http://target.com/backup      ← found
http://target.com/config.php  ← found
```

**dns** → finds subdomains
```
admin.target.com      ← found
dev.target.com        ← found
staging.target.com    ← found
```

**vhost** → finds virtual hosts on same IP
```
# Different from DNS — tests Host header not DNS
# Multiple websites hosted on same server/IP
admin.target.com → different site than target.com
```

---

## Difference: Subdomain vs Vhost

- DNS mode → queries actual DNS records
- Vhost mode → sends requests with different Host headers
  to same IP, finds sites not in public DNS
- Vhost finds more hidden targets than DNS mode

---

## Wordlists to Use
```bash
# Directory enumeration
/usr/share/wordlists/dirb/common.txt              # small fast
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  # thorough

# Subdomain and vhost
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Install seclists if missing
sudo apt install seclists
```

---

## Gotchas / Surprises

- Always specify -x extensions for web servers
  PHP sites → -x php, ASP sites → -x asp,aspx
- Vhost requires --append-domain flag or results are wrong
- More threads = faster but can crash unstable targets
  use -t 50 on stable targets, default on CTF machines
- Output to file with -o to review results later
- 403 results are interesting — directory exists but blocked

---

## My Understanding in Plain English

Gobuster automates the process of guessing hidden
directories, files and subdomains by trying every
word in a wordlist against the target. What takes
hours manually takes minutes with Gobuster.
Finding hidden admin panels, backup files and
dev subdomains is how real bugs get found.

---

## Related Notes

- [[Web Application Basics]]
- [[Hydra]]
- [[Burp Suite Basics]]
- [[Recon and Enumeration]]

---

## Recon and Enumeration

**Recon** → gathering info about target from a distance

- Passive → no direct contact (Google, WHOIS, Shodan, LinkedIn)
- Active → direct interaction (Nmap, Gobuster, Ping)

**Enumeration** → going deeper once you know something exists
```
Recon      → find port 80 is open
Enumeration → find every directory on that web server
```

**Full Attack Flow:**
```
Recon → Enumeration → Exploitation → Privilege Escalation → Post Exploitation
```

**Where Gobuster fits:**
- Nmap finds open ports → recon
- Gobuster finds hidden directories/subdomains → enumeration
- More enumeration = more attack surface = easier exploitation

Pentesters spend 60-70% of their time on recon and enumeration.
The more you find, the more you have to attack.

---
