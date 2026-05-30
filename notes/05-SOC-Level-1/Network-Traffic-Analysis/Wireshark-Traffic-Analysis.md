# Wireshark: Traffic Analysis
**Path:** SOC Level 1 - Network Traffic Analysis  
**Date:** 2026-05-30  
**Difficulty:** Medium

---

## Nmap Scan Detection

Nmap scans leave distinct fingerprints in pcap — recognizing them is a core SOC skill.

| Scan Type | What to Look For |
|---|---|
| TCP SYN (Half-open) | SYN → RST/ACK (no full handshake) |
| TCP Connect | Full 3-way handshake → immediate RST |
| UDP Scan | ICMP "Port Unreachable" responses |
| NULL Scan | Packet with no flags set |
| FIN Scan | Only FIN flag set |
| XMAS Scan | FIN + PSH + URG flags set |
| OS Detection | TTL values + TCP window size patterns |

### Wireshark Filters for Nmap
```
# SYN scan detection
tcp.flags.syn==1 and tcp.flags.ack==0

# NULL scan (no flags)
tcp.flags==0x000

# XMAS scan
tcp.flags.fin==1 and tcp.flags.push==1 and tcp.flags.urg==1

# RST flood (sign of port scan)
tcp.flags.reset==1
```

---

## ARP Poisoning & Man-in-the-Middle (MITM)

### How ARP Poisoning Works
1. Attacker sends fake ARP replies → maps attacker MAC to victim/gateway IP
2. Both victim and gateway update ARP cache with attacker's MAC
3. All traffic flows through attacker → MITM achieved

### Detection in Wireshark
- Multiple ARP replies for the same IP from different MACs
- Gratuitous ARP (unsolicited ARP reply)
- One MAC appearing for multiple IPs

```
# Show all ARP traffic
arp

# Gratuitous ARP (potential poisoning)
arp.isgratuitous == 1

# Detect duplicate IPs (same IP, different MACs)
arp.duplicate-address-detected
```

### MITM Signs in Traffic
- Victim traffic going to unexpected MAC
- SSL stripping attempts (HTTPS → HTTP downgrade)
- Credentials appearing in cleartext unexpectedly

---

## Identifying Hosts: DHCP, NetBIOS, Kerberos

### DHCP
- DHCP DISCOVER → OFFER → REQUEST → ACK (DORA)
- Hostname revealed in DHCP REQUEST packets
- Filter: `dhcp` or `bootp`
- `dhcp.option.hostname` → shows device hostname

### NetBIOS (NBNS)
- Windows name resolution (legacy, pre-DNS)
- Reveals hostnames on local network
- Filter: `nbns`
- Look at `nbns.name` field for hostnames

### Kerberos
- Windows domain authentication protocol
- Reveals usernames, domain names, service names
- Filter: `kerberos`
- Key fields: `kerberos.CNameString` (username), `kerberos.realm` (domain)
- AS-REQ packets contain the username requesting a ticket

---

## Tunneling Traffic: DNS and ICMP

### DNS Tunneling
- Data exfiltrated by encoding it in DNS query subdomains
- e.g. `c2VjcmV0ZGF0YQ==.evil.com` — base64 in subdomain

**Detection indicators:**
- Unusually long domain names (`dns.qry.name.len > 40`)
- High volume of DNS queries to single domain
- TXT record queries (used for C2)
- Non-standard query types

```
dns.qry.name.len > 40
dns.qry.type == 16        # TXT records
```

### ICMP Tunneling
- Data hidden in ICMP echo (ping) payload
- Normal ping payload = small, fixed pattern
- Tunneled ping = large or unusual payload

```
icmp
icmp.data_len > 64        # suspiciously large ICMP payload
```

---

## Cleartext Protocol Analysis: FTP

FTP sends everything in plaintext — credentials and files fully visible.

### FTP Flow
```
USER username  →  331 Password required
PASS password  →  230 Login successful
LIST           →  directory listing
RETR file.txt  →  file download
STOR file.txt  →  file upload
```

### Wireshark Filters
```
ftp                          # all FTP control traffic
ftp.request.command == "USER"
ftp.request.command == "PASS"
ftp-data                     # actual file transfer data
```

- Use **Follow TCP Stream** to see full session including credentials
- Use **File > Export Objects > FTP-DATA** to extract transferred files

---

## Cleartext Protocol Analysis: HTTP

### What's Visible in HTTP
- Full URLs, headers, cookies, form data (POST body)
- Credentials in Basic Auth (`Authorization: Basic base64`)
- Session tokens in cookies

### Key Filters
```
http.request.method == "POST"         # form submissions, logins
http.authorization                    # Basic Auth header
http.cookie                           # session cookies
http.request.uri contains "login"     # login endpoints
http.response.code == 401             # auth failures
http.response.code == 302             # redirects (post-login)
```

- **Follow HTTP Stream** → readable request + response
- **Export Objects > HTTP** → download files/images transferred

---

## Encrypted Protocol Analysis: Decrypting HTTPS

HTTPS is TLS-encrypted — but can be decrypted if you have the key.

### Decryption Methods in Wireshark
1. **Pre-Master Secret Log** (most common)
   - Set env variable: `SSLKEYLOGFILE=~/ssl-keys.log` before launching browser
   - In Wireshark: `Edit > Preferences > Protocols > TLS > (Pre)-Master-Secret log`
   - Point to the key log file → traffic decrypts automatically

2. **RSA Private Key** (only works if no PFS)
   - `Edit > Preferences > Protocols > TLS > RSA Keys`
   - Add server IP, port, protocol, and private key file

### After Decryption
- Filter: `http` will now show decrypted HTTPS as plain HTTP
- Credentials, tokens, API keys all visible

---

## Bonus: Hunt Cleartext Credentials

One-stop filters to find credentials across protocols:

```
# HTTP Basic Auth
http.authorization

# FTP credentials
ftp.request.command == "USER" or ftp.request.command == "PASS"

# Telnet (fully cleartext)
telnet

# SMTP auth
smtp.req.command == "AUTH"

# POP3 credentials
pop.request.command == "USER" or pop.request.command == "PASS"

# IMAP credentials  
imap contains "LOGIN"
```

**Workflow:** Filter → Follow Stream → Extract credentials

---

## Bonus: Actionable Results

### General Investigation Workflow
1. `Statistics > Protocol Hierarchy` — what protocols are present?
2. `Statistics > Conversations` — who is talking to whom?
3. `Statistics > Endpoints` — which hosts are most active?
4. Apply targeted display filters based on findings
5. Follow streams on suspicious conversations
6. Export objects if file transfers detected

### Threat Hunting Checklist in Wireshark
- [ ] ARP anomalies → MITM?
- [ ] DNS tunneling → exfiltration?
- [ ] ICMP large payloads → tunneling?
- [ ] Nmap-style SYN floods → reconnaissance?
- [ ] Cleartext credentials (FTP/HTTP/Telnet)?
- [ ] Kerberos AS-REQ → username enumeration?
- [ ] DHCP hostnames → asset inventory

---

## My Understanding in Plain English
This room is about using Wireshark for actual threat detection, not just packet reading. Each protocol tells a different story — ARP poisoning means someone is intercepting traffic, DNS/ICMP tunneling means data is being smuggled out, cleartext protocols expose credentials directly. The real skill is knowing which filter to apply and when, then following the stream to confirm. HTTPS isn't safe either if you have the key log file. The bonus tasks tie it all together into a practical SOC analyst workflow.

---

## Related Notes
- [[Wireshark-The-Basics]]
- [[Wireshark-Packet-Operations]]
- [[Network-Traffic-Basics]]
