---
tags: [concept, fundamentals, layer3]
aliases: [IP, IPv4, IP Address]
---

# IP Addressing

An IP address is a **logical, routable identity** (Layer 3 of the [[OSI Model]]) — unlike a MAC address, which is burned-in and only meaningful on the local segment ([[Switching and ARP]]).

## Anatomy of an IPv4 address

```text
192.168.20.37 /24
└─network──┘└host┘
Mask 255.255.255.0 → first 24 bits identify the NETWORK, last 8 the HOST.
```

Every configured host needs three values (usually delivered by [[DHCP]]):

| Value | Purpose | If wrong |
|---|---|---|
| IP + mask | Identity and subnet boundary | Duplicate-IP conflicts; wrong-subnet isolation |
| Default gateway | Where to send **off-subnet** traffic → [[Router]] | LAN works, internet dead |
| [[DNS]] server | Name → IP resolution | "Internet down" but pings to 8.8.8.8 succeed |

## The core decision every host makes

For each packet: *is the destination in my subnet?* (AND both IPs with the mask, compare — see [[Subnet Cheat Sheet]])
- **Yes** → deliver directly: ARP for destination's MAC → [[Switching and ARP]]
- **No** → send to default gateway → [[Routing]] takes over

This single rule explains most "can reach X but not Y" tickets.

## Static vs. dynamic — decision rule

| Assign statically (or via DHCP reservation) | Leave dynamic |
|---|---|
| Infrastructure: [[Router]], switches, [[Wireless Access Point|APs]] | Staff [[Desktops and Laptops]] |
| [[Server]], [[NAS]], [[Network Printer]] | Guest/BYOD devices |
| [[IP Cameras and NVR]], [[UPS]] management cards | Phones, tablets |

**Prefer DHCP reservations over device-side static config** — one place to manage, no fat-finger conflicts, survives device re-images. Record everything in IPAM → [[Documentation Standards]].

## Public vs. private

Private ranges ([[Private IP Ranges]]) live behind [[NAT]]; your public IP is on the [[Router]]/[[Firewall (NGFW)]] WAN interface (check with `curl ifconfig.me` or the WAN status page). Static public IPs (from the ISP) matter when hosting inbound services or site-to-site [[VPN Fundamentals|VPNs]].

## IPv6 in SMBs (brief)

128-bit, no NAT needed, addresses like `2001:db8::1`. Most SMBs run dual-stack passively (ISP-delegated) or disable it deliberately. **Rule:** if you don't manage IPv6, make sure the [[Firewall (NGFW)]] filters it too — an unfiltered IPv6 path bypasses all your IPv4 rules.

Related: [[Subnetting and CIDR]] · [[DHCP]] · [[NAT]] · [[No Internet Playbook]]
