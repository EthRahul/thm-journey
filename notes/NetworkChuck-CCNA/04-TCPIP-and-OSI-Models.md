# 04 - TCP/IP and OSI Models (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 3)
**Date:** May 19, 2026

## 🧠 Core Concept: Why do we need Networking Models?
* **The "Dark Ages":** In the early days, networks were completely proprietary. An IBM computer could only talk to an IBM network, and an Apple computer could only talk to Apple. They essentially spoke entirely different languages.
* **The Solution:** The industry desperately needed a universal standard. Two primary models emerged to define how data should be packaged, addressed, and transmitted: **TCP/IP** and **OSI**.
* **The Victor:** The **TCP/IP model** won the historical "protocol wars." It is the actual, functional architecture running on every single modern device today.
* **The Survivor:** Even though TCP/IP won the technical war, the 7-layer **OSI model** (Open Systems Interconnection) is still universally used by cybersecurity and network professionals as the standard reference terminology for troubleshooting and discussing networks.

## 📚 The 7 Layers of the OSI Model
The OSI model divides network communication into 7 distinct, isolated layers.
* **Layer 7 - Application:** The layer you interact with directly (Web Browsers, HTTP, DNS).
* **Layer 6 - Presentation:** Formatting, encryption, and compression (JPEG, TLS/SSL).
* **Layer 5 - Session:** Establishing and maintaining connections between software apps.
* **Layer 4 - Transport:** Deciding *how* to send the data (TCP for reliability, UDP for speed) and using Port Numbers.
* **Layer 3 - Network:** Logical addressing and routing (IP Addresses, Routers).
* **Layer 2 - Data Link:** Physical addressing on the local network (MAC Addresses, Switches).
* **Layer 1 - Physical:** The actual hardware (Cables, Fiber, Wi-Fi radio waves, Hubs).

**Mnemonics to memorize it:**
* Top to Bottom: **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing
* Bottom to Top: **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

## ⚔️ Offensive Security Perspective (Why Pentesters Care)
Knowing the OSI model isn't just for passing exams—it dictates your entire attack chain. Pentesters categorize their attacks based on what layer they are targeting:
* **Layer 2 Attacks (Data Link):** ARP Spoofing, MAC Flooding, VLAN Hopping. You are attacking the local switch infrastructure.
* **Layer 3 Attacks (Network):** IP Spoofing, Ping of Death, Route manipulation. You are manipulating how packets travel across geographical boundaries.
* **Layer 4 Attacks (Transport):** SYN Floods (DDoS), Port Scanning (Nmap stealth scans). You are exploiting how connections are established.
* **Layer 7 Attacks (Application):** SQL Injection, Cross-Site Scripting (XSS), Command Injection. *This is where your Web App Pentesting modules live!* When you exploit a web server using Burp Suite, you are operating entirely at Layer 7.
