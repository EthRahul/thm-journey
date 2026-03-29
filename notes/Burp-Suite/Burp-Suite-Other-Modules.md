# Burp Suite: Other Modules
**Path:** Jr Penetration Tester - Burp Suite
**Date:** 2026-03-29
**Difficulty:** Easy

---

## Decoder

Transforms encoded or hashed data into readable form and vice versa.
No need for external tools like base64decode.org anymore — Decoder
handles everything natively inside Burp.

### Encoding/Decoding
```
Paste any encoded text → select encoding type → instantly decoded

Supported formats:
URL encoding     → %20, %3D, %2F etc
HTML encoding    → &amp; &lt; &gt; etc
Base64           → dGVzdA==
ASCII Hex        → 74657374
Hex              → raw hex values
Octal            → octal representation
Binary           → binary string
Gzip             → compressed data
```

### How to Use
```
1. Burp → Decoder tab
2. Paste encoded text in top field
3. Click "Decode as" dropdown → select format
4. Decoded output appears below instantly
5. Can chain multiple decode operations
   e.g. URL decode → then Base64 decode
```

### Hashing
```
Decoder can also generate hashes of any input:
MD5, SHA-1, SHA-256, SHA-512

Use: Click "Hash" dropdown instead of "Decode as"
Useful for: identifying hash types, generating test hashes,
            verifying hash values
```

---

## Comparer

Compares two pieces of data and highlights the differences.
Like a diff tool but built into Burp — extremely useful for
spotting subtle changes between two responses.

### When to Use Comparer
```
Two responses with different lengths → find what changed
Before/after adding a payload       → see what the injection affected
Two user accounts responses         → spot privilege differences
Login success vs failure response   → identify what changes
```

### How to Use
```
1. Right-click any request/response in Burp → Send to Comparer
2. Add a second item the same way
3. Comparer tab → select both items → Compare
4. Choose: Words (highlights word-level diff)
              Bytes (highlights byte-level diff)
5. Yellow = modified, Red = deleted, Green = added
```

### Practical Example
```
Scenario: Two users — normal user and admin
1. Log in as normal user → capture response → send to Comparer
2. Log in as admin → capture response → send to Comparer
3. Compare words → spot admin-only fields or tokens
4. Use those differences to escalate privileges
```

---

## Sequencer

Analyses the randomness/entropy of tokens to determine
if they are predictable and therefore vulnerable to forgery.

### What it Tests
```
Session cookies    → are they truly random?
CSRF tokens        → can they be predicted?
Password reset tokens → are they sequential?
Any token          → entropy analysis
```

### Live Capture
```
1. Find a request that generates a token
   e.g. login request that sets a session cookie
2. Right-click → Send to Sequencer
3. Sequencer tab → select the token field to analyse
4. Click "Start live capture"
5. Burp fires hundreds of requests automatically
   and collects all the generated tokens
6. Click "Analyse now" when enough tokens collected (200+)
```

### Analysis Results
```
Overall result:    Excellent / Good / Poor / Extremely Poor
Bits of entropy:   higher = more random = more secure
                   lower  = predictable = vulnerable

If result is Poor → tokens are predictable → session hijacking possible
If result is Excellent → tokens are properly random → secure
```

### Manual Analysis
```
Already have a list of tokens? Use Manual load:
Paste tokens → Analyse → same entropy report generated
Useful when: you've collected tokens via other means
             or testing offline token samples
```

---

## Organizer

Stores and annotates HTTP requests for later reference.
Like a personal evidence locker inside Burp.

### When to Use
```
Found an interesting request during recon → save it
Want to document a vulnerability         → annotate and save
Need to reference a request later        → store it here
Building a report                        → collect evidence
```

### How to Use
```
Right-click any request → Send to Organizer
Add notes/annotations to each saved request
Colour code requests by severity or type
Export saved requests for reporting
```

---

## Module Comparison
```
Decoder    → transform data (encode/decode/hash)
Comparer   → diff two responses or requests
Sequencer  → test token randomness/entropy
Organizer  → save and annotate interesting requests
```

---

## Gotchas / Surprises

- Decoder can chain operations — URL decode first then Base64
  if data is double encoded
- Comparer Words view is usually more readable than Bytes view
  for HTTP responses
- Sequencer needs 100+ tokens minimum for reliable analysis
  — 200+ is better
- Poor entropy in Sequencer = critical finding in a pentest report
- Organizer is often overlooked but invaluable for report writing
- Hash in Decoder is one-way — you can generate hashes but
  cannot reverse them (that's what hash cracking tools are for)

---

## My Understanding in Plain English

Decoder is your Swiss Army knife for encoding — paste anything
encoded and it decodes it instantly, or encode your payloads
before sending. Comparer is a diff tool that shows exactly
what changed between two requests or responses — great for
spotting privilege differences between accounts. Sequencer
tests whether session tokens are truly random or predictable
— a low entropy score means sessions can be forged. Organizer
is a notepad for interesting requests you want to revisit during
reporting.

---

## Related Notes

- [[Burp-Suite-Basics]]
- [[Burp-Suite-Repeater]]
- [[Burp-Suite-Intruder]]
- [[Authentication-Bypass]]
