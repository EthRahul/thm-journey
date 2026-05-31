# 13 - Power over Ethernet (PoE) (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 12)
**Date:** May 31, 2026

## 🧠 Core Concept: What is PoE?
Power over Ethernet (PoE) allows a single Ethernet cable to provide both data connection and electrical power to devices. [00:00:09] This eliminates the need to hire electricians to run dedicated power lines to inconvenient places (like ceilings for Wi-Fi access points or security cameras).
* **PSE (Power Sourcing Equipment):** The device providing the power (typically a PoE-enabled network switch). [00:01:44]
* **PD (Powered Device):** The end device receiving the power (e.g., IP phone, camera, or wireless access point). [00:02:14]

## ⚡ PoE Standards and Types
As the demand for powering larger devices grew, the IEEE introduced higher-wattage standards.
* **802.3af (Type 1 / PoE):** Up to 15.4 Watts. [00:03:07] Used for basic IP phones and simple devices.
* **802.3at (Type 2 / PoE+):** Up to 30 Watts. [00:05:34] The most common standard today, used for advanced phones and standard access points.
* **802.3bt (Type 3 / UPOE / 4PPoE):** Up to 60 Watts. [00:06:50] Achieves this by sending power over all 4 twisted pairs in the cable. Can power smaller network switches.
* **802.3bt (Type 4 / PoE++):** Up to 90 Watts. [00:07:54] Capable of powering laptops via USB-C, smart building LED lighting, and even small HVAC systems.

## 🤝 Active vs. Passive PoE
* **Active PoE:** The switch intelligently negotiates with the device before sending power. [00:09:03] It uses protocols like **CDP** (Cisco Discovery Protocol) or **LLDP** (Link Layer Discovery Protocol) to ask the device how much wattage it needs. [00:09:42] If a non-PoE device (like a standard laptop) is plugged in, the switch safely provides zero power.
* **Passive PoE:** "Always on" power (often 24V). It does not negotiate. If you accidentally plug a standard non-PoE device into a passive port, the power can fry the device's network card.

## 💻 Cisco Switch Verification
You can verify the power draw on a Cisco switch using the command: `show power inline`. [00:14:40]
* **Power Budgets:** A switch does not have infinite power. A 24-port switch might have a total budget of 720 Watts. [00:16:14] If you plug in too many high-draw devices, the switch will run out of power and refuse to boot any new devices.
* **Err-Disabled State:** If a connected device attempts to maliciously or accidentally draw more power than the port is configured to allocate, the switch will shut the port down into an `err-disabled` state and generate a syslog alert.

## 🛡️ Security Engineering Perspective
PoE fundamentally alters the physical threat landscape for a SOC Analyst and Security Engineer:
* **The "Drop-and-Go" Rogue Device:** Before PoE, if an attacker wanted to hide a rogue Raspberry Pi or a malicious Wi-Fi pineapple in a corporate building, they had to find a hidden wall outlet for power. PoE allows them to simply plug their hardware into any active Ethernet jack (like behind a printer or above a drop ceiling), and the switch will power their attack box indefinitely.
* **Power-Budget DoS Attacks:** Because switches share a global power budget, an attacker could theoretically plug in malicious devices that intentionally draw maximum wattage, starving out legitimate critical infrastructure (like the building's security cameras or badge readers) that share the same switch.
* **Defense Strategy:** This makes **Layer 2 Port Security (802.1X)** and **MAC Filtering** absolutely mandatory. A port should never be left active or supplying power in a lobby or conference room unless the connecting device properly authenticates to the network.
