# Network Traffic Basics

**Path:** SOC Level 1 - Network Traffic Analysis
**Date:** 2026-05-21
**Difficulty:** Easy

---

## What is the Purpose of Network Traffic Analysis?

- **Network Traffic Analysis (NTA)** → intercepting, recording, and analysing network packets to detect anomalies, threats, and suspicious behaviour
- Every action on a network generates traffic — NTA captures and analyses that traffic
- Core skill for SOC analysts — most attacks leave traces in network traffic

### Why NTA Matters
| Purpose | Description |
|---|---|
| **Threat Detection** | Spot malware C2 beacons, data exfiltration, lateral movement |
| **Incident Response** | Reconstruct exactly what happened during an attack |
| **Threat Hunting** | Proactively search for hidden threats in traffic |
| **Baseline Behaviour** | Understand normal → detect abnormal |
| **Compliance** | Log and monitor traffic for regulatory requirements |
| **Forensics** | Evidence collection for post-incident investigation |

---

## What Network Traffic Can We Observe?

### Two Types of Network Data

**1. Full Packet Capture (PCAP)**
- Captures the **entire packet** — headers + payload
- Most detailed — can reconstruct full sessions, read cleartext data
- High storage requirement
- Tools: **Wireshark**, **tcpdump**, **NetworkMiner**

**2. Network Flow Data (NetFlow/IPFIX)**
- Captures **metadata only** — no payload content
- Who talked to whom, when, how much data, which port/protocol
- Much lower storage requirement
- Cannot read actual content but still reveals patterns
- Tools: **ntopng**, **SiLK**, **Zeek**

### What Fields Are Visible

| Layer | Observable Data |
|---|---|
| **Layer 2 (Data Link)** | MAC addresses, ARP, VLAN tags |
| **Layer 3 (Network)** | IP addresses, TTL, fragmentation |
| **Layer 4 (Transport)** | Ports, TCP flags (SYN, ACK, RST, FIN), sequence numbers |
| **Layer 7 (Application)** | HTTP headers, DNS queries, SMTP commands, FTP commands |

### Protocols Commonly Analysed
- **DNS** → domain lookups — reveals what sites/IPs target is communicating with
- **HTTP/HTTPS** → web traffic — user agents, URLs, file downloads
- **SMB** → file sharing — lateral movement indicator
- **FTP** → file transfer — often unencrypted
- **SMTP/POP3/IMAP** → email — phishing delivery, data exfiltration
- **ICMP** → ping — used in tunneling, reconnaissance
- **RDP/SSH** → remote access — lateral movement, brute force

---

## Network Traffic Sources and Flows

### Where Traffic is Captured

**1. Network TAP (Test Access Point)**
- Physical device that passively copies all traffic on a link
- No impact on network performance
- Cannot be detected by attackers
- Best for: full visibility on critical network segments

**2. SPAN Port (Switch Port Analyzer) / Port Mirroring**
- Switch configured to mirror traffic from one port to another
- Analyst connects monitoring tool to the SPAN port
- Cheaper than a TAP — uses existing infrastructure
- May drop packets under high load

**3. Inline (IDS/IPS)**
- Traffic passes through the monitoring device
- Can actively block as well as monitor
- Adds latency
- Used in IPS deployments

**4. Network Sensors / Security Onion**
- Dedicated appliances or VMs that capture and analyse traffic
- Often deployed at network perimeter and key internal segments

### Network Flow Concepts

**Flow** → single communication session defined by:
- Source IP + Destination IP
- Source Port + Destination Port
- Protocol (TCP/UDP/ICMP)
- Start time + Duration

**NetFlow Fields**
```
Src IP | Dst IP | Src Port | Dst Port | Protocol | Packets | Bytes | Duration
```

**Why Flow Data is Useful Without Payload**
- Encrypted traffic (HTTPS, SFTP) → payload unreadable → flow data still reveals patterns
- C2 beaconing: same dst IP, same interval, regular small packets → suspicious even without reading the content
- Large outbound transfer: same dst IP, massive byte count → potential exfiltration

---

## How Can We Observe Network Traffic?

### Key Tools

**Wireshark**
- GUI-based full packet capture and analysis
- Industry standard for packet analysis
- Reads PCAP files
- Key features:
  - **Display filters** → filter packets in real time
  - **Follow stream** → reconstruct full TCP/HTTP session
  - **Protocol dissectors** → auto-decode HTTP, DNS, SMB etc.
  - **Statistics** → conversations, protocol hierarchy, endpoints

**Common Wireshark Display Filters**
```wireshark
# Filter by IP
ip.addr == 10.10.10.5
ip.src == 10.10.10.5
ip.dst == 192.168.1.1

# Filter by protocol
http
dns
smb
ftp
icmp
tcp
udp

# Filter by port
tcp.port == 80
tcp.port == 443
udp.port == 53

# Filter by TCP flag
tcp.flags.syn == 1          # SYN packets (connection attempts)
tcp.flags.reset == 1        # RST packets (connection resets)

# HTTP specific
http.request.method == "POST"
http.response.code == 200
http contains "password"

# DNS specific
dns.qry.name contains "evil"

# Combine filters
ip.src == 10.10.10.5 && tcp.port == 4444
ip.addr == 10.10.10.5 || ip.addr == 10.10.10.6
```

**tcpdump**
- Command-line packet capture tool
- Lightweight — runs on servers without GUI
- Saves captures as PCAP files for Wireshark analysis

```bash
# Capture all traffic on interface
tcpdump -i eth0

# Capture and save to file
tcpdump -i eth0 -w capture.pcap

# Capture with filter
tcpdump -i eth0 'port 80'
tcpdump -i eth0 'host 10.10.10.5'
tcpdump -i eth0 'port 53 and udp'

# Read a PCAP file
tcpdump -r capture.pcap

# Show full packet content
tcpdump -i eth0 -X -s 0 'port 80'

# Capture N packets
tcpdump -i eth0 -c 100
```

**NetworkMiner**
- Passive network sniffer and packet analyser
- Automatically extracts files, images, credentials from PCAP
- Reconstructs sessions and hosts from captured traffic
- Great for forensic analysis of PCAP files

**Zeek (formerly Bro)**
- Network analysis framework
- Generates detailed logs from traffic (conn.log, dns.log, http.log, files.log)
- Used in Security Onion deployments
- Logs are easily searchable in SIEM

**Tshark**
- Command-line version of Wireshark
- Useful for scripting and automated analysis

```bash
# Read PCAP and display
tshark -r capture.pcap

# Filter like Wireshark
tshark -r capture.pcap -Y "http.request"

# Extract specific fields
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e tcp.dstport
```

---

## SOC Analyst Workflow for NTA

```
1. Capture or receive PCAP / flow data

2. Get overview first:
   → Statistics → Protocol Hierarchy (what protocols are present?)
   → Statistics → Conversations (who is talking to whom?)
   → Statistics → Endpoints (what IPs are active?)

3. Filter for suspicious traffic:
   → Known malicious IPs/domains
   → Unusual ports (4444, 1337, 8888 common C2 ports)
   → Cleartext credentials (FTP, HTTP, Telnet)
   → Large outbound transfers

4. Follow suspicious streams:
   → Right-click → Follow → TCP Stream
   → Read the full conversation in plaintext

5. Extract artifacts:
   → File → Export Objects → HTTP (extract downloaded files)
   → Check file hashes on VirusTotal

6. Document findings:
   → Source/destination IPs
   → Timestamps
   → Protocol used
   → What data was transferred
```

---

## Key Suspicious Traffic Patterns

| Pattern | Possible Meaning |
|---|---|
| Regular beaconing (same IP, same interval) | C2 communication |
| Large outbound transfer | Data exfiltration |
| Many connections to same IP on unusual port | C2 or scanning |
| DNS queries to random-looking domains | DGA (malware) |
| High volume of failed connections | Port scanning/brute force |
| ICMP traffic with large payloads | ICMP tunneling |
| Cleartext passwords in HTTP/FTP | Credential theft risk |
| SMB traffic to external IPs | Lateral movement / data exfiltration |

---

## My Understanding in Plain English

Network traffic is the single most complete record of what happened on a network. Every connection, every file download, every DNS query, every C2 beacon leaves a trace in packet captures or flow logs. Wireshark is your microscope — it lets you zoom in on individual packets and read the raw bytes. Flow data is your radar — it doesn't show you the content but shows you the patterns at scale across thousands of connections. The skill of NTA is knowing what normal looks like so you can spot abnormal — a server that suddenly starts making outbound connections to a random IP every 60 seconds isn't normal, and that pattern is visible in flow data even if the traffic is fully encrypted.

---

## Related Notes

- [[Introduction-to-SIEM]]
- [[Protocols-and-Servers-Combined]]
- [[Passive-Reconnaissance]]
- [[Active-Reconnaissance]]
- [[Introduction-to-EDR]]
