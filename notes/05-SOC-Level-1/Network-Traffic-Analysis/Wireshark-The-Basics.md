# Wireshark: The Basics

**Path:** SOC Level 1 - Network Traffic Analysis
**Date:** 2026-05-21
**Difficulty:** Easy

---

## Tool Overview

- **Wireshark** → open-source GUI packet analyser — industry standard for network forensics
- Captures live traffic OR analyses saved PCAP files
- Cross-platform: Windows, Linux, macOS
- Uses **libpcap/WinPcap** for packet capture at OS level

### Wireshark Interface Layout

```
┌──────────────────────────────────────────────────────┐
│  Menu Bar (File, Edit, View, Go, Capture, Analyse)   │
├──────────────────────────────────────────────────────┤
│  Main Toolbar (Start/Stop capture, open file)        │
├──────────────────────────────────────────────────────┤
│  Display Filter Bar  <- write filters here           │
├──────────────────────────────────────────────────────┤
│  Packet List Pane   <- all captured packets          │
│  (No. | Time | Source | Destination | Protocol | Info)│
├──────────────────────────────────────────────────────┤
│  Packet Details Pane <- expanded layers of selected  │
│  (Frame → Ethernet → IP → TCP → Application)        │
├──────────────────────────────────────────────────────┤
│  Packet Bytes Pane  <- raw hex + ASCII of packet     │
└──────────────────────────────────────────────────────┘
```

### Colour Coding (Default)
| Colour | Meaning |
|---|---|
| Light blue | UDP traffic |
| Light purple | TCP traffic |
| Green | HTTP traffic |
| Black | TCP errors, bad packets |
| Yellow | ARP, OSPF (routing) |
| Dark blue | DNS |
| Light grey | TCP SYN/FIN |

---

## Packet Dissection

- **Packet Dissection** → Wireshark breaking down a packet layer by layer
- Click any packet in the list → details pane shows all layers

### Layer Breakdown
```
Frame (Physical metadata)
  └── Ethernet II (MAC addresses)
        └── Internet Protocol (IP addresses, TTL)
              └── TCP/UDP (Ports, flags)
                    └── Application Layer (HTTP/DNS/FTP data)
```

### What Each Layer Shows

**Ethernet II**
- Source MAC + Destination MAC
- EtherType (IPv4, IPv6, ARP)

**Internet Protocol (IP)**
- Source IP + Destination IP
- TTL → hops remaining (useful for OS fingerprinting)
- Protocol number (TCP=6, UDP=17, ICMP=1)

**TCP/UDP**
- Source port + Destination port
- TCP flags: SYN, ACK, FIN, RST, PSH, URG
- Sequence/Acknowledgement numbers

**Application Layer**
- Actual data: HTTP headers, DNS query name, FTP command, credentials

---

## Packet Navigation

### Key Shortcuts
| Shortcut | Action |
|---|---|
| `Ctrl+F` | Find packet (string, hex, regex) |
| `Ctrl+G` | Go to specific packet number |
| `Ctrl+E` | Mark/unmark packet |
| `Ctrl+Alt+N` | Next packet matching filter |
| `Ctrl+Alt+B` | Previous packet matching filter |

### Find Packets (Ctrl+F)
- Search by: Display filter, Hex value, String, Regex
- Search in: Packet list, Packet details, Packet bytes
- **String search** → find passwords, flags, usernames in raw content

### Export Features
- **File → Export Objects → HTTP** → extract all files from HTTP traffic
- **File → Export Objects → FTP-DATA** → extract FTP transferred files
- **File → Export Objects → SMB** → extract SMB transferred files
- **File → Export Specified Packets** → save filtered subset as new PCAP

---

## Packet Filtering

### Two Types of Filters

**Capture Filters** → applied BEFORE/DURING live capture (BPF syntax)
```bash
host 10.10.10.5       # traffic to/from this IP
port 80               # HTTP only
tcp                   # TCP only
not arp               # exclude ARP
net 192.168.1.0/24    # entire subnet
```

**Display Filters** → applied AFTER capture (Wireshark syntax)
- Hides non-matching packets but doesn't delete them
- More flexible — use these most of the time

### Display Filter Operators
```
==    equals
!=    not equals
>     greater than
<     less than
contains   string contains value
&&    AND
||    OR
!     NOT
```

---

## Display Filter Cheatsheet

### IP Filters
```wireshark
ip.addr == 10.10.10.5          # to OR from IP
ip.src == 10.10.10.5           # FROM IP only
ip.dst == 10.10.10.5           # TO IP only
ip.src == 10.10.10.0/24        # entire subnet as source
!(ip.addr == 10.10.10.5)       # exclude IP
```

### Protocol Filters
```wireshark
tcp        udp        icmp
arp        http       dns
ftp        smb        ssh       telnet
```

### Port Filters
```wireshark
tcp.port == 80           # HTTP
tcp.port == 443          # HTTPS
tcp.port == 22           # SSH
tcp.port == 21           # FTP
udp.port == 53           # DNS
tcp.dstport == 4444      # common C2 port
```

### TCP Flag Filters
```wireshark
tcp.flags.syn == 1                           # SYN packets
tcp.flags.syn == 1 && tcp.flags.ack == 0     # SYN scan detection
tcp.flags.reset == 1                         # RST (connection reset)
tcp.flags.fin == 1                           # FIN (close)
```

### HTTP Filters
```wireshark
http.request                           # all requests
http.response                          # all responses
http.request.method == "POST"          # POST only (often credentials)
http.request.method == "GET"           # GET only
http.response.code == 200              # success
http.response.code == 404              # not found
http contains "password"               # contains keyword
http.request.uri contains "login"      # login page requests
```

### DNS Filters
```wireshark
dns                                    # all DNS
dns.qry.name == "evil.com"            # specific domain
dns.qry.name contains "evil"          # domains with keyword
dns.flags.response == 0               # queries only
dns.flags.response == 1               # responses only
```

### FTP Filters
```wireshark
ftp                                    # FTP control
ftp-data                               # FTP file transfers
ftp.request.command == "USER"          # FTP username
ftp.request.command == "PASS"          # FTP password (cleartext!)
```

---

## Follow Stream — Most Important Feature

- Right-click any packet → **Follow → TCP Stream** (or UDP/HTTP)
- Reconstructs the full conversation between two hosts
- Client data = **red**, Server data = **blue**
- Reads like a chat log — see full HTTP requests, FTP sessions, credentials
- Most useful for reading cleartext protocols and understanding what happened

---

## Statistics Menu — Key Tools

| Feature | Location | Use |
|---|---|---|
| Protocol Hierarchy | Statistics → Protocol Hierarchy | What protocols are in the capture? |
| Conversations | Statistics → Conversations | Who talked to whom, how much data? |
| Endpoints | Statistics → Endpoints | All active IPs/MACs |
| IO Graph | Statistics → IO Graph | Traffic volume over time |
| HTTP Requests | Statistics → HTTP → Requests | All HTTP requests |

---

## Investigation Workflow

```
1. Open PCAP → File → Open

2. Get overview:
   Statistics → Protocol Hierarchy
   Statistics → Conversations
   Statistics → Endpoints

3. Apply filters to focus:
   → Protocol filter first (http, dns, ftp)
   → Then IP filter to narrow down

4. Drill into suspicious packets:
   → Click packet → read details pane
   → Follow TCP Stream to read full conversation

5. Search for specific content:
   → Ctrl+F → String → "password", "flag", "THM"

6. Extract files:
   → File → Export Objects → HTTP

7. Document findings:
   → Packet numbers, timestamps, IPs, data transferred
```

---

## Common Investigation Scenarios

### Find Cleartext Credentials
```wireshark
# FTP credentials
ftp.request.command == "USER" || ftp.request.command == "PASS"

# HTTP POST login
http.request.method == "POST"
# → Follow TCP Stream to read form data

# Telnet (everything cleartext)
telnet
# → Follow TCP Stream
```

### Detect Port Scan
```wireshark
# Many SYN packets with no complete handshake
tcp.flags.syn == 1 && tcp.flags.ack == 0
# One source IP → many destination ports = scan
```

### Find C2 Beaconing
```wireshark
# Unusual outbound ports
tcp.dstport != 80 && tcp.dstport != 443 && tcp.dstport != 53
# Look for: same dst IP, regular intervals, small consistent packet sizes
```

### Find DNS Tunneling
```wireshark
dns
# Look for: long subdomain names, high query volume, unusual record types (TXT)
```

---

## My Understanding in Plain English

Wireshark is where network forensics actually happens. Every protocol, every conversation, every byte transferred is in the PCAP. The display filter is your most powerful tool — without it you're drowning in noise, with it you isolate exactly what you need in seconds. Follow Stream is the feature that takes you from metadata to actual content — one click reads the full HTTP conversation, sees FTP credentials in cleartext, or shows the C2 beacon check-in. The Statistics menu gives the high-level view before drilling down — Protocol Hierarchy tells you what's in the file, Conversations tells you who's talking, and from there you know exactly where to focus.

---

## Related Notes

- [[Network-Traffic-Basics]]
- [[Protocols-and-Servers-Combined]]
- [[Introduction-to-SIEM]]
- [[Passive-Reconnaissance]]
