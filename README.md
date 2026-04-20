# Home Lab Network Infrastructure
# Proxmox • pfSense • VLAN Segmentation • Network Security

Built a self-hosted enterprise-style home lab using Proxmox, pfSense, and a Cisco Catalyst 2960 to develop practical network engineering skills.

This project applies CCNA-level networking concepts in a live production-style environment rather than simulations, focusing on VLAN segmentation, firewall policy enforcement, management isolation, and network security hardening.

Technologies Used
Proxmox VE
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
Network Architecture

pfSense operates as the central router and firewall, connected to the Cisco Catalyst 2960 using an 802.1Q trunk for VLAN segmentation.

Traffic is separated into dedicated VLANs to enforce security boundaries, reduce attack surface, and isolate trusted devices from user, IoT, and management networks.

VLAN Design
VLAN	Purpose	Subnet
VLAN 10	Admin Devices	192.168.10.0/24
VLAN 20	Servers / Lab	192.168.20.0/24
VLAN 30	IoT Devices	192.168.30.0/24
VLAN 40	Users / Wi-Fi Clients	192.168.40.0/24
VLAN 99	Management Network	192.168.99.0/24
Security Design

Implemented security controls include:

Inter-VLAN isolation using least-privilege firewall rules
Restricted pfSense GUI and SSH access
Management-plane isolation using VLAN 99
DNS Resolver access control lists
DNSSEC enabled
SSH key-based authentication (Public Key Only)
Reduced unnecessary service exposure
Troubleshooting Experience

Real-world issues solved during the build:

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

(Diagrams will be added here)

Why This Project

This lab is designed to turn theory into practical engineering experience by building and troubleshooting real infrastructure.

Rather than only learning concepts like VLANs, routing, and segmentation in theory, this project demonstrates them in a live environment with real operational challenges and security considerations.
