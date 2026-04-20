<div align="center">

# 🖥️ Home Lab Network Infrastructure

### Proxmox • pfSense • VLAN Segmentation • Network Security

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=flat-square)
![Type](https://img.shields.io/badge/Type-Home%20Lab-blue?style=flat-square)
![Level](https://img.shields.io/badge/Level-CCNA-orange?style=flat-square)

*A self-hosted enterprise-style home lab built to develop practical network engineering skills using real hardware and software — not simulations.*

</div>

---

## 📋 Overview

Built using **Proxmox**, **pfSense**, and a **Cisco Catalyst 2960**, this lab applies CCNA-level networking concepts in a live production-style environment. The focus is on VLAN segmentation, firewall policy enforcement, management isolation, and network security hardening.

---

## 🛠️ Technologies Used

| Category | Tools |
|---|---|
| **Virtualisation** | Proxmox VE |
| **Firewall / Router** | pfSense Community Edition |
| **Switching** | Cisco Catalyst 2960 |
| **OS** | Ubuntu Server |
| **Protocols** | VLANs (802.1Q), SSH, DHCP, NAT, DNS |
| **Security** | Firewall Rules, DNS Resolver, Linux CLI Administration |

---

## 🏗️ Network Architecture

pfSense operates as the **central router and firewall**, connected to the Cisco Catalyst 2960 via an **802.1Q trunk** for VLAN segmentation.

Traffic is separated into dedicated VLANs to:
- Enforce security boundaries
- Reduce attack surface
- Isolate trusted devices from user, IoT, and management networks

### 📐 Network Diagram

<img width="607" height="1040" alt="VLAN drawio" src="https://github.com/user-attachments/assets/515cf71a-0d31-4295-be38-7bb246a0dab0" />

---

## 🌐 VLAN Design

| VLAN | Purpose | Subnet |
|------|---------|--------|
| **VLAN 10** | Admin Devices | `192.168.10.0/24` |
| **VLAN 20** | Servers / Lab | `192.168.20.0/24` |
| **VLAN 30** | IoT Devices | `192.168.30.0/24` |
| **VLAN 40** | Users / Wi-Fi Clients | `192.168.40.0/24` |
| **VLAN 99** | Management Network | `192.168.99.0/24` |

---

## 🔒 Security Design

### Implemented security controls:

- 🔐 Inter-VLAN isolation using least-privilege firewall rules
- 🔐 Restricted pfSense GUI and SSH access
- 🔐 Management-plane isolation using VLAN 99
- 🔐 DNS Resolver access control lists
- 🔐 DNSSEC enabled
- 🔐 SSH key-based authentication (Public Key Only)
- 🔐 Reduced unnecessary service exposure

### Zone-Based Firewall Policy

<img width="651" height="796" alt="ZB-Firewall drawio" src="https://github.com/user-attachments/assets/32f4ee73-2e1d-4fca-abc9-573e565108ee" />

---

## 🔧 Troubleshooting Experience

### Real-world issues solved during the build:

| Issue | Area |
|-------|------|
| DHCP failures — missing WAN lease | WAN / ISP connectivity |
| Interface link failures — `NO-CARRIER` | Physical / virtual NIC |
| Proxmox bridge misconfiguration | Virtualisation layer |
| Single-NIC routing bottlenecks | Hardware constraints |
| Firewall rule order conflicts | pfSense policy |
| VLAN management access restrictions | Switch / trunk config |

---

## 🚧 Current Focus & Roadmap

- [ ] Final pfSense hardening
- [ ] VPN deployment (WireGuard / OpenVPN)
- [ ] IDS/IPS deployment (Suricata / Snort)
- [ ] Monitoring and logging
- [ ] Dedicated second NIC implementation
- [ ] Network diagrams *(in progress)*

---

## 💡 Why This Project

This lab is designed to turn theory into **practical engineering experience** by building and troubleshooting real infrastructure.

Rather than only learning concepts like VLANs, routing, and segmentation in theory, this project demonstrates them in a **live environment** with real operational challenges and security considerations.

---

<div align="center">
<sub>Built and maintained by Gustavo | Ongoing development</sub>
</div>
