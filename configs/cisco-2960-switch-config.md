# 🔧 Cisco Catalyst 2960 — Switch Configuration

This document covers the full switch configuration for the Cisco Catalyst 2960 used in the home lab, including VLAN creation, trunk port setup, access port assignments, and verification outputs.

---

## 📋 Table of Contents

- [Device Overview](#device-overview)
- [VLAN Creation](#vlan-creation)
- [Trunk Port Configuration — Fa0/1 (pfSense)](#trunk-port-configuration--fa01-pfsense)
- [Access Port Configuration](#access-port-configuration)
- [Port Assignment Decision — Gi0/1 and Gi0/2](#port-assignment-decision--gi01-and-gi02)
- [Verification](#verification)
- [Lessons Learned](#lessons-learned)

---

## 📦 Device Overview

| Field | Detail |
|---|---|
| Device | Cisco Catalyst 2960 |
| Role | Layer 2 switching, VLAN segmentation |
| Uplink to pfSense | Fa0/1 (trunk) |
| Management VLAN | VLAN 99 |
| Connected VLANs | 10, 20, 30, 40, 99 |

### Final Port Assignment Summary

| Port | Name | Role | VLAN |
|---|---|---|---|
| `Fa0/1` | SERVER-pfSense | Trunk | All (10,20,30,40,99) |
| `Gi0/1` | ISP-ROUTER | Access | 99 (Management) |
| `Gi0/2` | ADMIN | Access | 10 |

> ⚠️ **Note:** Ports `Fa0/2` and `Fa0/3` were previously used for the admin PC and server respectively. These were migrated to `Gi0/1` and `Gi0/2` to eliminate Fast Ethernet bottlenecks. See [Port Assignment Decision](#port-assignment-decision--gi01-and-gi02) for full details.

---

## 🌐 VLAN Creation

VLAN creation was performed from global configuration mode. No screenshots were captured during this step — the commands below reflect the exact configuration visible in the `show vlan brief` verification output.

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

---

## 🔗 Trunk Port Configuration — Fa0/1 (pfSense)

`Fa0/1` connects the switch to pfSense and carries all VLANs via 802.1Q tagging. VLAN 99 is set as the native VLAN to carry untagged management traffic.

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

> ⚠️ **PortFast Warning:** Cisco issued a warning that `spanning-tree portfast trunk` should only be used on ports connected to a single host — not to hubs, switches, or bridges, as it can cause temporary bridging loops. This is acknowledged and accepted here as `Fa0/1` connects directly to pfSense only.

**Configuration screenshot:**

![Fa0/1 Trunk Configuration](<img width="752" height="155" alt="cisco-fa01-trunk-config" src="https://github.com/user-attachments/assets/f1771ced-cf4e-47bf-b905-ab59c6403a92" />
)

**Trunk verification:**

![Trunk Verification](<img width="750" height="146" alt="cisco-trunk-verify" src="https://github.com/user-attachments/assets/395010f2-593a-4a5a-99d7-e4d8e0e96a78" />)

---

## 🔌 Access Port Configuration

### Gi0/2 — Admin PC (VLAN 10)

```cisco
SWITCH(config)#interface gi0/2
SWITCH(config-if)#switchport mode access
SWITCH(config-if)#switchport access vlan 10
SWITCH(config-if)#description ADMIN
SWITCH(config-if)#no shutdown
SWITCH(config-if)#duplex full
SWITCH(config-if)#speed 1000
SWITCH(config-if)#exit
```

**Configuration screenshot:**

![Gi0/2 Access Port Config](<img width="1479" height="483" alt="cisco-gi02-access-config" src="https://github.com/user-attachments/assets/f3aee5e1-be44-4abc-9551-796d2752d880" />)

---

### Gi0/1 — ISP Router (VLAN 99)

```cisco
SWITCH(config)#interface gi0/1
SWITCH(config-if)#switchport mode access
SWITCH(config-if)#switchport access vlan 99
SWITCH(config-if)#description ISP-ROUTER
SWITCH(config-if)#no shutdown
SWITCH(config-if)#duplex full
SWITCH(config-if)#speed 1000
SWITCH(config-if)#exit
```

---

## ⚡ Port Assignment Decision — Gi0/1 and Gi0/2

### Problem

Initially the admin PC and server were connected to `Fa0/2` and `Fa0/3` respectively — Fast Ethernet ports running at 100Mbps full duplex.

| Port | Speed | Limitation |
|---|---|---|
| `Fa0/2` | 100Mbps | Bottleneck for admin traffic |
| `Fa0/3` | 100Mbps | Bottleneck for lab/server traffic |

This created a performance constraint that didn't reflect how production environments handle management and server traffic.

### Decision

Migrated both connections to the Gigabit uplink ports:

| Port | Speed | Role |
|---|---|---|
| `Gi0/1` | 1000Mbps | ISP Router — VLAN 99 |
| `Gi0/2` | 1000Mbps | Admin PC — VLAN 10 |

This eliminated the Fast Ethernet bottleneck and better reflects enterprise-grade port allocation where critical links use Gigabit interfaces.

---

## ✅ Verification

### VLAN Brief — `do sh vlan brief`

Confirms all VLANs are created, named correctly, and active.

![VLAN Brief](<img width="738" height="296" alt="cisco-vlan-brief" src="https://github.com/user-attachments/assets/4718404d-95df-4bec-af22-bd1e97b6700b" />)

### Interface Status — `do sh int status`

Confirms port assignments, VLAN membership, duplex, speed, and connection status.

![Interface Status](<img width="752" height="584" alt="cisco-int-status" src="https://github.com/user-attachments/assets/78eef069-f498-4751-8c32-9b71be717011" />)

### Trunk Status — `do sh int trunk`

Confirms `Fa0/1` is trunking with 802.1Q encapsulation, native VLAN 99, and all VLANs active and forwarding.

![Trunk Status — Fa0/1](<img width="750" height="146" alt="cisco-trunk-verify" src="https://github.com/user-attachments/assets/e419137b-4115-4af7-8801-72310eb9b3d1" />)

---

## 💡 Lessons Learned

- **Native VLAN on the trunk matters.** Setting VLAN 99 as native ensures management traffic is isolated from data VLANs and aligns with best practice for management plane separation.
- **PortFast on trunk ports requires caution.** The Cisco warning is real — this is only safe because `Fa0/1` connects to a single device (pfSense), not to another switch.
- **Hardware constraints shape design decisions.** Moving from Fa to Gi ports wasn't just a preference — it was necessary to remove a real bottleneck. That kind of constraint-driven decision is common in production environments.
- **Unused ports default to VLAN 1.** All unassigned ports remain on the default VLAN. In a hardened production environment these would be shut down and assigned to a dead VLAN.

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
