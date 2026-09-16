# Network Security Essentials
**Path:** SOC Level 1 - Network Security Monitoring  
**Date:** 2026-05-31  
**Difficulty:** Easy

---

## A Network — Overview

### Key Network Components
- **Router** — connects networks, routes traffic between them
- **Switch** — connects devices within a LAN, uses MAC addresses
- **Firewall** — filters traffic based on rules (perimeter defense)
- **IDS/IPS** — detects/prevents malicious traffic
- **DMZ** — demilitarized zone; public-facing servers live here, isolated from internal network

### Traffic Types
| Type | Description |
|---|---|
| North-South | Traffic entering/leaving the network (internet ↔ internal) |
| East-West | Traffic moving laterally inside the network (host ↔ host) |

> East-West is harder to monitor but critical — attackers move laterally after initial compromise.

### OSI Relevance for Security
| Layer | Examples | Security Concern |
|---|---|---|
| L3 Network | IP routing | IP spoofing, routing attacks |
| L4 Transport | TCP/UDP | Port scans, SYN floods |
| L7 Application | HTTP, DNS, FTP | Malware C2, data exfil, web attacks |

---

## Network Visibility

Visibility = knowing what is on your network and what it is doing.

### Types of Visibility
- **Full packet capture** — captures everything (pcap); storage-heavy
- **NetFlow / sFlow** — metadata only (src/dst IP, port, bytes, duration); lightweight
- **Logs** — firewall logs, IDS alerts, DNS logs, proxy logs

### Visibility Tools
| Tool | Purpose |
|---|---|
| Wireshark / NetworkMiner | Full packet analysis |
| Zeek (Bro) | Network metadata + scripted detection |
| Snort / Suricata | IDS/IPS — signature-based detection |
| ntopng | Real-time flow monitoring |
| Security Onion | All-in-one NSM platform |

### Blind Spots
- Encrypted traffic (HTTPS, TLS) — need decryption or metadata analysis
- East-West traffic — need internal sensors, not just perimeter
- Encrypted tunnels (DNS/ICMP tunneling) — need behavioral detection

---

## Network Perimeter

The perimeter = the boundary between your trusted internal network and untrusted external networks.

### Perimeter Devices
| Device | Role |
|---|---|
| Firewall | Allow/deny traffic by IP, port, protocol |
| IDS | Detect and alert on malicious traffic (passive) |
| IPS | Detect and block malicious traffic (inline, active) |
| Proxy | Intercepts outbound HTTP/S, can inspect/filter |
| WAF | Web Application Firewall — protects web apps at L7 |
| VPN Gateway | Encrypted tunnel for remote access |

### Firewall Rule Logic
```
Action  | Source IP   | Dest IP     | Port  | Protocol
ALLOW   | 0.0.0.0/0   | WebServer   | 443   | TCP
DENY    | 0.0.0.0/0   | InternalDB  | 3306  | TCP
```
- Rules processed **top-down** — first match wins
- Default deny at the bottom = best practice

### DMZ Architecture
```
Internet → Firewall → DMZ (Web/Mail servers) → Firewall → Internal LAN
```
- Public servers in DMZ, not directly on internal network
- Even if DMZ server is compromised, attacker still faces internal firewall

---

## Network Perimeters: Monitoring and Protecting

### What to Monitor at the Perimeter
- **Inbound:** port scans, exploit attempts, brute force, unusual geo-IPs
- **Outbound:** large data transfers, beaconing (C2), DNS tunneling, unusual ports
- **Lateral:** unexpected internal host communication, new services, privilege escalation indicators

### IDS vs IPS
| | IDS | IPS |
|---|---|---|
| Position | Out-of-band (mirror/tap) | Inline (traffic passes through) |
| Action | Alert only | Alert + Block |
| Risk | None to traffic | Can block legitimate traffic (false positives) |
| Examples | Snort (IDS mode), Zeek | Snort (IPS mode), Suricata |

### Detection Methods
| Method | How it works | Good for |
|---|---|---|
| Signature-based | Matches known attack patterns | Known malware, CVEs |
| Anomaly-based | Flags deviations from baseline | Zero-days, insider threats |
| Behaviour-based | Tracks patterns over time | APT, slow attacks |

---

## Perimeter Logs: Investigating the Breach

### Key Log Sources
| Source | What it tells you |
|---|---|
| Firewall logs | What was allowed/denied, src/dst IP, ports |
| IDS/IPS alerts | Triggered signatures, severity, affected hosts |
| DNS logs | Domains queried — C2 detection, exfiltration |
| Proxy logs | Outbound HTTP/S URLs, user-agents, response codes |
| NetFlow | Traffic volumes, connection patterns, durations |

### Firewall Log Fields (typical)
```
timestamp | action | src_ip | src_port | dst_ip | dst_port | protocol | bytes
```

### Investigation Workflow
```
1. Identify the alert / anomaly (IDS alert, SIEM trigger, user report)
2. Check firewall logs → was traffic allowed or denied?
3. Check DNS logs → any suspicious domains queried?
4. Check proxy logs → any outbound connections to bad IPs/domains?
5. Correlate timestamps across log sources
6. Escalate or contain based on findings
```

### Common Attack Patterns in Logs

**Port Scan:**
- Many connections from same src IP to many dst ports
- Short duration, no data transferred, lots of RST/ICMP unreachable

**Brute Force:**
- Many failed auth attempts (firewall blocks or IDS alerts)
- Same src IP → same dst IP:port, high frequency

**C2 Beaconing:**
- Regular outbound connections at fixed intervals (e.g. every 60s)
- Small data, same dst IP/domain, unusual port (e.g. 4444, 8080)

**Data Exfiltration:**
- Large outbound transfer, unusual destination
- DNS tunneling: high volume of DNS queries, long subdomains
- HTTPS to non-standard port or new/unknown domain

---

## My Understanding in Plain English
Network Security Essentials sets the foundation for SOC monitoring. The key mindset shift is thinking in terms of visibility — you can only defend what you can see. Perimeter devices (firewall, IDS/IPS, proxy) are your sensors, not just your blockers. Logs from these devices are the evidence trail — correlating firewall logs + DNS logs + proxy logs across timestamps is how you reconstruct an attack. The DMZ concept is important for understanding why a compromised web server doesn't automatically mean the internal network is gone. Detection methods (signature vs anomaly vs behaviour) explain why no single tool catches everything.

---

## Related Notes
- [[Introduction-to-SIEM]]
- [[Introduction-to-EDR]]
- [[Wireshark-Traffic-Analysis]]
- [[Network-Traffic-Basics]]
