# 🖥️ Proxmox Deployment & Virtualisation Setup

This document covers the deployment of Proxmox VE as the foundation of the home lab environment.

Proxmox was deployed as a bare-metal hypervisor to host:
- pfSense (router / firewall)
- Ubuntu Server virtual machines
- Supporting infrastructure services

This phase established the compute, networking, and virtualisation platform required for all later VLAN segmentation, firewall design, and infrastructure security work.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [Hardware Overview](#hardware-overview)
- [Installation](#installation)
- [Proxmox Network Design](#proxmox-network-design)
- [Bridge Configuration](#bridge-configuration)
- [Virtual Machines](#virtual-machines)
- [Issues Encountered](#issues-encountered)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## 🎯 Design Objectives

| Objective | Status |
|---|---|
| Deploy stable hypervisor environment | ✅ Complete |
| Host multiple virtual machines | ✅ Complete |
| Enable virtual networking | ✅ Complete |
| Support firewall and routing services | ✅ Complete |
| Provide reliable management access | ✅ Complete |

---

## 🖥️ Hardware Overview

Proxmox VE was installed directly onto dedicated bare-metal hardware.

| Component | Specification |
|---|---|
| **CPU** | Intel Core i7-8700K |
| **RAM** | 8GB |
| **Storage** | 1TB SSD |
| **NIC 1** | WAN / Proxmox management access |
| **NIC 2** | LAN / VLAN trunk (added later) |

> 📝 The lab started with a single NIC. A second PCIe NIC was added later as a key architectural upgrade — see [Proxmox Network Design](#proxmox-network-design) for the full story.

---

## 💿 Installation

Proxmox VE was installed as a bare-metal hypervisor using official installation media.

### Steps

1. Boot from Proxmox VE installation USB
2. Select target storage device (1TB SSD)
3. Configure initial management IP address
4. Set root password
5. Complete installation and reboot

After installation, Proxmox was managed through the web interface:

```
https://<proxmox-ip>:8006
```

And via SSH for command-line administration:

```bash
ssh root@<proxmox-ip>
```

---

## 🌐 Proxmox Network Design

### Phase 1 — Single NIC Design

The initial deployment used a single physical NIC, which required:

| Challenge | Impact |
|---|---|
| Shared WAN / LAN traffic | Routing complexity |
| VLAN-aware bridge design | More complex configuration |
| Management over same interface | Lockout risk during changes |
| Single point of failure | Harder to isolate faults |

While functional, this design introduced troubleshooting difficulty and management instability that shaped several early decisions in the build.

---

### Phase 2 — Dual NIC Architecture Upgrade

A second PCIe NIC was added to properly separate WAN and LAN traffic.

| Interface | Role | Connected To |
|---|---|---|
| **NIC 1** | WAN + Proxmox management | ISP router |
| **NIC 2** | LAN trunk | Cisco Catalyst 2960 |

**NIC 2 carries the following VLANs via trunk:**

| VLAN | Name | Subnet |
|---|---|---|
| VLAN 10 | Admin | `192.168.10.0/24` |
| VLAN 20 | Lab | `192.168.20.0/24` |
| VLAN 30 | IoT | `192.168.30.0/24` |
| VLAN 40 | Users | `192.168.40.0/24` |
| VLAN 99 | Management | `192.168.99.0/24` |

This upgrade significantly improved routing clarity, firewall architecture, VLAN stability, and management reliability — moving the lab much closer to a production-style infrastructure design.

---

## 🔗 Bridge Configuration

Proxmox uses Linux bridges to connect virtual machines to physical network interfaces.

### vmbr0 — WAN Bridge

| Setting | Value |
|---|---|
| Connected to | NIC 1 |
| Purpose | Proxmox management + pfSense WAN |
| Proxmox management IP | `192.168.1.100` |
| Gateway | `192.168.1.1` |

Used by:
- Proxmox web UI access (`https://192.168.1.100:8006`)
- pfSense WAN interface

---

### vmbr1 — LAN Trunk Bridge

| Setting | Value |
|---|---|
| Connected to | NIC 2 |
| Purpose | pfSense LAN interface + VLAN trunking |
| Management IP | None assigned |
| Layer 3 routing | Handled entirely by pfSense |

> 📝 `vmbr1` intentionally has no management IP assigned. pfSense performs all Layer 3 routing and firewall enforcement across this bridge. Assigning an IP here would create routing conflicts and undermine security boundaries.

---

## 🖥️ Virtual Machines

### Ubuntu Server

An Ubuntu Server VM was deployed to practice Linux administration and test internal network connectivity.

| Task | Detail |
|---|---|
| SSH access | Configured and tested |
| Package management | `apt` operations practiced |
| Filesystem navigation | Directory structure and permissions |
| User management | Permissions and access control |
| Service management | Start, stop, enable, status |

---

### pfSense

pfSense was deployed as the core virtual router and firewall — the central network control point for the entire lab.

| Responsibility | Detail |
|---|---|
| WAN / LAN routing | Inter-VLAN and internet routing |
| VLAN gateway | Default gateway for all VLAN subnets |
| DHCP services | Per-VLAN address assignment |
| Firewall policy | Zone-based access control |
| DNS control | Internal resolver with ACLs |
| Administrative access | Hardened SSH and GUI access |

> 📄 Full pfSense configuration details in [02 — pfSense Setup](02-pfsense-setup.md).

---

## ⚠️ Issues Encountered

### 1. Single NIC Limitations

| Field | Detail |
|---|---|
| **Problem** | Bridge complexity, management instability, routing bottlenecks |
| **Cause** | WAN and LAN traffic sharing a single physical NIC |
| **Resolution** | Installed second PCIe NIC — redesigned architecture around dedicated WAN / LAN separation |

---

### 2. Interface and Bridge Misconfiguration

| Field | Detail |
|---|---|
| **Problem** | `NO-CARRIER` errors, incorrect bridge assignments, unreachable VMs |
| **Cause** | Virtual NIC not mapped to correct Proxmox bridge |
| **Resolution** | Rebuilt bridge mappings and verified correct NIC assignment to `vmbr0` and `vmbr1` |

---

### 3. Physical Port Confusion

| Field | Detail |
|---|---|
| **Problem** | Trunk configured on wrong switch interface — VMs unreachable |
| **Cause** | Incorrect assumptions about which physical port connected to which device |
| **Resolution** | Verified Layer 1 physical connectivity before continuing Layer 2 / 3 troubleshooting |

> 💡 This became one of the most important operational lessons of the project — **always verify Layer 1 before diagnosing Layer 2 or 3 failures.**

---

## 📈 Result

- ✅ Stable hypervisor platform running on dedicated hardware
- ✅ Reliable remote management via web UI and SSH
- ✅ Clean WAN / LAN separation via dual NIC design
- ✅ Dedicated VLAN trunking through `vmbr1`
- ✅ Production-style firewall architecture in place
- ✅ Foundation established for all later network engineering work

---

## 💡 Lessons Learned

- **Stable management access comes before perfect segmentation.** Getting locked out of Proxmox mid-configuration is a real risk — management access should be secured and tested before making network changes.
- **Layer 1 mistakes can look like Layer 3 failures.** Several issues that appeared to be routing or firewall problems traced back to incorrect physical port assumptions. Always verify the physical layer first.
- **Virtual networking must match physical topology.** Proxmox bridge assignments must accurately reflect how physical NICs connect to external hardware — mismatches cause silent failures.
- **Additional hardware simplifies architecture dramatically.** Adding a second NIC removed an entire class of complexity from the design and made the lab significantly easier to manage and troubleshoot.
- **Proper separation improves both security and maintainability.** Dedicated WAN and LAN interfaces are not just a performance improvement — they are a security boundary.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| pfSense Setup | [02-pfsense-setup.md](02-pfsense-setup.md) |
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| Cisco Switch Config | [cisco-2960-switch-config.md](../configs/cisco-2960-switch-config.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
