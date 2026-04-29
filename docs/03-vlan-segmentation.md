# 🌐 VLAN Segmentation Design

This document covers the VLAN architecture designed and implemented across pfSense, Proxmox, and the Cisco Catalyst 2960 switch.

The goal was to move from a flat home network into a segmented, security-focused environment using enterprise-style VLAN separation, controlled inter-VLAN routing, and least-privilege firewall policy.

This became one of the most valuable parts of the project due to the real troubleshooting required across switching, routing, and virtual networking layers.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [VLAN Architecture](#vlan-architecture)
- [Role of Each VLAN](#role-of-each-vlan)
- [Network Design](#network-design)
- [Cisco Switch Configuration](#cisco-switch-configuration)
- [Proxmox Integration](#proxmox-integration)
- [pfSense Integration](#pfsense-integration)
- [Firewall Policy](#firewall-policy)
- [Design Decisions](#design-decisions)
- [Issues Encountered](#issues-encountered)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## 🎯 Design Objectives

| Objective | Status |
|---|---|
| Segment network by device type and trust level | ✅ Complete |
| Isolate IoT devices from trusted infrastructure | ✅ Complete |
| Separate management plane from user traffic | ✅ Complete |
| Enforce all inter-VLAN routing through pfSense firewall | ✅ Complete |
| Protect administrative systems and management access | ✅ Complete |
| Support secure remote access via WireGuard | ✅ Complete |
| Improve security posture and operational visibility | ✅ Complete |

---

## 🧱 VLAN Architecture

| VLAN | Name | Subnet | Purpose |
|---|---|---|---|
| **VLAN 10** | Admin | `192.168.10.0/24` | Trusted administrative devices |
| **VLAN 20** | Servers | `192.168.20.0/24` | Ubuntu VMs and internal services |
| **VLAN 30** | IoT | `192.168.30.0/24` | Untrusted smart devices |
| **VLAN 40** | Users | `192.168.40.0/24` | Daily-use client devices and Wi-Fi |
| **VLAN 99** | Management | `192.168.99.0/24` | Network device management plane |

---

## 🔍 Role of Each VLAN

### VLAN 10 — Admin

The primary operator network. Used for trusted administrative devices with full access to Proxmox, pfSense, and all infrastructure management interfaces.

| Access | Detail |
|---|---|
| pfSense GUI and SSH | Full access |
| Proxmox management | Full access |
| Infrastructure control | Full access |
| Trust level | Highest |

---

### VLAN 20 — Servers

Used for Ubuntu VMs, future self-hosted services, and internal infrastructure. Access from other VLANs is tightly controlled through firewall policy.

| Access | Detail |
|---|---|
| Internet access | Permitted for updates |
| Access from Users (VLAN 40) | HTTP/HTTPS only |
| Access from IoT (VLAN 30) | Blocked |
| Trust level | Medium — infrastructure controlled |

---

### VLAN 30 — IoT

Used for smart devices and low-trust network appliances. Intentionally isolated from all internal infrastructure.

| Access | Detail |
|---|---|
| Internet access | Permitted |
| DNS | pfSense resolver only |
| Access to internal VLANs | Fully blocked |
| Trust level | Lowest — untrusted |

> IoT devices often lack security updates and represent the highest unmanaged risk on a network. Full isolation prevents lateral movement into trusted systems.

---

### VLAN 40 — Users

Used for household devices — laptops, phones, TVs, consoles, and Wi-Fi clients connected through the EE Hub access point.

| Access | Detail |
|---|---|
| Internet access | Permitted |
| Access to Servers (VLAN 20) | HTTP/HTTPS only |
| Access to Admin / Management | Blocked |
| Trust level | Low — standard user devices |

---

### VLAN 99 — Management

Dedicated management plane for network infrastructure — switch management, future AP management, and infrastructure control.

| Access | Detail |
|---|---|
| Accessible from | VLAN 10 (Admin) only |
| Used for | Switch and infrastructure management |
| User / IoT access | Fully blocked |
| Trust level | Infrastructure only |

---

## 🔗 Network Design

### pfSense

| Role | Detail |
|---|---|
| Layer 3 router | Inter-VLAN routing |
| Firewall | Least-privilege policy enforcement |
| DHCP server | Per-VLAN address assignment |
| DNS resolver | Internal name resolution with ACLs |

All inter-VLAN communication must pass through pfSense — this ensures security policy is enforced consistently and prevents silent trust between networks.

---

### Cisco Catalyst 2960

| Role | Detail |
|---|---|
| Layer 2 switch | Physical segmentation |
| VLAN distribution | Access and trunk port management |
| Trunk point | Carries all VLANs to pfSense via single uplink |

---

### Proxmox

| Role | Detail |
|---|---|
| Virtualisation platform | Hosts pfSense and Ubuntu VMs |
| NIC 1 (`vmbr0`) | WAN path |
| NIC 2 (`vmbr1`) | LAN trunk — all internal VLANs |
| Management interface | `vmbr1.10` — tagged VLAN 10 |

---

## 🔌 Cisco Switch Configuration

### Trunk Port — Gi0/2

Carries all tagged VLAN traffic between the switch and Proxmox host (pfSense LAN).

```cisco
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk native vlan 99
```

---

### Access Port Examples

#### Gi0/1 — EE Router (VLAN 40 Access Point)

```cisco
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 40
```

#### Gi0/2 — Admin Workstation

```cisco
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 10
```

---

### Port Assignment Summary

| Port | Device | VLAN |
|---|---|---|
| `Gi0/1` | EE Router — Wi-Fi AP | VLAN 40 |
| `Gi0/2` | Proxmox / pfSense trunk | Trunk |
| `Gi0/3` | Admin workstation | VLAN 10 |
| `Fa0/3` | Ubuntu Server / Lab | VLAN 20 |

---

## 🖥️ Proxmox Integration

### vmbr1 — VLAN-Aware Trunk Bridge

```text
LAN trunk bridge
→ Cisco switch trunk
→ VLAN-aware enabled
→ carries all internal VLAN traffic
```

| Setting | Value |
|---|---|
| Bridge | `vmbr1` |
| Physical NIC | NIC 2 (`enp1s0f0`) |
| VLAN-aware | Enabled |
| bridge-vids | 10, 20, 30, 40, 99 |

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
| Management IP | `192.168.10.200/24` |
| Gateway | `192.168.10.1` |
| Web UI | `https://192.168.10.200:8006` |

Moving management to an explicit tagged interface resolved the native VLAN conflict and restored stable Proxmox GUI access.

---

## 🧱 pfSense Integration

### Parent Interface

```text
vtnet1
```

All VLAN subinterfaces created from this parent:

| Interface | VLAN | Gateway |
|---|---|---|
| `vtnet1.10` | VLAN 10 — Admin | `192.168.10.1` |
| `vtnet1.20` | VLAN 20 — Servers | `192.168.20.1` |
| `vtnet1.30` | VLAN 30 — IoT | `192.168.30.1` |
| `vtnet1.40` | VLAN 40 — Users | `192.168.40.1` |
| `vtnet1.99` | VLAN 99 — Management | `192.168.99.1` |

---

## 🔒 Firewall Policy

All inter-VLAN traffic is routed through pfSense and subject to least-privilege firewall rules.

### Access Matrix

| Source VLAN | Destination | Access |
|---|---|---|
| VLAN 10 — Admin | All VLANs | ✅ Full access |
| VLAN 20 — Servers | Internet | ✅ Permitted |
| VLAN 30 — IoT | Internet | ✅ Permitted |
| VLAN 30 — IoT | All internal VLANs | ❌ Blocked |
| VLAN 40 — Users | VLAN 20 Servers | ✅ HTTP/HTTPS only |
| VLAN 40 — Users | Admin / Management | ❌ Blocked |
| Any | pfSense GUI / SSH | ❌ Admin VLAN only |
| Default | Any | ❌ Deny all |

> 📄 Full firewall rule details in [04 — Firewall Policy](04-firewall-policy.md).

---

## 🧠 Design Decisions

### Why VLAN segmentation?

Flat networks allow unrestricted lateral communication — any compromised device can reach any other. Segmentation limits blast radius, improves control, enables policy enforcement, and mirrors real enterprise network design.

### Why centralise routing through pfSense?

Routing all inter-VLAN traffic through pfSense ensures every flow hits firewall policy. This prevents silent trust between networks and gives full visibility over internal traffic.

### Why isolate IoT?

IoT devices frequently lack security updates and represent the highest unmanaged risk on a home network. Full isolation to internet-only access prevents lateral movement into trusted infrastructure.

### Why a dedicated management VLAN?

Separating management traffic from user and data traffic protects the infrastructure control plane. Unauthorised access to management interfaces is one of the most serious risks in any network environment.

---

## ⚠️ Issues Encountered

### 1. Native VLAN Mismatch

| Field | Detail |
|---|---|
| **Problem** | Proxmox management interface unreachable after VLAN migration |
| **Cause** | Untagged management traffic conflicting with native VLAN 99 on the trunk |
| **Resolution** | Moved Proxmox management to explicit tagged interface `vmbr1.10` |

---

### 2. Wrong Physical NIC Mapping

| Field | Detail |
|---|---|
| **Problem** | pfSense VLAN gateways unreachable — no inter-VLAN routing |
| **Cause** | `vmbr1` mapped to the wrong physical NIC — not connected to the switch trunk |
| **Resolution** | Corrected bridge-to-NIC assignment, verified physical cabling |

---

### 3. FastEthernet Trunk Bottleneck

| Field | Detail |
|---|---|
| **Problem** | Poor Wi-Fi throughput and unstable VLAN routing |
| **Cause** | Critical trunk running on `Fa0/1` at 100Mbps with heavy CRC errors |
| **Resolution** | Migrated trunk link to GigabitEthernet — restored full throughput and stable performance |

---

### 4. DHCP Failures

| Field | Detail |
|---|---|
| **Problem** | Devices not receiving IP addresses on correct VLANs |
| **Cause** | VLAN path incorrect — traffic not reaching pfSense DHCP scopes |
| **Resolution** | Validated switch access port assignments and pfSense DHCP configuration per VLAN |

> 💡 **Key insight:** Physical connectivity errors frequently present as Layer 2 or Layer 3 failures. Always verify Layer 1 before diagnosing routing or firewall issues.

---

## 📈 Result

- ✅ Fully segmented network with five distinct trust zones
- ✅ All inter-VLAN routing enforced through pfSense firewall
- ✅ IoT devices isolated from all trusted infrastructure
- ✅ Management plane separated from user and data traffic
- ✅ Proxmox management stable on tagged `vmbr1.10` interface
- ✅ Trunk migrated to GigabitEthernet — full throughput restored
- ✅ Enterprise-style switching and segmentation design in place

The environment moved from basic connectivity into intentional, production-style security architecture.

---

## 💡 Lessons Learned

- **VLANs are a security control, not just an organisational tool.** The value is in the enforcement, not the labelling.
- **Layer 1 mistakes look like Layer 3 failures.** Physical port verification should always come before logical troubleshooting.
- **Native VLAN assumptions cause hard-to-diagnose lockouts.** Explicit tagged interfaces remove ambiguity entirely.
- **Stable management access must come before segmentation changes.** Losing access mid-migration is a real risk — validate the management path first.
- **Dedicated WAN/LAN separation simplifies everything downstream.** The dual NIC upgrade made VLAN troubleshooting dramatically easier.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| Proxmox Setup | [01-proxmox-setup.md](01-proxmox-setup.md) |
| pfSense Setup | [02-pfsense-setup.md](02-pfsense-setup.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| Cisco Switch Config | [cisco-2960-switch-config.md](../configs/cisco-2960-switch-config.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
