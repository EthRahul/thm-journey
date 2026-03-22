# OWASP Top 10 2025: Application Design Flaws
**Path:** CyberSec_101 - OWASP Top 10 (2025)
**Date:** 2026-03-19
**Difficulty:** Easy

---

## A02: Security Misconfiguration

- Default credentials left unchanged
- Unnecessary services or endpoints exposed
- Verbose error messages leaking stack traces
- Debug mode left ON in production (Werkzeug console)
- Misconfigured cloud storage (S3, Azure Blob)
- Exposed API endpoints without authentication

Real example from room:
```
# Debug console exposed → full server access
http://target.com/console

# Verbose error leaking sensitive data
http://target.com/api/user/invalidinput
→ stack trace reveals internal code and secrets
```

How to find it:
- Gobuster to find hidden endpoints
- Send invalid input to trigger verbose errors
- Check JS files for hardcoded secrets
- Try /admin, /debug, /console, /config endpoints

---

## A03: Software Supply Chain Failures

- Using third party libraries with known vulnerabilities
- Outdated frameworks or dependencies
- Unverified packages from public repositories
- Compromised build pipelines or CI/CD systems

Real world examples:
- Log4Shell → vulnerability in Log4j library used by millions
- SolarWinds → compromised build pipeline affected thousands

How to identify vulnerable components:
```bash
# Check response headers for version info
curl -I http://target.com
# X-Powered-By: Express 4.17.1
# Server: Apache/2.4.49 → check CVE for this version
```

---

## A04: Cryptographic Failures

- Sensitive data transmitted over HTTP not HTTPS
- Weak encryption algorithms (MD5, SHA1, DES, ECB mode)
- Hardcoded encryption keys in source code
- Keys stored in client side JS files
```
Weak hashing   → MD5, SHA1 (crackable)
Strong hashing → bcrypt, SHA256, Argon2

Weak encryption  → DES, AES-ECB
Strong encryption → AES-GCM, AES-CBC with random IV
```

Real example from room:
```javascript
// Found in public JS file
const SECRET_KEY = "my-secret-key-16";
const ENCRYPTION_MODE = "ECB";
```

Decrypt AES-ECB with found key:
```bash
echo "encryptedBase64" | base64 -d | openssl enc -aes-128-ecb -d \
-K $(echo -n "my-secret-key-16" | xxd -p) -nopad 2>/dev/null
```

---

## A06: Insecure Design

- Security not considered during design phase
- Missing rate limiting on sensitive endpoints
- Flawed business logic that can be abused
- Trusting client side data for security decisions

Examples:
```
# No rate limiting → brute force possible
# Business logic flaw → negative quantity in cart
# Predictable password reset tokens
# No limit on OTP attempts
```

Difference from misconfiguration:
```
Misconfiguration → correct design, wrong implementation
Insecure Design  → flawed from the beginning
                   no config fix can solve it
```

---

## Bug Bounty Relevance
```
A02 → exposed debug endpoints, verbose errors → easy to find
A03 → outdated libraries → check CVE database
A04 → hardcoded keys in JS → high severity payout
A06 → business logic bugs → hard to find but high value
```

---

## Gotchas / Surprises

- ECB mode is weak — same plaintext = same ciphertext
- Hardcoded secrets in JS = anyone can view page source
- Verbose errors in production = instant finding
- Insecure design cannot be patched — must be redesigned
- Supply chain vulns affect you even with perfect code

---

## My Understanding in Plain English

These are design level failures baked into how the
application was built. Misconfiguration leaves doors
open by mistake. Cryptographic failures use weak locks.
Supply chain failures trust unverified third party code.
Insecure design has fundamental logic flaws that
no amount of patching can fix.

---

## Related Notes

- [[OWASP Top 10 IAAA Failures]]
- [[Web Application Basics]]
- [[Burp Suite Basics]]
- [[JavaScript Essentials]]
- [[Gobuster Basics]]
