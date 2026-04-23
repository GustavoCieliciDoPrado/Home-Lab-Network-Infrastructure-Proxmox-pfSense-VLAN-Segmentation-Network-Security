# 🌐 VLAN Segmentation Design

## Overview

This document outlines the VLAN segmentation strategy used to separate network traffic into logical security zones.

The goal was to:

* Reduce attack surface
* Prevent lateral movement
* Isolate high-risk devices (IoT)
* Separate administrative and user traffic
* Enforce controlled inter-VLAN communication via pfSense

This design mirrors real-world enterprise segmentation rather than flat home networking.

---

# 🎯 Design Objectives

| Objective                        | Status     |
| -------------------------------- | ---------- |
| Segment network by device type   | ✅ Complete |
| Isolate IoT devices              | ✅ Complete |
| Separate management plane        | ✅ Complete |
| Enforce routing through firewall | ✅ Complete |
| Improve security and visibility  | ✅ Complete |

---

# 🧱 VLAN Architecture

| VLAN    | Name       | Subnet          | Purpose                   |
| ------- | ---------- | --------------- | ------------------------- |
| VLAN 10 | Admin      | 192.168.10.0/24 | Trusted admin devices     |
| VLAN 20 | Lab        | 192.168.20.0/24 | Servers and testing       |
| VLAN 30 | IoT        | 192.168.30.0/24 | Untrusted devices         |
| VLAN 40 | Users      | 192.168.40.0/24 | Daily-use clients         |
| VLAN 99 | Management | 192.168.99.0/24 | Network device management |

---

# 🔗 Network Design

## pfSense

Acts as:

* Layer 3 router
* Firewall
* DHCP server
* DNS resolver

All inter-VLAN communication must pass through pfSense for security enforcement.

---

## Cisco Catalyst 2960

Acts as:

* Layer 2 switch
* VLAN segmentation point
* trunk distribution point

This provides physical separation of network trust zones.

---

## Proxmox

Acts as:

* virtualization platform
* pfSense host
* infrastructure control layer

With the dual-NIC upgrade:

* NIC1 → WAN / router side
* NIC2 → LAN / VLAN trunk side

This significantly improved VLAN reliability and routing clarity.

---

# 🔌 Trunk Configuration

A dedicated trunk link was configured between the Cisco switch and the Proxmox host (pfSense LAN side).

This carries multiple VLANs over a single physical interface.

### Trunk Port

## Switch Port

Gi0/1

## Connected To

Proxmox Host → NIC2 → vmbr1 → pfSense LAN

## Allowed VLANs

* VLAN 10
* VLAN 20
* VLAN 30
* VLAN 40
* VLAN 99

This ensures pfSense receives VLAN-tagged traffic for all internal networks.

---

# 🧩 Access Port Assignments

Devices are assigned to VLANs based on operational role.

| Port  | Device                  | VLAN     |
| ----- | ----------------------- | -------- |
| Gi0/1 | Proxmox / pfSense trunk | Trunk    |
| Gi0/2 | Main PC (Admin)         | VLAN 10  |
| Fa0/2 | ISP Router / WAN side   | External |
| Fa0/3 | Ubuntu Server / Lab     | VLAN 20  |

This preserves clear separation between infrastructure, users, and untrusted devices.

---

# 🔐 Segmentation Strategy

## VLAN 10 — Admin

### Purpose

Trusted administrator devices

### Access

* pfSense GUI
* Proxmox management
* SSH access
* full administrative visibility

This is the primary management network.

---

## VLAN 20 — Lab

### Purpose

Servers and infrastructure testing

### Access

* internet access
* updates
* controlled internal access

Protected from unnecessary user and IoT access.

---

## VLAN 30 — IoT

### Purpose

Untrusted smart devices

### Access

* internet only
* DNS to pfSense

Fully isolated from trusted internal networks.

This significantly reduces attack surface.

---

## VLAN 40 — Users

### Purpose

Daily-use client devices

### Access

* internet access
* limited approved internal services

Restricted from infrastructure control.

---

## VLAN 99 — Management

### Purpose

Network device management

Used for:

* switch management
* future AP management
* dedicated infrastructure control

Separated from user-facing networks.

---

# 🧠 Design Decisions

## Why use VLAN segmentation?

Flat networks allow unrestricted communication.

Segmentation:

* limits attack spread
* improves control
* enables policy enforcement
* improves visibility

This is standard enterprise design.

---

## Why centralize routing through pfSense?

This ensures:

* all inter-VLAN traffic hits firewall policy
* security is enforced consistently
* visibility exists for all traffic flows

This prevents silent trust between networks.

---

## Why isolate IoT?

IoT devices:

* often lack updates
* cannot be trusted
* represent the highest unmanaged security risk

Isolation prevents lateral movement into trusted systems.

---

## Why use a dedicated management VLAN?

Separating management traffic:

* reduces exposure of infrastructure
* prevents unauthorized administrative access
* mirrors production network design

This protects the management plane.

---

# ⚠️ Challenges Encountered

| Issue                   | Cause                      | Resolution                       |
| ----------------------- | -------------------------- | -------------------------------- |
| VLAN access failures    | Incorrect trunk config     | Corrected switch trunk placement |
| Devices not getting IP  | DHCP misconfiguration      | Verified VLAN DHCP scopes        |
| Connectivity issues     | Wrong physical switch port | Validated Layer 1 before Layer 2 |
| Management lockout risk | Single-NIC limitations     | Upgraded to dual-NIC design      |

This phase reinforced that physical connectivity errors often appear as logical network failures.

---

# 📈 Result

The final design achieved:

* fully segmented network architecture
* clean trust zone separation
* centralized routing and security enforcement
* stable inter-VLAN communication
* reduced attack surface
* production-style network design

The environment moved from basic connectivity into intentional security architecture.

---

# 💡 Lessons Learned

* VLANs are for security, not just organization
* Layer 1 mistakes can look like Layer 3 failures
* Stable management access must come before segmentation perfection
* Dedicated WAN/LAN separation improves everything
* Clean architecture simplifies troubleshooting dramatically

---

# 🔗 Related Documentation

* Proxmox setup → 01-proxmox-setup.md
* pfSense setup → 02-pfsense-setup.md
* Firewall policy → 04-firewall-policy.md

---

# Why This Matters

This proves practical experience with:

* VLAN design
* Cisco switching
* trunking
* segmentation strategy
* firewall-enforced routing
* enterprise-style security boundaries

These are core skills for:

* junior network engineering
* datacenter operations
* NOC roles
* infrastructure engineering
