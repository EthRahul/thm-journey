# 06 - Upper OSI Layers & Ports (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 5)
**Date:** May 19, 2026

## 🧠 Core Concept: The Upper Layers (App, Presentation, Session)
While routers and switches handle the bottom layers, the upper layers of the OSI model happen primarily within your computer's software and operating system.
* **Layer 7 (Application):** The interface for a network-aware program. When your web browser wants to see a website, it sends an HTTP `GET` request. 
* **Layer 6 (Presentation):** Responsible for data formatting and encryption. It ensures the data is a recognized file type (like HTML, JPEG, or PDF). Critically, this is also where encryption protocols like SSL/TLS encrypt your data before sending it down the stack.
* **Layer 5 (Session):** Opens, maintains, and tears down the communication channel between your application and the destination server. (e.g., managing multiple browser tabs talking to different servers).

## 🚚 The Transport Layer (Layer 4): TCP vs. UDP
The Transport Layer determines *how* the data gets shipped across the network. 
* **TCP (Transmission Control Protocol):** Built for **Reliability**. TCP is "naggy." Before sending data, it performs a **Three-Way Handshake** (SYN -> SYN-ACK -> ACK) to guarantee a connection. It numbers every segment, and if the destination doesn't send an Acknowledgement (ACK) that it received it, TCP will resend it.
* **UDP (User Datagram Protocol):** Built for **Speed**. UDP is "reckless." It does not establish a connection or check if you received the data; it just blasts it at you.
* *Real-World Example (YouTube):* Loading the YouTube webpage (the HTML, buttons, thumbnails) uses **TCP** because you need every line of code to render the page properly. But once the video starts playing, it switches to **UDP**. If a few packets drop, the video might glitch for a millisecond, but it keeps playing. If it used TCP, the video would constantly freeze to re-download dropped frames.

## 🚪 Port Numbers (Addressing the Application)
If an IP address gets your packet to the correct building (the server), the **Port Number** gets it to the correct apartment (the specific service running on that server).
* **Well-Known Ports (0 - 1023):** Standardized ports listening on servers.
    * Port 80 (HTTP)
    * Port 443 (HTTPS)
    * Port 21 (FTP)
    * Port 22 (SSH)
    * Port 69 (TFTP - Uses UDP!)
* **Ephemeral Ports:** When your laptop requests a webpage from YouTube (Destination Port 443), your browser generates a random, temporary "source port" (e.g., 57095) so YouTube knows exactly which browser tab to send the video back to.

## ⚔️ Offensive Security Perspective (Why Pentesters Care)
* **Nmap Scanning (Layer 4 Recon):** When you run a port scan using Nmap, you are directly interacting with Layer 4. A standard `nmap -sS` (Stealth Scan) sends a TCP SYN packet. If the target server replies with a SYN-ACK, Nmap knows the port is open, but instead of completing the 3-way handshake, Nmap maliciously drops the connection to avoid being logged.
* **Service Enumeration:** Knowing common ports is mandatory. If you see Port 22 open on a web server, you know there is a potential entry point to brute force SSH credentials.
* **Wireshark Analysis:** Capturing traffic allows you to see the exact ports being used. If you capture a victim's traffic and see communication leaving their machine going to an unknown IP on Port 4444 (a common Metasploit default port), you have likely identified a reverse shell.
