# 11 - Hybrid Cloud & Microservices (CCNA 200-301)
**Source:** NetworkChuck CCNA Playlist (EP 10)
**Date:** May 29, 2026

## 🧠 Core Concept: On-Premises vs. The Cloud
* **On-Premises (On-Prem):** You own, power, and manage the physical servers in your own data center. [00:01:20]
    * *CapEx (Capital Expenditure):* Requires massive upfront costs (like buying a $100k server).
    * *Pros:* Absolute control, strict compliance, low latency for local users.
* **Public Cloud:** Renting someone else's servers on demand (like AWS, Azure, GCP).
    * *OpEx (Operational Expenditure):* Paying per hour/minute of compute time. [00:02:06]
    * *Pros:* Elasticity. You can spin up thousands of servers during a traffic spike and destroy them moments later.

## 📦 Microservices, Containers, & Kubernetes
Modern application development has shifted away from large, monolithic applications.
* **Monolith vs. Microservices:** Instead of one massive app running on a single Virtual Machine (VM), the app is broken into tiny, independent services (e.g., the login service, the checkout service, the search service).
* **Containers:** These microservices run inside isolated containers. [00:03:51] Unlike VMs which require an entire heavy operating system, containers just hold the app code and its exact dependencies, allowing them to boot incredibly fast.
* **Kubernetes (K8s):** The orchestration tool that automates, networks, and manages these containers. [00:04:05]
* *Note:* While these are traditionally "Cloud-Native" tools, modern hyper-converged infrastructure allows enterprises to run Kubernetes and containers directly in their On-Prem data centers.

## 🌩️ Hybrid-Cloud & Multi-Cloud
* **Hybrid-Cloud:** Using a mix of On-Premises infrastructure (for secure/compliant data) and Public Cloud (for elastic, web-facing apps). [00:06:14]
* **Multi-Cloud:** Using multiple cloud providers simultaneously (AWS + Azure + GCP) to avoid vendor lock-in or leverage specific features from each. [00:07:51]
* *The Management Problem:* Managing multiple portals is an administrative nightmare. Solutions like VMware Cloud Foundation allow engineers to use a single pane of glass to manage infrastructure across On-Prem, AWS, and Azure seamlessly—even live-migrating virtual machines to the cloud via vMotion. [00:10:36]

## 🛡️ Security Engineering Perspective (Defending the Cloud)
Securing a Hybrid/Multi-Cloud environment completely fractures traditional network defense.
* **Loss of Visibility:** In a physical data center, a SOC Analyst can mirror a switch port and see every packet on the wire. In AWS or Azure, you do not own the underlying network. Security Engineers must rely on Cloud provider flow logs (like VPC Flow Logs) and API monitoring to detect threats.
* **Container Security:** Containers are ephemeral (temporary). If an attacker breaches a container, executes malware, and the container spins down 10 seconds later, the forensic evidence vanishes. DevSecOps engineers must scan container images for vulnerabilities during the CI/CD pipeline before they ever reach production.
* **The IAM Perimeter:** In the cloud, the firewall is no longer the primary perimeter; Identity and Access Management (IAM) is. An attacker doesn't need to break through a router if they can steal an AWS access key and legitimately ask the API to delete the entire database.
