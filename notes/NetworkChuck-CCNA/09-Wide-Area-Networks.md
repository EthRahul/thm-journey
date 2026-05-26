# 09 - Wide Area Networks (WAN) & Connectivity (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 8)
**Date:** May 26, 2026

## 🧠 Core Concept: LAN vs. WAN
* **LAN (Local Area Network):** The network *inside* your contained environment (your house, the corporate headquarters, or the data center). 
* **WAN (Wide Area Network):** The connections used to bridge those isolated LANs together across large geographical distances (e.g., connecting a coffee shop in Seattle to a data center in Dallas). 
* *Note:* The public Internet is a WAN, but a corporate WAN does not necessarily have to use the public internet. It can be entirely private.

## 🏛️ Legacy WAN Technologies
You will still see these in the wild, though they are being phased out due to cost and lack of scalability:
* **Leased Lines (T1 / T3):** Renting a dedicated, physical copper cable from a provider that connects Site A directly to Site B. 
    * *Pros:* 100% private, guaranteed bandwidth, no shared traffic.
    * *Cons:* Expensive, very slow by modern standards (T1 is 1.54 Mbps), and requires a massive, unscalable mesh of cables if you have many branch offices.
* **Frame Relay & ATM:** Older packet-switching technologies used as alternatives to leased lines. (Mostly obsolete for modern CCNA).

## 🚀 Modern Private WAN (MPLS & Metro Ethernet)
* **MPLS (Multiprotocol Label Switching):** The enterprise standard for the last two decades. Instead of running dedicated cables to every single branch office, all branch offices connect locally to the provider's MPLS "cloud."
    * **How it works:** The provider uses a Layer 2.5 "Label" to identify your corporate traffic. This creates a Virtual Private Network (VPN) *without* needing encryption, simply by logically separating your traffic from other customers on the provider's backbone.
    * **CE vs. PE Routers:** You maintain the Customer Edge (CE) router at your office, which plugs into the Provider Edge (PE) router.
* **Metro Ethernet (Metro E):** Blazing fast (1 Gbps - 10 Gbps) fiber connections typically used to connect a corporate HQ directly to a Data Center within the same metropolitan city.
    * **E-Line:** Point-to-Point (like a modern, super-fast leased line).
    * **E-LAN:** Point-to-Multipoint (Acts like a giant switch in the sky connecting multiple sites).
    * **E-Tree:** Hub and Spoke topology.

## 🌐 Public WAN (Internet VPN & SD-WAN)
Because MPLS and Metro E are expensive, many companies use standard public internet lines for their remote branches.
* **Site-to-Site VPN:** Because the public internet is untrusted, branch routers must encapsulate and mathematically encrypt their traffic (usually via IPsec) before sending it over the public internet to the HQ router.
* **The Problem with Public Internet:** Traffic is subject to public congestion. You cannot guarantee QoS (Quality of Service) to prioritize important traffic (like VoIP phone calls).
* **SD-WAN (Software-Defined WAN):** The modern replacement for MPLS. It intelligently routes traffic over multiple cheaper internet connections simultaneously, optimizing paths in real-time to mimic the reliability of expensive private lines.

## 🛡️ Security Engineering Perspective
As a Security Engineer or SOC Analyst, the type of WAN dictates your threat model and monitoring strategy:
* **Trusting the Provider (MPLS):** MPLS is a *private* network, but it is **not encrypted** by default. You are trusting the ISP's label-switching to keep your data safe. If a highly secure environment (like banking) uses MPLS, a Security Engineer will still enforce IPsec encryption *over* the MPLS link to achieve true Zero Trust.
* **The Cloud Shift:** Historically, branch offices routed all internet traffic back through the HQ's massive firewall before going out to the web (Hub-and-Spoke). With modern apps living in AWS and Azure, backhauling traffic to HQ causes massive latency. Security Engineers are now deploying cloud-based firewalls (SASE - Secure Access Service Edge) so branches can securely connect straight to the cloud without touching the corporate data center.
