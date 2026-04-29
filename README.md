# 🖥️ Home Lab Network Infrastructure

### Proxmox • pfSense • Cisco Catalyst 2960 • VLAN Segmentation • WireGuard VPN • Network Security

![Status](https://img.shields.io/badge/Status-Active-green?style=flat-square)
![Type](https://img.shields.io/badge/Type-Home%20Lab-blue?style=flat-square)
![Level](https://img.shields.io/badge/Level-CCNA-orange?style=flat-square)

*A self-hosted enterprise-style home lab built to develop practical network engineering skills using real hardware and software — not simulations.*

---

## 📋 Overview

Built using **Proxmox VE**, **pfSense**, and a **Cisco Catalyst 2960**, this lab applies CCNA-level networking concepts in a live production-style environment. The focus is on VLAN segmentation, firewall policy enforcement, management isolation, WireGuard remote access, and security hardening.

pfSense operates directly on the Openreach ONT, replacing the ISP double-NAT architecture and acting as the true network edge.

---

## 🛠️ Technologies Used

| Category | Tools |
| --- | --- |
| **Virtualisation** | Proxmox VE |
| **Firewall / Router** | pfSense Community Edition |
| **Switching** | Cisco Catalyst 2960 |
| **OS** | Ubuntu Server |
| **Remote Access** | WireGuard VPN |
| **Protocols** | VLANs (802.1Q), SSH, DHCP, NAT, DNS, PPPoE |
| **Security** | Zone-based firewall, DNSSEC, SSH key auth, ACLs |

---

## 🏗️ Network Architecture

```text
Openreach ONT
→ pfSense (Edge Router / Firewall / DHCP / DNS / VPN)
→ Cisco Catalyst 2960 (Managed Switch / VLAN Trunk)
→ VLAN Segmentation
→ Proxmox VE Host → Ubuntu VMs / Services
→ EE Router (Access Point — VLAN 40 Wi-Fi only)
→ User Devices
```

pfSense connects directly to the Openreach ONT via **PPPoE**, eliminating the ISP router from the routing path entirely and resolving the previous double-NAT issues.

### 📐 Network Diagram

<!-- Replace with your diagram image link -->
![Network Diagram](docs/diagrams/network-diagram.png)

---

## 🌐 VLAN Design

| VLAN | Purpose | Subnet |
| --- | --- | --- |
| **VLAN 10** | Admin Devices | `192.168.10.0/24` |
| **VLAN 20** | Servers / Lab | `192.168.20.0/24` |
| **VLAN 30** | IoT Devices | `192.168.30.0/24` |
| **VLAN 40** | Users / Wi-Fi Clients | `192.168.40.0/24` |
| **VLAN 99** | Management Network | `192.168.99.0/24` |

---

## 🔒 Security Design

### Firewall Policy

- Least-privilege inter-VLAN access rules
- IoT and user devices isolated from admin and server networks
- Users permitted HTTP/HTTPS to servers only
- pfSense GUI and SSH locked to VLAN 99 (management) only
- WAN exposure reviewed and unnecessary services disabled

### DNS Hardening

- DNS Resolver ACLs restricting query sources
- DNSSEC enabled
- DNS traffic enforcement per VLAN

### SSH Hardening

- Public key authentication only — password login disabled
- Management access restricted to trusted VLAN

### Zone-Based Firewall Policy

<!-- Replace with your diagram image link -->
![Firewall Zone Diagram](docs/diagrams/zone-firewall.png)

---

## 🔐 WireGuard VPN

Deployed WireGuard for secure remote access to the lab:

- pfSense-hosted WireGuard server
- Remote peer configured on MacBook
- Access scoped to management VLAN and infrastructure only
- Validated externally using mobile hotspot to confirm true remote access
- Required resolving PPPoE edge routing, NAT, and ISP router conflicts before functioning correctly

---

## 🖥️ Linux & Server Administration

Hands-on Ubuntu Server administration carried out throughout this project:

- CLI navigation, file permissions, filesystem management
- Package management with `apt`
- Service management with `systemctl` and `journalctl`
- SSH key generation and hardened remote access
- Proxmox VM and LXC container deployment
- Virtual bridge and network interface configuration
- Snapshot and backup management
- Real-world Windows recovery using SFC and CHKDSK

---

## 🔧 Troubleshooting Experience

The most valuable part of this project. Every issue below was diagnosed and resolved without external assistance.

| Issue | Layer | Area |
| --- | --- | --- |
| DHCP failures — missing WAN lease | L3 | WAN / ISP connectivity |
| `NO-CARRIER` interface failures | L1 | Physical / virtual NIC |
| Proxmox bridge misconfiguration | L2 | Virtualisation networking |
| Double-NAT caused by ISP router | L3 | Routing architecture |
| Failed WireGuard handshakes | L3/L4 | VPN / NAT traversal |
| VLAN tagging and trunk/native VLAN mismatches | L2 | Switch configuration |
| Proxmox management loss during VLAN migration | L2/L3 | Bridge and VLAN config |
| Incorrect bridge-to-NIC assignments | L1/L2 | Proxmox networking |
| Switch port CRC errors | L1 | Physical cabling |
| FastEthernet trunk bottleneck | L1 | Interface speed limitation |
| Trunk link migrated to Gigabit interfaces | L1 | Hardware upgrade |

---

## 🚧 Current Focus & Roadmap

- [ ] Dynamic DNS deployment
- [ ] Monitoring stack — Grafana / Zabbix / LibreNMS
- [ ] Centralised logging / Syslog
- [ ] IDS/IPS — Suricata or Snort
- [ ] Self-hosted internal services
- [ ] Backup automation
- [ ] Formal infrastructure diagrams

---

## 📁 Documentation

| Document | Description |
| --- | --- |
| [01 — Proxmox Setup](docs/01-proxmox-setup.md) | Hypervisor installation and network bridge configuration |
| [02 — pfSense Setup](docs/02-pfsense-setup.md) | Firewall and router initial configuration |
| [03 — VLAN Segmentation](docs/03-vlan-segmentation.md) | 802.1Q trunk setup and VLAN design |
| [04 — Firewall Policy](docs/04-firewall-policy.md) | Zone-based firewall rules and inter-VLAN policy |
| [05 — WireGuard VPN](docs/05-wireguard-vpn.md) | Remote access deployment and troubleshooting |
| [06 — SSH Hardening](docs/06-ssh-hardening.md) | Public key authentication and access controls |
| [07 — Lessons Learned](docs/07-lessons-learned.md) | What broke, how I fixed it, and what I'd do differently |

---

## 💡 Why This Project

This lab is designed to turn theory into **practical engineering experience** by building and troubleshooting real infrastructure.

Rather than learning concepts like VLANs, routing, and segmentation in theory only, this project demonstrates them in a **live environment** — with real operational failures, security constraints, and recovery decisions.

The skills developed here are directly applicable to:

- Network Engineering
- Infrastructure / Systems Administration
- Datacenter Operations
- NOC / Cloud Infrastructure roles

---

Built and maintained by Gustavo | Ongoing development
