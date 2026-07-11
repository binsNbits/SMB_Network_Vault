---
tags: [reference]
aliases: [RFC 1918, Reserved IPs, Private Addresses]
---

# Private IP Ranges

Private (RFC 1918) addresses are used **inside** the LAN and are not routable on the internet — the [[Router]] translates them via [[NAT]]. Foundation: [[IP Addressing]].

## RFC 1918 private space

| Range | CIDR | Addresses | Typical SMB use |
|---|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16,777,216 | Larger orgs, multi-site (`10.<site>.<vlan>.x` scheme) |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1,048,576 | Mid-size; avoids conflicts with home 192.168 nets (good for VPN) |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65,536 | Small offices, consumer defaults |

## Other reserved ranges you WILL encounter

| Range | Meaning | When you see it |
|---|---|---|
| 169.254.0.0/16 | APIPA / link-local | Client failed to get a [[DHCP]] lease — a diagnostic red flag |
| 127.0.0.0/8 | Loopback | `127.0.0.1` = "this host"; test local services |
| 100.64.0.0/10 | CGNAT | ISP shares one public IP among customers — breaks inbound port forwards → consider [[SD-WAN Appliance]] or VPN out |
| 224.0.0.0/4 | Multicast | mDNS (224.0.0.251), streaming, some discovery protocols |
| 0.0.0.0 | "Any / unspecified" | Default route target, listen-on-all-interfaces |
| 255.255.255.255 | Limited broadcast | DHCP DISCOVER packets |

## Practical scheme for an SMB (example)

| VLAN | Purpose | Subnet | Gateway |
|---|---|---|---|
| 1 | Management | 10.0.10.0/24 | 10.0.10.1 |
| 20 | Staff | 10.0.20.0/24 | 10.0.20.1 |
| 30 | VoIP | 10.0.30.0/24 | 10.0.30.1 |
| 40 | Guest | 10.0.40.0/24 | 10.0.40.1 |
| 50 | Cameras/IoT | 10.0.50.0/24 | 10.0.50.1 |
| 60 | Servers | 10.0.60.0/24 | 10.0.60.1 |

**Why this shape:** the third octet mirrors the VLAN ID — instantly readable in logs. See [[VLANs]] and [[Network Segmentation]] for why these groups are separated.

> [!tip] VPN planning
> Never use 192.168.0.0/24 or 192.168.1.0/24 at the office — remote workers' home routers use the same range and [[VPN Fundamentals|VPN]] routing breaks (overlapping subnets). Pick something like 172.22.x.x or 10.x.x.x.

Related: [[Subnet Cheat Sheet]] · [[Subnetting and CIDR]] · [[DHCP]]
