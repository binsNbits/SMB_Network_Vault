---
tags: [index, moc]
aliases: [Index, MOC, Start Here]
---

# 🏠 SMB Network Knowledge Base — Home

This vault is a **neural network of knowledge** for managing SMB networks. Every note links to related notes — open **Graph View** (`Ctrl/Cmd+G`) to see the web. Follow `[[wikilinks]]` to traverse from any device to the concepts it depends on and the playbooks that use it.

**How to use this vault**
1. New to networking? Start with [[OSI Model]] → [[IP Addressing]] → [[Subnetting and CIDR]] → [[VLANs]].
2. Working on a device? Jump straight to its note under *Devices* — each note links back to prerequisite concepts.
3. Something broken? Go to *Playbooks* — symptom-driven, step-by-step, with reasons for every step.
4. Need a number fast? *Reference* holds ports, subnets, default IPs, CLI crib sheets.

---

## 📡 Devices (ordered: internet edge → endpoints)

### Edge & Connectivity
1. [[Modem]]
2. [[Router]]
3. [[Firewall (NGFW)]]
4. [[VPN Appliance]]
5. [[SD-WAN Appliance]]

### Core & Distribution
6. [[Managed Switch]]
7. [[Unmanaged Switch]]
8. [[PoE Switch]]
9. [[Layer 3 Switch]]

### Wireless
10. [[Wireless Access Point]]
11. [[Wireless LAN Controller]]
12. [[Mesh Node and Range Extender]]

### Servers & Storage
13. [[Server]]
14. [[NAS]]
15. [[SAN]]
16. [[Backup Appliance]]

### Endpoints & Peripherals
17. [[Desktops and Laptops]]
18. [[VoIP Phones and PBX]]
19. [[Network Printer]]
20. [[IP Cameras and NVR]]
21. [[POS Terminals]]

### Supporting Infrastructure
22. [[UPS]]
23. [[Network Rack and Patch Panel]]
24. [[DNS DHCP Print Servers]]
25. [[IoT Devices]]

---

## 🧠 Concepts (foundations every device note builds on)

| Layer | Notes |
|---|---|
| Models | [[OSI Model]] |
| Addressing | [[IP Addressing]] · [[Subnetting and CIDR]] · [[NAT]] |
| Core services | [[DHCP]] · [[DNS]] |
| Layer 2 | [[Switching and ARP]] · [[VLANs]] · [[Spanning Tree Protocol]] · [[Power over Ethernet]] |
| Layer 3 | [[Routing]] |
| Wireless | [[Wi-Fi Fundamentals]] |
| Security | [[Firewall Concepts]] · [[VPN Fundamentals]] · [[Network Segmentation]] · [[Email Authentication]] |
| Operations | [[Device Access Methods]] · [[SNMP and Monitoring]] · [[QoS]] · [[High Availability]] · [[Backup Strategy]] |

---

## 🛠 Playbooks (symptom → resolution)

- [[New Site Deployment]] — greenfield build, end to end
- [[No Internet Playbook]] — total outage triage
- [[Slow Network Playbook]] — performance degradation
- [[Wi-Fi Problems Playbook]] — coverage, roaming, auth failures
- [[VPN Troubleshooting Playbook]] — tunnels down or flapping
- [[Email Deliverability Playbook]] — mail in spam, bounces, domain spoofing
- [[Firmware Update Procedure]] — safe upgrade workflow
- [[Configuration Backup Procedure]] — before you touch anything
- [[Security Incident Basics]] — first response

---

## 📖 Reference (fast lookup)

- [[Common Ports]] — protocol/port table
- [[Private IP Ranges]] — RFC 1918 and reserved space
- [[Subnet Cheat Sheet]] — mask ↔ hosts ↔ CIDR
- [[CLI Quick Reference]] — Cisco-style command crib
- [[Email Authentication Cheat Sheet]] — SPF/DKIM/DMARC syntax, lookups, providers
- [[Vendor Default IPs and Logins]] — factory access details
- [[Documentation Standards]] — labeling, IPAM, diagrams

## 📐 Templates

- [[Device Note Template]] — for documenting new gear
- [[Site Note Template]] — for documenting a location
- [[Email Deliverability Engagement Template]] — per-client deliverability fix worksheet

> [!tip] Golden rules
> 1. **Back up config before every change** → [[Configuration Backup Procedure]]
> 2. **Change one thing at a time**, verify, then proceed.
> 3. **Document as you go** → [[Documentation Standards]]
