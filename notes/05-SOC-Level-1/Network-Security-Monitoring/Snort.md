# Snort
**Path:** SOC Level 1 - Network Security Monitoring  
**Date:** 2026-06-04  
**Difficulty:** Hard

---

## Introduction to IDS/IPS

### What is Snort?
- Open-source **Network Intrusion Detection and Prevention System (NIDS/NIPS)**
- Developed by Martin Roesch (1998), now maintained by Cisco Talos
- Works by inspecting network traffic against a ruleset and alerting/blocking matches
- Used in SOC environments for real-time threat detection and pcap analysis

### IDS vs IPS
| | IDS | IPS |
|---|---|---|
| Position | Out-of-band (mirror/tap) | Inline (traffic passes through it) |
| Action | Alert only — passive | Alert + Block — active |
| Risk | No impact on traffic | Can block legitimate traffic |
| Snort mode | `-A` alert modes | `-Q` inline mode |

### Types of Detection
| Method | How |
|---|---|
| Signature-based | Matches known patterns — fast, low false positives |
| Anomaly-based | Detects deviation from baseline — catches zero-days |
| Policy-based | Flags violations of defined security policies |

---

## First Interaction with Snort

### Verify Installation
```bash
snort -V                    # check version
snort --help                # list all options
```

### Test Config File
```bash
sudo snort -c /etc/snort/snort.conf -T
# -c = config file path
# -T = test config, don't run
```

### Key Config File Locations
```
/etc/snort/snort.conf       # main config
/etc/snort/rules/           # rules directory
/var/log/snort/             # default log output
```

---

## Operation Mode 1: Sniffer Mode

Snort reads and displays packets live — like tcpdump. No logging, no alerting.

```bash
sudo snort -v               # verbose — show packet headers
sudo snort -d               # show packet data (payload)
sudo snort -e               # show data link layer headers
sudo snort -X               # show full packet in hex + ASCII
sudo snort -i eth0          # specify interface
```

### Combine flags
```bash
sudo snort -v -d -e         # headers + payload + data link
sudo snort -X -i eth0       # full hex dump on eth0
```

---

## Operation Mode 2: Packet Logger Mode

Snort captures and saves packets to disk.

```bash
sudo snort -dev -l /var/log/snort          # log to directory
sudo snort -dev -l /var/log/snort -h 192.168.1.0/24   # log specific network
sudo snort -r /var/log/snort/snort.log     # read back saved log
sudo snort -r snort.log -n 10             # read only first 10 packets
```

### Log file formats
- Default: binary tcpdump format (`.log` file)
- ASCII mode: `-K ASCII` flag → human-readable but larger
```bash
sudo snort -dev -K ASCII -l /var/log/snort
```

---

## Operation Mode 3: IDS/IPS Mode

Snort runs against a ruleset and generates alerts.

```bash
# IDS mode — alert on matches
sudo snort -c /etc/snort/snort.conf -A console

# IDS mode on specific interface
sudo snort -c /etc/snort/snort.conf -i eth0 -A console

# IPS inline mode (requires DAQ)
sudo snort -c /etc/snort/snort.conf -Q --daq afpacket -i eth0:eth1
```

### Alert Modes (`-A`)
| Mode | Output |
|---|---|
| `console` | Prints alerts to terminal in real time |
| `cmg` | CMG format — full packet header + payload |
| `fast` | Timestamp, message, src/dst IP:port |
| `full` | Most detailed alert format |
| `none` | No alerts (logging only) |

```bash
sudo snort -c /etc/snort/snort.conf -A fast -l /var/log/snort
sudo snort -c /etc/snort/snort.conf -A console -i eth0
```

---

## Operation Mode 4: PCAP Investigation

Run Snort against a saved pcap file — core SOC analyst workflow.

```bash
# Single pcap
sudo snort -c /etc/snort/snort.conf -r capture.pcap -A console

# Multiple pcaps
sudo snort -c /etc/snort/snort.conf --pcap-list="file1.pcap file2.pcap"

# All pcaps in a directory
sudo snort -c /etc/snort/snort.conf --pcap-dir=/path/to/pcaps/

# Show stats only
sudo snort -c /etc/snort/snort.conf -r capture.pcap --stats-file=stats.txt
```

### Key flags for pcap investigation
```bash
-n 100          # process only first 100 packets
-X              # show hex dump of each packet
-v              # verbose output
--log-tcpdump   # output as tcpdump format
```

---

## Snort Rule Structure

### Rule Anatomy
```
[action] [protocol] [src_ip] [src_port] -> [dst_ip] [dst_port] ([options])
```

### Example Rules
```
# Alert on any ICMP traffic
alert icmp any any -> any any (msg:"ICMP Packet Detected"; sid:1000001; rev:1;)

# Alert on HTTP traffic to specific IP
alert tcp any any -> 192.168.1.10 80 (msg:"HTTP to target"; sid:1000002; rev:1;)

# Alert on specific content in TCP payload
alert tcp any any -> any 80 (msg:"GET request detected"; content:"GET"; sid:1000003; rev:1;)

# Alert on DNS queries
alert udp any any -> any 53 (msg:"DNS Query"; sid:1000004; rev:1;)
```

### Rule Actions
| Action | What it does |
|---|---|
| `alert` | Generate alert + log packet |
| `log` | Log packet only, no alert |
| `pass` | Ignore the packet |
| `drop` | Block + log (IPS mode) |
| `reject` | Block + send TCP RST / ICMP unreachable |

### Protocol Options
`tcp` `udp` `icmp` `ip`

### IP/Port Syntax
```
any                     # match any IP or port
192.168.1.0/24         # CIDR notation
!192.168.1.0/24        # NOT this network
[192.168.1.1,10.0.0.1] # list of IPs
any -> any             # one direction
any <> any             # bidirectional
```

### Key Rule Options
| Option | Purpose | Example |
|---|---|---|
| `msg` | Alert message | `msg:"SSH Brute Force";` |
| `sid` | Unique rule ID (>1000000 for custom) | `sid:1000001;` |
| `rev` | Rule revision number | `rev:1;` |
| `content` | Match string in payload | `content:"admin";` |
| `nocase` | Case-insensitive content match | `content:"GET"; nocase;` |
| `pcre` | Regex match | `pcre:"/evil\.php/i";` |
| `flags` | TCP flag match | `flags:S;` (SYN only) |
| `dsize` | Packet data size | `dsize:>1000;` |
| `ttl` | Match TTL value | `ttl:128;` |
| `threshold` | Rate limiting / frequency | `threshold: type limit, track by_src, count 5, seconds 60;` |
| `priority` | Alert priority 1-10 | `priority:1;` |
| `classtype` | Alert classification | `classtype:web-application-attack;` |

### Content Modifiers
```
content:"GET";          # exact string match
content:"|47 45 54|";   # hex match (GET in hex)
content:"GET"; offset:0; depth:3;   # match at specific byte position
content:"admin"; nocase;            # case insensitive
```

---

## Snort2 Operation Logic: Key Points

### Rule Processing Order
1. Pass rules (highest priority)
2. Drop rules
3. Alert rules
4. Log rules

### Important Snort2 Behaviours
- Rules are processed **top to bottom** — order matters
- `sid` must be **unique** — duplicate SIDs cause errors
- Custom rules: SID range **1,000,000 – 2,000,000**
- Local rules file: `/etc/snort/rules/local.rules`
- Always test config after adding rules: `snort -c snort.conf -T`

### Adding Custom Rules
```bash
# Edit local rules file
sudo nano /etc/snort/rules/local.rules

# Add your rule, save, then test
sudo snort -c /etc/snort/snort.conf -T

# Run with custom rules
sudo snort -c /etc/snort/snort.conf -A console -i eth0
```

### Reading Alert Logs
```bash
# Default alert file
cat /var/log/snort/alert

# Live monitoring
tail -f /var/log/snort/alert
```

### Alert Log Format (fast mode)
```
[timestamp] [**] [sid:rev] "msg" [**] [Classification] [Priority]
{protocol} src_ip:src_port -> dst_ip:dst_port
```

---

## Quick Reference: Most Used Commands

```bash
# Test config
sudo snort -c /etc/snort/snort.conf -T

# Live IDS on interface
sudo snort -c /etc/snort/snort.conf -i eth0 -A console

# Investigate pcap
sudo snort -c /etc/snort/snort.conf -r file.pcap -A console

# Sniffer — see packets live
sudo snort -v -d -e -i eth0

# Log packets
sudo snort -dev -l /var/log/snort -i eth0

# Read saved log
sudo snort -r /var/log/snort/snort.log -v
```

---

## My Understanding in Plain English
Snort has three personalities: a packet sniffer (just shows traffic), a packet logger (saves traffic), and an IDS/IPS (matches traffic against rules and alerts/blocks). The core SOC use case is running Snort against a pcap file to see which rules trigger — that's the PCAP Investigation mode. Rule writing is the most important skill: every rule follows the same structure — action, protocol, IPs, ports, then options in parentheses. The `content` keyword matches payload strings, `flags` matches TCP flags, `threshold` controls alert frequency. Custom rules go in `local.rules` with SIDs above 1,000,000. Always test config with `-T` before running live.

---

## Related Notes
- [[Network-Security-Essentials]]
- [[Network-Discovery-Detection]]
- [[Wireshark-Traffic-Analysis]]
- [[Introduction-to-SIEM]]
