# NetworkMiner
**Path:** SOC Level 1 - Network Traffic Analysis  
**Date:** 2026-05-31  
**Difficulty:** Easy

---

## What is NetworkMiner?

- Network forensics analysis tool (NFAT) — passive sniffer + pcap analyzer
- Focus: **artifact extraction** and **host-centric** view (vs Wireshark's packet-centric view)
- OS: Windows native (runs on Linux via Mono)
- Free version + commercial (Professional) version
- Used in: network forensics, incident response, SOC analysis

### NetworkMiner vs Wireshark

| Feature | NetworkMiner | Wireshark |
|---|---|---|
| Primary use | Artifact extraction, host profiling | Deep packet inspection |
| View | Host-centric | Packet-centric |
| Filtering | Limited | Powerful display filters |
| File extraction | Automatic | Manual (Export Objects) |
| Credential hunting | Automatic | Manual filtering |
| Best for | Quick triage, forensics | Detailed analysis |

> Use both together — NetworkMiner for fast triage, Wireshark for deep dive.

---

## NetworkMiner in Forensics

- Reconstructs sessions and extracts artifacts automatically from pcap
- Does NOT need live traffic — load a `.pcap` file directly
- Key strength: presents findings in organized tabs without manual filtering

### Loading a pcap
`File > Open` → select `.pcap` / `.pcapng` file → auto-parsed instantly

---

## Tool Overview — Tabs

### Hosts Tab
- Lists all detected hosts with IP, MAC, OS (fingerprinted), hostname
- OS detection via TCP/IP stack fingerprinting (TTL, window size, options)
- Expand a host to see open ports, sent/received data, associated sessions

### Frames Tab
- Raw packet view (like Wireshark's packet list)
- Less useful than Wireshark for deep inspection

### Parameters Tab
- Extracted HTTP parameters (GET/POST values, cookies, form fields)
- Great for finding credentials and session tokens quickly

### Credentials Tab ⭐
- Auto-extracted credentials from cleartext protocols
- Captures: HTTP Basic Auth, FTP, SMTP, POP3, IMAP, Telnet
- Shows: protocol, client IP, server IP, username, password

### Sessions Tab
- All reconstructed TCP/UDP sessions
- Shows src/dst IP, ports, duration, bytes exchanged

### DNS Tab
- All DNS queries and responses in one view
- Useful for spotting C2 domains, tunneling, suspicious lookups

### Files Tab ⭐
- **Automatically extracts all transferred files** from pcap
- HTTP, FTP, SMB file transfers all parsed
- Right-click → Open folder to access extracted file
- Saves to: `NetworkMiner_<version>/AssembledFiles/`

### Images Tab
- Subset of Files tab — only images
- Visual preview of images transferred over HTTP/FTP

### Messages Tab
- Reconstructed emails (SMTP, POP3, IMAP)
- Shows sender, recipient, subject, body

### Anomalies Tab
- Flags unusual or suspicious traffic automatically
- Check this first for quick wins

---

## Version Differences (Free vs Professional)

| Feature | Free | Professional |
|---|---|---|
| pcap loading | ✅ | ✅ |
| File extraction | ✅ | ✅ |
| Credential extraction | ✅ | ✅ |
| OS fingerprinting | ✅ | ✅ |
| Geo-IP mapping | ❌ | ✅ |
| Advanced filtering | ❌ | ✅ |
| PCAP-over-IP | ❌ | ✅ |
| Export to CSV | ❌ | ✅ |
| Script/CLI support | ❌ | ✅ |

> Free version is enough for THM and most SOC triage tasks.

---

## Practical Workflow with NetworkMiner

```
1. Load pcap → File > Open
2. Check Hosts tab → identify all IPs, OS, hostnames
3. Check Credentials tab → any cleartext creds?
4. Check Files tab → any files transferred? Malware?
5. Check DNS tab → suspicious domains?
6. Check Anomalies tab → anything flagged?
7. Check Parameters tab → HTTP form data, cookies?
8. For anything suspicious → switch to Wireshark for deeper analysis
```

---

## Key Points for SOC Use

- NetworkMiner is **passive** — cannot inject or modify traffic
- Great for **first-pass triage** of a pcap before going into Wireshark
- Credentials tab saves huge time vs manual FTP/HTTP filtering in Wireshark
- Files tab can recover malware samples, documents, images from captures
- OS fingerprinting helps with **asset inventory** and **anomaly detection** (unknown OS on network)

---

## My Understanding in Plain English
NetworkMiner is basically a forensic autopsy tool for pcap files. Where Wireshark shows you every raw packet, NetworkMiner automatically organizes everything into useful categories — hosts, credentials, files, DNS, sessions. The Credentials and Files tabs are the biggest time-savers: instead of writing filters to hunt creds or manually exporting objects, NetworkMiner does it automatically on load. The workflow is: use NetworkMiner for fast triage to get an overview and grab artifacts, then drop into Wireshark when you need to dig into specific packets or apply complex filters.

---

## Related Notes
- [[Wireshark-The-Basics]]
- [[Wireshark-Packet-Operations]]
- [[Wireshark-Traffic-Analysis]]
- [[Network-Traffic-Basics]]
