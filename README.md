# Home Lab Network Infrastructure
# Proxmox • pfSense • VLAN Segmentation • Network Security
<br>
Built a self-hosted enterprise-style home lab using Proxmox, pfSense, and a Cisco Catalyst 2960 to develop practical network engineering skills.
<br>
This project applies CCNA-level networking concepts in a live production-style environment rather than simulations, focusing on VLAN segmentation, firewall policy enforcement, management isolation, and network security hardening.
<br>
<h2>Technologies Used</h2>
<li>Proxmox VE
pfSense Community Edition
Cisco Catalyst 2960 Switch
Ubuntu Server
VLANs (802.1Q)
SSH
DNS Resolver
Firewall Rules
DHCP
NAT
Linux CLI Administration
</li>
<br>
<h2>Network Architecture</h2>
<br>
pfSense operates as the central router and firewall, connected to the Cisco Catalyst 2960 using an 802.1Q trunk for VLAN segmentation.
<br>
Traffic is separated into dedicated VLANs to enforce security boundaries, reduce attack surface, and isolate trusted devices from user, IoT, and management networks.
<br>
<h2>VLAN Design</h2>
<li>
VLAN	Purpose	Subnet <br>
VLAN 10	Admin Devices	192.168.10.0/24 <br>
VLAN 20	Servers / Lab	192.168.20.0/24 <br>
VLAN 30	IoT Devices	192.168.30.0/24 <br>
VLAN 40	Users / Wi-Fi Clients	192.168.40.0/24 <br>
VLAN 99	Management Network	192.168.99.0/24 <br>
</li>
<h2>Security Design</h2>

Implemented security controls include:
<li>
Inter-VLAN isolation using least-privilege firewall rules
Restricted pfSense GUI and SSH access
Management-plane isolation using VLAN 99
DNS Resolver access control lists
DNSSEC enabled
SSH key-based authentication (Public Key Only)
Reduced unnecessary service exposure
</li>
<br>
<h2>Troubleshooting Experience</h2>

Real-world issues solved during the build:
<li>
DHCP failures (missing WAN lease)
Interface link failures (NO-CARRIER)
Proxmox bridge misconfiguration
Single-NIC routing bottlenecks
Firewall rule order conflicts
VLAN management access restrictions
Current Focus
Final pfSense hardening
VPN deployment (WireGuard / OpenVPN)
IDS/IPS deployment (Suricata / Snort)
Monitoring and logging
Dedicated second NIC implementation
Network Diagrams
</li>
(Diagrams will be added here)

<h2>Why This Project</h2>
<br>
This lab is designed to turn theory into practical engineering experience by building and troubleshooting real infrastructure.
<br>
Rather than only learning concepts like VLANs, routing, and segmentation in theory, this project demonstrates them in a live environment with real operational challenges and security considerations.
