# 02 - What is a Switch? (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (Day 1)
**Date:** May 1, 2026

## 🧠 Core Concept: Hubs vs. Switches
* **The Hub (Legacy):** A hub is a "dumb" device. When a computer sends a frame of data into port 1, the hub simply copies and shouts that data out of every other port. This causes massive network congestion and **collisions** (when two devices talk at the same time).
* **The Switch (Modern):** A switch is an "intelligent" device. It inspects incoming traffic, learns where devices live, and forwards the data *only* to the specific port where the destination device is plugged in.

## 📍 How a Switch Works (The MAC Address Table)
A switch learns the layout of a network dynamically using its **MAC Address Table** (also known as a CAM table).
1. A computer (MAC: AAAA) sends a message to another computer.
2. The switch receives the frame and looks at the **Source MAC**. It records: *"MAC AAAA is plugged into Port 1."*
3. The switch looks at the **Destination MAC**. If it doesn't know where that MAC is yet, it temporarily acts like a hub and floods the message to all ports.
4. When the destination replies, the switch records its MAC too. Over time, it builds a complete map of the network.

## 💻 First Cisco IOS Commands
If you log into a Cisco switch CLI, you start in user mode. 
* `enable` -> Moves you into **Privileged EXEC mode** (like becoming root/admin in Linux).
* `show mac address-table` -> Displays the internal map the switch has built of all connected MAC addresses and their corresponding physical ports.

## ⚔️ Offensive Security Perspective (Why Pentesters Care)
* **Packet Sniffing Difficulty:** In the old days, a pentester could plug into a hub, open Wireshark, and capture everyone's passwords because the hub broadcasted everything. Modern switches prevent this by isolating traffic. If you plug your Kali machine into port 4, you only see traffic meant for port 4.
* **MAC Flooding Attack (CAM Overflow):** To bypass this isolation, an attacker uses tools (like `macof`) to blast the switch with thousands of fake MAC addresses per second. 
* **The Result:** The switch's internal memory fills up completely. When a switch's MAC table overflows, it panics and "fails open"—reverting its behavior back to a dumb hub. It starts broadcasting all traffic to all ports, allowing the attacker to open Wireshark and sniff the entire network's sensitive data.
