# 🔥 Firewall Policy & Access Control

This document outlines the firewall design used to control traffic between VLANs and enforce security boundaries across the home lab environment.

Rather than allowing unrestricted communication between all devices, pfSense was configured to:
- Restrict unnecessary lateral movement
- Isolate high-risk networks
- Protect infrastructure services
- Preserve administrative access
- Enforce least-privilege communication

This moves the environment from simple VLAN separation into intentional security architecture.

---

## 📋 Table of Contents

- [Design Objectives](#design-objectives)
- [Security Model](#security-model)
- [VLAN Security Zones](#vlan-security-zones)
- [Core Policy Design](#core-policy-design)
- [Zone-Based Firewall Policy Summary](#zone-based-firewall-policy-summary)
- [DNS Security Policy](#dns-security-policy)
- [Administrative Access Policy](#administrative-access-policy)
- [Default Deny Enforcement](#default-deny-enforcement)
- [Challenges Encountered](#challenges-encountered)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## 🎯 Design Objectives

| Objective | Status |
|---|---|
| Restrict inter-VLAN communication | ✅ Complete |
| Protect management interfaces | ✅ Complete |
| Isolate IoT devices | ✅ Complete |
| Preserve internet access where needed | ✅ Complete |
| Centralise security policy enforcement | ✅ Complete |

---

## 🧱 Security Model

### Default Principle — Deny by Default

Traffic is not trusted simply because it exists inside the network. All access must be **intentionally and explicitly allowed**.

| Principle | Detail |
|---|---|
| Default stance | Deny all traffic unless explicitly permitted |
| Rule evaluation | Top-down — first match wins |
| Trust model | Zero implicit trust between VLANs |
| Enforcement point | All inter-VLAN traffic passes through pfSense |

This follows standard enterprise security practice and prevents silent lateral movement between network segments.

> ⚠️ **Rule order is critical.** A misplaced allow rule above a deny rule will bypass intended security controls. Every rule was placed deliberately with least-privilege in mind.

---

## 🌐 VLAN Security Zones

| VLAN | Name | Trust Level | Role |
|---|---|---|---|
| **VLAN 10** | Admin | ⭐ High Trust | Primary management and administration |
| **VLAN 20** | Lab / Servers | 🔶 Medium Trust | Infrastructure services and testing |
| **VLAN 30** | IoT | 🔴 Untrusted | Isolated untrusted endpoints |
| **VLAN 40** | Users | 🔷 Semi-trusted | Daily-use client devices |
| **VLAN 99** | Management | 🔒 Restricted | Network device management only |

Each VLAN is treated as a separate trust boundary. All routing between them passes through pfSense for policy enforcement.

---

## 🔐 Core Policy Design

### VLAN 10 — Admin *(High Trust)*

| Traffic | Direction | Action |
|---|---|---|
| Internet access | Outbound | ✅ Allowed |
| pfSense GUI | VLAN 10 → pfSense | ✅ Allowed |
| Proxmox management | VLAN 10 → Proxmox | ✅ Allowed |
| SSH administration | VLAN 10 → infrastructure | ✅ Allowed |
| Switch management | VLAN 10 → VLAN 99 | ✅ Allowed |
| Access to Lab / Servers | VLAN 10 → VLAN 20 | ✅ Allowed |
| DNS | VLAN 10 → pfSense | ✅ Allowed |

**Reason:** Trusted administrator devices require full visibility and control access. VLAN 10 is the primary operational network — access is broad but deliberate.

---

### VLAN 20 — Lab / Servers *(Medium Trust)*

| Traffic | Direction | Action |
|---|---|---|
| Internet access | Outbound | ✅ Allowed |
| Package updates / DNS | VLAN 20 → pfSense | ✅ Allowed |
| Admin access inbound | VLAN 10 → VLAN 20 | ✅ Allowed |
| Access to Admin network | VLAN 20 → VLAN 10 | ❌ Blocked |
| Access to IoT | VLAN 20 → VLAN 30 | ❌ Blocked |
| Access to Users | VLAN 20 → VLAN 40 | ❌ Blocked |
| Access to Management | VLAN 20 → VLAN 99 | ❌ Blocked |

**Reason:** Servers should be reachable from trusted admin devices but not broadly exposed. Lab VMs should not be able to pivot into management infrastructure.

---

### VLAN 30 — IoT *(Untrusted)*

| Traffic | Direction | Action |
|---|---|---|
| Internet access | Outbound | ✅ Allowed |
| DNS | VLAN 30 → pfSense only | ✅ Allowed |
| Access to Admin | VLAN 30 → VLAN 10 | ❌ Blocked |
| Access to Lab | VLAN 30 → VLAN 20 | ❌ Blocked |
| Access to Users | VLAN 30 → VLAN 40 | ❌ Blocked |
| Access to Management | VLAN 30 → VLAN 99 | ❌ Blocked |

**Reason:** IoT devices are treated as fully untrusted endpoints. They frequently lack security patches and cannot be managed like trusted devices. Full isolation prevents a compromised IoT device from reaching any critical internal system.

---

### VLAN 40 — Users *(Semi-trusted)*

| Traffic | Direction | Action |
|---|---|---|
| Internet access | Outbound — HTTP/HTTPS only | ✅ Allowed |
| DNS | VLAN 40 → pfSense only | ✅ Allowed |
| Access to Admin | VLAN 40 → VLAN 10 | ❌ Blocked |
| Access to Lab | VLAN 40 → VLAN 20 | ⚠️ Limited — approved services only |
| Access to IoT | VLAN 40 → VLAN 30 | ❌ Blocked |
| Access to Management | VLAN 40 → VLAN 99 | ❌ Blocked |

**Reason:** Daily-use client devices should remain productive without gaining unnecessary internal access. Internet is restricted to HTTP/HTTPS (ports 80/443) to reduce outbound exposure.

---

### VLAN 99 — Management *(Restricted)*

| Traffic | Direction | Action |
|---|---|---|
| Admin access inbound | VLAN 10 → VLAN 99 | ✅ Allowed |
| Switch management | VLAN 99 internal | ✅ Allowed |
| Access from Users | VLAN 40 → VLAN 99 | ❌ Blocked |
| Access from IoT | VLAN 30 → VLAN 99 | ❌ Blocked |
| Access from Lab | VLAN 20 → VLAN 99 | ❌ Blocked |
| Internet access | Outbound | ❌ Blocked |

**Reason:** The management plane must be completely isolated from user-facing networks. Only VLAN 10 (Admin) is permitted to reach management infrastructure. This mirrors enterprise out-of-band management design.

---

## 📊 Zone-Based Firewall Policy Summary

| Source → Destination | VLAN 10 Admin | VLAN 20 Lab | VLAN 30 IoT | VLAN 40 Users | VLAN 99 Mgmt | Internet |
|---|---|---|---|---|---|---|
| **VLAN 10 Admin** | — | ✅ | ✅ | ✅ | ✅ | ✅ |
| **VLAN 20 Lab** | ❌ | — | ❌ | ❌ | ❌ | ✅ |
| **VLAN 30 IoT** | ❌ | ❌ | — | ❌ | ❌ | ✅ |
| **VLAN 40 Users** | ❌ | ⚠️ | ❌ | — | ❌ | ✅ HTTP/S |
| **VLAN 99 Mgmt** | ❌ | ❌ | ❌ | ❌ | — | ❌ |

> ⚠️ = Limited access to approved services only

### Firewall Policy Diagram

![Zone-Based Firewall Policy](../diagrams/ZB-Firewall_drawio.png)

---

## 🌐 DNS Security Policy

DNS was treated as a security control, not just a utility service.

| Control | Detail |
|---|---|
| DNS Resolver | pfSense Unbound DNS |
| DNSSEC | Enabled |
| External DNS bypass | Blocked via firewall rules |
| Per-VLAN ACLs | Configured — each VLAN restricted to pfSense resolver |
| Purpose | Prevent DNS-based security boundary bypass |

Allowing devices to use external resolvers (e.g. `8.8.8.8`) bypasses internal DNS visibility and allows potential security boundary circumvention. Forcing all VLANs through the pfSense resolver preserves control and visibility.

---

## 🔑 Administrative Access Policy

pfSense GUI and SSH access are restricted to the Admin VLAN only.

| Service | Permitted Source | Action |
|---|---|---|
| pfSense WebGUI | VLAN 10 only | ✅ Allowed |
| SSH to pfSense | VLAN 10 only — key-based | ✅ Allowed |
| SSH from other VLANs | All other VLANs | ❌ Blocked |
| WebGUI from other VLANs | All other VLANs | ❌ Blocked |

> 📄 Full SSH hardening details in [06 — SSH Hardening](06-ssh-hardening.md).

---

## 🚫 Default Deny Enforcement

pfSense enforces a default deny rule at the bottom of every interface rule set.

| Setting | Value |
|---|---|
| Default rule | Block all — any source, any destination |
| Logging | Enabled on default deny |
| Scope | Applied per VLAN interface |
| Override | Explicit allow rules above default deny only |

Any traffic not explicitly permitted by a rule above the default deny is silently dropped. This means new devices added to the network have no implicit access to anything — access must be deliberately granted.

---

## ⚠️ Challenges Encountered

| Issue | Cause | Resolution |
|---|---|---|
| Rule order conflicts | Allow rule placed above intended deny | Reviewed and reordered rules — most permissive rules moved below restrictive ones |
| IoT internet access failing | Missing outbound NAT rule for VLAN 30 | Verified NAT scope included all VLAN subnets |
| Admin locked out of pfSense GUI | Management access rule misconfigured | Corrected source interface on GUI allow rule |
| DNS bypass not blocked | Firewall rule missing for outbound UDP/TCP 53 | Added explicit block rule for external DNS from all VLANs |
| Users reaching Lab services | Overly broad allow rule on VLAN 40 | Tightened rule to specific destination IPs and ports only |

---

## 📈 Result

- ✅ Least-privilege access enforced across all VLANs
- ✅ IoT fully isolated from all internal networks
- ✅ Management plane accessible only from Admin VLAN
- ✅ DNS bypass blocked — all VLANs forced through pfSense resolver
- ✅ Default deny enforced at every VLAN interface
- ✅ Administrative access restricted to key-based SSH from Admin VLAN only
- ✅ Internet access preserved where operationally required

The firewall moved from a basic pass-through into a deliberate, production-style security policy.

---

## 💡 Lessons Learned

- **Rule order matters more than rule content.** A correctly written allow rule in the wrong position will silently bypass intended security controls.
- **DNS is a security control.** Unrestricted outbound DNS allows devices to bypass internal visibility — blocking external DNS resolvers is an essential hardening step.
- **Default deny must be tested, not assumed.** Configuring a default deny rule is not enough — each VLAN must be tested to confirm isolation actually works as intended.
- **Least privilege is harder to maintain than it sounds.** It requires knowing exactly what each device needs and resisting the temptation to create broad allow rules for convenience.
- **Firewall policy and VLAN design are inseparable.** The segmentation is only as strong as the rules enforcing it — a flat firewall policy with VLANs provides almost no real security benefit.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| Proxmox Setup | [01-proxmox-setup.md](01-proxmox-setup.md) |
| pfSense Setup | [02-pfsense-setup.md](02-pfsense-setup.md) |
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| SSH Hardening | [06-ssh-hardening.md](06-ssh-hardening.md) |
| Lessons Learned | [05-lessons-learned.md](05-lessons-learned.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
