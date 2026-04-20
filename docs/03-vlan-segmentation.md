# 🌐 VLAN Segmentation Design

This document outlines the VLAN segmentation strategy used to separate network traffic into logical security zones.

The goal was to:
- Reduce attack surface
- Prevent lateral movement
- Isolate high-risk devices (IoT)
- Separate administrative and user traffic
- Enforce controlled inter-VLAN communication via pfSense

This design mirrors real-world enterprise segmentation rather than a flat home network.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [VLAN Architecture](#vlan-architecture)
- [Network Design](#network-design)
- [Trunk Configuration](#trunk-configuration)
- [VLAN Configuration — Switch](#vlan-configuration--switch)
- [Access Port Assignments](#access-port-assignments)
- [Segmentation Strategy](#segmentation-strategy)
- [Design Decisions](#design-decisions)
- [Challenges Encountered](#challenges-encountered)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## 🎯 Design Objectives

| Objective | Status |
|---|---|
| Segment network by device type | ✅ Complete |
| Isolate IoT devices | ✅ Complete |
| Separate management plane | ✅ Complete |
| Enforce routing through firewall | ✅ Complete |
| Improve security and visibility | ✅ Complete |

---

## 🧱 VLAN Architecture

| VLAN | Name | Subnet | Purpose |
|---|---|---|---|
| **VLAN 10** | Admin | `192.168.10.0/24` | Trusted admin devices |
| **VLAN 20** | Lab | `192.168.20.0/24` | Servers and testing |
| **VLAN 30** | IoT | `192.168.30.0/24` | Untrusted devices |
| **VLAN 40** | Users | `192.168.40.0/24` | Daily-use clients |
| **VLAN 99** | Management | `192.168.99.0/24` | Network device management |

---

## 🔗 Network Design

**pfSense** acts as:
- Layer 3 router
- Firewall
- DHCP server per VLAN

**Cisco Catalyst 2960** operates as:
- Layer 2 switch
- VLAN segmentation point

All inter-VLAN routing is handled centrally by pfSense, ensuring every cross-VLAN packet passes through firewall policy enforcement.

---

## 🔌 Trunk Configuration

A trunk link was configured between the Cisco switch and pfSense to carry multiple VLANs over a single physical interface.

| Setting | Value |
|---|---|
| Interface | `Fa0/1` |
| Mode | Trunk (802.1Q) |
| Native VLAN | 99 (Management) |
| Allowed VLANs | 10, 20, 30, 40, 99 |

### Configuration

```cisco
SWITCH(config)#interface fa0/1
SWITCH(config-if)#switchport mode trunk
SWITCH(config-if)#switchport trunk native vlan 99
SWITCH(config-if)#switchport trunk allowed vlan 10,20,30,40,99
SWITCH(config-if)#no shutdown
SWITCH(config-if)#spanning-tree portfast trunk
SWITCH(config-if)#description SERVER-pfSense
SWITCH(config-if)#exit
```

### Verification — `do show interfaces trunk`

<img width="752" height="155" alt="cisco-fa01-trunk-config" src="https://github.com/user-attachments/assets/cf98c8ef-89a2-4603-aef7-d56a17e101c7" />

---

## 🧩 VLAN Configuration — Switch

### VLAN Creation

```cisco
SWITCH(config)#vlan 10
SWITCH(config-vlan)#name MAIN
SWITCH(config-vlan)#exit

SWITCH(config)#vlan 20
SWITCH(config-vlan)#name LAB
SWITCH(config-vlan)#exit

SWITCH(config)#vlan 30
SWITCH(config-vlan)#name IoT-devices
SWITCH(config-vlan)#exit

SWITCH(config)#vlan 40
SWITCH(config-vlan)#name Users
SWITCH(config-vlan)#exit

SWITCH(config)#vlan 99
SWITCH(config-vlan)#name Management
SWITCH(config-vlan)#exit
```

### Verification — `do show vlan brief`

<img width="738" height="296" alt="cisco-vlan-brief" src="https://github.com/user-attachments/assets/94844b78-963f-4e92-8e41-b7d976806870" />

---

## 🔌 Access Port Assignments

Devices were assigned to VLANs based on role and trust level:

| Port | Device | VLAN | Speed |
|---|---|---|---|
| `Fa0/1` | pfSense (trunk) | All VLANs | 100Mbps |
| `Gi0/1` | ISP Router | VLAN 99 — Management | 1000Mbps |
| `Gi0/2` | Admin PC | VLAN 10 — Admin | 1000Mbps |

> 📝 **Note:** The admin PC and ISP router were migrated from `Fa0/2` / `Fa0/3` to `Gi0/1` / `Gi0/2` to eliminate Fast Ethernet bottlenecks. Full details in [Cisco Switch Config](../configs/cisco-2960-switch-config.md).

### Verification — `do show interfaces status`

<img width="752" height="584" alt="cisco-int-status" src="https://github.com/user-attachments/assets/3c6905f4-ad01-4698-82c6-6cf2f51f60d1" />

---

## 🔐 Segmentation Strategy

Each VLAN was treated as a separate trust zone with distinct access permissions:

| VLAN | Trust Level | Access Policy |
|---|---|---|
| **VLAN 10 — Admin** | Trusted | Full access across the network |
| **VLAN 20 — Lab** | Protected | Only required access allowed — blocked from users and IoT |
| **VLAN 30 — IoT** | Untrusted | Internet-only — fully isolated from all internal networks |
| **VLAN 40 — Users** | Semi-trusted | Limited access — restricted from infrastructure |
| **VLAN 99 — Management** | Restricted | Switch and pfSense management only — not accessible from user networks |

---

## 🧠 Design Decisions

### Why use VLAN segmentation?

Flat networks allow unrestricted communication between all devices. Segmentation limits attack spread, improves control and visibility, and enables policy enforcement at the firewall level.

### Why use pfSense for routing?

Centralising routing through pfSense ensures all inter-VLAN traffic passes through firewall rules, security policies are consistently enforced, and traffic flows are visible and logged.

### Why isolate IoT devices?

IoT devices often lack regular security updates, cannot be fully trusted, and represent high-risk endpoints. Isolation prevents them from accessing or pivoting to critical internal systems even if compromised.

### Why use a dedicated management VLAN?

Separating management traffic onto VLAN 99 reduces exposure of infrastructure devices, prevents unauthorised administrative access from user or IoT networks, and mirrors enterprise network design standards.

---

## ⚠️ Challenges Encountered

| Issue | Cause | Resolution |
|---|---|---|
| VLAN access failures | Incorrect trunk allowed VLAN list | Fixed allowed VLANs on trunk interface |
| Devices not receiving IP | DHCP misconfiguration on pfSense interfaces | Verified and corrected pfSense VLAN interface config |
| Connectivity issues | Proxmox bridge misconfiguration | Rebuilt virtual bridge layout |
| Management lockout risk | No management isolation initially | Introduced VLAN 99 early in the redesign |

---

## 📈 Result

- ✅ Fully segmented network architecture
- ✅ Controlled communication between VLANs
- ✅ Reduced attack surface across all zones
- ✅ Clear separation of trust levels
- ✅ Centralised routing and security enforcement through pfSense

This phase transformed the network from a basic flat setup into a structured, security-focused design that mirrors enterprise segmentation principles.

---

## 💡 Lessons Learned

- **VLANs are about security, not just organisation.** The primary value is enforcing trust boundaries, not tidying up the network.
- **Trunk configuration must be precise.** A single missing VLAN ID or wrong native VLAN breaks connectivity in ways that are hard to diagnose quickly.
- **Management access must be isolated early.** Retrofitting VLAN 99 after initial setup created temporary lockout risks that could have been avoided.
- **Testing segmentation is critical.** Configuring VLANs is not enough — each zone must be tested to confirm isolation actually works as intended.
- **Design matters as much as configuration.** Getting the architecture right before touching the CLI saves significant troubleshooting time.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| Cisco Switch Configuration | [cisco-2960-switch-config.md](../configs/cisco-2960-switch-config.md) |
| pfSense Setup | [02-pfsense-setup.md](02-pfsense-setup.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| SSH Hardening | [06-ssh-hardening.md](06-ssh-hardening.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
