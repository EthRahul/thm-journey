# Wireshark: Packet Operations
**Path:** SOC Level 1 - Network Traffic Analysis  
**Date:** 2026-05-30  
**Difficulty:** Medium

---

## Statistics Menu

### Capture File Properties (`Statistics > Capture File Properties`)
- File hash, capture duration, interface info
- Packet count, avg packet size, avg speed
- Quick overview before deep dive

### Protocol Hierarchy (`Statistics > Protocol Hierarchy`)
- Shows all protocols as a tree with % distribution
- Spot unexpected protocols (e.g. DNS at 80% → suspicious)

### Conversations (`Statistics > Conversations`)
- Lists all unique communication pairs (IP↔IP, port↔port)
- Tabs: Ethernet, IPv4, IPv6, TCP, UDP
- Sort by bytes to find heaviest talkers

### Endpoints (`Statistics > Endpoints`)
- Per-host view (vs per-pair in Conversations)
- GeoIP mapping possible if MaxMind DB loaded

### Protocol-Specific Stats
- `Statistics > HTTP` → request/response breakdown
- `Statistics > DNS` → query types, response codes

---

## Packet Filtering — Principles

- **Display filters** (what you see) vs **Capture filters** (what is captured) — different syntax
- Display filters use `==`, `!=`, `contains`, `matches` (regex)
- Combine with `and`, `or`, `not`

---

## Protocol Filters

| Filter | Purpose |
|---|---|
| `http` | All HTTP traffic |
| `dns` | All DNS traffic |
| `tcp` | All TCP |
| `udp` | All UDP |
| `icmp` | Ping traffic |
| `ftp` | FTP traffic |
| `ssh` | SSH traffic |

---

## IP / Port Filters

| Filter | Purpose |
|---|---|
| `ip.addr == 10.0.0.1` | Traffic to OR from IP |
| `ip.src == 10.0.0.1` | Traffic FROM IP |
| `ip.dst == 10.0.0.1` | Traffic TO IP |
| `tcp.port == 80` | TCP port 80 (either direction) |
| `tcp.dstport == 443` | TCP destination port 443 |
| `udp.port == 53` | DNS (UDP) |

---

## HTTP Filters

| Filter | Purpose |
|---|---|
| `http.request` | HTTP requests only |
| `http.response` | HTTP responses only |
| `http.request.method == "POST"` | POST requests |
| `http.response.code == 200` | 200 OK responses |
| `http.host contains "evil"` | Host header match |

---

## DNS Filters

| Filter | Purpose |
|---|---|
| `dns.qry.name == "example.com"` | Specific domain query |
| `dns.flags.response == 0` | DNS queries only |
| `dns.flags.response == 1` | DNS responses only |

---

## Advanced Filtering

### `contains` — substring match
```
http.host contains "google"
dns.qry.name contains "evil"
```

### `matches` — regex match
```
http.request.uri matches "\.(php|asp|aspx)$"
```

### `in` — set match
```
tcp.port in {80 443 8080}
```

### Other Useful Filters
```
frame.len > 1000                    # packets larger than 1000 bytes
tcp.analysis.retransmission         # retransmitted packets
http.authorization                  # HTTP Basic Auth credentials
tcp.flags.syn==1 and tcp.flags.ack==0   # SYN only (port scan detection)
dns.qry.name.len > 50               # long DNS queries (exfil indicator)
```

---

## Follow Stream
- Right-click any packet → **Follow → TCP / UDP / HTTP Stream**
- Reconstructs full conversation in readable form
- Use for: credentials, commands, file transfers

---

## Export Objects
- `File > Export Objects > HTTP` → extracts files sent over HTTP
- Also works for SMB, DICOM, IMF

---

## My Understanding in Plain English
Packet Operations is about efficiently finding what matters in a large pcap. The Statistics menu gives a high-level picture fast — Protocol Hierarchy and Conversations are the first things to check. Display filters are the core skill: narrow millions of packets to exactly what you need. Advanced filters like `contains`, `matches`, and `in` handle complex searches. Follow Stream and Export Objects are the most practical features for extracting real data — credentials, transferred files, commands.

---

## Related Notes
- [[Wireshark-The-Basics]]
- [[Network-Traffic-Basics]]
