# 10 - Home Network Security & IP Addressing (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 9) + Supplemental SOC Material
**Date:** May 26, 2026

## 🧠 Core Concept: SOHO Network Vulnerabilities
A Small Office / Home Office (SOHO) network usually consists of a single all-in-one router, switch, and access point. These are highly vulnerable entry points:
* **External Threats:** Hackers constantly scan public IP blocks using tools like Nmap to find open ports (e.g., an exposed Remote Desktop port 3389 or a forgotten HTTP port 80).
* **Internal Threats (IoT):** Smart bulbs, TVs, and appliances frequently phone home to unknown servers. Because default firewalls typically allow all outbound traffic, a compromised IoT device acts as a permanent backdoor into the internal network.

## 🛡️ 6 Steps to Harden a Router
1.  **Enable the Firewall:** Ensure the built-in SPI (Stateful Packet Inspection) firewall is toggled on to block unprompted inbound traffic.
2.  **Disable Port Forwarding:** Close all external holes. If you need remote access to an internal server, use a VPN, not port forwarding.
3.  **Disable Remote Management:** Never allow the router's admin portal to be accessed from the public WAN.
4.  **Change Default Credentials:** Immediately change default logins (`admin/admin`) to complex passwords.
5.  **Update Firmware:** Hardware manufacturers patch critical security flaws regularly.
6.  **Secure the Wi-Fi:** Use WPA2/WPA3, change the default SSID (which gives away the router model), and put guests on a completely isolated Guest Network.

---

## 🌐 Supplemental: IP Addressing & Subnetting
To effectively implement security controls like VLAN isolation, you must understand how IP addresses are structured and assigned.

### IPv4 vs. IPv6
* **IPv4:** The legacy standard. It is 32 bits long, written in four octets (e.g., `192.168.1.1`). Because it only provides about 4.3 billion addresses, the world ran out of them. We survive using NAT (Network Address Translation), which allows multiple private internal devices to share a single public IP address.
* **IPv6:** The modern standard. It is 128 bits long, written in hexadecimal (e.g., `2001:0db8:85a3:0000:0000:8a2e:0370:7334`). It provides an effectively infinite number of public IPs, theoretically eliminating the strict need for NAT.

### How IPs are Assigned
1.  **Static IP:** Manually configured by a network engineer. Used for servers, printers, and routers so their address never changes, ensuring clients can always find them.
2.  **DHCP (Dynamic Host Configuration Protocol):** A server (or your router) automatically hands out IP address "leases" to clients from a predefined pool. 
    * *The DORA Process:* **D**iscover (Client shouts for a server) -> **O**ffer (Server offers an IP) -> **R**equest (Client accepts) -> **A**cknowledge (Server confirms).

### Subnetting (The Basics)
Subnetting takes a large network block and chops it into smaller, isolated sub-networks (which physically map to VLANs).
* **Subnet Mask:** Tells the computer which part of the IP address is the "Network" (the street) and which part is the "Host" (the specific house).
* *Example (`/24` CIDR Notation):* An IP of `192.168.1.50` with a mask of `255.255.255.0` (`/24`). 
    * The Network portion is `192.168.1.0`.
    * The Host portion is `.50`.
    * This standard subnet can hold 254 usable devices. If an enterprise needs more segmentation for security, engineers manipulate the binary bits in the subnet mask to create multiple smaller networks (e.g., slicing a `/24` into four `/26` networks, each holding 62 hosts).

## ⚔️ Security Engineering Perspective
* **Asset Discovery:** The `nmap -sT [Subnet/24]` and `nmap -O [IP]` commands are your best friends in a SOC. You cannot protect a network if you don't know what is plugged into it. Scanning your own subnets regularly detects rogue IoT devices or unauthorized servers.
* **Zero Trust and Isolation:** A Security Engineer handles untrusted devices using VLANs. By putting trusted devices (corporate laptops) on VLAN 10 and untrusted IoT devices on VLAN 20, you isolate them. Enabling **Client Isolation** ensures devices on the same wireless network cannot even ping or communicate with each other.
* **DHCP Starvation & Spoofing:** Attackers can flood a network with fake DHCP requests to exhaust the IP pool, or spin up a rogue DHCP server to assign malicious IP addresses to legitimate clients (routing their traffic to the attacker for a Man-in-the-Middle attack). As a defender, you prevent this by configuring **DHCP Snooping** on enterprise switches to block untrusted DHCP offers.
