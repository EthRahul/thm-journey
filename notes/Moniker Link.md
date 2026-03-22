# Room Name
**Path:** CyberSec_101 - Exploitation Basics
**Date:** 10/3/26
**Difficulty:** Easy

---

## Commands Used
| Command                        | What it does |
| ------------------------------ | ------------ |
| none this room — theory based) |              |

---

## New Concepts

- CVE-2024-21413 is a vulnerability in Microsoft Outlook 
- It bypasses Outlook's Protected View by using a Moniker Link
- A Moniker Link tricks Outlook into making an outbound connection, leaking NTLM credentials to an attacker 
- NTLM credentials can then be cracked offline or used in pass-the-hash attacks
---

## Gotchas / Surprises
- Protected View in Outlook can be bypassed just by previewing an email — no click needed 
- This affects fully patched systems unless the specific CVE patch is applied- 

---

## Related Notes
- [[NTLM]] 
[[Credential Harvesting]] 
 [[Exploitation Basics]]

## My Understanding (in plain English)
An attacker sends a crafted email → Outlook auto-connects 
to attacker server → leaks Windows credentials without 
victim clicking anything.