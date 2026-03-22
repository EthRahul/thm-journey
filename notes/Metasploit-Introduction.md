# Metasploit Introduction
**Path:** CyberSec_101 - Exploitation Basics
**Date:** 2026-03-10
**Difficulty:** Easy

---

## Commands Used

- `msfconsole` → Launch Metasploit console
- `search <keyword>` → Search for a module
- `use <module>` → Select a module
- `info` → Show details about current module
- `show options` → Show required settings
- `set <OPTION> <value>` → Set a value e.g. set RHOSTS 10.10.x.x (setg - set to all modules)
- `run` / `exploit` → Execute the module
- `back` → Return to main prompt
- `show payloads` → List compatible payloads

---

## New Concepts

- Metasploit is a framework with modules for every exploitation phase
- 3 steps: Find exploit → Customize it → Execute it
- Module types:
  - Auxiliary → Scanning and info gathering
  - Exploit → Exploits the vulnerability
  - Payload → Code that runs after exploitation
  - Post → Post-exploitation actions
  - Encoder → Obfuscates payload to avoid detection
- RHOSTS = target IP
- LHOST = your machine IP

---

## Gotchas / Surprises

- Must set ALL required options before running
- Use show options to see what is missing
- run and exploit do the same thing
- Use search then use the number shown not the full path

---

## My Understanding in Plain English

Metasploit is like a weapon loadout system. Pick your
weapon (exploit), attach your bullet (payload), aim at
the target (set RHOSTS), and fire (run).

---

## Related Notes

- [[Exploitation Basics]]
- [[Moniker Link]]
