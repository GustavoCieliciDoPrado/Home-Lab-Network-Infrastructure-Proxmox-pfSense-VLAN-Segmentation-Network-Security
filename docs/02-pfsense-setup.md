# 🧱 pfSense Deployment & Configuration

This document covers the deployment and configuration of pfSense as the central router and firewall for the home lab environment.

pfSense was used to:
- Route traffic between VLANs
- Provide DHCP services per VLAN
- Enforce firewall policies
- Control access between network segments
- Enable secure internet connectivity

This phase transformed the lab from isolated virtual machines into a fully routed and controlled network.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [pfSense Deployment](#pfsense-deployment)
- [Interface Configuration](#interface-configuration)
- [VLAN Interface Setup](#vlan-interface-setup)
- [DHCP Configuration](#dhcp-configuration)
- [NAT Configuration](#nat-configuration)
- [DNS Resolver Setup](#dns-resolver-setup)
- [Initial Connectivity Testing](#initial-connectivity-testing)
- [Issues Encountered](#issues-encountered)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## 🎯 Design Objectives

| Objective | Status |
|---|---|
| Provide internet access to all VLANs | ✅ Complete |
| Enable inter-VLAN routing | ✅ Complete |
| Centralise firewall control | ✅ Complete |
| Implement DHCP services per VLAN | ✅ Complete |
| Prepare for segmentation and security hardening | ✅ Complete |

---

## 🖥️ pfSense Deployment

pfSense was deployed as a virtual machine within Proxmox VE.

| Setting | Value |
|---|---|
| Hypervisor | Proxmox VE |
| VM Type | Virtual Router / Firewall |
| Edition | pfSense Community Edition |
| WAN Interface | Connected to ISP router via DHCP |
| LAN Interface | Trunk link to Cisco Catalyst 2960 |

> 📝 pfSense runs as a VM rather than on dedicated hardware. This introduced single-NIC constraints that shaped several design decisions throughout the build — see [Issues Encountered](#issues-encountered) for details.

---

## 🌐 Interface Configuration

### WAN Interface

| Setting | Value |
|---|---|
| Connection | ISP router |
| IP Assignment | DHCP |
| Purpose | Internet access for all internal VLANs via NAT |

### LAN Interface — Trunk

| Setting | Value |
|---|---|
| Connection | Cisco Catalyst 2960 via `Fa0/1` |
| Mode | 802.1Q trunk |
| VLANs Carried | 10, 20, 30, 40, 99 |
| Purpose | Layer 3 gateway for all VLAN subnets |

The LAN interface carries all VLANs over a single trunk, allowing pfSense to act as the default gateway for every network segment from a single physical interface.

---

## 🔀 VLAN Interface Setup

VLAN subinterfaces were created within pfSense to enable routing between segmented networks. Each interface acts as the default gateway for its subnet.

| VLAN | Interface Name | Gateway IP | Subnet |
|---|---|---|---|
| VLAN 10 | Admin | `192.168.10.1` | `192.168.10.0/24` |
| VLAN 20 | Lab | `192.168.20.1` | `192.168.20.0/24` |
| VLAN 30 | IoT | `192.168.30.1` | `192.168.30.0/24` |
| VLAN 40 | Users | `192.168.40.1` | `192.168.40.0/24` |
| VLAN 99 | Management | `192.168.99.1` | `192.168.99.0/24` |

---

## 📡 DHCP Configuration

DHCP was configured on each VLAN interface to automatically assign IP addresses to connected devices.

| VLAN | DHCP Range |
|---|---|
| VLAN 10 — Admin | `192.168.10.100` – `192.168.10.200` |
| VLAN 20 — Lab | `192.168.20.100` – `192.168.20.200` |
| VLAN 30 — IoT | `192.168.30.100` – `192.168.30.200` |
| VLAN 40 — Users | `192.168.40.100` – `192.168.40.200` |

> 📝 VLAN 99 (Management) does not use DHCP. Management devices use static IPs to ensure consistent, predictable access to infrastructure.

---

## 🌍 NAT Configuration

pfSense was configured with **Automatic Outbound NAT**, translating internal VLAN traffic to the WAN IP address for internet access.

| Setting | Value |
|---|---|
| NAT Mode | Automatic Outbound NAT |
| Source | All internal VLAN subnets |
| Translation | WAN IP address |
| Purpose | Internet access for all VLANs |

This allows devices across all VLANs to reach the internet while remaining on private RFC1918 address space internally.

---

## 🔐 DNS Resolver Setup

pfSense DNS Resolver was configured to provide internal name resolution across all VLANs.

| Setting | Value |
|---|---|
| Resolver | pfSense Unbound DNS |
| Scope | All VLAN interfaces |
| ACLs | Configured per VLAN — restricted access |
| DNSSEC | Enabled |
| External DNS bypass | Blocked via firewall rules |

Restricting DNS to the internal resolver prevents devices from bypassing security controls by pointing to external resolvers. This became a key part of the security hardening phase.

> 📄 Full DNS hardening details covered in [06 — SSH Hardening](06-ssh-hardening.md) and [04 — Firewall Policy](04-firewall-policy.md).

---

## 🧪 Initial Connectivity Testing

After setup, the following tests were performed to verify the deployment:

| Test | Method | Expected Result |
|---|---|---|
| Inter-VLAN connectivity | Ping between VLAN hosts | Response from gateway |
| Gateway reachability | Ping pfSense VLAN interface IP | Response confirmed |
| Internet access | Ping `8.8.8.8` | Response from Google DNS |
| DNS resolution | Resolve `google.com` | IP returned successfully |

---

## ⚠️ Issues Encountered

### 1. No WAN IP — DHCP Failure

| Field | Detail |
|---|---|
| **Problem** | pfSense was not receiving a WAN IP from the ISP router |
| **Cause** | Incorrect WAN interface assignment in pfSense |
| **Resolution** | Verified physical connection, reassigned WAN interface, restarted DHCP request |

### 2. NO-CARRIER Interface Errors

| Field | Detail |
|---|---|
| **Problem** | Interfaces displayed `NO-CARRIER` — no link established |
| **Cause** | Incorrect Proxmox bridge assignment; virtual NIC not mapped to correct bridge |
| **Resolution** | Verified Proxmox bridge configuration and reattached interfaces correctly |

### 3. No Internet Access from VLANs

| Field | Detail |
|---|---|
| **Problem** | Devices could reach pfSense gateway but not the internet |
| **Cause** | Missing NAT configuration; firewall rules blocking outbound traffic |
| **Resolution** | Verified NAT settings, adjusted outbound firewall rules, confirmed gateway config |

### 4. Single NIC Bottleneck

| Field | Detail |
|---|---|
| **Problem** | Performance issues and routing complexity with shared NIC |
| **Cause** | pfSense WAN and LAN traffic sharing a single physical NIC via VLAN trunking |
| **Resolution** | Redesigned traffic flow using VLAN trunking; dedicated second NIC planned as future improvement |

---

## 📈 Result

- ✅ Fully functional routed network across all VLANs
- ✅ Internet access operational via NAT
- ✅ Centralised routing and firewall control through pfSense
- ✅ DHCP operational across all VLAN segments
- ✅ DNS Resolver serving all internal networks
- ✅ Foundation in place for VLAN segmentation and firewall policy enforcement

---

## 💡 Lessons Learned

- **Interface assignment is critical.** Misassigning WAN or LAN interfaces in pfSense causes failures that are difficult to diagnose without methodical verification.
- **Virtual networking must match physical topology.** Proxmox bridge configuration must accurately reflect how physical and virtual interfaces connect — mismatches cause `NO-CARRIER` errors that look like hardware failures.
- **DHCP failures are often upstream issues.** When pfSense doesn't receive a WAN IP, the problem is usually the interface assignment or the upstream connection, not pfSense itself.
- **NAT is essential for internet access.** Without correct outbound NAT, internal devices can reach the gateway but go no further — a common and easily missed misconfiguration.
- **Single NIC designs introduce real complexity.** Running WAN and LAN on the same physical interface works but creates bottlenecks and makes troubleshooting harder. Dedicated interfaces are always preferable.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| SSH Hardening | [06-ssh-hardening.md](06-ssh-hardening.md) |
| Cisco Switch Config | [cisco-2960-switch-config.md](../configs/cisco-2960-switch-config.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
