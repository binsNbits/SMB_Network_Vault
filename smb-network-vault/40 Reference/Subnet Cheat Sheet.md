---
tags: [reference]
aliases: [Subnet Table, CIDR Table, Netmask Table]
---

# Subnet Cheat Sheet

Full explanation: [[Subnetting and CIDR]]. Usable hosts = 2^(host bits) − 2 (network + broadcast reserved).

| CIDR | Subnet Mask | Total IPs | Usable Hosts | Typical use |
|---|---|---|---|---|
| /30 | 255.255.255.252 | 4 | 2 | Point-to-point links (router↔router) |
| /29 | 255.255.255.248 | 8 | 6 | Tiny DMZ, ISP static blocks |
| /28 | 255.255.255.240 | 16 | 14 | Small server segment |
| /27 | 255.255.255.224 | 32 | 30 | Management [[VLANs|VLAN]] |
| /26 | 255.255.255.192 | 64 | 62 | Department subnet |
| /25 | 255.255.255.128 | 128 | 126 | Medium office |
| **/24** | **255.255.255.0** | **256** | **254** | **The SMB workhorse — one VLAN per /24** |
| /23 | 255.255.254.0 | 512 | 510 | Large Wi-Fi client pool |
| /22 | 255.255.252.0 | 1024 | 1022 | Big guest network |
| /16 | 255.255.0.0 | 65,536 | 65,534 | Summarization only — never one flat /16 LAN |
| /8 | 255.0.0.0 | 16.7M | — | 10.0.0.0/8 allocation planning |

## Increment trick (for /25–/30)

Subtract the "interesting octet" from 256 to get the block size:
- /26 → 256−192 = **64** → networks start at .0, .64, .128, .192
- /27 → 256−224 = **32** → .0, .32, .64, .96 …
- /30 → 256−252 = **4** → .0, .4, .8 …

**Example:** Which subnet is 192.168.1.75/26 in? Blocks of 64 → .64 network, range .64–.127, broadcast .127, usable .65–.126.

## Quick answers

- **First usable** = network + 1 (usually the gateway → [[Router]])
- **Last usable** = broadcast − 1
- **Two hosts same subnet?** Same result when you AND each IP with the mask → they talk directly via [[Switching and ARP]]. Different result → traffic goes to the gateway → [[Routing]].

Related: [[Private IP Ranges]] · [[IP Addressing]] · [[DHCP]]
