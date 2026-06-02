# 14 - Fiber Optics & Transceivers (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 13)
**Date:** June 2, 2026

## 🧠 Core Concept: Transmitting Light
Unlike copper cables that use electrical voltages to transmit binary data, Fiber Optic cables use rapid pulses of light (photons) sent down an incredibly thin strand of glass. 
* A pulse of light = binary `1`.
* No light = binary `0`.
Because fiber uses light instead of electricity, it is completely immune to Electromagnetic Interference (EMI) and Crosstalk, and can transmit data at blistering speeds over massive distances without signal degradation.

## 🌈 Single-Mode vs. Multimode Fiber
* **SMF (Single-Mode Fiber):** Uses a highly focused laser to shoot a single, direct beam of light straight down the glass core. 
    * *Core size:* Extremely small (around 9 microns).
    * *Use case:* Long-haul connections (e.g., spanning across an entire city or connecting data centers miles apart). Can transmit up to 100 km without losing the signal.
    * *Color code:* Usually a **Yellow** jacket.
* **MMF (Multimode Fiber):** Uses LEDs to shoot multiple beams of light down the core simultaneously. The beams bounce off the walls of the core at different angles.
    * *Core size:* Larger (50 to 62.5 microns).
    * *Use case:* Shorter distances (e.g., inside the same building or Server Rack to Rack). Max distance is usually around 500 meters. 
    * *Color code:* Usually **Orange** or **Aqua**.

## 🔌 Connectors & Transceivers
You cannot plug a glass fiber directly into a standard RJ45 copper switch port. You need a transceiver to convert the switch's electrical signals into light pulses.
* **SFP (Small Form-factor Pluggable):** The most common transceiver. It slides into the empty modular slot on an enterprise switch. Standard SFPs support up to 1 Gbps.
* **SFP+:** The upgraded version. It is identical in physical size to an SFP, but supports 10 Gbps speeds.
* **QSFP / QSFP+ (Quad SFP):** Four optical channels combined to achieve 40 Gbps to 100 Gbps speeds. These are heavily used in modern Spine-Leaf Data Center architectures.
* **Fiber Connectors:** * **LC (Lucent Connector):** The smallest and most popular modern connector. It clicks in place like an RJ45.
    * **SC (Standard/Subscriber Connector):** A larger, square connector that pushes in and pulls out.
    * **ST (Straight Tip):** An older connector that pushes in and twists to lock (similar to a BNC connector).

## 🛡️ Security Engineering Perspective
For a Security Engineer and DevSecOps professional, transitioning from Copper to Fiber significantly hardens the physical threat model.
* **Immunity to Wiretapping:** Copper wires leak electromagnetic radiation, theoretically allowing a highly sophisticated attacker to passively "sniff" traffic without touching the cable. Fiber optics emit zero radiation. To tap a fiber line, an attacker must physically cut and splice the glass, which instantly drops the connection and alerts the SOC to a Layer 1 failure.
* **Zero EMI & DoS Resilience:** Because fiber ignores electrical interference, attackers cannot use EMPs (Electromagnetic Pulses) or localized radio jammers to cause physical-layer Denial of Service (DoS) attacks on critical data center links. 
* **The SFP Vendor Lock-In Trap:** From an operations perspective, many enterprise switches (like Cisco) are hard-coded to reject third-party SFP modules. If a security incident requires you to rapidly rebuild a compromised switch stack or replace a burnt-out transceiver, plugging an incompatible SFP will fail silently or lock the port. Knowing your exact hardware dependencies is critical for swift Incident Response.
