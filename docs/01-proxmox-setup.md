# 🖥️ Proxmox Deployment & Virtualisation Setup

This document covers the deployment and redesign of Proxmox VE as the foundation of the home lab environment.

Proxmox provides the virtualisation platform for pfSense, Ubuntu virtual machines, and future self-hosted services. The host was later redesigned to support proper VLAN-aware networking and stable management access after pfSense was migrated to the edge-router role.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [Hardware Overview](#hardware-overview)
- [Installation](#installation)
- [Virtual Machines](#virtual-machines)
- [Proxmox Network Design](#proxmox-network-design)
- [Bridge Configuration](#bridge-configuration)
- [Issues Encountered](#issues-encountered)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## 🎯 Design Objectives

| Objective | Status |
|---|---|
| Deploy stable hypervisor environment | ✅ Complete |
| Host multiple virtual machines | ✅ Complete |
| Enable VLAN-aware virtual networking | ✅ Complete |
| Support firewall and routing services | ✅ Complete |
| Provide reliable management access | ✅ Complete |
| Separate WAN and LAN on dedicated NICs | ✅ Complete |

---

## 🖥️ Hardware Overview

Proxmox VE was installed directly onto dedicated bare-metal hardware.

| Component | Specification |
|---|---|
| **CPU** | Intel Core i7-8700K |
| **RAM** | 8GB |
| **Storage** | 1TB SSD |
| **NIC 1** | WAN path for pfSense |
| **NIC 2** | LAN trunk to Cisco Catalyst 2960 |

### Network Interfaces

| Interface | Bridge | Role |
|---|---|---|
| `nic0` | `vmbr0` | WAN path for pfSense |
| `enp1s0f0` | `vmbr1` | LAN trunk to Cisco switch |
| `enp1s0f1` | — | Additional testing / migration path |

> 📝 The lab started with a single NIC. A second PCIe NIC was added later as a key architectural upgrade — separating WAN and LAN traffic onto dedicated physical interfaces.

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

## 🖥️ Virtual Machines

### pfSense

Primary infrastructure VM — the central control point for the entire network.

| Responsibility | Detail |
|---|---|
| WAN / LAN routing | Inter-VLAN and internet routing |
| VLAN gateway | Default gateway for all VLAN subnets |
| DHCP services | Per-VLAN address assignment |
| Firewall policy | Zone-based access control |
| DNS control | Internal resolver with ACLs |
| WireGuard VPN | Remote administrative access |
| Edge routing | Direct PPPoE connection to Openreach ONT |

> 📄 Full pfSense configuration details in [02 — pfSense Setup](02-pfsense-setup.md).

---

### Ubuntu Server

Ubuntu Server VMs were deployed for Linux administration practice, SSH workflows, internal connectivity testing, and future service hosting.

| Task | Detail |
|---|---|
| SSH access | Configured and tested |
| Package management | `apt` operations |
| Filesystem navigation | Directory structure and permissions |
| User and permission management | `chmod`, `chown`, user groups |
| Service management | `systemctl` start, stop, enable, status |
| System recovery | Hands-on troubleshooting and repair workflows |

---

## 🌐 Proxmox Network Design

### Phase 1 — Single NIC Design

The initial deployment used a single physical NIC, which introduced the following constraints:

| Challenge | Impact |
|---|---|
| Shared WAN / LAN traffic | Routing complexity and instability |
| VLAN-aware bridge required | More complex and fragile configuration |
| Management over same interface | High lockout risk during changes |
| Single point of failure | Difficult to isolate faults |

While functional, this design introduced troubleshooting difficulty and management instability that shaped several early decisions in the build.

---

### Phase 2 — Dual NIC Architecture Upgrade

A second PCIe NIC was added to properly separate WAN and LAN traffic onto dedicated physical interfaces.

| NIC | Bridge | Role | Connected To |
|---|---|---|---|
| NIC 1 | `vmbr0` | WAN path | Openreach ONT (via pfSense PPPoE) |
| NIC 2 | `vmbr1` | LAN trunk | Cisco Catalyst 2960 |

**NIC 2 carries the following VLANs via trunk:**

| VLAN | Name | Subnet |
|---|---|---|
| **VLAN 10** | Admin | `192.168.10.0/24` |
| **VLAN 20** | Lab | `192.168.20.0/24` |
| **VLAN 30** | IoT | `192.168.30.0/24` |
| **VLAN 40** | Users | `192.168.40.0/24` |
| **VLAN 99** | Management | `192.168.99.0/24` |

> This upgrade significantly improved routing clarity, VLAN stability, and management reliability — moving the lab into a production-style dual-NIC architecture.

---

## 🔗 Bridge Configuration

Proxmox uses Linux bridges to connect virtual machines to physical network interfaces.

---

### vmbr0 — WAN Bridge

```text
WAN bridge
→ pfSense WAN only
→ no Proxmox management IP
```

| Setting | Value |
|---|---|
| Connected to | NIC 1 |
| Purpose | pfSense WAN interface only |
| Management IP | None assigned |

> 📝 `vmbr0` carries only pfSense WAN traffic. No Proxmox management IP is assigned here — management was intentionally separated to avoid routing conflicts and reduce attack surface.

---

### vmbr1 — LAN Trunk Bridge

```text
LAN trunk bridge
→ Cisco switch trunk
→ VLAN-aware
→ carries all internal VLAN traffic
```

| Setting | Value |
|---|---|
| Connected to | NIC 2 |
| Purpose | pfSense LAN trunk + VLAN transport |
| VLAN-aware | Enabled |
| Management IP | None assigned directly |
| Layer 3 routing | Handled entirely by pfSense |

> 📝 `vmbr1` intentionally has no management IP assigned. pfSense handles all Layer 3 routing and firewall enforcement across this bridge. Assigning an IP here would create routing conflicts and undermine security boundaries.

---

### vmbr1.10 — Tagged Management Interface

```text
Tagged management interface
→ VLAN 10
→ Proxmox management IP
```

| Setting | Value |
|---|---|
| Interface | `vmbr1.10` |
| VLAN | 10 (Admin) |
| Proxmox management IP | `192.168.10.200/24` |
| Gateway | `192.168.10.1` |

```text
https://192.168.10.200:8006
```

This resolved the major management access issues encountered during VLAN migration and aligned Proxmox with the production VLAN design.

---

## ⚠️ Issues Encountered

### 1. Incorrect Bridge Assignment

| Field | Detail |
|---|---|
| **Problem** | Management IP placed on WAN-facing bridge — lost GUI access and routing stability |
| **Cause** | `vmbr0` incorrectly assigned both WAN and management roles |
| **Resolution** | Separated WAN and LAN bridges correctly, moved management to tagged VLAN interface `vmbr1.10` |

---

### 2. Native VLAN Mismatch

| Field | Detail |
|---|---|
| **Problem** | Proxmox management interface unreachable after VLAN migration |
| **Cause** | Untagged management traffic conflicting with native VLAN 99 on the trunk |
| **Resolution** | Moved management to explicit tagged interface `vmbr1.10` — eliminating native VLAN dependency |

---

### 3. Wrong Physical NIC Path

| Field | Detail |
|---|---|
| **Problem** | pfSense VLAN failures, unreachable VLAN gateways |
| **Cause** | `vmbr1` mapped to the wrong physical NIC — not connected to the switch trunk |
| **Resolution** | Verified Layer 1 physical connectivity, corrected NIC-to-bridge mapping |

> 💡 **Key insight:** Layer 1 physical connectivity errors frequently present as Layer 2 or Layer 3 failures. Always verify the physical layer before diagnosing routing or firewall issues.

---

### 4. Single NIC Limitations

| Field | Detail |
|---|---|
| **Problem** | Bridge complexity, management instability, routing bottlenecks |
| **Cause** | WAN and LAN traffic sharing a single physical NIC |
| **Resolution** | Installed second PCIe NIC — redesigned architecture around dedicated WAN / LAN separation |

---

## 📈 Result

- ✅ Stable hypervisor platform running on dedicated hardware
- ✅ Reliable remote management via `https://192.168.10.200:8006`
- ✅ Clean WAN / LAN separation via dual NIC design
- ✅ VLAN-aware bridge with dedicated trunk on `vmbr1`
- ✅ Tagged management interface via `vmbr1.10`
- ✅ Production-style firewall architecture in place
- ✅ Foundation established for all later network engineering work

---

## 💡 Lessons Learned

- **Stable management access comes before perfect segmentation.** Getting locked out of Proxmox mid-configuration is a real risk — management access should be secured and tested before making network changes.
- **Layer 1 mistakes can look like Layer 3 failures.** Several issues that appeared to be routing or firewall problems traced back to incorrect physical port assumptions. Always verify the physical layer first.
- **Virtual networking must match physical topology.** Proxmox bridge assignments must accurately reflect how physical NICs connect to external hardware — mismatches cause silent failures.
- **Additional hardware simplifies architecture dramatically.** Adding a second NIC removed an entire class of complexity and made the lab significantly easier to manage and troubleshoot.
- **Native VLAN assumptions are dangerous.** Explicit tagged interfaces remove ambiguity and prevent hard-to-diagnose management lockouts.

---

## 🗺️ Future Improvements

- [ ] Backup automation and snapshot scheduling
- [ ] Monitoring and alerting integration
- [ ] Additional service VMs
- [ ] Infrastructure documentation diagrams
- [ ] Snapshot and recovery strategy improvements

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
