# 📋 Changelog

This file tracks the progression of the Home Lab Network Infrastructure project as the environment evolves from a basic virtualisation setup into a production-style segmented network with security-focused design.

The goal is continuous improvement, practical troubleshooting, and applying real-world infrastructure engineering principles using live hardware and services.

---

## 📊 Project Progress

| Phase | Description | Status |
|---|---|---|
| Phase 1 | Proxmox Foundation | ✅ Complete |
| Phase 2 | Network Infrastructure Build | ✅ Complete |
| Phase 3 | Security Hardening | ✅ Complete |
| Phase 4 | VPN Deployment | 🔄 Planned |
| Phase 5 | IDS / IPS | 🔄 Planned |
| Phase 6 | Monitoring and Logging | 🔄 Planned |
| Phase 7 | Hardware Improvements | 🔄 Planned |

---

## ✅ Phase 1 — Proxmox Foundation
`April 2026`

### Completed

- Installed Proxmox VE on dedicated bare-metal hardware
- Created Ubuntu Server virtual machines
- Configured SSH access to Linux VMs
- Developed Linux administration skills across filesystem navigation, permissions management, CLI operations, and service management

### Real-World Troubleshooting

| Issue | Tools Used | Resolution |
|---|---|---|
| Windows boot failure | `SFC`, `CHKDSK` | Diagnosed and recovered from boot corruption and filesystem issues |

### Outcome

> Established the virtualisation platform and management environment for the full lab build.

---

## ✅ Phase 2 — Network Infrastructure Build
`April 2026`

### Completed

- Integrated Cisco Catalyst 2960 switch into the lab environment
- Configured VLAN segmentation across the network
- Built Proxmox virtual networking using `vmbr0` and `vmbr1`
- Deployed pfSense Community Edition as virtual firewall and router

### pfSense Configuration

| Component | Details |
|---|---|
| WAN / LAN | Interface assignment and routing |
| DHCP | Per-VLAN address assignment |
| VLAN Interfaces | 802.1Q subinterfaces configured |
| Inter-VLAN Routing | pfSense routing between segments |
| NAT | Outbound internet access |
| DNS Resolver | Internal name resolution |

### Real-World Troubleshooting

| Issue | Area | Resolution |
|---|---|---|
| WAN DHCP failures — missing lease | WAN / ISP connectivity | Resolved interface and ISP handoff config |
| Interface `NO-CARRIER` failures | Physical / virtual NIC | Identified and corrected bridge assignments |
| Misconfigured bridges and network interfaces | Proxmox networking | Rebuilt virtual bridge layout |
| Single-NIC routing limitations | Hardware constraints | Redesigned traffic flow within constraints |

### Outcome

> Created a working segmented network architecture with functional routing and firewall control.

---

## ✅ Phase 3 — Security Hardening
`April 2026`

### DNS Security

| Control | Detail |
|---|---|
| DNS Resolver ACLs | Restricted resolver access by VLAN |
| Internal DNS enforcement | VLANs forced to use pfSense resolver |
| Reduced resolver abuse risk | External DNS bypassing blocked |

### Administrative Access Security

| Control | Detail |
|---|---|
| pfSense GUI access | Restricted to Admin VLAN only |
| Admin VLAN isolation | VLAN 10 designated for management |
| Management plane separation | VLAN 99 dedicated management network |

### SSH Hardening

| Control | Detail |
|---|---|
| Public key authentication | RSA 4096-bit key pair deployed |
| Password authentication | Disabled entirely |
| SSH access scope | Restricted to Admin VLAN only |
| Management attack surface | Significantly reduced |

> 📄 Full details: [06 — SSH Hardening](docs/06-ssh-hardening.md)

### Firewall Policy Design

| Control | Detail |
|---|---|
| Inter-VLAN access | Least-privilege rules enforced |
| IoT isolation | VLAN 30 blocked from all internal segments |
| Server network protection | VLAN 20 restricted from user and IoT access |
| User VLAN restrictions | VLAN 40 limited to HTTP/HTTPS outbound only |
| Default policy | Deny all traffic unless explicitly allowed |

### Outcome

> Moved the lab from functional networking into production-style security design.

---

## 🔄 Current Focus — Final pfSense Hardening

- [ ] Final firewall policy refinement
- [ ] Administrative access review
- [ ] Rule cleanup and validation
- [ ] Management path verification

---

## 🗺️ Planned Phases

### Phase 4 — VPN Deployment

| Item | Detail |
|---|---|
| WireGuard | Primary VPN deployment target |
| OpenVPN | Secondary evaluation |
| Objective | Secure remote administrative access and management testing |

### Phase 5 — IDS / IPS

| Item | Detail |
|---|---|
| Suricata | Primary IDS/IPS deployment target |
| Snort | Secondary evaluation |
| Objective | Threat visibility, intrusion detection and monitoring |

### Phase 6 — Monitoring and Logging

| Item | Detail |
|---|---|
| Syslog centralisation | Aggregate logs from all network components |
| Traffic monitoring | Visibility across VLANs |
| Alerting | Threshold-based notification setup |

### Phase 7 — Hardware Improvements

| Item | Detail |
|---|---|
| Dedicated second NIC | Physical interface separation |
| Improved physical layout | Reduced reliance on virtual routing |
| Objective | Remove single-NIC constraints from current design |

---

## 🏁 Long-Term Goal

Build a production-style infrastructure lab that demonstrates practical skills across:

| Discipline | Focus Area |
|---|---|
| Datacentre Operations | Hardware, virtualisation, physical infrastructure |
| Network Engineering | Routing, switching, segmentation, VPN |
| Systems Administration | Linux, Windows, service management |
| Infrastructure Security | Firewall policy, hardening, IDS/IPS |
| Enterprise Troubleshooting | Real failure diagnosis and recovery |

> This project is designed to turn CCNA theory into operational engineering experience.

---

<div align="center">
<sub>Part of the <a href="README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
