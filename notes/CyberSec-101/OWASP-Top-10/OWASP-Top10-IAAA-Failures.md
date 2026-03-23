# OWASP Top 10 2025: IAAA Failures
**Path:** CyberSec_101 - OWASP Top 10 (2025)
**Date:** 2026-03-19
**Difficulty:** Easy

---

## What is IAAA
```
I → Identification   → who are you claiming to be?
A → Authentication   → prove who you are
A → Authorization    → what are you allowed to do?
A → Accountability   → logging what you did
```

Every web application should implement all 4 layers.
Failure in any one = vulnerability.

---

## A01: Broken Access Control

- #1 vulnerability on OWASP Top 10
- Occurs when users can access resources beyond their permissions
- Most common and most rewarding bug in bug bounty

**Types of Broken Access Control:**

IDOR (Insecure Direct Object Reference):
```
# Change your ID to access another user's data
https://site.com/user?id=123  →  change to id=124
https://site.com/invoice/download?id=1001 → change to 1002
https://site.com/api/user/123/data → change 123 to 124
```

Horizontal privilege escalation:
```
# Access same level user's data
User A accessing User B's profile/orders/messages
Same role, different account
```

Vertical privilege escalation:
```
# Access higher privileged functions
Regular user accessing admin panel
https://site.com/admin → accessible without admin role
https://site.com/admin/users → manage all users
```

Missing function level access control:
```
# Hidden admin endpoints accessible directly
https://site.com/admin/deleteUser
https://site.com/api/admin/getAllUsers
```

**How to test for Broken Access Control:**
- Log in as User A → copy resource URL
- Log in as User B → paste URL → if accessible = IDOR
- Log out → try accessing authenticated pages directly
- Change role parameter in requests → user=admin
- Increment/decrement IDs in URLs and API calls

---

## A07: Authentication Failures

- Weaknesses in login, session management, password handling
- Allows attackers to compromise passwords or session tokens

**Common Authentication Failures:**

Weak credentials:
```
- Default credentials not changed (admin:admin)
- Weak passwords allowed (123456, password)
- No account lockout after failed attempts
- No rate limiting → brute force possible
```

Session management issues:
```
- Session token in URL (leaks in logs/history)
- Session token not invalidated after logout
- Predictable session tokens
- Missing secure and HttpOnly cookie flags
- No session expiry timeout
```

Missing MFA:
```
- No multi-factor authentication on sensitive accounts
- MFA bypassable via response manipulation
```

Username enumeration:
```
# Different responses reveal valid usernames
"User not found"    → username doesn't exist
"Wrong password"    → username exists
Both should say: "Invalid credentials"
```

**How to test Authentication:**
- Try default credentials on login pages
- Check if account locks after multiple failed attempts
- Inspect session cookie flags in Burp
- Try accessing pages after logout using old session token
- Check if username enumeration is possible via error messages

---

## A09: Logging & Alerting Failures

- When applications don't log security events properly
- Attackers can operate undetected for long periods
- Critical for incident response and forensics

**What should be logged:**
```
- Failed login attempts
- Successful logins with timestamp and IP
- Access control failures
- Input validation failures
- Admin actions
- API calls with authentication failures
```

**What logging failures enable:**
```
- Attackers brute force without triggering alerts
- Data exfiltration goes unnoticed
- No evidence trail for forensic investigation
- Attackers maintain persistence undetected
```

**From attacker perspective:**
```
- No logs = no evidence = harder to detect
- Exploit during off hours when no one monitors
- Cover tracks by deleting logs if accessible
- timestomp to modify file timestamps
```

---

## Bug Bounty Relevance
```
A01 Broken Access Control → most common payout
    → test every ID parameter you find
    → test every endpoint while logged out
    
A07 Authentication Failures → easy wins
    → check for rate limiting on login
    → check session token after logout
    → look for username enumeration
    
A09 Logging Failures → usually informational severity
    → mention in reports as supporting finding
```

---

## Gotchas / Surprises

- Broken Access Control is #1 because developers
  focus on authentication but forget authorization
- IDOR doesn't require any special tools — just
  changing a number in a URL
- Session tokens should NEVER appear in URLs
- After logout old session tokens should be invalid
- Username enumeration is a valid bug bounty finding
- Logging failures alone rarely pay bounties but
  strengthen the impact of other findings

---

## My Understanding in Plain English

IAAA failures happen when apps correctly identify
and authenticate users but fail to check what they
are authorized to do. Broken Access Control means
you can access things you shouldn't. Authentication
failures mean the login process itself is weak.
Logging failures mean attackers can do all of this
without anyone noticing.

---

## Related Notes

- [[Web Application Basics]]
- [[Burp Suite Basics]]
- [[SQLi]]
- [[OWASP Top 10]]

---

## OWASP Top 10 2025 — Correct Official List
```
A01 → Broken Access Control          (same as 2021)
A02 → Security Misconfiguration      (was A05 in 2021, moved up)
A03 → Software Supply Chain Failures (NEW in 2025)
A04 → Cryptographic Failures         (was A02 in 2021)
A05 → Injection                      (was A03 in 2021)
A06 → Insecure Design                (was A04 in 2021)
A07 → Authentication Failures        (same as 2021)
A08 → Data Integrity Failures        (same as 2021)
A09 → Security Logging & Alerting Failures (same position)
A10 → Mishandling of Exceptional Conditions (NEW in 2025)
```

Key changes from 2021 to 2025:
- Security Misconfiguration jumped from A05 to A02
- Two new categories added: A03 and A10
- SSRF absorbed into A01 Broken Access Control
- A10 covers verbose errors, fail-open logic, stack trace leaks
