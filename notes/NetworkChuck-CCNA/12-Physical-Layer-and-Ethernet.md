# 12 - Physical Layer, Ethernet & Pinouts (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 11)
**Date:** May 29, 2026

## 🧠 Core Concept: How Copper Transmits Data
At Layer 1 of the OSI model, data is just electrical voltages. To send a binary `1`, the computer might send +2.5 volts down the copper wire. To send a binary `0`, it might send -2.5 volts. The receiving NIC (Network Interface Card) reads these rapid voltage changes and translates them back into binary data.

## 🛡️ Cable Protection & Types
Ethernet cables have two main enemies: **EMI** (Electromagnetic Interference from other electrical devices) and **Crosstalk** (interference between the wires inside the cable itself).
* **Twisted Pairs:** Wires inside an Ethernet cable are twisted together specifically to cancel out EMI and Crosstalk.
* **UTP (Unshielded Twisted Pair):** The most common cable (used in homes and standard offices). It only has an outer plastic jacket.
* **STP (Shielded Twisted Pair):** Contains an internal metallic shield around the twisted pairs. Used in industrial environments with heavy EMI (like factories).
* **Plenum Cable:** A special fire-rated jacket. If it burns, it does not emit toxic fumes. Required by fire code when running cables through ceilings or HVAC spaces.

## 🚀 Ethernet Speed Standards & Wires
* **10Base-T (Cat3):** 10 Mbps. Uses only 4 wires (2 pairs).
* **100Base-TX (Fast Ethernet / Cat5):** 100 Mbps. Uses only 4 wires. 
* **1000Base-T (Gigabit / Cat5e):** 1 Gbps. Uses all 8 wires (4 pairs). 
    * *Limitation:* The max length of a Cat5e cable is **100 meters**. Any longer, and the signal degrades, causing "Late Collisions."
* **10GBASE-T (Cat6a):** 10 Gbps using 8 wires.
* **40GBASE-T (Cat8):** 40 Gbps using 8 wires.

## 🔌 Pinouts: Straight-Through vs. Crossover
When terminating an RJ45 connector, the standard color order is **T568B** (White/Orange, Orange, White/Green, Blue, White/Blue, Green, White/Brown, Brown).
* **Transmit (Tx) vs. Receive (Rx):** In older standards (10/100), PCs transmit on pins 1 and 2, and receive on pins 3 and 6. Switches do the exact opposite (Listen on 1/2, Transmit on 3/6).
* **Straight-Through Cable:** Used to connect different devices (PC to Switch). Both ends use the T568B standard.
* **Crossover Cable:** Historically required to connect *like* devices (PC to PC, or Switch to Switch) so their Tx and Rx pins cross over to meet each other properly. (One end uses T568B, the other uses T568A).
* **Auto-MDIX (The Modern Solution):** You rarely need crossover cables today. Modern Gigabit switches have a software feature called Auto-MDIX, which automatically detects what pins the connected device is using and flips its own Tx/Rx pins internally to match.

## ⚔️ Security Engineering Perspective
As a Security Engineer, Layer 1 (Physical) is the foundation of your threat model:
* **Wiretapping:** Because UTP cables are unshielded, highly sophisticated attackers can use inductive taps to read the electromagnetic radiation leaking from the copper wires without actually cutting the cable, passively sniffing unencrypted Layer 7 traffic.
* **Physical Port Security:** If you don't implement Layer 2 port security (like 802.1X MAC filtering) on the switches in the lobby, an attacker can simply plug their own rogue laptop directly into a wall jack and bypass your external firewall entirely.
* **Compliance & Availability:** Part of DevSecOps and Infrastructure Security is compliance. Running non-Plenum cables in a drop-ceiling violates fire codes, which can shut down a business. Furthermore, knowing that cables max out at 100 meters helps you troubleshoot availability issues (like late collisions) that mimic DDoS attacks.
