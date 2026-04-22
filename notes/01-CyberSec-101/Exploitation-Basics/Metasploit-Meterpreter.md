# Metasploit Meterpreter
**Path:** CyberSec_101 - Exploitation Basics
**Date:** 2026-03-11
**Difficulty:** Easy

---

## Commands Used

- `sysinfo` → Get target system information
- `getuid` → Show current user on target
- `getpid` → Show current process ID
- `ps` → List all running processes on target
- `migrate <PID>` → Move Meterpreter into another process
- `hashdump` → Dump all user password hashes
- `getsystem` → Attempt privilege escalation to SYSTEM
- `shell` → Drop into native OS shell
- `background` → Send session to background
- `sessions -l` → List all active sessions
- `sessions -i <number>` → Re-enter a session
- `upload <file> <path>` → Upload file to target
- `download <file>` → Download file from target
- `search -f <filename>` → Search for a file on target
- `cat <file>` → Read contents of a file
- `pwd` → Print current directory on target
- `ls` → List files in current directory
- `cd <directory>` → Change directory on target
- `edit <file>` → Edit a file on target
- `execute -f <program>` → Execute a program on target
- `screenshot` → Take screenshot of target desktop
- `keyscan_start` → Start keylogger
- `keyscan_dump` → Dump captured keystrokes
- `keyscan_stop` → Stop keylogger
- `load kiwi` → Load Kiwi module for credential dumping
- `creds_all` → Dump all credentials using Kiwi
- `run post/multi/recon/local_exploit_suggester` → Find local privilege escalation exploits
- `timestomp <file> -m <date>` → Modify file timestamps to cover tracks

---

## New Concepts

- Meterpreter runs entirely in memory so it leaves no files on disk
- This makes it harder to detect by antivirus
- Meterpreter communicates over encrypted channel back to attacker
- Process migration moves Meterpreter into a more stable or privileged process
- Always migrate into a stable process like explorer.exe or svchost.exe
- Migrating into a process owned by SYSTEM gives you SYSTEM privileges
- Meterpreter flavors exist for different situations:
  - windows/meterpreter/reverse_tcp → staged Windows payload
  - windows/meterpreter_reverse_tcp → stageless Windows payload
  - linux/x86/meterpreter/reverse_tcp → staged Linux payload
  - php/meterpreter_reverse_tcp → PHP web shell payload
- Post exploitation modules are run after you have access:
  - Recon → gather more info about target
  - Privilege escalation → get higher access
  - Persistence → maintain access after reboot
  - Credential dumping → steal passwords and hashes
- Kiwi module is Meterpreter version of Mimikatz for credential dumping
- local_exploit_suggester automatically finds privilege escalation paths
- Timestomping modifies file timestamps to hide attacker activity

---

## Gotchas / Surprises

- hashdump requires SYSTEM level privileges, run getsystem first
- hashdump shows SAM(security account manager) encrypted data which stores credential in windows in NTLM format 
- If getsystem fails try migrate into a SYSTEM owned process first
- shell drops you into OS shell but you lose Meterpreter commands there
- Type exit in OS shell to return to Meterpreter
- Migrating into wrong process can kill your session, choose carefully
- background vs exit: background keeps session alive, exit kills it
- load kiwi must be done before running creds_all

---

## Post Exploitation Flow

1. Get Meterpreter session via exploit
2. Run sysinfo and getuid to understand target
3. Run getsystem to escalate privileges
4. Run hashdump to dump password hashes
5. Run ps and migrate into stable process
6. Run local_exploit_suggester for more privesc options
7. Use upload/download to transfer files
8. Cover tracks with timestomp
9. clear tracks with clearev command 

---

## My Understanding in Plain English

Meterpreter is your Swiss army knife once you are inside
a target. It lives in memory, talks back to you over an
encrypted channel, and gives you powerful commands to
move around, steal credentials, escalate privileges and
cover your tracks without touching the disk.

---

## Related Notes

- [[Metasploit Introduction]]
- [[Metasploit Exploitation]]
- [[Exploitation Basics]]
