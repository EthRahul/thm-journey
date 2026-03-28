# Burp Suite: Intruder
**Path:** Jr Penetration Tester - Burp Suite
**Date:** 2026-03-28
**Difficulty:** Easy

---

## What is Intruder

Intruder is Burp's automated request fuzzer. It takes a captured
HTTP request, lets you mark positions in it, then fires a wordlist
or number range at those positions automatically — observing each
response to find vulnerabilities.
```
Think of it as: Repeater + automation + wordlists
```

Community Edition is rate-limited to ~1 req/sec.
Pro version has no rate limit.

---

## Positions

Positions are the parts of the request you want Intruder to replace
with payloads. Marked using § symbols.
```
§value§   → Intruder will replace this with each payload

# Example
GET /support/ticket/§788§ HTTP/1.1

# Adding positions
Auto § → Intruder guesses likely positions
Add §  → manually highlight text and wrap in §
Clear § → remove all markers
```

---

## Payloads

What gets inserted into the § positions.
```
Simple list  → custom wordlist, paste or load from file
Numbers      → numeric range (From / To / Step)
Runtime file → load from file at attack time
Username generator → generates usernames from full names
```

### Numbers payload config (used for IDOR):
```
Type: Sequential
From: 1
To:   200
Step: 1
Base: Decimal
```

---

## Attack Types

### Sniper (most common)
One payload set, one position at a time.
If multiple positions exist, cycles through each one independently.
```
Positions: §user§ and §pass§
Payload:   [a, b, c]
Requests:  user=a pass=original
           user=b pass=original
           user=original pass=a
           ...
```
Use for: fuzzing single parameters, IDOR enumeration.

### Battering Ram
One payload set, inserts same payload into ALL positions simultaneously.
```
Positions: §user§ and §pass§
Payload:   [admin, test]
Requests:  user=admin pass=admin
           user=test  pass=test
```
Use for: when username and password must match (same value).

### Pitchfork
Multiple payload sets, one per position, iterates in parallel.
```
Positions: §user§ and §pass§
Payload 1: [admin, user]
Payload 2: [pass123, letmein]
Requests:  user=admin pass=pass123
           user=user  pass=letmein
```
Use for: credential stuffing with known username:password pairs.

### Cluster Bomb
Multiple payload sets, tests every combination (cartesian product).
```
Positions: §user§ and §pass§
Payload 1: [admin, user]
Payload 2: [pass123, letmein]
Requests:  user=admin pass=pass123
           user=admin pass=letmein
           user=user  pass=pass123
           user=user  pass=letmein
```
Use for: brute forcing login forms where both fields are unknown.
Warning: combinations grow fast — 100 users × 100 passwords = 10,000 requests.

---

## Attack Type Summary
```
Sniper       → 1 list, 1 position at a time     → IDOR, fuzzing
Battering Ram → 1 list, all positions same value → same value needed
Pitchfork    → multiple lists, parallel          → credential stuffing
Cluster Bomb → multiple lists, all combinations  → brute force login
```

---

## Practical Example — IDOR via Sniper

Enumerate ticket IDs to find tickets belonging to other users:
```
1. Capture GET /support/ticket/788 in Proxy
2. Send to Intruder (Ctrl+I)
3. Clear § → highlight 788 → Add §
4. Payloads tab → Type: Numbers → From:1 To:200 Step:1
5. Start Attack
6. Sort results by Length column
7. Different length = different content = someone else's ticket
```

---

## Practical Example — Brute Force Login via Cluster Bomb
```
1. Capture POST /login with username and password fields
2. Send to Intruder
3. Clear § → mark §username§ and §password§
4. Attack type: Cluster Bomb
5. Payload 1: username wordlist
6. Payload 2: password wordlist
7. Start Attack
8. Sort by Status or Length
9. 302 redirect or different length = successful login
```

---

## Reading Attack Results
```
Status code  → 302 instead of 200 = redirect = likely success
Length       → different from baseline = different response content
Response tab → check manually for flags or error messages
```

Always establish a baseline first — send one normal request
and note its status code and length. Anything that deviates
from that baseline is worth investigating.

---

## Payload Processing

Rules applied to each payload before it gets sent.
```
Add prefix    → prepend text to every payload
Add suffix    → append text to every payload
Match/Replace → substitute strings in the payload
Hash          → MD5/SHA hash the payload before sending
URL encode    → encode special characters
```
Useful when the app expects hashed passwords or encoded values.

---

## Payload Encoding

Bottom of Payloads tab — URL encodes special characters in payloads.
```
Enabled by default → encodes: /\=<>?+&*;{}[]^#
Disable if         → your payload is already encoded
                   → or encoding breaks the injection
```

---

## Remediation
```
- Rate limiting on login endpoints
- Account lockout after N failed attempts
- CAPTCHA on sensitive forms
- Unpredictable IDs (UUIDs not sequential integers)
- Proper authorization checks on every resource
```

---

## Gotchas / Surprises

- Community Edition is throttled — 200 requests takes ~3 minutes
- Always clear § before adding your own positions or auto-adds
  will create extra unwanted positions
- Cluster Bomb request count = payload1 size × payload2 size
- Sort by Length first — status codes alone can be misleading
- Intruder URL-encodes payloads by default — disable if your
  payload already contains encoded characters
- For IDOR — use Numbers payload type not Simple list

---

## My Understanding in Plain English

Intruder is essentially a for-loop for HTTP requests. You pick
a position in your request, give it a list of values, and it
fires every value automatically while logging every response.
The four attack types control how multiple positions are handled —
Sniper is one at a time, Cluster Bomb is every combination.
The key skill is reading the results table — you're looking for
the response that stands out from the crowd by length or status code.

---

## Related Notes

- [[Burp-Suite-Repeater]]
- [[Burp-Suite-Basics]]
- [[Authentication-Bypass]]
- [[IDOR]]
