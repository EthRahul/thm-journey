# 05 - Packet Flow and Encapsulation (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 4)
**Date:** May 19, 2026

## 🧠 Core Concept: Encapsulation
When your computer sends a message (like requesting a website), it doesn't just shoot raw data across the cable. The data must be packaged with instructions so the network knows what to do with it. This process of wrapping data in layers of headers is called **Encapsulation** (like putting a letter inside an envelope, and then putting that envelope inside a bigger shipping box).

## 📦 The Journey of a Message (PDUs)
As data moves down the OSI model, it changes its name based on the headers attached to it. These are known as Protocol Data Units (PDUs):
1. **Layer 7 (Application):** The raw **Data** (e.g., an HTTPS GET request asking for `networkchuck.coffee`).
2. **Layer 4 (Transport):** The computer adds a Layer 4 Header (specifying TCP or UDP, and port numbers like 443). The data is now called a **Segment**.
3. **Layer 3 (Network):** A Layer 3 Header is added (containing the Source and Destination IP Addresses). The segment is now called a **Packet**.
4. **Layer 2 (Data Link):** A Layer 2 Header and Trailer are added (containing Source and Destination MAC Addresses). The packet is now called a **Frame**.

## 🚦 How Network Devices Read the Layers
When the Frame travels across the network wire, different devices process it differently:
* **Switches (Layer 2):** A switch only opens the outermost "envelope" (Layer 2). It reads the MAC addresses, forwards the frame to the correct port, and leaves the rest of the package untouched.
* **Routers (Layer 3):** A router opens the Layer 2 envelope and discards it. It then reads the Layer 3 IP address to determine the best path forward. Once it decides where to route the packet, it **re-encapsulates** it into a *brand new* Layer 2 Frame with new MAC addresses before sending it to the next hop.
* **The Destination Server:** Once the server receives it, it performs **De-encapsulation**—stripping away Layer 2, then Layer 3, then Layer 4, until the web server software can finally read the Layer 7 Application Data.

## ⚔️ Offensive Security Perspective (Why Pentesters Care)
* **Wireshark & Packet Sniffing:** When you capture traffic in Wireshark, you are looking directly at these encapsulated layers. Wireshark parses the Frame, Packet, Segment, and Data so you can read them. If the Layer 4 port is 80 (HTTP), the Layer 7 data is in plain text. If it is 443 (HTTPS), the Layer 7 data is encrypted.
* **Header Manipulation:** Hackers exploit the fact that devices trust these headers blindly.
    * **Layer 3 Manipulation:** By changing the Source IP in the L3 header before encapsulating the packet, an attacker can perform **IP Spoofing**.
    * **Layer 7 Manipulation:** In Web App Pentesting (using tools like Burp Suite), you intercept the traffic at Layer 7 before the browser encrypts and encapsulates it. Modifying HTTP headers, cookies, or payloads at this stage is exactly how you execute attacks like SQL Injection or Session Hijacking.
