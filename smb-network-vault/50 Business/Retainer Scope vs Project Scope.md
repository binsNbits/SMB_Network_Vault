---
tags: [business, ops, networking]
aliases: [Networking Scope, Scope Boundary, What the Retainer Covers]
---

# Retainer Scope vs Project Scope

Networking splits into **retainer scope (keep-lights-on)** vs. **project scope (design/build)**. The dividing line: **if it requires new hardware, cabling, or design work, it's a project** ([[Project Pricing]]).

## Included in the retainer (Core and up)

- Troubleshooting connectivity: Wi-Fi drops, "internet is down," slow-network complaints → [[No Internet Playbook]], [[Slow Network Playbook]], [[Wi-Fi Problems Playbook]]
- Config changes on **existing** gear: firewall rules and port forwards ([[Firewall (NGFW)]]), Wi-Fi password rotations, guest network settings ([[Wireless Access Point]]), DHCP reservations ([[DHCP]]), DNS records ([[DNS]])
- Firmware/patching of centrally manageable network devices → [[Firmware Update Procedure]]
- VPN user adds/removes during onboarding/offboarding ([[VPN Fundamentals]])
- Printer/scanner/VoIP connectivity issues ([[Network Printer]], [[VoIP Phones and PBX]])
- **Vendor wrangling** — calling the ISP, phone vendor, copier company on the client's behalf. A big share of real-world "networking" work, explicitly in the Core tier ([[Retainer Tiers]])
- Documenting the network (IPs, [[VLANs]], credentials, ISP account info) per [[Documentation Standards]]

## Excluded — priced as projects

Office network setup: $2,500–$8,000 + hardware at cost+15% ([[Project Pricing]]):

- New or renovated office buildouts: cabling, switch/AP placement, rack work → [[New Site Deployment]], [[Network Rack and Patch Panel]]
- Wholesale equipment refreshes: consumer router → business [[Firewall (NGFW)]], adding [[Managed Switch|managed switches]]/APs
- Site-to-site VPNs, multi-site WAN design ([[VPN Appliance]], [[SD-WAN Appliance]])
- Structural redesigns: [[Network Segmentation|segmentation]]/[[VLANs|VLAN]] projects, new server room

> [!tip] Boundary rule of thumb
> A change an experienced tech completes **remotely or in under ~1 hour on-site with existing equipment** = retainer. **Anything with a bill of materials** = project quote. This protects the ~2–5 hrs/month per-client time budget that makes the solo model work ([[Solo Operations and Boundaries]]).
