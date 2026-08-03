# 🔐 ZTUTP Solutions — Zero Trust Wi-Fi Network Implementation

> **Academic case study:** design and implementation of a **Zero Trust** architecture for a corporate WLAN, developed in three phases — from a theoretical attack-surface analysis to a functional lab with **pfSense + Cloudflare Zero Trust**.

**Technological University of Panama** · Faculty of Computer Systems Engineering · B.S. in Cybersecurity · Cybersecurity III Course

---

## 📋 Table of Contents

- [Project Summary](#-project-summary)
- [Part I — Analysis and Conceptual Prototype](#part-i--analysis-and-conceptual-prototype)
- [Part II — Packet Tracer Prototype](#part-ii--packet-tracer-prototype)
- [Part III — Real Lab (pfSense + Cloudflare)](#part-iii--real-lab-pfsense--cloudflare)
- [Test Evidence](#-test-evidence)
- [Tool Stack](#-tool-stack)
- [NIST SP 800-207 Principles Applied](#-nist-sp-800-207-principles-applied)
- [Conclusions](#-conclusions)
- [References](#-references)

---

## 🎯 Project Summary

The case study simulates **ZTUTP Solutions**, a fictitious cybersecurity company tasked with modernizing the infrastructure of a client with **500 employees** and an attack surface exposed by BYOD devices and Wi-Fi access with no identity control.

The goal was to eliminate the **implicit trust** model and replace it with continuous verification, applying **NIST SP 800-207** principles, across three incremental stages:

| Part | Focus | Deliverable |
|---|---|---|
| **I** | Attack surface analysis + conceptual design | High-level Zero Trust architecture |
| **II** | Simulated prototype | Functional topology in **Cisco Packet Tracer** (802.1X, RADIUS, ACLs) |
| **III** | Real implementation | Lab with **pfSense**, **Docker**, **Cloudflare Tunnel**, and **Cloudflare Access** |

---

## Part I — Analysis and Conceptual Prototype

Three critical exposure areas were identified: financial servers, BYOD devices, and Wi-Fi access points. From there, the attack lifecycle was modeled (discovery → infiltration → lateral movement), and a reference architecture was designed with **ZTNA Gateway**, **IAM**, **MFA**, and **MDM/EDR** as the control plane.

<p align="center">
  <img src="screenshots/02_pagina8.png" width="600" alt="Proposed Zero Trust solution architecture">
  <br><em>Proposed architecture: internal flow (BYOD) and external flow (untrusted client)</em>
</p>

<p align="center">
  <img src="screenshots/03_pagina9.png" width="600" alt="Zero Trust access flow with a BYOD device">
  <br><em>Step-by-step Zero Trust access flow for a BYOD device</em>
</p>

---

## Part II — Packet Tracer Prototype

An 802.1X WLAN was built with **RADIUS/AAA** authentication, VLAN segmentation, and **microsegmentation ACLs** that block lateral movement between the BYOD segment, the servers, and the control plane.

<p align="center">
  <img src="screenshots/05_pagina11.png" width="500" alt="Zero Trust Road Map applied to the case study">
  <br><em>4-phase implementation roadmap (visibility → identity → NAC → microsegmentation)</em>
</p>

<p align="center">
  <img src="screenshots/07_pagina15.png" width="600" alt="Zero Trust WLAN prototype topology in Packet Tracer">
  <br><em>Final Packet Tracer topology: Core Router, Core Switch, WLC, AP, RADIUS server, and VLAN-segmented servers</em>
</p>

**Segmentation test results:**

| Scenario | Source → Destination | Result |
|---|---|---|
| A | BYOD Laptop → Router (VLAN gateway) | ✅ Successful ping |
| B | BYOD Laptop → Financial Server | ⛔ `Destination host unreachable` |
| C | BYOD Laptop → RADIUS Server (control plane) | ⛔ `Destination host unreachable` |

---

## Part III — Real Lab (pfSense + Cloudflare)

The final phase brought the concept into a real, functional environment, **with no DMZ**, demonstrating that in Zero Trust, protection depends on **identity and policy**, not on where the resource sits on the network.

<p align="center">
  <img src="screenshots/08_pagina24.png" width="600" alt="Real Zero Trust lab topology">
  <br><em>Simplified topology: pfSense as a deny-by-default firewall and Ubuntu Server with Docker (Nginx + cloudflared)</em>
</p>

<p align="center">
  <img src="screenshots/09_pagina25.png" width="600" alt="Detailed lab topology in PNETLab">
  <br><em>Detailed lab diagram in PNETLab: deny-by-default rules and network summary</em>
</p>

<p align="center">
  <img src="screenshots/10_pagina28.png" width="600" alt="Cloudflare Zero Trust panel">
  <br><em>"Get started with Zero Trust" panel in Cloudflare, the starting point for the ZTNA layer</em>
</p>

<p align="center">
  <img src="screenshots/11_pagina29.png" width="400" alt="Creating a tunnel in Cloudflare">
  <br><em>Creating the Cloudflare Tunnel used to publish the server without opening any ports</em>
</p>

---

## ✅ Test Evidence

Six tests validated the **deny-by-default** behavior of the real lab:

<p align="center">
  <img src="screenshots/12_pagina30.png" width="450" alt="OTP code received by email">
  <br><em>Test 1 — Authorized access: OTP code sent by Cloudflare Access</em>
</p>

<p align="center">
  <img src="screenshots/13_pagina30.png" width="450" alt="Portal accessed after authentication">
  <br><em>Test 1 — Portal visible only after passing identity verification</em>
</p>

<p align="center">
  <img src="screenshots/14_pagina31.png" width="450" alt="Access denied by Cloudflare Access">
  <br><em>Test 2 — Unauthorized access: Cloudflare Access denies the login</em>
</p>

<p align="center">
  <img src="screenshots/15_pagina32.png" width="450" alt="Ping blocked to the server">
  <br><em>Test 4 — Lateral movement blocked: ping from the LAN to the server (100% lost)</em>
</p>

<p align="center">
  <img src="screenshots/16_pagina33.png" width="450" alt="Failed RDP connection">
  <br><em>Test 4 — Remote Desktop (RDP) attempt rejected by pfSense's deny-by-default rules</em>
</p>

| # | Test | Expected Result |
|---|---|---|
| 1 | Authorized access with OTP | ✅ Access granted |
| 2 | Unauthorized access | ⛔ Access Denied |
| 3 | Direct access to the server (nmap) | ⛔ Filtered/closed ports |
| 4 | Lateral movement (ping/RDP/SSH) | ⛔ Blocked by pfSense |
| 5 | Authorized service only (curl) | ✅ Responds only via the Cloudflare tunnel |
| 6 | Session revocation and expiration | ⛔ Re-authentication required |

---

## 🛠 Tool Stack

| Component | Tool | Function |
|---|---|---|
| Firewall | pfSense CE 2.6.0 | Segmentation, deny-by-default, NAT |
| Web Server | Nginx (Docker) | Protected internal application |
| Zero Trust Tunnel | cloudflared (Docker) | Encrypted outbound connection, no open ports |
| Access Control | Cloudflare Access | Identity-based authentication + OTP |
| IAM (prototype) | Keycloak | Centralized identity management |
| MFA | Google Authenticator / privacyIDEA | Multi-factor verification |
| MDM/EDR | OpenEDR / Wazuh | Device posture monitoring |
| Microsegmentation (real) | OPNsense / pfSense | Segment isolation |
| OS / containers | Ubuntu Server 22.04 + Docker Compose | Service host |

---

## 📐 NIST SP 800-207 Principles Applied

- **Deny-by-Default** — all traffic is blocked unless there's an explicit rule or policy.
- **Identity-based access with MFA** — mandatory authentication before any access.
- **Least privilege** — the tunnel exposes only the specific authorized resource, never the whole network.
- **Continuous verification and session expiration** — 15-minute sessions with immediate revocation.

---

## 🧾 Conclusions

The lab demonstrated that a complete Zero Trust architecture can be implemented with **free and open-source tools**, at a total cost of roughly **$1 USD/year** (the domain only). The intentional removal of the DMZ confirmed the model's core principle: a resource's location on the network does not determine its trust level — protection depends on verified identity and the policy applied to each request.

---

## 📚 References

- Rose, S., Borchert, O., Mitchell, S., & Connelly, S. (2020). *Zero Trust Architecture*. NIST SP 800-207. https://csrc.nist.gov/pubs/sp/800/207/final
- Cloudflare, Inc. (2026). *Cloudflare Zero Trust Documentation*. https://developers.cloudflare.com/cloudflare-one/
- Netgate. (2026). *pfSense Documentation*. https://docs.netgate.com/pfsense/en/latest/
- Mazebolt. (2025). *Zero trust model security challenges*. https://mazebolt.com/blog/zero-trust-security-challenges
- CISA. (2023). *Zero Trust Maturity Model*. https://www.cisa.gov/zero-trust-maturity-model

---

<p align="center"><em>Academic project — Cybersecurity III, Technological University of Panama.</em></p>