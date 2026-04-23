# 🧱 pfSense Deployment & Configuration

This document covers the deployment and configuration of pfSense as the central router and firewall for the home lab environment.

pfSense was used to:
- Route traffic between VLANs
- Provide DHCP services per VLAN
- Enforce firewall policies
- Control access between network segments
- Enable secure internet connectivity

This phase transformed the lab from isolated virtual machines into a fully routed, segmented, and security-focused network.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [pfSense Deployment](#pfsense-deployment)
- [Network Architecture Evolution](#network-architecture-evolution)
- [Interface Configuration](#interface-configuration)
- [VLAN Interface Setup](#vlan-interface-setup)
- [DHCP Configuration](#dhcp-configuration)
- [NAT Configuration](#nat-configuration)
- [DNS Resolver Setup](#dns-resolver-setup)
- [Connectivity Testing](#connectivity-testing)
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
| Achieve clean WAN / LAN separation | ✅ Complete |

---

## 🖥️ pfSense Deployment

pfSense was deployed as a virtual machine inside Proxmox VE, acting as the core control point for all network traffic.

| Role | Detail |
|---|---|
| Router | Inter-VLAN and internet routing |
| Firewall | Zone-based policy enforcement |
| DHCP Server | Per-VLAN address assignment |
| DNS Resolver | Internal name resolution with ACLs |
| VLAN Gateway | Default gateway for all VLAN subnets |
| Security Boundary | Hardened administrative access |

---

## 🌐 Network Architecture Evolution

### Phase 1 — Single NIC Design

The original deployment used a single physical NIC on the Proxmox host, requiring WAN and LAN traffic to share the same interface.

| Challenge | Impact |
|---|---|
| Shared WAN / LAN traffic | Routing complexity |
| VLAN-aware bridge required | More complex virtual network design |
| Management over same path | Increased lockout risk during changes |
| Single failure point | Harder to isolate faults during troubleshooting |

While functional, this design introduced instability that required rethinking the architecture.

---

### Phase 2 — Dual NIC Architecture Upgrade

A second PCIe NIC was installed in the Proxmox host, enabling proper physical separation between WAN and LAN traffic.

| Interface | Bridge | Role | Connected To |
|---|---|---|---|
| NIC 1 | `vmbr0` | WAN + Proxmox management | ISP / home router |
| NIC 2 | `vmbr1` | LAN trunk — all internal VLANs | Cisco Catalyst 2960 |

This upgrade significantly improved routing clarity, management reliability, firewall architecture, and overall troubleshooting — aligning the lab with production-style infrastructure design.

---

## 🔌 Interface Configuration

### WAN Interface

| Setting | Value |
|---|---|
| Bridge | `vmbr0` → NIC 1 |
| IP Assignment | DHCP from ISP router |
| Typical Address | `192.168.1.x` |
| Purpose | Internet access + upstream routing |

---

### LAN Interface — Trunk

| Setting | Value |
|---|---|
| Bridge | `vmbr1` → NIC 2 |
| Mode | 802.1Q trunk |
| Purpose | Parent interface for all VLAN subinterfaces |
| Layer 3 routing | Handled entirely by pfSense |

`vmbr1` carries all internal VLAN traffic to and from the Cisco switch. pfSense performs all routing and firewall enforcement across this bridge — no management IP is assigned to the bridge itself.

---

## 🔀 VLAN Interface Setup

VLAN subinterfaces were created inside pfSense on top of the LAN trunk interface, each acting as an independent routed security zone.

| VLAN | Interface Name | Gateway IP | Subnet |
|---|---|---|---|
| VLAN 10 | Admin | `192.168.10.1` | `192.168.10.0/24` |
| VLAN 20 | Lab | `192.168.20.1` | `192.168.20.0/24` |
| VLAN 30 | IoT | `192.168.30.1` | `192.168.30.0/24` |
| VLAN 40 | Users | `192.168.40.1` | `192.168.40.0/24` |
| VLAN 99 | Management | `192.168.99.1` | `192.168.99.0/24` |

---

## 📡 DHCP Configuration

DHCP was enabled individually on each VLAN interface to provide automatic IP assignment while preserving static ranges for infrastructure devices.

| VLAN | DHCP Range |
|---|---|
| VLAN 10 — Admin | `192.168.10.100` – `192.168.10.199` |
| VLAN 20 — Lab | `192.168.20.100` – `192.168.20.199` |
| VLAN 30 — IoT | `192.168.30.100` – `192.168.30.199` |
| VLAN 40 — Users | `192.168.40.100` – `192.168.40.199` |

> 📝 VLAN 99 (Management) uses static IP assignment only. Infrastructure devices require consistent, predictable addresses — DHCP on the management plane introduces unnecessary risk.

---

## 🌍 NAT Configuration

| Setting | Value |
|---|---|
| NAT Mode | Automatic Outbound NAT |
| Source | All internal VLAN subnets |
| Translation | WAN IP address |
| Purpose | Internet access for all VLANs |

Automatic Outbound NAT translates all internal RFC1918 traffic to the WAN IP, restoring full internet connectivity across all segmented networks without manual NAT rule management.

---

## 🔐 DNS Resolver Setup

| Setting | Value |
|---|---|
| Resolver | pfSense Unbound DNS |
| Scope | All VLAN interfaces |
| ACLs | Configured per VLAN |
| DNSSEC | Enabled |
| External DNS bypass | Blocked via firewall rules |

Forcing all VLANs to use the internal resolver prevents devices from bypassing security controls by pointing to external DNS servers. This became a key part of the security hardening phase.

> 📄 Full DNS and firewall policy details in [04 — Firewall Policy](04-firewall-policy.md).

---

## 🧪 Connectivity Testing

After each configuration change, the following tests were performed to validate the deployment:

| Test | Method | Expected Result |
|---|---|---|
| Gateway reachability | Ping pfSense VLAN interface IP | Response confirmed |
| Inter-VLAN routing | Ping between VLAN hosts | Controlled by firewall policy |
| Internet access | Ping `8.8.8.8` | Response from Google DNS |
| DNS resolution | Resolve `google.com` | IP returned successfully |
| DHCP lease | Connect device to VLAN | IP assigned from correct range |
| Proxmox management | Access web UI post-changes | `https://<proxmox-ip>:8006` reachable |

---

## ⚠️ Issues Encountered

### 1. No WAN IP — DHCP Failure

| Field | Detail |
|---|---|
| **Problem** | pfSense not receiving WAN IP from ISP router |
| **Cause** | Incorrect interface mapping — WAN assigned to wrong bridge |
| **Resolution** | Reassigned WAN interface to `vmbr0` and validated upstream connectivity |

---

### 2. NO-CARRIER Interface Errors

| Field | Detail |
|---|---|
| **Problem** | Interfaces showing `NO-CARRIER` — no link established |
| **Cause** | Incorrect bridge assignment; virtual NIC not mapped to correct `vmbr` |
| **Resolution** | Verified NIC attachment in Proxmox and corrected bridge mapping |

---

### 3. VLAN Routing Failure

| Field | Detail |
|---|---|
| **Problem** | Inter-VLAN traffic not routing correctly |
| **Cause** | Trunk port configured on wrong switch interface; incorrect pfSense LAN path |
| **Resolution** | Validated physical switch connections and corrected trunk placement on `Fa0/1` |

> 💡 This reinforced the importance of verifying Layer 1 physical connectivity before attempting Layer 2 or Layer 3 troubleshooting.

---

### 4. WebGUI Access Failure

| Field | Detail |
|---|---|
| **Problem** | pfSense web UI inaccessible after VLAN redesign |
| **Cause** | Browser session corruption following management access changes |
| **Resolution** | Cleared browser site data and restarted pfSense GUI services |

> 💡 This demonstrated that authentication and access failures are not always network or firewall problems — application state can mimic infrastructure issues.

---

## 📈 Result

- ✅ Stable internet access across all VLANs
- ✅ Reliable inter-VLAN routing via pfSense
- ✅ Clean WAN / LAN separation via dual NIC design
- ✅ Centralised firewall enforcement across all segments
- ✅ DHCP operational per VLAN with static ranges preserved
- ✅ DNS Resolver serving all networks with ACL enforcement
- ✅ Reduced management risk through architectural separation

pfSense moved from a functional lab router into a production-style network security platform.

---

## 💡 Lessons Learned

- **Single-NIC designs create avoidable complexity.** The dual NIC upgrade removed an entire class of routing and management problems — hardware investment pays off in architectural clarity.
- **Physical connectivity must be verified before logical troubleshooting.** Several issues that looked like routing failures traced directly back to incorrect physical port assumptions.
- **Stable management access must come before hardening.** Locking yourself out of pfSense mid-configuration is a real operational risk — always verify the management path before making network changes.
- **Browser state can mimic network failures.** A corrupted session can look exactly like a firewall or authentication problem — clear application state before assuming infrastructure is broken.
- **WAN / LAN separation dramatically improves the design.** Dedicated interfaces are not just a performance improvement — they are a security and operational boundary.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| Proxmox Setup | [01-proxmox-setup.md](01-proxmox-setup.md) |
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| Cisco Switch Config | [cisco-2960-switch-config.md](../configs/cisco-2960-switch-config.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
