---
tags: [concept, fundamentals, layer3]
aliases: [Subnetting, CIDR, Subnet Mask]
---

# Subnetting and CIDR

Subnetting divides IP space into isolated broadcast domains. CIDR (`/24`) notation counts the network bits of the mask. Quick tables: [[Subnet Cheat Sheet]].

## Why subnet at all?

1. **Containment** — broadcasts ([[DHCP]] discovers, ARP) stay inside the subnet; one chatty device can't flood the whole company.
2. **Security boundaries** — traffic *between* subnets must cross a [[Router]]/[[Layer 3 Switch]], where [[Firewall Concepts|rules]] can inspect it → the mechanism behind [[Network Segmentation]].
3. **Organization** — logs and IPAM stay readable → [[Documentation Standards]].

## Reading a subnet (worked example)

`10.0.20.0/24` (mask 255.255.255.0):

| Value | Address | Meaning |
|---|---|---|
| Network | 10.0.20.0 | Identifier — not assignable |
| First usable | 10.0.20.1 | Convention: the gateway |
| Last usable | 10.0.20.254 | |
| Broadcast | 10.0.20.255 | "All hosts here" — not assignable |
| Usable hosts | 254 | 2⁸ − 2 |

## Sizing decisions (conditionals)

- **Default: /24 per [[VLANs|VLAN]].** 254 hosts fits most SMB segments and the math is trivial.
- **Point-to-point link (router↔router)?** → /30 (2 usable) or /31.
- **More than ~200 Wi-Fi clients on one SSID?** → /23 (510 hosts) *or better*: split into more VLANs — huge broadcast domains degrade Wi-Fi ([[Wi-Fi Fundamentals]]).
- **Planning multi-site?** Allocate non-overlapping blocks up front (site1 = 10.1.0.0/16, site2 = 10.2.0.0/16). Overlaps break site-to-site [[VPN Fundamentals|VPNs]] and are painful to renumber later.

## Classic symptoms of subnetting mistakes

| Symptom | Likely cause |
|---|---|
| Two hosts on same switch can't talk | Different subnets/masks — no router between them |
| Host reaches some LAN IPs but not others | Wrong mask (e.g., /25 configured where /24 intended — half the range appears "remote") |
| VPN connects but no traffic flows | Overlapping subnets at both ends → [[VPN Troubleshooting Playbook]] |
| Intermittent conflicts | DHCP pool overlaps someone's static assignments → [[DHCP]] |

Related: [[IP Addressing]] · [[Private IP Ranges]] · [[Routing]]
