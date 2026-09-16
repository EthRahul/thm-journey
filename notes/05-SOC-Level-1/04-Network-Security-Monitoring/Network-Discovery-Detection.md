# Network Discovery Detection
**Path:** SOC Level 1 - Network Security Monitoring  
**Date:** 2026-05-31  
**Difficulty:** Easy

---

## Network Discovery

Network discovery = attacker mapping the target network to find live hosts, open ports, services, and OS before exploitation.

### Discovery Goals (Attacker POV)
1. Which hosts are alive?
2. What ports/services are open?
3. What OS is running?
4. What vulnerabilities exist?

### Common Discovery Tools Used by Attackers
| Tool | Purpose |
|---|---|
| Nmap | Port scanning, OS/service detection |
| Masscan | Ultra-fast port scanning |
| Netdiscover | ARP-based host discovery |
| Angry IP Scanner | GUI-based host/port scanner |
| Nessus / OpenVAS | Vulnerability scanning |

---

## External vs Internal Scanning

### External Scanning
- Attacker is **outside** the network (internet-facing)
- Targets: public IPs, exposed services (web, SSH, RDP, VPN)
- Detection point: **perimeter firewall / IDS**
- Indicators: port scans from unknown/foreign IPs, high connection rate to multiple ports

### Internal Scanning
- Attacker is **already inside** the network (post-compromise)
- Targets: internal hosts, subnets, lateral movement paths
- Detection point: **internal IDS, east-west monitoring, SIEM**
- More dangerous — attacker has higher trust level
- Indicators: a single internal host scanning many others, ARP sweeps, unusual SMB/RDP connections

### Key Difference
| | External | Internal |
|---|---|---|
| Attacker position | Outside perimeter | Inside network |
| Detection difficulty | Easier (perimeter tools) | Harder (needs internal visibility) |
| Risk level | Lower (blocked by firewall) | Higher (bypasses perimeter) |

---

## Horizontal vs Vertical Scanning

### Horizontal Scanning (Wide scanning)
- Scan **one port across many hosts**
- Goal: find all hosts with a specific service open
- Example: scan port 22 across entire /24 subnet → find all SSH servers
- Indicator: same src IP → many dst IPs, same dst port

```
nmap -p 22 192.168.1.0/24
```

### Vertical Scanning (Deep scanning)
- Scan **many ports on one host**
- Goal: enumerate all services on a specific target
- Example: full port scan of a single server
- Indicator: same src IP → same dst IP, many dst ports

```
nmap -p- 192.168.1.10
```

### Combined (Matrix Scan)
- Many ports across many hosts
- Most noisy — easiest to detect
- Tools like Masscan do this at high speed

---

## The Mechanics of Scanning

### Host Discovery Techniques
| Technique | How | Wireshark Filter |
|---|---|---|
| ICMP Ping sweep | ICMP echo to each IP | `icmp.type == 8` |
| ARP sweep | ARP who-has for each IP (LAN only) | `arp.opcode == 1` |
| TCP SYN ping | SYN to port 80/443 | `tcp.flags.syn==1 and tcp.flags.ack==0` |
| UDP ping | UDP to closed port → ICMP unreachable | `icmp.type == 3` |

### Port Scan Types & Signatures

| Scan Type | Flags Sent | Open Response | Closed Response | Filter |
|---|---|---|---|---|
| SYN scan | SYN | SYN-ACK | RST | `tcp.flags.syn==1 and tcp.flags.ack==0` |
| Connect scan | SYN | Full handshake | RST | Full 3-way then RST |
| NULL scan | None (0x000) | No response | RST | `tcp.flags==0x000` |
| FIN scan | FIN | No response | RST | `tcp.flags.fin==1` |
| XMAS scan | FIN+PSH+URG | No response | RST | `tcp.flags.fin==1 and tcp.flags.push==1 and tcp.flags.urg==1` |
| UDP scan | UDP | No response | ICMP unreachable | `icmp.type==3` |

> NULL / FIN / XMAS scans use unusual flag combos to evade basic stateless firewalls.

### OS Fingerprinting
- **Passive:** analyse TTL, TCP window size, TCP options in existing traffic
- **Active:** Nmap `-O` sends crafted packets and analyses responses

| OS | Typical TTL |
|---|---|
| Windows | 128 |
| Linux | 64 |
| Cisco / Network devices | 255 |

### Service Version Detection
- Nmap `-sV` — grabs banners after port found open
- SSH version string, HTTP Server header, FTP banner all reveal software + version
- Detection: probe packets to open ports immediately after handshake

---

## Detection Indicators Summary

### Behavioural Patterns in Logs
- Single src IP → many dst ports on same host → **vertical scan**
- Single src IP → same port across many hosts → **horizontal scan**
- High RST count → scanner hitting closed ports
- Sequential IP or port targeting
- Rapid connections with no data transferred
- Traffic spikes during off-hours

### Wireshark Filters for Detection
```
# Ping sweep
icmp.type == 8

# ARP sweep
arp.opcode == 1

# SYN scan
tcp.flags.syn==1 and tcp.flags.ack==0

# High RST (closed port responses)
tcp.flags.reset==1

# NULL scan
tcp.flags==0x000

# XMAS scan
tcp.flags.fin==1 and tcp.flags.push==1 and tcp.flags.urg==1

# UDP scan (ICMP port unreachable responses)
icmp.type==3 and icmp.code==3
```

---

## My Understanding in Plain English
Network discovery is the recon phase — attackers map the network before attacking it. External scans are caught at the perimeter; internal scans (post-compromise lateral movement) are far more dangerous and need east-west visibility inside the network. Horizontal scanning finds who has a service; vertical scanning finds all services on a target. Each scan type has a unique packet signature — NULL/FIN/XMAS scans use weird flag combos specifically to slip past basic firewalls. As a SOC analyst, you're looking for these patterns: sequential ports, high RST counts, unusual TCP flags, and rapid connection attempts from a single host.

---

## Related Notes
- [[Network-Security-Essentials]]
- [[Wireshark-Traffic-Analysis]]
- [[Nmap-Basic-Port-Scans]]
- [[Nmap-Advanced-Port-Scans]]
