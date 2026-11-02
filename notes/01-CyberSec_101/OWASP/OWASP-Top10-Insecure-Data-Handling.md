# OWASP Top 10 2025: Insecure Data Handling
**Path:** CyberSec_101 - OWASP Top 10 (2025)
**Date:** 2026-03-20
**Difficulty:** Easy

---

## A04: Cryptographic Failures

- Weak algorithms: MD5, SHA1, DES, AES-ECB
- Hardcoded keys in JS files
- Data transmitted over HTTP not HTTPS
```
Weak   → MD5, SHA1, DES, AES-ECB
Strong → bcrypt, SHA256, AES-GCM
```
 
 Can use Cyberchef also for decrypting the encrypted text 
 
Decrypt AES-ECB when key is found:
```bash
echo "base64text" | base64 -d | openssl enc -aes-128-ecb -d \
-K $(echo -n "found-key" | xxd -p) -nopad 2>/dev/null
```

---

## A05: Injection

App takes user input and passes it directly into a system
that can execute commands — database, shell, template engine.

**Types covered in room:**

SQL Injection → manipulates database queries
```sql
' OR 1=1--                                    ← login bypass
' UNION SELECT username,password FROM users-- ← data extraction
```

Command Injection → executes OS commands on server
```bash
# Normal: ping 8.8.8.8
# Injected: 8.8.8.8; whoami
# Separators: ; | && ||
```

Server Side Template Injection (SSTI) → injects into template engine
```
# Template engines render dynamic content
# Injecting expressions executes server side code
{{7*7}}        ← test payload, returns 49 if vulnerable
{{config}}     ← dump config
```

AI Prompt Injection → manipulates AI model behaviour
```
# Injecting instructions into AI prompts
# Makes AI ignore its original instructions
# Growing threat as AI is integrated into apps
```

**How to prevent:**
- Use prepared statements for SQL
- Never pass user input directly to shell
- Input validation and sanitisation
- Escape dangerous characters

---

## A08: Software or Data Integrity Failures

**Software Integrity:**
- Using unverified third party libraries
- No SRI hash on CDN scripts
```html
<!-- Safe - browser verifies hash -->
<script src="cdn.js" integrity="sha384-abc123..."></script>
```

**Data Integrity — Insecure Deserialization:**
- Pickle executes code during deserialization
- `__reduce__` runs automatically on deserialization
```python
import pickle, subprocess, base64

class Exploit(object):
    def __reduce__(self):
        return (subprocess.check_output, (['cat', 'flag.txt'],))

print(base64.b64encode(pickle.dumps(Exploit())).decode())
```

Safe alternatives: JSON, YAML with safe_load()

---

## Gotchas / Surprises

- os.system returns exit code not output → use subprocess
- SSTI test payload → {{7*7}} if returns 49 = vulnerable
- AI prompt injection is a new and growing attack type
- Injection appears on OWASP 2025 list twice → that critical
- Prepared statements completely prevent SQLi

---

## Related Notes

- [[SQL Fundamentals]]
- [[SQLMap Basics]]
- [[OWASP Top 10 Application Design Flaws]]
- [[Shells Overview]]
