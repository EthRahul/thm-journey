# Detecting Web Attacks

**Path:** SOC Level 1 - Web Security Monitoring
**Date:** 2026-09-05
**Difficulty:** Easy

---

## What Are We Detecting?

Web attacks are one of the most common ways attackers gain a foothold, since public-facing sites and apps usually sit in front of databases and other valuable infrastructure. This room splits web attacks into two broad classes, then covers three ways to detect them: **logs**, **network traffic**, and **WAFs**.

| Class | Where It Happens | Analyst Visibility |
|---|---|---|
| **Client-Side** | Inside the victim's own browser/device | Little to none — logs and network captures rarely show it |
| **Server-Side** | On the web server / application / backend | Good — every request leaves a trail in logs and network traffic |

---

## Client-Side Attacks

- Abuse weaknesses in **user behavior** or the user's device/browser rather than the server itself
- Rising use of third-party plugins/dynamic content keeps expanding this attack surface
- Example: a hidden, invisible element on a page silently loads a malicious site in the background and steals the victim's session cookies — nothing about the visible page looks wrong

| Attack | Description |
|---|---|
| **XSS (Cross-Site Scripting)** — most common client-side attack | Malicious script gets stored/reflected by a trusted site and runs in the victim's browser (e.g. an unfiltered comment box storing `<script>...</script>`) |
| **CSRF (Cross-Site Request Forgery)** | Victim's browser is tricked into firing off unauthorized requests while still authenticated |
| **Clickjacking** | Invisible elements are overlaid on legitimate content so the user "clicks" something they never intended to |

**SOC limitation:** since the malicious code runs client-side, it often generates no unusual HTTP request or network traffic — server logs and packet captures give little to no visibility. Detecting these usually needs browser-side controls or endpoint monitoring instead.

---

## Server-Side Attacks

- Exploit weaknesses in the **web server, application code, or backend** — misconfigurations, flawed input handling, vulnerable server logic
- Unlike client-side attacks, these **do** leave evidence: every request is processed and (usually) logged by the server, and travels across the network where it can be captured

| Attack | Description |
|---|---|
| **Brute-force** | Repeated automated login attempts across many username/password combos (e.g. T-Mobile, 2021 — 50M+ customer records exposed via brute-force) |
| **SQL Injection (SQLi)** | Abuses queries built by string concatenation instead of parameterization, letting an attacker alter the SQL logic to read/dump/modify the database (e.g. MOVEit, 2023 — 2,700+ organizations affected) |
| **Command Injection** | Unsanitized user input gets passed straight to the OS, letting attackers run system commands with the application's own privileges |

---

## Log-Based Detection

Every request hitting a web server can leave a trace in **access** and **error logs**.

### Access Log Fields

| Field | Suspicious Indicator |
|---|---|
| **Client IP Address** | Known-malicious IP, or outside the expected geographic range |
| **Timestamp + Requested Page** | Requests at unusual hours, or many requests in a short burst |
| **Status Code** | Repeated `404`s (scanning for pages), or an unexpected `302`/`200` after failed attempts |
| **Response Size** | Significantly smaller/larger than a typical response |
| **Referrer** | Referring pages that don't match normal site navigation |
| **User-Agent** | Outdated browsers or known attack tools (`sqlmap`, `wpscan`, `FFUF`) |

### A Typical Attack Sequence in Logs

1. **Directory fuzz** — attacker probes for hidden pages/forms; repeated `200`s mark valid finds
2. **Brute-force** — rapid repeated `POST` requests to a login form; the request that finally returns `302 Found` (redirect) is the successful login
3. **SQL injection** — once authenticated, the attacker submits payloads like `' OR '1'='1` or `1' OR 'a'='a` on a search/form field; if the app builds SQL dynamically instead of using parameterized queries, the database can be dumped

### Log Limitations

- Access logs generally do **not** capture the body of `POST` requests (e.g. submitted credentials/payloads) — you can see *that* a login happened, not the actual password
- `GET` query strings may or may not be logged depending on server software/config
- Example log line: `10.10.10.100 [12/Aug/2025:14:32:10] "POST /login HTTP/1.1" 200 532 "/home.html" "Mozilla/5.0"` — shows method, page, and status, but nothing about what was submitted

---

## Network-Based Detection

Network traffic capture is far more verbose than logs — it can reveal full HTTP headers, `POST` bodies, cookies, and uploaded/downloaded files (unless the traffic is encrypted, e.g. HTTPS/SSH, where only metadata is visible without the decryption keys).

- **Wireshark filter examples used in this room:**
  - `ip.dst == 10.10.20.200` — isolate traffic to the target server
  - `http.user_agent` — pull out the attacking tool's User-Agent
  - `http.response.code == 302` — jump straight to the successful login attempt
- **Follow → HTTP Stream** reconstructs the full request/response — this is how the actual submitted username/password or SQLi payload becomes visible, and how the returned (dumped) database contents can be read in cleartext
- Revisiting the same attack sequence from the logs task in Wireshark exposes what the logs couldn't: the real credentials used in the brute-force, and the actual data returned by the SQLi payload

---

## Web Application Firewall (WAF)

WAFs sit in front of the application as the first line of defense — inspecting full request packets (similar to Wireshark, but capable of decrypting TLS) and deciding to allow or block based on rules.

### Rule Categories

| Rule Type | Description | Example |
|---|---|---|
| **Block common attack patterns** | Known malicious payloads/indicators | Block User-Agent `sqlmap` |
| **Deny known malicious sources** | IP reputation, threat intel, geo-blocking | Block IPs from a recent botnet campaign |
| **Custom-built rules** | Tailored to the specific application | Allow only `GET`/`POST` to `/login` |
| **Rate-limiting / abuse prevention** | Caps request frequency | Limit login attempts to 5/minute/IP |

- **Rule syntax used in this room:** `IF field-name CONTAINS value THEN action` — e.g. `IF User-Agent CONTAINS "BotTHM" THEN block`
- **Challenge-response:** instead of outright blocking, a WAF can serve a CAPTCHA to verify a human is behind the request — useful when a rule risks false-positiving on legitimate traffic (bot traffic makes up a large share of all web traffic)
- **Threat intel integration:** many WAFs ship built-in rule sets covering the OWASP Top 10 and pull live threat-intel feeds (malicious IPs, botnets, VPNs/anonymizers, known APT infrastructure) to auto-update blocking rules

---

## SOC Investigation Workflow for Web Attacks

```
1. Start with logs → spot the pattern (fuzzing → brute-force → injection)
   but note the gaps: no POST bodies, no actual payload content

2. Pivot to network capture (PCAP) for the same time window
   → Follow HTTP Stream to recover full requests/responses
   → Recover credentials, SQLi payloads, and dumped data

3. Check whether a WAF rule would have caught it
   → known attack pattern? → signature rule
   → known-bad source? → IP reputation / geo-block
   → app-specific abuse? → custom rule or rate-limit

4. Correlate across all three sources (logs + network + WAF)
   instead of treating each alert in isolation
```

---

## Key Terms

| Term | Meaning |
|---|---|
| **Client-Side Attack** | Exploits the user's browser/device — hard to see from server logs or network traffic |
| **Server-Side Attack** | Exploits the server/app/backend — leaves evidence in logs and on the wire |
| **XSS** | Malicious script injected into a trusted site, executes in the victim's browser |
| **SQLi** | Injecting SQL through unsanitized input to manipulate or dump a database |
| **Access Log** | Server-side record of each request — IP, timestamp, status, user-agent, etc. |
| **WAF** | Web Application Firewall — inspects and blocks/allows requests by rule |
| **Follow HTTP Stream** | Wireshark feature that reconstructs a full request/response conversation |
| **Rate-Limiting** | Capping how many requests a source can make in a given time window |

---

## My Understanding in Plain English

Detecting a web attack is really about layering three different views of the same event. Logs tell you *that* something happened — a fuzz scan, a burst of failed logins, a weird form submission — but they're often blind to the actual content, especially anything sent in a POST body. Network captures fill that gap: if you can grab the PCAP for the same window, following the HTTP stream shows you the real password that got brute-forced or the real SQLi payload and what it pulled out of the database. A WAF is the piece that turns this analysis into prevention — once you know the pattern (a bad User-Agent, a malicious IP, a suspicious query string), you write a rule so the next attempt gets blocked or challenged before it ever reaches the app. Client-side attacks are the odd one out — because they run entirely in the victim's browser, none of this — logs, network captures, WAF rules — reliably sees them at all.

---

## Related Notes

- [[Web-Security-Essentials]]
- [[Network-Traffic-Basics]]
- [[Intro-to-Log-Analysis]]
- [[Wireshark-The-Basics]]
- [[OWASP-Top-10]]
