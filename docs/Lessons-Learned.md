# 💡 Lessons Learned

This document captures the most important technical and operational lessons from building and troubleshooting the home lab environment.

The most valuable experience did not come from successful deployment — it came from failure, recovery, and understanding why systems broke. This phase transformed the project from a simple lab exercise into practical infrastructure engineering experience.

---

## 📋 Table of Contents

- [Key Lessons](#key-lessons)
- [If I Were Starting Over](#if-i-were-starting-over)
- [Challenges Summary](#challenges-summary)
- [Final Reflection](#final-reflection)

---

## 📚 Key Lessons

---

### 1. 🏗️ Architecture Matters More Than Individual Fixes

> **Key takeaway:** Most problems were not caused by a single bad setting — they were caused by the overall network design.

| Problem Observed | Actual Root Cause |
|---|---|
| WireGuard handshake failures | Double NAT — not WireGuard config |
| Management access failures | Incorrect Proxmox bridge design — not GUI issues |
| Poor internet performance | FastEthernet trunk bottleneck — not pfSense or DHCP |

Fixing the underlying architecture solved these problems permanently. Fixing individual settings did not.

---

### 2. 🔌 Layer 1 Problems Can Look Like Layer 7 Problems

> **Key takeaway:** Always verify physical connectivity before assuming software or configuration issues.

CRC errors, bad cables, wrong ports, and physical NIC mismatches caused symptoms that initially appeared to be firewall or routing failures.

| Symptom | Actual Cause |
|---|---|
| Poor internet performance | Switch port CRC errors |
| Broken VLAN gateways | Incorrect physical NIC mapping |
| Complete routing failure | Trunk traffic on the wrong interface |

---

### 3. 🌐 Native VLAN Mistakes Are Severe

> **Key takeaway:** Never rely on native VLAN assumptions — use explicit tagged interfaces.

Proxmox management initially failed because untagged traffic was placed on a trunk whose native VLAN did not match the intended management network.

**Impact:**

- Proxmox GUI unreachable
- VLAN communication broken
- Gateway access failed

**Fix:** Explicit tagged management interface using `vmbr1.10` instead of relying on native VLAN behaviour. This immediately restored stable management access.

---

### 4. 🔄 Double NAT Creates Hidden Problems

> **Key takeaway:** ISP routers in the path create problems that cannot be fixed by tuning — the architecture must change.

| Problem | Caused By |
|---|---|
| WireGuard handshakes never completing | Broken inbound UDP path behind ISP router |
| Unreliable remote access | Double NAT preventing correct packet forwarding |
| Unnecessary troubleshooting complexity | ISP router masking the true network path |

**Fix:** Moving pfSense directly to the ONT as the true edge router eliminated all of these issues in a single architectural change.

---

### 5. ⚡ FastEthernet Becomes the Silent Bottleneck

> **Key takeaway:** Performance troubleshooting must include full path analysis — not just endpoint testing.

Even with correct VLANs and pfSense working properly, the network still performed poorly because the critical trunk path was running at:

```text
FastEthernet — 100 Mbps + CRC errors
```

Migrating to GigabitEthernet immediately restored full throughput. The bottleneck was invisible until the full path was analysed.

---

### 6. 🔀 One Gateway Means One Gateway

> **Key takeaway:** Split routing across multiple bridges causes instability — keep gateway paths clean and singular.

Attempting to manage routing across multiple Proxmox bridges caused major instability. Stable networking required:

- Clear WAN/LAN bridge separation
- A single defined default gateway
- Explicit tagged management path — no ambiguity

This principle applies everywhere in infrastructure design. Ambiguous routing paths create failure modes that are difficult to diagnose.

---

### 7. 🛡️ Recovery Strategy Is Part of the Design

> **Key takeaway:** Stability-first thinking prevents outages from becoming disasters.

Several changes temporarily caused complete loss of access. Recovery was only possible because of:

| Control | Why It Mattered |
|---|---|
| Console access | Provided out-of-band recovery path |
| Staged rollback planning | Allowed reverting individual changes |
| One change at a time | Isolated the cause of each failure |
| Known-good fallback paths | Prevented total lockout |

Making changes without a recovery path is not engineering — it is gambling.

---

### 8. 🔐 Security Is Mostly Good Design

> **Key takeaway:** Most security improvements came from architecture, not from adding more firewall rules.

| Security Improvement | How It Was Achieved |
|---|---|
| VLAN isolation | Network segmentation design |
| Management plane protection | Dedicated management VLAN + access control |
| Reduced WAN exposure | pfSense as edge router — direct ONT connection |
| Strong remote access | WireGuard VPN-first model |
| Hardened authentication | SSH key-only — password auth removed |
| Least-privilege routing | Explicit inter-VLAN firewall policy |

Good architecture creates good security. Rules enforcing bad architecture provide almost no real protection.

---

### 9. 📄 Documentation Prevents Repeated Pain

> **Key takeaway:** If the architecture exists only in your head, recovery under pressure becomes guesswork.

Repeated troubleshooting made one thing clear — undocumented systems are fragile systems. Documenting the following turned troubleshooting from guessing into engineering:

- VLAN design and subnet allocation
- Proxmox bridge mappings
- Firewall policy and rule intent
- Switch port roles and trunk configuration
- WAN/LAN paths and NAT behaviour

---

### 10. 🔧 Real Troubleshooting Teaches Faster Than Tutorials

> **Key takeaway:** Solving real failures builds deeper understanding than theory alone — and it is what employers actually care about.

The most valuable learning came from resolving genuine failures:

| Failure Encountered | Skill Developed |
|---|---|
| Broken inter-VLAN routing | Full-stack network path analysis |
| Lost management access | Recovery planning and console skills |
| Physical layer faults | Layer 1 diagnosis before assuming software issues |
| ISP router limitations | WAN architecture and edge routing design |
| WireGuard handshake failures | VPN architecture and NAT troubleshooting |
| Firewall rule conflicts | Policy design and rule ordering |

---

## 🔄 If I Were Starting Over

| What I'd Change | Why |
|---|---|
| Plan physical NIC layout before virtualising pfSense | Avoids single-NIC routing constraints from the start |
| Allocate VLAN 99 for management from day one | Prevents management plane exposure during early setup |
| Document firewall rules as they were created | Retroactive documentation loses context and intent |
| Build a physical topology diagram before cabling | Forces failure domain thinking before problems occur |
| Test inter-VLAN routing before adding security rules | Easier to isolate config issues from security policy issues |
| Validate trunk speed and CRC errors early | Would have caught the FastEthernet bottleneck weeks earlier |

---

## ⚠️ Challenges Summary

| Challenge | Root Cause | Resolution |
|---|---|---|
| WireGuard never handshaking | Double NAT / broken inbound UDP | Moved pfSense to WAN edge |
| Proxmox management unreachable | Native VLAN mismatch on trunk | Explicit tagged interface `vmbr1.10` |
| Poor internet throughput | FastEthernet trunk + CRC errors | Migrated trunk to GigabitEthernet |
| Routing instability | Multiple default gateways in Proxmox | Single gateway + clean bridge separation |
| Security boundaries failing | Overly broad firewall rules | Enforced least-privilege per VLAN |
| Remote access unreliable | ISP router interference | pfSense as true edge router |

---

## 💭 Final Reflection

This project started as a simple Proxmox lab and evolved into a full infrastructure environment:

| Capability | Detail |
|---|---|
| Edge routing | pfSense on ONT — direct WAN assignment |
| Network segmentation | VLAN design with least-privilege firewall policy |
| Enterprise switching | Cisco trunk and access port configuration |
| Secure remote access | WireGuard VPN-first administration |
| Performance optimisation | GigabitEthernet trunk — CRC error resolution |
| Production-style security | DNS hardening, SSH hardening, management isolation |

The most important lesson was this:

```text
Infrastructure engineering is not about getting things working.
It is about understanding why they fail — and designing systems that fail less.
```

That mindset shift is the most transferable outcome of the entire project — and the one most applicable to working in a real engineering environment.

---

## 🔗 Related Documentation

| Document | Link |
|---|---|
| Proxmox Setup | [01-proxmox-setup.md](01-proxmox-setup.md) |
| pfSense Setup | [02-pfsense-setup.md](02-pfsense-setup.md) |
| VLAN Segmentation | [03-vlan-segmentation.md](03-vlan-segmentation.md) |
| Firewall Policy | [04-firewall-policy.md](04-firewall-policy.md) |
| WireGuard VPN | [05-wireguard-vpn.md](05-wireguard-vpn.md) |

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
