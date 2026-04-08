# TryHackMe: Net Sec Challenge
**Path:** Jr Penetration Tester - Network Security
**Date:** 2026-04-08
**Tools Used:** Nmap, Telnet, Hydra, FTP, Curl

---

## Executive Summary
This challenge room tests fundamental network security skills, requiring full-port enumeration, banner grabbing, and service brute-forcing. The environment consists of hidden services on non-standard ports and requires chaining reconnaissance data to gain authenticated access.

---

## Methodology & Solutions

### 1. Initial Reconnaissance (Port Scanning)
To answer the first three questions, a comprehensive port scan is required. Since services are hidden on non-standard ports, a full 65,535 port scan is mandatory.
```bash
sudo nmap -sS -p- -T4 <MACHINE_IP> -v
Q: What is the highest port number being open less than 10,000?

Answer: 8080

Concept: Nmap reveals all open ports. Filtering the output shows 8080 is the highest under the 10k threshold.

Q: There is an open port outside the common 1000 ports; it is above 10,000. What is it?

Answer: 10021

Concept: By scanning -p-, we force Nmap to look beyond its default top 1000 ports, revealing hidden services.

Q: How many TCP ports are open?

Answer: 6

Concept: Counting the total number of open states returned in the initial SYN scan.

2. Service Enumeration & Banner Grabbing
Once the open ports are identified, we need to determine what is running on them and extract hidden flags from their banners.

Bash
sudo nmap -sV -sC -p <LIST_OF_OPEN_PORTS> <MACHINE_IP>
Q: What is the flag hidden in the HTTP server header?

Answer: THM{web_server_25352}

Concept: Web servers often leak information in their HTTP headers. This can be grabbed using Nmap's -sV, or manually via curl -I http://<MACHINE_IP>:<PORT> or telnet <MACHINE_IP> <PORT>.

Q: What is the flag hidden in the SSH server header?

Answer: THM{946219583339}

Concept: SSH servers broadcast a banner before authentication. Connecting manually via nc <MACHINE_IP> <PORT> or ssh -v <MACHINE_IP> reveals the custom banner flag.

Q: We have an FTP server listening on a nonstandard port. What is the version of the FTP server?

Answer: vsftpd 3.0.5

Concept: Port 10021 is running FTP. Nmap's service version detection (-sV) interacts with the protocol to extract the exact software version.

3. Exploitation & Brute-Forcing
We have an FTP service on port 10021 and two potential usernames (eddie, quinn). We use Hydra to brute-force the password.

Bash
# 1. Create a users.txt file with 'eddie' and 'quinn'
# 2. Run Hydra against the non-standard FTP port
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://<MACHINE_IP>:10021
Q: What is the flag hidden in one of these two account files and accessible via FTP?

Answer: THM{321452667098}

Concept: Once Hydra finds the valid credential pair, log into the FTP server (ftp <MACHINE_IP> 10021). Use ls to list the files and get <filename> to download the file containing the flag to the local machine.

4. Web Challenge
Q: Browsing to http://MACHINE_IP:8080 displays a small challenge that will give you a flag once you solve it. What is the flag?

Answer: THM{f7443f99}

Concept: This requires interacting with the web application hosted on port 8080. Challenges like this typically involve triggering specific network behaviors (like running an Nmap scan against the machine to bypass a filter) or inspecting the page source to retrieve the final flag.
