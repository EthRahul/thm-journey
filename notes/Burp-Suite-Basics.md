# Burp Suite: The Basics
**Path:** CyberSec_101 - Web Hacking
**Date:** 2026-03-15
**Difficulty:** Easy

---

## Commands Used

- `burpsuite` → Launch Burp Suite on Kali Linux

---

## What is Burp Suite

- Burp Suite is a web application pentesting framework
- Acts as a proxy — sits between your browser and target server
- Intercepts, inspects and modifies HTTP/HTTPS requests and responses
- Made by PortSwigger
- Three editions:
  - Community → free, limited features
  - Professional → paid, full features, most used in industry
  - Enterprise → automated scanning for organizations

---

## Features of Burp Community

- Proxy → intercept and modify live HTTP traffic
- Repeater → manually resend and modify requests
- Intruder → automated fuzzing and brute forcing (rate limited in community)
- Decoder → encode/decode data (Base64, URL, HTML etc)
- Comparer → compare two pieces of data side by side
- Sequencer → analyze randomness of tokens
- Target → site map and scope management

---

## The Dashboard

- Active tasks panel → shows running scans/tasks
- Event log → logs Burp activity and errors
- Issue activity → shows vulnerabilities found (Pro only)
- Advisory → details about found issues

---

## Navigation

- Keyboard shortcut to switch tabs: `Ctrl + Shift + D` → Dashboard
- Right click any request anywhere → Send to Repeater/Intruder

---

## Introduction to Burp Proxy

- Proxy intercepts traffic between browser and server
- Default listener: `127.0.0.1:8080`
- Intercept ON → every request pauses and waits
- Intercept OFF → traffic flows normally, still logged
- HTTP History → log of all requests even when intercept is off
- WebSockets History → log of WebSocket traffic

Proxy workflow:
```
Browser → FoxyProxy → Burp Proxy (8080) → Target Server
Browser ← FoxyProxy ← Burp Proxy (8080) ← Target Server
```

---

## FoxyProxy Setup (Firefox)

1. Install FoxyProxy extension in Firefox
2. Click FoxyProxy icon → Options → Add new proxy
3. Set:
   - Title: Burp
   - Proxy Type: HTTP
   - IP: 127.0.0.1
   - Port: 8080
4. Save → click FoxyProxy icon → select Burp profile
5. Now all Firefox traffic routes through Burp

---

## Intercept Workflow (Correct Way)
```
Intercept OFF by default
       ↓
Browse normally to find the request you want
       ↓
Turn Intercept ON
       ↓
Trigger the action (login, search, submit)
       ↓
Burp captures request → inspect/modify → Forward
       ↓
Turn Intercept OFF again
```

Key point: One page load = many requests.
Keep clicking Forward until page loads,
or just turn Intercept OFF to let all through.

---

## Site Map and Issue Definitions

- Target tab → Site Map → shows all discovered endpoints
- Every URL Burp has seen is recorded here
- Right click any host → Add to scope
- Issue Definitions → database of 100+ vulnerability types
  with descriptions (useful reference even in community)

---

## Scoping and Targeting

- Scope limits Burp to only your target domain
- Prevents capturing traffic from every site you visit
- Target tab → Scope → Add target URL
- In Proxy Settings → enable "And URL is in target scope"
- Now only target traffic is intercepted and logged

Setting scope:
```
Target tab → Scope Settings → Include in scope
→ Add → enter target URL
→ Proxy Settings → check "only intercept in-scope requests"
```

---

## Proxying HTTPS

- Burp uses its own CA certificate to decrypt HTTPS
- Without it → browser shows SSL error for HTTPS sites
- Install Burp CA cert in Firefox:
```
1. With FoxyProxy on, visit: http://burpsuite
   or http://burp in Firefox
2. Click "CA Certificate" to download
3. Firefox → Settings → search "certificates"
4. View Certificates → Authorities → Import
5. Select downloaded .der file
6. Check "Trust this CA to identify websites"
7. OK → now HTTPS sites work through Burp
```

---

## The Burp Suite Browser

- Burp has its own built in Chromium browser
- Proxy tab → Open Browser
- Already configured to use Burp proxy automatically
- CA cert pre-installed — works with HTTPS immediately
- Useful when you don't want to configure Firefox

---

## Repeater (Extra — Very Important)

- Right click any request → Send to Repeater
- Repeater tab → modify request → Send → see response
- Best tool for manually testing vulnerabilities
- Can resend same request hundreds of times with changes
- Used for SQLi, XSS, IDOR, parameter tampering

---

## Decoder (Extra — Very Useful)

- Decoder tab → paste any encoded string
- Can decode: Base64, URL encoding, HTML entities,
  Hex, ASCII, Gzip
- Can encode too — useful for crafting payloads
- Example:
  - `YWRtaW4=` → Base64 decode → `admin`
  - `admin` → Base64 encode → `YWRtaW4=`

---

## Gotchas / Surprises

- One page = many requests → keep forwarding or turn intercept off
- HTTPS sites need CA certificate installed or they wont load
- HTTP History logs everything even when intercept is off
  — very useful for reviewing past requests
- Scope must be set or Burp captures ALL browser traffic
- Intruder is rate limited in Community edition
  — use for small wordlists only
- Right click is your best friend in Burp —
  right click any request to send it anywhere

---

## My Understanding in Plain English

Burp Suite sits between your browser and the target.
Every request you make passes through it first.
You can pause, read, modify and resend any request.
This is how you manually test for web vulnerabilities —
intercept a login request, change the username to
admin, modify parameters, inject payloads, all before
the request even reaches the server.

---

## Related Notes

- [[Web Application Basics]]
- [[SQLi]]
- [[XSS]]
- [[CSRF]]
- [[IDOR]]
