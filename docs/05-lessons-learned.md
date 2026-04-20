# 📖 Lessons Learned

This project taught me far more than just configuring VLANs and firewall rules — it showed me how real infrastructure behaves when theory meets production-style environments.

Building this lab forced me to troubleshoot real networking failures, design around hardware limitations, and think about security from an operational perspective rather than just a textbook one.

---

## 1. 🔥 Firewall Rule Order Matters More Than Expected

> **Key takeaway:** Security policy is only as good as its rule order.

One of the biggest lessons was understanding how pfSense evaluates firewall rules. Rules are processed **top to bottom**, and a single misplaced allow rule can completely bypass intended security controls.

I learned that security is not only about creating rules, but about designing rule order correctly and following **least-privilege principles**. This became especially important when isolating IoT devices and restricting access between VLANs.

---

## 2. 🖥️ Single NIC Labs Create Real Constraints

> **Key takeaway:** Infrastructure design is shaped by hardware constraints, not just ideal architecture diagrams.

Running pfSense virtualised with limited physical NICs created real design challenges. Using a single NIC required careful bridge design inside Proxmox and proper VLAN trunking to avoid routing bottlenecks and management access issues.

This reinforced why **dedicated management interfaces** are important in production environments — and why you should plan your physical layout before you virtualise.

---

## 3. 🌐 DNS Control Is a Security Feature

> **Key takeaway:** DNS is not just a utility — it is part of security policy enforcement.

At first, DNS felt like a simple service. I later realised that unrestricted DNS allows users and devices to bypass security boundaries by pointing to external resolvers.

Restricting DNS access using **pfSense DNS Resolver ACLs** and forcing VLANs to use internal DNS significantly improved network control and visibility. This fundamentally changed how I think about DNS as a security control.

---

## 4. 🔑 SSH Hardening Is Essential

> **Key takeaway:** Convenience and security are often in direct conflict — default to security.

Enabling SSH access was convenient, but it also created unnecessary risk. Switching from password authentication to **public-key-only authentication** made access significantly more secure and aligned the lab more closely with enterprise best practices.

This also deepened my understanding of Linux authentication, key management, and operational security hygiene.

---

## 5. 🔧 Troubleshooting Builds Real Skill

> **Key takeaway:** Fixing what breaks teaches more than initial setup ever could.

The most valuable part of this project was not the final design — it was resolving real failures as they happened:

| Issue Encountered | Area |
|---|---|
| Missing WAN DHCP lease | WAN / ISP connectivity |
| Interface `NO-CARRIER` failures | Physical / virtual NIC |
| Proxmox bridge misconfiguration | Virtualisation layer |
| VLAN access failures | Switch / trunk configuration |
| Firewall rule conflicts | pfSense policy |
| Management plane lockout risk | Security / access control |

Resolving these issues forced me to understand how the **full stack interacts**: hypervisor, switch, firewall, routing, and endpoint behaviour. That experience is what turned this from a lab exercise into practical engineering work.

---

## 6. 🔄 If I Were Starting Over

Knowing what I know now, I would approach the build differently:

| What I'd Change | Why |
|---|---|
| Plan physical NIC layout before virtualising pfSense | Avoids single-NIC routing constraints from the start |
| Allocate VLAN 99 for management from day one | Prevents management plane exposure during early setup |
| Document firewall rules as I created them | Retroactive documentation misses context and intent |
| Build a physical topology diagram before cabling | Forces you to think through failure domains early |
| Test inter-VLAN routing before adding security rules | Easier to isolate config issues from security policy issues |

---

## 💭 Final Reflection

This project changed how I approach infrastructure. Instead of thinking only in terms of configuration, I now think in terms of:

| Mindset Shift | What It Replaced |
|---|---|
| Security boundaries | "Just configure and connect" |
| Failure domains | Assuming things will work |
| Operational access control | Root access everywhere |
| Recovery planning | Build it and forget it |
| Long-term maintainability | One-time setup thinking |

That mindset shift is probably the most valuable outcome of the entire lab — and the one most applicable to working in a real engineering environment.

---

<div align="center">
<sub>Part of the <a href="../README.md">Home Lab Network Infrastructure</a> project</sub>
</div>
