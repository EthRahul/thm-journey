# Man-in-the-Middle Detection
**Path:** SOC Level 1 - Network Security Monitoring  
**Date:** 2026-06-02  
**Difficulty:** Easy

---

## MITM Attacks — An Overview

A Man-in-the-Middle attack = attacker secretly positions themselves between two communicating parties, intercepting and potentially modifying traffic.

### What an Attacker Can Do in MITM
- **Eavesdrop** — read cleartext traffic (credentials, data)
- **Modify** — alter data in transit
- **Replay** — resend captured packets
- **Downgrade** — force HTTPS → HTTP (SSL stripping)
- **Inject** — insert malicious content into traffic

### MITM Attack Flow
```
Normal:   Victim ←————————————→ Gateway/Server
MITM:     Victim ←→ Attacker ←→ Gateway/Server
```

### Common MITM Techniques
| Technique | How |
|---|---|
| ARP Spoofing | Fake ARP replies poison victim's ARP cache |
| DNS Spoofing | Fake DNS responses redirect to attacker's IP |
| SSL Stripping | Downgrades HTTPS to HTTP |
| Evil Twin (Wi-Fi) | Rogue AP mimics legitimate Wi-Fi |
| DHCP Spoofing | Fake DHCP server sets attacker as gateway |

---

## Detecting ARP Spoofing

### How ARP Spoofing Works
1. Attacker sends unsolicited (gratuitous) ARP replies
2. Tells victim: "The gateway's IP is at MY MAC"
3. Tells gateway: "The victim's IP is at MY MAC"
4. Both update ARP cache → all traffic routes through attacker

### ARP Spoofing Indicators in Wireshark

**1. Duplicate IP → Multiple MACs**
- Same IP address resolving to more than one MAC address
- Wireshark flags this automatically: *"Duplicate IP address detected"*

**2. Gratuitous ARP (unsolicited replies)**
- ARP reply sent without a request
- `arp.isgratuitous == 1`

**3. High volume of ARP replies from one host**
- One MAC sending ARP replies for many different IPs

### Wireshark Filters
```
# All ARP traffic
arp

# Gratuitous ARP
arp.isgratuitous == 1

# ARP replies only
arp.opcode == 2

# Wireshark's built-in duplicate IP detection
arp.duplicate-address-detected

# ARP from specific suspicious MAC
eth.src == aa:bb:cc:dd:ee:ff and arp
```

### ARP Cache Check (on live host)
```bash
arp -a          # view ARP table
# Red flag: two different IPs sharing the same MAC → one is spoofed
```

---

## Unmasking DNS Spoofing

### How DNS Spoofing Works
1. Attacker intercepts DNS query from victim
2. Sends fake DNS response before legitimate server replies
3. Victim's DNS cache stores attacker's IP for the domain
4. All connections to that domain go to attacker

### DNS Spoofing vs DNS Tunneling
| | DNS Spoofing | DNS Tunneling |
|---|---|---|
| Goal | Redirect victim to fake site | Exfiltrate data / C2 |
| Direction | Attacker → Victim | Victim → Attacker's DNS server |
| Indicator | Fake response IP | Long query names, high volume |

### DNS Spoofing Indicators
- DNS response IP doesn't match known legitimate IP
- Multiple different responses for the same domain query
- Response arrives suspiciously fast (attacker pre-empts real server)
- TTL values unusually low or inconsistent

### Wireshark Filters
```
# All DNS traffic
dns

# DNS responses only
dns.flags.response == 1

# Specific domain queries
dns.qry.name == "targetdomain.com"

# Spot multiple different answers for same query
# → filter dns, then sort by dns.qry.name and compare dns.a values
```

### Verification
- Compare DNS response IP against known good IP (e.g. from another trusted source)
- Check if response IP is in an unexpected ASN/country

---

## Spotting SSL Stripping in Action

### How SSL Stripping Works
1. Victim requests `http://site.com`
2. Attacker intercepts before HTTPS redirect
3. Attacker makes HTTPS connection to real server
4. Attacker serves HTTP (unencrypted) to victim
5. Victim sees HTTP, thinks it's fine — attacker sees all data

```
Victim ←—HTTP—→ Attacker ←—HTTPS—→ Real Server
```

### SSL Stripping Indicators

**1. HTTP traffic to sites that should be HTTPS**
- Banking, login pages, email served over plain HTTP
- Check: is this domain expected to use HTTPS?

**2. HTTP 200 OK responses for login pages**
- Legitimate sites redirect HTTP → HTTPS (301/302)
- If login page loads over HTTP with 200 → suspicious

**3. Mixed content / missing HTTPS indicators**
- In pcap: credentials (passwords) visible in cleartext on port 80

**4. sslstrip tool artifacts**
- Rewrites `https://` links to `http://` in HTML
- Look for modified `<a href>` tags in HTTP responses

### Wireshark Filters
```
# HTTP traffic on port 80 to domains that should use HTTPS
http and tcp.port == 80

# POST requests over HTTP (credentials in cleartext)
http.request.method == "POST" and tcp.port == 80

# HTTP 200 on login-related URIs
http.response.code == 200 and http.request.uri contains "login"

# Look for credentials in HTTP POST body
http.authorization
```

### Prevention (context)
- **HSTS** (HTTP Strict Transport Security) — browser enforces HTTPS, defeats SSL stripping
- Sites without HSTS are vulnerable even with valid SSL certs

---

## MITM Detection Summary

| Attack | Key Indicator | Wireshark Filter |
|---|---|---|
| ARP Spoofing | Duplicate IP → different MACs | `arp.duplicate-address-detected` |
| ARP Spoofing | Gratuitous ARP flood | `arp.isgratuitous == 1` |
| DNS Spoofing | Unexpected IP in DNS response | `dns.flags.response == 1` |
| SSL Stripping | Login/HTTPS site served over HTTP | `http.request.method=="POST" and tcp.port==80` |

### General MITM Investigation Workflow
```
1. Check for ARP anomalies → arp.duplicate-address-detected
2. Check for gratuitous ARP flood → arp.isgratuitous == 1
3. Check DNS responses → unexpected IPs for known domains?
4. Check for HTTP POST on port 80 for sensitive sites → SSL stripping?
5. Follow TCP stream on suspicious sessions → credentials in cleartext?
6. Correlate MAC addresses → is one MAC appearing for multiple IPs?
```

---

## My Understanding in Plain English
MITM attacks are about positioning — the attacker gets between two hosts and becomes an invisible relay. ARP spoofing is the most common LAN technique: poison the ARP cache so traffic routes through the attacker. DNS spoofing redirects at the domain level. SSL stripping is the most dangerous because it silently downgrades encrypted traffic to cleartext. Detection relies on spotting the anomalies each technique leaves: duplicate MACs for same IP (ARP), unexpected IPs in DNS responses (DNS spoofing), and credentials flowing over port 80 to sites that should be HTTPS (SSL stripping). In a SOC context, ARP and DNS anomalies in logs + pcap are the primary evidence.

---

## Related Notes
- [[Network-Security-Essentials]]
- [[Network-Discovery-Detection]]
- [[Data-Exfiltration-Detection]]
- [[Wireshark-Traffic-Analysis]]
