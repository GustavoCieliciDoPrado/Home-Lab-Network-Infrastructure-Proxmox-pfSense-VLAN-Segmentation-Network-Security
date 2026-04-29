# 🧱 pfSense Deployment & Configuration

This document covers the deployment, migration, and hardening of pfSense as the core network appliance for the home lab environment.

pfSense evolved from an internal virtual firewall behind the ISP router into the primary edge router for the entire environment, handling WAN connectivity, routing, VLAN segmentation, DHCP, firewall policy, DNS, and WireGuard remote access.

This migration removed double-NAT, improved stability, and enabled proper enterprise-style network design.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [pfSense Deployment](#pfsense-deployment)
- [Network Architecture Evolution](#network-architecture-evolution)
- [WAN Migration](#wan-migration)
- [Interface Configuration](#interface-configuration)
- [VLAN Interface Setup](#vlan-interface-setup)
- [DHCP Configuration](#dhcp-configuration)
- [NAT Configuration](#nat-configuration)
- [Firewall Policy](#firewall-policy)
- [DNS Resolver Setup](#dns-resolver-setup)
- [SSH Hardening](#ssh-hardening)
- [EE Router Conversion](#ee-router-conversion)
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
| Migrate pfSense to true edge router | ✅ Complete |
| Eliminate ISP double-NAT | ✅ Complete |
| Deploy WireGuard remote access VPN | ✅ Complete |
| Harden DNS, SSH, and management access | ✅ Complete |

---

## 🖥️ pfSense Deployment

pfSense was deployed as a virtual machine inside Proxmox VE, acting as the central control point for all network traffic.

| Role | Detail |
|---|---|
| Router | Inter-VLAN and internet routing |
| Firewall | Zone-based policy enforcement |
| DHCP Server | Per-VLAN address assignment |
| DNS Resolver | Internal name resolution with ACLs |
| VLAN Gateway | Default gateway for all VLAN subnets |
| WireGuard VPN | Secure remote administrative access |
| Edge Router | Direct PPPoE connection to Openreach ONT |

---

## 🌐 Network Architecture Evolution

### Phase 1 — Internal Firewall Behind ISP Router

The original deployment placed pfSense behind the EE router, creating the following constraints:

| Challenge | Impact |
|---|---|
| Double-NAT | Broken WireGuard handshakes, unreliable port forwarding |
| ISP router limitations | Reduced routing control and visibility |
| Shared WAN / LAN NIC | Routing complexity on Proxmox host |
| Management over shared path | Increased lockout risk during changes |

While functional for basic lab work, this design prevented proper VPN deployment and introduced unnecessary routing complexity.

---

### Phase 2 — Dual NIC Architecture Upgrade

A second PCIe NIC was installed in the Proxmox host, enabling physical separation between WAN and LAN traffic.

| NIC | Bridge | Role | Connected To |
|---|---|---|---|
| NIC 1 | `vmbr0` | WAN path | ISP router (initial) → Openreach ONT (final) |
| NIC 2 | `vmbr1` | LAN trunk — all internal VLANs | Cisco Catalyst 2960 |

---

### Phase 3 — Edge Router Migration

pfSense was moved from behind the EE router to directly in front of the Openreach ONT via PPPoE.

| Before | After |
|---|---|
| Openreach ONT → EE Router → pfSense | Openreach ONT → pfSense (PPPoE) |
| Double-NAT | Single NAT — pfSense owns the WAN IP |
| WireGuard handshakes failing | WireGuard stable and validated |
| ISP router controlling routing path | pfSense as sole edge device |

---

## 🔄 WAN Migration

### Previous Architecture

```text
Openreach ONT
→ EE Router (NAT + DHCP)
→ pfSense WAN
→ Internal VLANs
```

Problems:
- Double-NAT blocking WireGuard UDP handshakes
- ISP router intercepting and modifying inbound traffic
- No direct control over WAN IP assignment
- Unreliable remote access

---

### Final Architecture

```text
Openreach ONT
→ pfSense WAN (PPPoE)
→ Cisco Catalyst 2960
→ VLANs / Proxmox / AP / Clients
```

Benefits:
- pfSense owns the public WAN IP directly
- Clean single-NAT path
- Stable WireGuard VPN
- Full routing and firewall control at the edge

---

### WAN Interface Configuration

| Setting | Value |
|---|---|
| Interface | WAN |
| IPv4 Type | PPPoE |
| PPPoE Username | `bthomehub@btbroadband.com` |
| PPPoE Password | ISP credentials |
| Result | Direct public WAN IP assigned to pfSense |

---

## 🔌 Interface Configuration

### WAN Interface

| Setting | Value |
|---|---|
| Bridge | `vmbr0` → NIC 1 |
| IP Assignment | PPPoE — direct from Openreach ONT |
| Purpose | Internet access and edge routing |

---

### LAN Interface — Trunk

| Setting | Value |
|---|---|
| Parent Interface | `vtnet1` |
| Bridge | `vmbr1` → NIC 2 |
| Mode | 802.1Q trunk |
| Purpose | Parent interface for all VLAN subinterfaces |
| Layer 3 routing | Handled entirely by pfSense |

`vmbr1` carries all internal VLAN traffic to and from the Cisco switch. pfSense performs all routing and firewall enforcement across this bridge — no management IP is assigned to the bridge itself.

---

## 🔀 VLAN Interface Setup

VLAN subinterfaces were created inside pfSense on top of the LAN trunk interface, each acting as an independent routed security zone.

| VLAN | Interface | Gateway IP | Subnet |
|---|---|---|---|
| **VLAN 10** | Admin | `192.168.10.1` | `192.168.10.0/24` |
| **VLAN 20** | Servers | `192.168.20.1` | `192.168.20.0/24` |
| **VLAN 30** | IoT | `192.168.30.1` | `192.168.30.0/24` |
| **VLAN 40** | Users / Wi-Fi | `192.168.40.1` | `192.168.40.0/24` |
| **VLAN 99** | Management | `192.168.99.1` | `192.168.99.0/24` |

All VLANs are tagged on the Cisco trunk and routed by pfSense.

---

## 📡 DHCP Configuration

DHCP was enabled individually on each VLAN interface to provide automatic IP assignment while preserving static ranges for infrastructure devices.

| VLAN | DHCP Range |
|---|---|
| VLAN 10 — Admin | `192.168.10.100` – `192.168.10.199` |
| VLAN 20 — Servers | `192.168.20.100` – `192.168.20.199` |
| VLAN 30 — IoT | `192.168.30.100` – `192.168.30.199` |
| VLAN 40 — Users | `192.168.40.100` – `192.168.40.199` |

> 📝 VLAN 99 (Management) uses static IP assignment only. Infrastructure devices require consistent, predictable addresses — DHCP on the management plane introduces unnecessary risk.

This replaced DHCP services previously provided by the EE router.

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

## 🔒 Firewall Policy

Implemented least-privilege access between all VLANs.

### Inter-VLAN Access Rules

| Rule | Detail |
|---|---|
| IoT isolation | VLAN 30 blocked from Admin, Management, and Servers |
| User restrictions | VLAN 40 limited to HTTP/HTTPS outbound only |
| Server protection | VLAN 20 blocked from user and IoT access |
| Management isolation | VLAN 99 accessible only from trusted Admin devices |
| pfSense GUI and SSH | Locked to VLAN 10 (Admin) only |
| Default policy | Deny all traffic unless explicitly permitted |

### WAN Security

| Rule | Detail |
|---|---|
| Inbound access | Tightly restricted — only WireGuard UDP permitted |
| Unnecessary services | Disabled on WAN interface |
| Attack surface | Significantly reduced |

> 📄 Full firewall policy details in [04 — Firewall Policy](04-firewall-policy.md).

---

## 🔐 DNS Resolver Setup

| Setting | Value |
|---|---|
| Resolver | pfSense Unbound DNS |
| Scope | All VLAN interfaces |
| ACLs | Configured per VLAN |
| DNSSEC | Enabled |
| External DNS bypass | Blocked via firewall rules |

Forcing all VLANs to use the internal resolver prevents devices from bypassing security controls by pointing to external DNS servers.

---

## 🔑 SSH Hardening

| Setting | Value |
|---|---|
| Authentication method | RSA 4096-bit public key only |
| Password authentication | Disabled entirely |
| SSH access scope | Restricted to Admin VLAN 10 only |

> 📄 Full details in [06 — SSH Hardening](06-ssh-hardening.md).

---

## 📶 EE Router Conversion

After pfSense was moved to the network edge, the EE Hub was repurposed as a Wi-Fi access point only.

### Previous Role

```text
EE Hub: router + NAT + DHCP + Wi-Fi
```

### New Role

```text
EE Hub: Wi-Fi access point only
→ Connected to VLAN 40 access port on Cisco switch
→ DHCP disabled
→ Static management IP assigned
→ NAT and routing role removed
```

| Change | Detail |
|---|---|
| DHCP | Disabled — pfSense provides DHCP for VLAN 40 |
| NAT | Removed — pfSense handles all NAT at the edge |
| Wi-Fi | Retained for household user devices |
| Connection | Switch VLAN 40 access port → EE Hub LAN port |

This preserved full household Wi-Fi connectivity while eliminating double-NAT and ISP router interference.

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
| WireGuard tunnel | Handshake + remote SSH | Validated over mobile hotspot |
| Proxmox management | Access web UI post-changes | `https://192.168.10.200:8006` reachable |

---

## ⚠️ Issues Encountered

### 1. WireGuard Handshake Failures

| Field | Detail |
|---|---|
| **Problem** | WireGuard tunnel failing to establish — no handshake |
| **Cause** | ISP router double-NAT intercepting inbound UDP traffic |
| **Resolution** | Moved pfSense directly to ONT via PPPoE — eliminated ISP router from WAN path |

---

### 2. No WAN IP — PPPoE Failure

| Field | Detail |
|---|---|
| **Problem** | pfSense not receiving WAN IP after migration to ONT |
| **Cause** | Incorrect interface mapping — WAN assigned to wrong bridge |
| **Resolution** | Reassigned WAN interface to `vmbr0` and validated PPPoE credentials |

---

### 3. VLAN Gateway Failures

| Field | Detail |
|---|---|
| **Problem** | Inter-VLAN traffic not routing — VLAN gateways unreachable |
| **Cause** | Trunk port configured on wrong switch interface; incorrect pfSense LAN parent interface |
| **Resolution** | Validated physical switch connections and corrected trunk placement |

> 💡 **Key insight:** Always verify Layer 1 physical connectivity before attempting Layer 2 or Layer 3 troubleshooting.

---

### 4. FastEthernet Trunk Bottleneck

| Field | Detail |
|---|---|
| **Problem** | Poor Wi-Fi throughput and inter-VLAN performance |
| **Cause** | Critical trunk running on `Fa0/1` (100Mbps) with heavy CRC errors |
| **Resolution** | Migrated trunk to GigabitEthernet interface — restored full throughput |

---

### 5. WebGUI Access Failure

| Field | Detail |
|---|---|
| **Problem** | pfSense web UI inaccessible after management access changes |
| **Cause** | Browser session corruption following GUI protocol and VLAN changes |
| **Resolution** | Cleared browser site data and restarted pfSense GUI services |

> 💡 **Key insight:** Authentication and access failures are not always network or firewall problems — application state can mimic infrastructure issues.

---

## 📈 Result

- ✅ pfSense operating as true edge router with direct PPPoE WAN
- ✅ Stable internet access across all VLANs
- ✅ Reliable inter-VLAN routing with least-privilege firewall enforcement
- ✅ DHCP operational per VLAN with static ranges preserved
- ✅ DNS Resolver serving all networks with ACL and DNSSEC enforcement
- ✅ WireGuard VPN stable and validated from external network
- ✅ EE Hub repurposed as access point — double-NAT eliminated
- ✅ SSH hardened to public key authentication only
- ✅ pfSense GUI and SSH access locked to Admin VLAN

pfSense moved from a functional lab router into a production-style network security platform.

---

## 💡 Lessons Learned

- **ISP routers impose hidden constraints.** Moving pfSense to the edge removed an entire class of problems — NAT traversal, VPN handshakes, and routing stability all improved immediately.
- **Single-NIC designs create avoidable complexity.** The dual NIC upgrade removed routing and management problems that software configuration alone could not resolve.
- **Physical connectivity must be verified before logical troubleshooting.** Several issues that looked like routing failures traced directly back to incorrect physical port assumptions.
- **Stable management access must come before hardening.** Locking yourself out of pfSense mid-configuration is a real operational risk — always verify the management path before making changes.
- **Browser state can mimic network failures.** A corrupted session can look exactly like a firewall or authentication problem — clear application state before assuming infrastructure is broken.
- **Access point conversion is straightforward but sequencing matters.** Disabling DHCP on the EE Hub before pfSense was ready to serve VLAN 40 would have cut off household Wi-Fi — plan the migration order carefully.

---

## 🗺️ Future Improvements

- [ ] Dynamic DNS deployment
- [ ] IDS/IPS — Suricata or Snort
- [ ] Monitoring stack integration — Grafana / Zabbix
- [ ] Centralised logging and Syslog
- [ ] Backup validation and recovery testing
- [ ] Formal network diagrams

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| Proxmox Setup | [01-proxmox-setup.md](01-proxmox-setup.md) |
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| SSH Hardening | [06-ssh-hardening.md](06-ssh-hardening.md) |
| Cisco Switch Config | [cisco-2960-switch-config.md](../configs/cisco-2960-switch-config.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
