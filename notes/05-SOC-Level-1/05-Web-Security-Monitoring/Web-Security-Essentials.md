# Web Security Essentials

**Path:** SOC Level 1 - Web Security Monitoring
**Date:** 2026-09-05
**Difficulty:** Easy

---

## Why Web Applications Are a Prime Target

- Apps shifted **desktop → web** over three decades: 1990s desktop apps (slow internet) → 2000s dynamic web apps (email, banking, social) → 2010s cloud/SaaS boom → today, almost everything runs in-browser
- The shift buys accessibility, faster updates, and lower resource use on the client side — but it also means the app is **always online, always exposed**, and usually wired straight into backend systems (databases, internal APIs), so a compromised web app is often step one of a much bigger attack

### Risk Split

| Perspective | Risk |
|---|---|
| **Web App Owner** | App must be secured 24/7, reachable from anywhere, owns legal/practical responsibility for user data, has to keep pace with constantly evolving threats |
| **Web App User** | Personal data lives inside the app (possibly insecurely stored), one breached account/browser can cascade to other services, exposure risks identity theft or financial loss |

### Real-World Breaches

| Breach | Year | Root Cause | Impact |
|---|---|---|---|
| Equifax | 2017 | Unpatched Apache vulnerability (CVE-2017-5638) | ~150M customer records exposed |
| Capital One | 2019 | Misconfigured Web Application Firewall (WAF) | 100M+ customers' financial/personal data exposed |

---

## How a Web Service Actually Works

- **Request-response cycle:** browser sends a request → server processes it and verifies access → server returns a response (webpage, image, search results, account data)
- Attackers abuse this cycle by flooding the server with requests, bypassing access controls, or tricking the server into running unintended commands

### The 3 Core Components of Any Web Service

| Component | Role |
|---|---|
| **Application** | The code, images, styles, icons — dictates how the site looks and functions |
| **Web Server** | Hosts the application, listens for requests, sends back responses |
| **Host Machine** | The underlying OS (Linux/Windows) that the web server and application run on |

### Common Web Servers

| Web Server | Typical Use |
|---|---|
| **Apache** | Most common choice for simple sites/blogs, especially WordPress |
| **Nginx** | High-performance, industry standard — used by Netflix, Airbnb, GitHub |
| **IIS** | Microsoft's web server, common in enterprise/Windows environments |

---

## What's Essential in a Web Request/Response Flow

Every request that hits a web server can be recorded as an **access log** entry. This is the web equivalent of flow data — even without seeing full page content, these fields are usually enough to reconstruct what happened:

```
Client IP | Timestamp | Method (GET/POST) | Requested URL/Resource | Response Status Code | User-Agent
```

- **GET** → retrieve a resource (e.g. loading a page)
- **POST** → submit data to the server (e.g. a login form)

**Example benign sequence (client `10.10.10.100`):**
```
GET  /index.html      → loads homepage
GET  /login.html      → loads login page
POST /login.html      → submits credentials
GET  /myaccount.html  → loads account page after auth
```

Nothing here looks malicious on its own — but this is exactly the kind of sequence an analyst reconstructs from logs when investigating account takeover, credential stuffing, or session abuse.

---

## Protecting Each Layer

Controls either provide **visibility** (e.g. logging — see what happened) or **mitigation** (actively stop/limit an attack in progress).

| Layer | Protections |
|---|---|
| **Application** | Secure coding (avoid insecure functions, proper error handling, strip sensitive info from output), input validation/sanitization (blocks injection), role-based access control |
| **Web Server** | Logging (access logs), Web Application Firewall (filters/blocks malicious traffic), CDN (reduces direct exposure, often bundles a WAF) |
| **Host Machine** | Least privilege (don't run services as root), system hardening (disable unused services/ports), antivirus (endpoint-level malware blocking) |
| **All Three** | Strong authentication, patch management (keep app dependencies, web server, and host OS up to date) |

---

## Defense Systems

### Content Delivery Network (CDN)
- Caches and serves content from edge servers close to the user → lower latency, and acts as a buffer in front of the real (origin) server
- **Security benefits:** IP masking (hides origin server IP), DDoS absorption (large capacity soaks up flood traffic), enforced HTTPS/TLS by default, many CDNs bundle an integrated WAF (Cloudflare, AWS CloudFront, Azure Front Door)

### Web Application Firewall (WAF)
- Inspects incoming HTTP traffic and blocks/logs requests against a rule set — like a bouncer checking everyone at the door

| WAF Type | Description |
|---|---|
| **Cloud-based / Reverse Proxy** | Sits in front of the web server; easy to deploy, scales well |
| **Host-based** | Installed directly on the web server; per-application control |
| **Network-based** | Physical/virtual appliance at the network perimeter; suited to enterprise scale |

| Detection Method | How It Works | Example |
|---|---|---|
| **Signature-based** | Matches known attack patterns/payloads | Request with User-Agent `sqlmap/1.8.1` |
| **Heuristic-based** | Analyzes context/behavior of the request | Long query string with special chars: `search?q=%3Cscript%20(1)` |
| **Anomaly/Behavioral** | Flags deviation from normal traffic | Single IP making repeated login attempts rapidly |
| **Location/IP Reputation** | Uses geolocation + threat intel to block | Request from an IP outside the normal business region |

### Antivirus (AV)
- Endpoint-level protection (desktops, laptops, servers), mostly **signature-based** against known malware
- Web attacks mostly hit the application layer, not the host — but AV still matters for catching malicious uploads (web shells, post-exploitation tools)
- Just one layer in a defense-in-depth strategy, not a complete solution on its own

---

## Key Terms

| Term | Meaning |
|---|---|
| **Mitigation** | Any action taken to actively stop or limit damage from a threat |
| **Access Log** | Server-side record of every request: client IP, timestamp, resource, status code, user agent |
| **Least Privilege** | Running services/users with the minimum permissions needed, nothing more |
| **Patch Management** | Keeping application dependencies, web server, and host OS updated against known vulnerabilities |
| **WAF** | Web Application Firewall — inspects and filters HTTP traffic against rules |
| **CDN** | Content Delivery Network — distributes/caches content globally and shields the origin server |
| **Host Machine** | The OS + environment (Linux/Windows) that runs the web server and application |
| **Defense-in-Depth** | Layering multiple, independent controls (app, server, host) so no single failure exposes the whole system |

---

## Practice Scenario

- Hands-on hardening exercise on a sample site (**Secure-A-Site**) — applied the Task 4/5 best practices across all three layers: Web Application, Web Server, Host Machine
- Completed and captured a flag for each layer once it was properly secured

---

## My Understanding in Plain English

A web app isn't one thing — it's three stacked layers (the code, the server running it, and the machine underneath), and each layer needs its own defenses. The app layer needs clean, validated code; the server layer needs logging plus something like a WAF or CDN standing in front of it; and the host layer needs to be locked down so a compromise in the app doesn't turn into full control of the machine. None of these tools — WAF, CDN, AV — is a silver bullet on its own; they're only effective stacked together, which is the whole idea behind defense-in-depth. Access logs matter because they're the paper trail: even a completely normal-looking GET/POST sequence is exactly what you'd pull up first when investigating whether an attacker walked through the front door.

---

## Related Notes

- [[Network-Traffic-Basics]]
- [[HTTP-In-Detail]]
- [[Web-Application-Basics]]
