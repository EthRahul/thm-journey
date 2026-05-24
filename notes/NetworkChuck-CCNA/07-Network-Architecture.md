# 07 - Network Architecture & Design (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 6)
**Date:** May 24, 2026

## 🧠 Core Concept: Single Points of Failure
* A poorly designed "noob" network daisy-chains switches together. If one cable is cut (or chewed by a dog), every switch down the line loses connectivity.
* **Redundancy** is the primary goal of enterprise network design. You must build pathways so that if a cable, switch, or router physically fails, traffic can automatically route around the dead zone without the business losing money.

## 🏢 Two-Tier Architecture (The Collapsed Core)
This is the most common design for small-to-medium businesses or single corporate buildings.
1. **Access Layer (Tier 1):** The switches your endpoints (computers, IP phones, servers, IoT devices) plug directly into. 
2. **Distribution Layer (Tier 2):** The aggregator. All Access switches connect up to the Distribution switches. 
    * These utilize **Multi-Layer (Layer 3) Switches**—massive devices that have the port density of a switch but can process IP addresses and perform routing just like a router.
    * They handle the heavy lifting: Inter-VLAN routing, network policies, and routing protocols.
    * In a Two-Tier model, the Distribution Layer also absorbs the responsibilities of the Core Layer (hence the term **"Collapsed Core"**).

## 🏙️ Three-Tier Architecture (Campus Design)
When a business expands to multiple buildings (a campus), connecting every distribution switch to every other distribution switch creates a messy, unscalable web.
1. **Access Layer:** Endpoints.
2. **Distribution Layer:** Building aggregators.
3. **Core Layer (Tier 3):** The absolute network backbone. 
    * The Core layer connects the separate Distribution layers of different buildings together.
    * Its ONLY job is to move massive amounts of data as fast as humanly possible with ultra-low latency. You do not burden the core switch with policies; you just let it push packets.

## 🛡️ Security Engineering Perspective (Why the Blue Team Cares)
As a Security Engineer or SOC Analyst, you don't just look at this map to see how data flows; you look at it to see where to place your defenses and sensors.
* **Access Layer Security:** This is where internal threats (and physical attackers) plug into the wall. As a security engineer, you harden this layer by implementing **Port Security**, 802.1X (Network Access Control), and disabling unused ports to prevent rogue devices from joining.
* **Distribution Layer Security:** Because traffic between different departments (VLANs) passes through here, this is your primary security checkpoint. This is where you write **Access Control Lists (ACLs)** to ensure the Guest WiFi VLAN cannot communicate with the Internal Server VLAN. It is also the ideal place to mirror traffic to your SOC's Intrusion Detection System (IDS).
* **Core Layer Reliability:** The core layer represents pure **Availability** (the 'A' in the CIA Triad). A DDoS attack that overwhelms the core brings the entire corporate campus down. Security engineers must ensure redundant routing paths exist around the core. 
