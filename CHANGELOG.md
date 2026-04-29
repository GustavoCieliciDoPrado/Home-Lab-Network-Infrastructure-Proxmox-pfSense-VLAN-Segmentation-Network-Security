# 📋 Changelog

This file tracks the progression of the Home Lab Network Infrastructure project as the environment evolved from a basic virtualisation setup into a production-style segmented network with security-focused design.

The goal is continuous improvement, practical troubleshooting, and applying real-world infrastructure engineering principles using live hardware and services.

---

## 📊 Project Progress

| Phase | Description | Status |
|---|---|---|
| Phase 1 | Proxmox Deployment | ✅ Complete |
| Phase 2 | Cisco Switch Integration & VLAN Design | ✅ Complete |
| Phase 3 | pfSense Deployment & Routing | ✅ Complete |
| Phase 4 | Troubleshooting & Management Recovery | ✅ Complete |
| Phase 5 | Dual NIC Architecture Upgrade | ✅ Complete |
| Phase 6 | Security Hardening | ✅ Complete |
| Phase 7 | Edge Routing Migration & ISP Architecture Redesign | ✅ Complete |
| Phase 8 | WireGuard VPN Deployment | ✅ Complete |
| Phase 9 | IDS / IPS | 🔄 Planned |
| Phase 10 | Monitoring and Logging | 🔄 Planned |
| Phase 11 | Dynamic DNS | 🔄 Planned |
| Phase 12 | Hardware Improvements | 🔄 Planned |

---

## ✅ Phase 1 — Proxmox Deployment
`April 2026`

### Completed

- Installed Proxmox VE as a bare-metal hypervisor on dedicated hardware (i7-8700K, 8GB RAM, 1TB SSD)
- Created initial Ubuntu Server virtual machines
- Established SSH access for Linux administration
- Configured basic virtual networking using Proxmox bridges
- Built initial foundation for pfSense deployment

### Outcome

> Created the virtualisation platform required for all later networking, firewall, and segmentation work.

---

## ✅ Phase 2 — Cisco Switch Integration & VLAN Design
`April 2026`

### Completed

- Integrated Cisco Catalyst 2960 switch into the lab environment
- Designed and implemented VLAN segmentation strategy:

| VLAN | Name | Subnet |
|---|---|---|
| **VLAN 10** | Admin | `192.168.10.0/24` |
| **VLAN 20** | Lab | `192.168.20.0/24` |
| **VLAN 30** | IoT | `192.168.30.0/24` |
| **VLAN 40** | Users | `192.168.40.0/24` |
| **VLAN 99** | Management | `192.168.99.0/24` |

- Configured access ports and trunk links on the 2960
- Built initial switch-to-Proxmox trunk design

### Outcome

> Moved from flat networking into structured Layer 2 segmentation.

---

## ✅ Phase 3 — pfSense Deployment & Routing
`April 2026`

### Completed

- Deployed pfSense Community Edition as a virtual firewall / router inside Proxmox
- Configured WAN and LAN interfaces
- Created VLAN subinterfaces and gateway assignments per VLAN
- Enabled DHCP services per VLAN with dedicated address ranges
- Restored internet access across all segmented networks via NAT
- Implemented inter-VLAN routing centralised through pfSense

### Outcome

> Established centralised routing, firewall enforcement, and network segmentation.

---

## ✅ Phase 4 — Troubleshooting & Management Recovery
`April 2026`

This phase was unplanned but became one of the most valuable parts of the project. Real infrastructure failures were diagnosed and resolved across the full stack.

### Issues Resolved

| Issue | Cause | Resolution |
|---|---|---|
| DHCP failures | Incorrect VLAN path and interface mismatch | Validated switch access ports and pfSense DHCP scopes |
| `NO-CARRIER` interface errors | Incorrect bridge assignment and NIC mapping | Rebuilt Proxmox bridge design and verified physical NIC mapping |
| Switch trunk failure | Trunk configured on wrong physical port | Verified Layer 1 physical connectivity and corrected trunk placement |
| Proxmox management lockout | Management migration attempted before stable path validation | Restored stable management access on `192.168.1.100` |
| pfSense WebGUI login loop | Browser session corruption after GUI protocol and VLAN changes | Cleared browser site data and restarted GUI services |
| SSH hardening validation | Password login intentionally disabled | Verified successful public-key-only authentication |

> 💡 **Key insight:** Layer 1 physical connectivity errors frequently present as Layer 2 or Layer 3 failures. Always verify the physical layer before diagnosing routing or firewall issues. Browser state can also mimic authentication failures — clear application state before assuming infrastructure is broken.

### Outcome

> Stabilised management plane and reinforced operational troubleshooting discipline across the full stack.

---

## ✅ Phase 5 — Dual NIC Architecture Upgrade
`April 2026`

### Completed

- Installed second PCIe NIC in the Proxmox host
- Separated WAN and LAN traffic onto dedicated physical interfaces

### Final Architecture

| NIC | Bridge | Role | Connected To |
|---|---|---|---|
| NIC 1 | `vmbr0` | WAN + Proxmox management | Home router / ISP |
| NIC 2 | `vmbr1` | LAN trunk — all internal VLANs | Cisco Catalyst 2960 |

### Improvements Delivered

| Area | Before | After |
|---|---|---|
| WAN / LAN separation | Shared single NIC | Dedicated physical interfaces |
| VLAN stability | Complex single-NIC trunking | Clean dedicated trunk on NIC 2 |
| Management access | Shared with data traffic | Isolated on NIC 1 |
| Troubleshooting clarity | Difficult to isolate faults | Clear separation of failure domains |
| Architecture alignment | Single-NIC workaround | Production-style dual-NIC design |

> This was the single biggest infrastructure improvement of the entire project. Adding a second NIC removed an entire class of complexity that no amount of software configuration could fully resolve.

### Outcome

> The environment moved from a constrained single-NIC lab into a cleaner, production-style architecture with proper WAN / LAN separation.

---

## ✅ Phase 6 — Security Hardening
`April 2026`

### DNS Security

| Control | Detail |
|---|---|
| DNS Resolver ACLs | Restricted resolver access per VLAN |
| Internal DNS enforcement | All VLANs forced to use pfSense resolver |
| External DNS bypass | Blocked via firewall rules |
| DNSSEC | Enabled |

### Administrative Access Security

| Control | Detail |
|---|---|
| pfSense GUI access | Restricted to Admin VLAN 10 only |
| Admin VLAN isolation | VLAN 10 designated as primary management network |
| Management plane separation | VLAN 99 dedicated to infrastructure management |

### SSH Hardening

| Control | Detail |
|---|---|
| Authentication method | RSA 4096-bit public key only |
| Password authentication | Disabled entirely |
| SSH access scope | Restricted to Admin VLAN only |
| Attack surface | Significantly reduced |

> 📄 Full details: [06 — SSH Hardening](docs/06-ssh-hardening.md)

### Firewall Policy Design

| Control | Detail |
|---|---|
| Inter-VLAN access | Least-privilege rules enforced |
| IoT isolation | VLAN 30 blocked from all internal segments |
| Server network protection | VLAN 20 restricted from user and IoT access |
| User VLAN restrictions | VLAN 40 limited to HTTP/HTTPS outbound only |
| Default policy | Deny all traffic unless explicitly allowed |

> 📄 Full details: [04 — Firewall Policy](docs/04-firewall-policy.md)

### Outcome

> Moved the lab from functional networking into production-style security design with enforced least-privilege access across all zones.

---

## ✅ Phase 7 — Edge Routing Migration & ISP Architecture Redesign
`April 2026`

This phase eliminated the ISP double-NAT constraint and moved pfSense to the true network edge — a prerequisite for stable VPN deployment and proper remote access.

### Completed

- Removed ISP router from the routing path entirely
- Connected pfSense WAN interface directly to Openreach ONT via PPPoE
- Configured pfSense as the sole edge router, firewall, DHCP server, and future VPN endpoint
- Converted EE Hub from router to Wi-Fi access point only (VLAN 40)
- Eliminated double-NAT and resolved associated routing instability

### Issues Resolved During Migration

| Issue | Cause | Resolution |
|---|---|---|
| WireGuard handshake failures | ISP router intercepting and NATting VPN traffic | Moved pfSense to edge, removing ISP router from path |
| Double-NAT routing conflicts | pfSense sitting behind ISP router | Direct PPPoE connection to ONT |
| VLAN 40 Wi-Fi disruption | EE Hub role change during migration | Reconfigured as access point, preserved household connectivity |
| FastEthernet trunk bottleneck | Critical trunk running on `Fa0/1` (100Mbps) | Migrated trunk to GigabitEthernet uplink |
| CRC errors and packet corruption | Physical fault on `Fa0/1` port | Diagnosed via `show interfaces`, moved to clean Gi port |

> 💡 **Key insight:** ISP routers fundamentally block certain traffic types including VPN handshakes. Moving to edge routing is not just a performance improvement — it removes an entire category of constraints that cannot be worked around in software.

### Outcome

> pfSense now operates as the true network edge. All routing, NAT, firewall enforcement, and VPN termination is handled directly by pfSense with no upstream interference.

---

## ✅ Phase 8 — WireGuard VPN Deployment
`April 2026`

### Completed

- Installed and configured WireGuard VPN server on pfSense
- Generated server keypair and configured WireGuard interface and tunnel addressing
- Added MacBook as a remote peer with dedicated allowed IPs
- Configured pfSense firewall rules to permit WireGuard tunnel traffic
- Configured routing to allow remote peer access to management VLAN and infrastructure

### Validation

| Test | Method | Result |
|---|---|---|
| Tunnel establishment | WireGuard handshake confirmation | ✅ Successful |
| Remote management access | SSH to Proxmox and pfSense over tunnel | ✅ Confirmed |
| True external validation | Mobile hotspot — no local network involvement | ✅ Confirmed |
| DNS resolution over tunnel | Internal hostname resolution via pfSense | ✅ Confirmed |

> 💡 **Key insight:** VPN handshake failures that present as configuration issues are often caused by upstream NAT interference. Resolving the edge routing architecture in Phase 7 was a prerequisite for this phase to succeed.

### Outcome

> Secure remote administrative access to the full lab infrastructure is now operational and validated from an external network.

---

## 🔄 Current Focus

- [ ] Final firewall policy refinement and rule cleanup
- [ ] Administrative access review
- [ ] Management path verification
- [ ] Dynamic DNS deployment

---

## 🗺️ Planned Phases

### Phase 9 — IDS / IPS

| Item | Detail |
|---|---|
| Suricata | Primary IDS/IPS deployment target |
| Snort | Secondary evaluation |
| Objective | Threat visibility, intrusion detection and monitoring |

---

### Phase 10 — Monitoring and Logging

| Item | Detail |
|---|---|
| Grafana / Zabbix / LibreNMS | Traffic and performance monitoring across all VLANs |
| Syslog centralisation | Aggregate logs from all network components |
| Alerting | Threshold-based notification setup |

---

### Phase 11 — Dynamic DNS

| Item | Detail |
|---|---|
| Objective | Maintain stable remote access regardless of ISP WAN IP changes |
| Integration | pfSense DDNS client pointing to chosen provider |

---

### Phase 12 — Hardware Improvements

| Item | Detail |
|---|---|
| Backup automation | Scheduled Proxmox VM snapshots and offsite backup |
| DNS hardening | Further resolver hardening and DNSSEC validation |
| Objective | Operational resilience and recovery planning |

---

## 🔧 Current State

| Component | Status |
|---|---|
| Proxmox management access | ✅ Stable |
| pfSense GUI access | ✅ Stable |
| VLAN segmentation | ✅ Operational |
| Inter-VLAN routing | ✅ Operational |
| DHCP services | ✅ Operational per VLAN |
| Internet access | ✅ Across all VLANs |
| SSH public-key-only access | ✅ Enforced |
| Firewall policy enforcement | ✅ Active |
| Dual NIC WAN / LAN separation | ✅ Complete |
| PPPoE edge routing | ✅ Complete |
| EE Hub converted to access point | ✅ Complete |
| WireGuard VPN | ✅ Operational |

---

## 💡 Operational Lessons Learned

- **Stable management access comes before perfect segmentation.** Getting locked out mid-build is a real risk — verify the management path before making changes.
- **Layer 1 failures often appear as Layer 3 problems.** Physical verification prevents hours of wasted logical troubleshooting.
- **Browser state can mimic authentication failures.** Clear application state before assuming infrastructure is broken.
- **Additional hardware can simplify architecture more than software workarounds.** The dual NIC upgrade solved problems that configuration alone could not.
- **ISP routers impose hidden constraints.** Moving to edge routing removes an entire class of problems — NAT traversal, VPN handshakes, and routing stability all improved immediately.
- **Clean configuration discipline prevents future operational pain.** Document as you build — retroact
