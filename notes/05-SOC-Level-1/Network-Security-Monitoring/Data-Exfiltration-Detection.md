# Data Exfiltration Detection
**Path:** SOC Level 1 - Network Security Monitoring  
**Date:** 2026-06-02  
**Difficulty:** Easy

---

## Data Exfil: Overview, Techniques, and Indicators

Data exfiltration = unauthorized transfer of data from a target to an attacker-controlled destination.

### Exfil Stages (in Kill Chain context)
1. **Collection** — attacker gathers target data (files, creds, DB dumps)
2. **Staging** — compresses/encrypts data to reduce size and avoid detection
3. **Exfiltration** — sends data out via a covert channel

### Common Exfil Techniques
| Technique | Channel Used | Stealth Level |
|---|---|---|
| DNS Tunneling | DNS queries | High |
| ICMP Tunneling | Ping packets | High |
| HTTP/S Exfil | Web traffic | Medium-High |
| FTP Upload | FTP | Low |
| Email (SMTP) | Mail protocol | Medium |
| Cloud storage | HTTPS to Google Drive, Dropbox etc. | High |

### General Indicators of Exfiltration
- Unusually large outbound data transfers
- Connections to new/unknown external IPs or domains
- Traffic on non-standard ports
- Outbound traffic at unusual hours
- Repeated connections to same external host
- Encoded or compressed data in protocol payloads

---

## Detection: Data Exfil through DNS Tunneling

### How It Works
- Data encoded (Base64/hex) and embedded in DNS query subdomains
- e.g. `aGVsbG8=.c2VjcmV0.attacker.com` → each query carries a chunk of data
- Responses carry data back (TXT records used for C2)
- Bypasses firewalls because DNS (port 53) is almost always allowed outbound

### Detection Indicators
- DNS queries with **unusually long subdomains** (normal: <30 chars)
- **High volume** of DNS queries to a single domain
- DNS **TXT record** queries (rare in normal traffic)
- Queries with **random-looking / Base64-looking** subdomains
- DNS responses with **large TXT payloads**

### Wireshark Filters
```
# Long subdomain queries
dns.qry.name.len > 40

# TXT record queries (used for C2/exfil)
dns.qry.type == 16

# All DNS traffic to inspect
dns

# Filter specific suspicious domain
dns.qry.name contains "attacker"
```

### Log-Based Detection
- High DNS query rate from single internal host
- Queries to newly registered domains (low reputation)
- DNS to external resolvers instead of internal DNS server

---

## Detection: Data Exfil through FTP

### How It Works
- Attacker uploads files using FTP STOR command to external FTP server
- FTP is cleartext — credentials and data fully visible
- Often uses port 21 (control) and passive ports for data transfer

### Detection Indicators
- Outbound FTP connections to external/unknown IPs
- FTP STOR commands (upload) — especially large files
- FTP connections outside business hours
- Credentials in FTP control stream

### Wireshark Filters
```
# All FTP control traffic
ftp

# Upload command
ftp.request.command == "STOR"

# Login attempts
ftp.request.command == "USER"
ftp.request.command == "PASS"

# FTP data channel (actual file content)
ftp-data
```

### Key Actions
- Follow TCP Stream on FTP session → see full session including filename and credentials
- `File > Export Objects > FTP-DATA` → recover the exfiltrated file

---

## Detection: Data Exfil via HTTP

### How It Works
- Data sent via HTTP **POST** requests to attacker-controlled web server
- Can blend with normal web traffic
- Data often Base64-encoded in POST body or custom headers
- HTTPS makes payload invisible without decryption

### Detection Indicators
- Large HTTP POST to unknown/external IP or domain
- POST requests to IPs rather than domain names (no legitimate hostname)
- Unusual User-Agent strings
- HTTP POSTs to non-standard ports (e.g. 8080, 4444)
- Repeated POSTs to same URI with large body sizes
- Base64-looking data in POST body or headers

### Wireshark Filters
```
# All POST requests
http.request.method == "POST"

# Large HTTP transfers
http.content_length > 5000

# Requests to IP directly (no domain)
http.host matches "^\d+\.\d+\.\d+\.\d+$"

# Specific URI patterns
http.request.uri contains "upload"
http.request.uri contains "submit"

# Unusual ports with HTTP
tcp.port == 8080 or tcp.port == 4444
```

### HTTPS Exfil
- Payload hidden by TLS encryption
- Detection relies on: connection metadata, destination reputation, volume, frequency
- If SSL key log available → decrypt and inspect as HTTP

---

## Detection: Data Exfiltration via ICMP

### How It Works
- Data hidden in ICMP Echo Request (ping) **payload field**
- Normal ping payload = small fixed pattern (e.g. `abcdefghijklmnop...`)
- Tunneled ping = large, random, or encoded payload
- Tools: `ptunnel`, `icmptunnel`, custom scripts

### Detection Indicators
- ICMP packets with **unusually large payloads** (normal ping = 32–64 bytes)
- ICMP payload contains **non-standard / random data** instead of fixed pattern
- High **volume** of ICMP packets to single external IP
- ICMP traffic to **external** hosts (normal ping rarely leaves LAN)
- Consistent ICMP traffic at regular intervals (beaconing)

### Wireshark Filters
```
# All ICMP traffic
icmp

# Large ICMP payloads (normal is 32 bytes on Windows, 48 on Linux)
icmp and data.len > 64

# ICMP echo requests only
icmp.type == 8

# ICMP to external IPs (adjust subnet as needed)
icmp and !ip.dst == 192.168.0.0/16
```

### Inspecting ICMP Payload
- Click ICMP packet in Wireshark → expand **Data** field in packet details
- Normal: repeating ASCII pattern (`abcdefghijklmnopqrstuvwabcdefghi`)
- Suspicious: random bytes, Base64 string, or structured data

---

## Comparison: Exfil Channels at a Glance

| Channel | Port | Encrypted | Detection Difficulty | Key Filter |
|---|---|---|---|---|
| DNS | 53 UDP | No | High | `dns.qry.name.len > 40` |
| ICMP | N/A | No | High | `data.len > 64` |
| HTTP | 80 | No | Medium | `http.request.method=="POST"` |
| HTTPS | 443 | Yes | Hard | Metadata + reputation |
| FTP | 21 | No | Low | `ftp.request.command=="STOR"` |

---

## General Detection Workflow

```
1. Baseline normal traffic → know what's typical for your network
2. Alert on anomalies:
   - Unusual outbound volume (NetFlow / firewall logs)
   - Connections to new external IPs/domains
   - Non-standard protocol usage on standard ports
3. Drill into pcap with Wireshark:
   - Apply protocol filter
   - Follow Stream
   - Check payload content
4. Cross-reference with threat intel:
   - Is the destination IP/domain known malicious?
   - Is the domain newly registered?
5. Escalate / contain if confirmed
```

---

## My Understanding in Plain English
Data exfil detection is about catching attackers smuggling data out through channels that look legitimate. DNS and ICMP are the sneakiest — both are normally allowed through firewalls and look harmless, but data hides in query names and ping payloads. FTP is the easiest to catch because it's cleartext. HTTP POST exfil blends with web traffic but stands out when it's going to raw IPs or unusual ports. The core detection skill is knowing what "normal" looks like for each protocol, so anomalies — long DNS names, big ICMP payloads, large POSTs to unknown hosts — immediately stand out.

---

## Related Notes
- [[Network-Security-Essentials]]
- [[Network-Discovery-Detection]]
- [[Wireshark-Traffic-Analysis]]
- [[Network-Traffic-Basics]]
