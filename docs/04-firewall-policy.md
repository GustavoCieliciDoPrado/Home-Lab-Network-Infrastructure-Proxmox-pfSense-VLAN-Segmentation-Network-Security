# 🔥 Firewall Policy Design — pfSense

> Enforcing segmentation, least-privilege access, and controlled inter-VLAN communication across the home lab environment.

---

## Overview

This document covers the firewall architecture used to enforce network segmentation inside the home lab. The goal was not simply connectivity — but **secure, intentional traffic control** between trusted, semi-trusted, and untrusted zones.

All rules were designed to mirror real-world production security practices rather than typical home networking defaults.

---

## 🎯 Security Objectives

| Objective | Status |
|---|---|
| Restrict unnecessary inter-VLAN access | ✅ Complete |
| Isolate IoT devices from trusted networks | ✅ Complete |
| Protect server VLAN from user/IoT traffic | ✅ Complete |
| Restrict pfSense management access | ✅ Complete |
| Enforce default deny policy | ✅ Complete |

---

## 🌐 VLAN Trust Zones

| VLAN | Name | Trust Level | Purpose |
|---|---|---|---|
| VLAN 10 | Admin | Trusted | Main administrator devices |
| VLAN 20 | Servers | Protected | Ubuntu server / lab services |
| VLAN 30 | IoT | Untrusted | Smart devices / isolated endpoints |
| VLAN 40 | Users | Semi-Trusted | Daily-use clients |
| VLAN 99 | Management | Restricted | Network device management |

---

## 🧱 Security Design Principles

Firewall rules were built using the following principles:

- **Least privilege access** — only what is explicitly required is permitted
- **Default deny** — all traffic is blocked unless an explicit allow rule exists
- **Administrative isolation** — management plane separated from data plane
- **Reduced attack surface** — unused services and ports blocked by default
- **Lateral movement prevention** — VLANs cannot communicate without explicit policy
- **DNS enforcement** — all DNS forced through pfSense resolver

This approach mirrors real production security design rather than simple home networking.

---

## 🔁 Inter-VLAN Access Policy

### VLAN 10 — Admin

**Allowed:**
- Full access to all VLANs
- pfSense GUI access
- SSH access to all management interfaces

> **Rationale:** Administrative devices require full visibility and management capability across all network zones.

---

### VLAN 20 — Servers

**Allowed:**
- Outbound internet access
- Required updates and package management

**Blocked:**
- Direct access to Admin VLAN
- Direct access to IoT VLAN
- Unrestricted user access

> **Rationale:** Servers are protected assets. They should not be freely accessible endpoints — access is granted on a need-only basis.

---

### VLAN 30 — IoT

**Allowed:**
- Internet access only
- DNS queries to pfSense resolver

**Blocked:**
- Admin VLAN
- Server VLAN
- User VLAN
- All management interfaces

> **Rationale:** IoT devices represent the highest unmanaged security risk in any network. Full isolation is mandatory.

---

### VLAN 40 — Users

**Allowed:**
- Internet access
- Limited access to approved services only

**Blocked:**
- Direct administrative access
- IoT lateral movement paths

> **Rationale:** Users require productivity access without the ability to reach infrastructure control planes or untrusted devices.

---

## 🛡️ Example Rule Logic

### Block IoT → Internal Networks

| Field | Value |
|---|---|
| Interface | VLAN 30 |
| Source | VLAN30 subnet |
| Destination | RFC1918 networks |
| Protocol | Any |
| Action | **Block** |
| Description | Prevent IoT lateral movement |

---

### Allow Admin → pfSense SSH

| Field | Value |
|---|---|
| Interface | VLAN 10 |
| Source | `192.168.10.0/24` |
| Destination | pfSense LAN IP |
| Port | `22` |
| Protocol | TCP |
| Action | **Pass** |

---

## 📌 Rule Order — Lessons Learned

> **Rule order matters more than expected.**

pfSense processes rules **top → bottom**. A misplaced allow rule can completely bypass intended security controls — and it will do so silently.

This became especially apparent when testing:

- DNS restriction enforcement
- IoT VLAN isolation
- SSH access controls
- Admin GUI access restriction

Firewall design became less about *adding rules* and more about **correct rule placement**. A correct rule in the wrong position is effectively no rule at all.

---

## 🧪 Testing & Validation

All security controls were validated post-configuration:

- [x] SSH blocked from all non-Admin VLANs
- [x] pfSense GUI inaccessible outside VLAN 10
- [x] IoT devices unable to reach any internal subnet
- [x] Servers isolated from user traffic
- [x] DNS forced through pfSense resolver
- [x] Internet access maintained where required by policy

> Security is only complete after verification — not after configuration.

---

## 📈 Results

| Area | Before | After |
|---|---|---|
| Inter-VLAN access | Mostly open | Controlled |
| IoT exposure | High | Isolated |
| Admin access | Broad | Restricted |
| Management plane | Shared | Segmented |
| Firewall design | Functional | Security-driven |

---

## 💡 Lessons Learned

- Firewall rules are **architecture**, not just access lists
- Least privilege is harder to implement — but far safer
- IoT isolation is **mandatory**, not optional
- DNS control is a security feature, not a convenience
- Rule testing is as important as rule creation

> This phase transformed the lab from working networking into real infrastructure security design.

---

*Part of the [Home Lab Repository](../README.md) — documenting hands-on security and networking practice.*
