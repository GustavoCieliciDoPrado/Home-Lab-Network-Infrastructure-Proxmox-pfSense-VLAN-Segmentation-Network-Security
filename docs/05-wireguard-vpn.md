# 🔒 WireGuard VPN

This document covers the deployment of WireGuard VPN for secure remote administration of the home lab environment.

The goal was to enable reliable, secure access to pfSense, Proxmox, and internal infrastructure from outside the home network — without exposing management services directly to the internet.

This became one of the most valuable parts of the project. Deployment required resolving WAN architecture issues, ISP-router limitations, double NAT problems, and VLAN-aware routing design before WireGuard could function correctly.

---

## 📋 Table of Contents

- [Objectives](#objectives)
- [Initial Problem](#initial-problem)
- [Root Cause](#root-cause)
- [Permanent Fix](#permanent-fix)
- [WireGuard Deployment](#wireguard-deployment)
- [Firewall Rules](#firewall-rules)
- [Validation](#validation)
- [Dynamic DNS](#dynamic-dns)
- [Challenges Encountered](#challenges-encountered)
- [Result](#result)
- [Lessons Learned](#lessons-learned)

---

## 🎯 Objectives

| Goal | Detail |
|---|---|
| Secure remote access | Admin access to management infrastructure without public exposure |
| VPN-first model | No direct internet exposure of Proxmox or pfSense |
| MacBook remote administration | External device as primary remote admin client |
| True off-site validation | Mobile hotspot testing to confirm real external reachability |

---

## ⚠️ Initial Problem

### Original Network Path

```text
Internet
→ EE Router
→ pfSense
→ Internal VLANs
```

With pfSense sitting behind the ISP router, inbound WireGuard traffic could never reach it reliably.

| Symptom | Detail |
|---|---|
| Latest Handshake | Never |
| TX | Increasing — client sending traffic |
| RX | 0 — pfSense never receiving initiation packets |

Attempted workarounds that failed:

| Attempt | Outcome |
|---|---|
| Port forwarding on ISP router | Unreliable — ISP router remained the bottleneck |
| DMZ configuration | Did not resolve inbound UDP path |
| Alternate ports (e.g. UDP 443) | Same result — handshake never completed |
| pfSense firewall tuning | Not the root cause |

---

## 🔍 Root Cause

The problem was not WireGuard configuration. The actual cause was architectural:

```text
EE Router + double NAT + broken inbound UDP path
→ pfSense never received WireGuard initiation packets
```

> ⚠️ **Key insight:** No amount of firewall or VPN tuning can fix a broken network path upstream. The ISP router was the real problem.

---

## 🔧 Permanent Fix

pfSense was moved to the WAN edge, replacing the ISP router for all routing and NAT.

### New Architecture

```text
Openreach ONT
→ pfSense WAN (PPPoE)
→ Cisco Switch
→ VLANs
→ Proxmox / Infrastructure
```

| Improvement | Detail |
|---|---|
| Double NAT removed | pfSense holds the public WAN assignment directly |
| Stable inbound UDP path | No ISP-router interference |
| Proper VPN architecture | pfSense as true edge router and VPN endpoint |

This immediately resolved all WireGuard handshake failures.

---

## 🚀 WireGuard Deployment

### pfSense Tunnel Configuration

| Setting | Detail |
|---|---|
| Package | WireGuard for pfSense |
| Listen port | UDP 51820 |
| Keys | Private/public key pair generated on pfSense |
| WAN firewall rule | UDP 51820 allowed inbound to WAN address |

### MacBook Peer Configuration

```ini
[Interface]
PrivateKey = <client-private-key>
Address    = <vpn-tunnel-ip>

[Peer]
PublicKey  = <pfsense-public-key>
Endpoint   = <public-wan-ip-or-ddns>:51820
AllowedIPs = <management-network-ranges>
```

| Setting | Detail |
|---|---|
| Endpoint | Public WAN IP (or DDNS hostname) |
| AllowedIPs | Management network ranges only — not full tunnel |
| Access granted | pfSense GUI, Proxmox, internal infrastructure |
| Direct WAN exposure | ❌ None |

---

## 🔐 Firewall Rules

### WAN Rule

| Setting | Value |
|---|---|
| Protocol | UDP |
| Destination | WAN address |
| Port | 51820 |
| Action | ✅ Allow |

### VPN Interface Access Control

| Access | Action |
|---|---|
| Administrative VLANs | ✅ Allowed |
| Infrastructure management paths | ✅ Allowed |
| Full network tunnel | ❌ Not permitted |
| Unrestricted internal access | ❌ Not permitted |

VPN clients are granted access equivalent to VLAN 10 (Admin) — full management access without bypassing segmentation policy.

---

## ✅ Validation

### Testing Method

All validation was performed over a **mobile hotspot**, not the local LAN. This confirmed true external reachability rather than simulated remote access.

| Test | Result |
|---|---|
| WireGuard handshake | ✅ Successful |
| RX / TX | ✅ Both active |
| pfSense GUI access | ✅ Reachable remotely |
| Proxmox access | ✅ Reachable remotely |
| Direct WAN exposure | ✅ None — VPN-only path confirmed |

---

## 🌐 Dynamic DNS

Because the public WAN IP may change, Dynamic DNS is planned to remove the static IP dependency.

| Current state | Static IP configured as endpoint |
|---|---|
| Target state | DDNS hostname as endpoint |

```ini
Endpoint = homelab.duckdns.org:51820
```

This remains part of future improvement work and will make the VPN resilient to IP changes without requiring peer reconfiguration.

---

## ⚠️ Challenges Encountered

| Issue | Cause | Resolution |
|---|---|---|
| Handshake never completing | Double NAT blocking inbound UDP | Moved pfSense to WAN edge — removed ISP router |
| TX increasing, RX = 0 | Initiation packets not reaching pfSense | Resolved by eliminating double NAT |
| Port forwarding unreliable | ISP router handling forwarding inconsistently | Bypassed entirely — pfSense now owns WAN |
| DMZ approach failing | ISP router still intercepting traffic | Same fix — full architectural change required |

---

## 📈 Result

- ✅ Secure remote access working — pfSense and Proxmox reachable off-site
- ✅ Stable WireGuard handshakes — resolved by architectural fix, not VPN tuning
- ✅ No public exposure of management services — VPN-first access model enforced
- ✅ Mobile hotspot validation confirmed true external reachability
- ✅ VLAN segmentation preserved through VPN access

This transformed remote access from unreliable ISP-router workarounds into a clean, production-style VPN architecture.

---

## 💡 Lessons Learned

- **VPN issues are often not VPN issues.** The handshake failures had nothing to do with WireGuard configuration — the root cause was double NAT and a broken inbound UDP path at the ISP router layer.
- **Architecture must be fixed before services can work reliably.** No amount of firewall tuning or port forwarding resolves a fundamentally broken network path upstream.
- **Test with real external networks.** Local LAN testing cannot validate true remote reachability. Mobile hotspot testing was essential to confirm the VPN was working correctly end-to-end.
- **Least-privilege applies to VPN too.** AllowedIPs was scoped to management networks only — not a full tunnel — preserving segmentation policy through the VPN.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| pfSense Setup | [02-pfsense-setup.md](02-pfsense-setup.md) |
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| Firewall Hardening | [04b-firewall-hardening.md](04b-firewall-hardening.md) |
| Lessons Learned | [06-lessons-learned.md](06-lessons-learned.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
