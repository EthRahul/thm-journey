# 02 - Routers, MAC Addresses, and ARP (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (Video 2)
**Date:** May 1, 2026

## 🧠 Core Concept: Switches vs. Routers
* **Switches Create Networks:** They connect devices *within* the same Local Area Network (LAN). They operate using **MAC Addresses** to send data locally.
* **Routers Connect Networks:** They link *different* LANs together, acting as the gateway to the WAN (Internet). They operate using **IP Addresses** to navigate global traffic. 

## 📍 Addressing: MAC vs. IP
* **MAC Address (Physical Address):** Hardcoded into a device's Network Interface Card (NIC) by the manufacturer. It is permanent, much like a fingerprint or a hardware serial number. Switches rely entirely on MACs to deliver frames.
* **IP Address (Logical Address):** Assigned to a device dynamically or statically based on the network it currently resides in. It acts like a mailing address—if your laptop moves from your home Wi-Fi to a coffee shop network, its IP address changes. Routers use IPs to route packets between completely different geographic networks.

## 🔎 The Critical Protocol: ARP (Address Resolution Protocol)
* **The Problem:** A computer knows the IP address of the device it wants to talk to, but switches cannot read IPs; they only understand MAC addresses. How does the computer find out the destination's MAC?
* **The Solution (How ARP Works):**
    1. The sending computer shouts out to the entire local network (an **ARP Broadcast**): *"Who has IP address 192.168.1.50? Tell me your MAC address!"*
    2. Every device connected to the switch hears the shout.
    3. Only the specific device holding that IP responds: *"That is me! Here is my MAC address."*
    4. The sending computer saves this mapping in its **ARP Cache (ARP Table)** to avoid shouting the next time it needs to communicate.

## 🚪 The Default Gateway (Your Way Out)
* When your computer wants to talk to an IP address *outside* your local network (like a TryHackMe VPN server), it realizes the IP is on a different subnet.
* Because your switch cannot route outside the LAN, the computer forwards the data to the **Default Gateway** (which is your local Router).
* The router inspects the IP, determines the best exit path using its routing table, and sends the packet out toward the internet.

## 📖 DNS (Domain Name System) - Sneak Peek
* Humans memorize words (like `google.com`). Computers only understand numbers (like `142.250.190.46`).
* DNS acts as the "phonebook" of the internet. It translates the human-readable domain name into the correct IP address so your router knows exactly where to route the traffic.

## ⚔️ Offensive Security Perspective (Why Pentesters Care)
* **ARP Spoofing / Poisoning:** ARP is an old protocol that relies entirely on trust; it does not verify identities. As an attacker on a LAN, you can broadcast fake ARP replies. By telling the victim machine, *"Hey, my Kali Linux MAC address belongs to the Router's IP,"* the victim will unknowingly send all their internet traffic directly to you. This enables massive **Man-in-the-Middle (MitM)** attacks, allowing you to intercept passwords, session cookies, and sensitive data.
* **Routing Tables & Pivoting:** When you gain a reverse shell on a web server, that server often has two network interfaces (one facing the public internet, one facing the internal corporate network). By checking the compromised machine's internal routing table (`route` or `ip route`), you can map out internal subnets and pivot your attacks deeper into the network.
