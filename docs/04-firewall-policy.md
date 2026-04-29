# 🔥 Firewall Hardening

This document covers the security hardening implemented within pfSense to reduce attack surface, enforce least-privilege access, and protect management infrastructure across the home lab.

Rather than simply establishing connectivity, pfSense was configured to:
- Minimise unnecessary WAN exposure
- Protect administrative services
- Isolate low-trust devices from critical systems
- Enforce least-privilege inter-VLAN access
- Support secure remote administration via WireGuard

This moves the environment from basic connectivity into a network with production-style security controls similar to real infrastructure environments.

---

## 📋 Table of Contents

- [Security Goals](#security-goals)
- [WAN Exposure Review](#wan-exposure-review)
- [WAN Firewall Policy](#wan-firewall-policy)
- [VLAN Firewall Policy](#vlan-firewall-policy)
- [Zone-Based Policy Summary](#zone-based-policy-summary)
- [DNS Resolver Hardening](#dns-resolver-hardening)
- [SSH Hardening](#ssh-hardening)
- [Management Plane Protection](#management-plane-protection)
- [WireGuard Security](#wireguard-security)
- [Lessons Learned](#lessons-learned)
- [Result](#result)

---

## 🎯 Security Goals

| Objective | Status |
|---|---|
| Minimise unnecessary WAN exposure | ✅ Complete |
| Protect administrative services | ✅ Complete |
| Isolate low-trust devices from critical systems | ✅ Complete |
| Enforce least-privilege inter-VLAN access | ✅ Complete |
| Reduce attack surface | ✅ Complete |
| Improve visibility and operational security | ✅ Complete |
| Support secure remote administration via WireGuard | ✅ Complete |

---

## 🌐 WAN Exposure Review

### Previous Risk

Before pfSense became the edge router, the ISP router handled internet-facing traffic and introduced unreliable forwarding, double NAT, and poor visibility. After migrating pfSense to the WAN edge, a full review of exposed services was required.

### Hardening Actions

Services disabled or removed:

| Service | Action |
|---|---|
| Web management from untrusted networks | ❌ Removed |
| Unused inbound services | ❌ Removed |
| Unnecessary external-facing services | ❌ Removed |

Only required services remained exposed.

---

## 🔐 WAN Firewall Policy

### Inbound Policy

| Approach | Detail |
|---|---|
| Default stance | Deny all inbound traffic |
| Override | Allow only explicit requirements |

| Service | Action |
|---|---|
| WireGuard VPN listener | ✅ Allowed |
| Explicitly required administrative paths | ✅ Allowed |
| Everything else | ❌ Denied |

This follows least-privilege design — nothing is exposed unless there is a deliberate operational reason.

---

## 🧱 VLAN Firewall Policy

### Security Model

Access between VLANs is controlled using explicit policy rather than open trust.

| VLAN | Trust Level | Role |
|---|---|---|
| **VLAN 10** | ⭐ High Trust | Administrative network |
| **VLAN 20** | 🔶 Medium Trust | Lab / Servers |
| **VLAN 30** | 🔴 Untrusted | IoT devices |
| **VLAN 40** | 🔷 Semi-trusted | Users / Wi-Fi |
| **VLAN 99** | 🔒 Restricted | Management plane |

### Rules Implemented

**Blocked traffic:**

| Source → Destination | Action |
|---|---|
| VLAN 30 → VLAN 10 | ❌ Blocked |
| VLAN 30 → VLAN 99 | ❌ Blocked |
| VLAN 40 → VLAN 10 | ❌ Blocked |
| VLAN 40 → VLAN 99 | ❌ Blocked |
| Non-admin → pfSense GUI | ❌ Blocked |
| Non-admin → SSH management | ❌ Blocked |

**Controlled access:**

| Source → Destination | Action |
|---|---|
| VLAN 40 → VLAN 20 | ⚠️ HTTP/HTTPS only |
| Approved management access | ✅ Trusted devices only |
| Infrastructure access | ✅ Administrative systems only |

This prevents lateral movement and protects the control plane.

---

## 📊 Zone-Based Policy Summary

| Source → Destination | VLAN 10 Admin | VLAN 20 Lab | VLAN 30 IoT | VLAN 40 Users | VLAN 99 Mgmt | Internet |
|---|---|---|---|---|---|---|
| **VLAN 10 Admin** | — | ✅ | ✅ | ✅ | ✅ | ✅ |
| **VLAN 20 Lab** | ❌ | — | ❌ | ❌ | ❌ | ✅ |
| **VLAN 30 IoT** | ❌ | ❌ | — | ❌ | ❌ | ✅ |
| **VLAN 40 Users** | ❌ | ⚠️ | ❌ | — | ❌ | ✅ HTTP/S |
| **VLAN 99 Mgmt** | ❌ | ❌ | ❌ | ❌ | — | ❌ |

> ⚠️ = Limited access to approved services only

A zone-based firewall policy was implemented using pfSense VLAN interfaces to enforce least-privilege communication between network segments. Administrative systems retain secure access to infrastructure, while user and IoT VLANs are isolated from management services and sensitive systems. Access between VLANs is explicitly permitted only where operationally required, reducing attack surface and improving overall network security.

---

## 🌐 DNS Resolver Hardening

DNS was treated as a security control, not just a utility service.

| Control | Detail |
|---|---|
| DNS Resolver | pfSense Unbound DNS |
| Access Control Lists | Configured per VLAN |
| DNSSEC | Enabled |
| External DNS bypass | Blocked via firewall rules |
| Resolver access | Restricted to pfSense only |

**Benefits:**

- Trusted internal DNS resolution
- Reduced abuse potential
- Improved integrity of DNS responses

Allowing devices to use external resolvers (e.g. `8.8.8.8`) bypasses internal DNS visibility and allows potential security boundary circumvention. Forcing all VLANs through the pfSense resolver preserves control and visibility.

---

## 🔑 SSH Hardening

Administrative access was hardened to eliminate password-based authentication.

| Setting | Value |
|---|---|
| Authentication method | Public key only |
| Password authentication | ❌ Disabled |
| Permitted sources | VLAN 10 (Admin) only |

**Benefits:**

- Stronger administrative access control
- Reduced brute-force exposure
- Better operational security posture

> 📄 Full SSH hardening details in [06 — SSH Hardening](06-ssh-hardening.md).

---

## 🛡️ Management Plane Protection

All management interfaces are restricted to trusted VLANs and approved devices only.

| Service | Permitted Source | Action |
|---|---|---|
| pfSense WebGUI | VLAN 10 only | ✅ Allowed |
| SSH to pfSense | VLAN 10 only — key-based | ✅ Allowed |
| Switch management | VLAN 10 → VLAN 99 | ✅ Allowed |
| Proxmox management | VLAN 10 only | ✅ Allowed |
| Access from all other VLANs | Any management service | ❌ Blocked |

This separates infrastructure control from normal user activity and mirrors enterprise out-of-band management design.

---

## 🔒 WireGuard Security

WireGuard was deployed as the primary secure remote access method. Rather than exposing administrative services directly to the internet, all remote access is tunnelled through VPN first.

```text
Remote admin access
→ VPN first
→ Infrastructure access second
```

| Control | Detail |
|---|---|
| WAN listener | WireGuard only |
| Direct admin exposure | ❌ None |
| Remote access path | VPN → Admin VLAN → Infrastructure |

This significantly reduces the external attack surface.

---

## ⚠️ Lessons Learned

- **Security is architecture, not rules.** The biggest improvements came from correct VLAN design, proper trunking, management isolation, and removal of double NAT — not simply adding more firewall rules.
- **Rule order matters more than rule content.** A correctly written allow rule in the wrong position will silently bypass intended security controls.
- **DNS is a security control.** Unrestricted outbound DNS allows devices to bypass internal visibility — blocking external DNS resolvers is an essential hardening step.
- **Default deny must be tested, not assumed.** Configuring a default deny rule is not enough — each VLAN must be tested to confirm isolation actually works as intended.
- **Least privilege is harder to maintain than it sounds.** It requires knowing exactly what each device needs and resisting the temptation to create broad allow rules for convenience.

---

## 📈 Result

- ✅ Reduced WAN exposure — only WireGuard listener exposed
- ✅ Secure management access — key-based SSH from Admin VLAN only
- ✅ Strong VLAN isolation — least-privilege routing enforced across all segments
- ✅ DNS bypass blocked — all VLANs forced through pfSense resolver
- ✅ Hardened SSH — password authentication removed
- ✅ Reliable remote administration through WireGuard VPN

This transformed the environment from a basic home network into a controlled infrastructure environment with production-style security design.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| Proxmox Setup | [01-proxmox-setup.md](01-proxmox-setup.md) |
| pfSense Setup | [02-pfsense-setup.md](02-pfsense-setup.md) |
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| SSH Hardening | [06-ssh-hardening.md](06-ssh-hardening.md) |
| Lessons Learned | [05-lessons-learned.md](05-lessons-learned.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
