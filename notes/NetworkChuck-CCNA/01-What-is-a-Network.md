# 01 - What is a Network? (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (Video 1)
**Date:** May 1, 2026

## 🧠 Core Concept: The Definition of a Network
A network, at its most fundamental level, is simply **two or more computers connected together so they can share resources.** 

If you connect two laptops together with an Ethernet cable so they can share a file, you have built a network.

## 🏗️ The Evolution of Networks
* **The Standalone Era (Sneakernet):** Before networking, sharing data meant saving it to a floppy disk or USB drive and physically walking it over to another computer. 
* **The LAN (Local Area Network):** Connecting computers within the same building or room (like your home Wi-Fi or a school computer lab) using switches and cables.
* **The WAN (Wide Area Network):** Connecting LANs together across large geographical distances (e.g., connecting an office in New York to an office in London). The Internet is the largest WAN in existence.

## ⚔️ Offensive Security Perspective (Why this matters for Pentesting)
As an offensive security researcher, understanding what a network is dictates *how* you attack it. 

* **The Goal:** Pentesters aren't just hacking single computers; they are exploiting the *connections* between them. 
* **The "Resources":** When a network shares a resource (like a printer, a file share, or a database), it creates a pathway. If an attacker compromises one device on the network, they will attempt to pivot across those established pathways to access the shared resources of other, more secure machines. 
* **The Scope:** Before running any exploit, you must enumerate the network to understand its boundaries (Are you attacking a local LAN, or are you attacking public-facing WAN infrastructure?).

## 📝 Key Terminology
* **Host:** Any device connected to a network (a laptop, a server, a smartphone, a smart TV).
* **Switch:** A device that connects multiple hosts together within the *same* local network (LAN).
* **Router:** A device that connects multiple *different* networks together (e.g., connecting your home LAN to the public Internet WAN).
