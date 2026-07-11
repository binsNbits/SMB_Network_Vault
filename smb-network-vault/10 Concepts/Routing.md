---
tags: [concept, layer3]
aliases: [Routes, Routing Table, Static Route, Default Route]
---

# Routing

Routing moves packets **between** subnets (Layer 3 of the [[OSI Model]]). A router receives a packet, looks up the destination in its **routing table**, and forwards out the matching interface. Done by the [[Router]], [[Firewall (NGFW)]], or [[Layer 3 Switch]].

## Reading a routing table

```text
S*   0.0.0.0/0 [1/0] via 203.0.113.1        ← default route ("everything else") → ISP
C    10.0.20.0/24 is directly connected, Vlan20
C    10.0.60.0/24 is directly connected, Vlan60
S    10.2.0.0/16 [1/0] via 10.255.0.2       ← static: branch office via VPN/WAN link
```

**Longest prefix wins:** a packet to 10.0.20.5 matches both /24 and the default — the /24 (more specific) is used. `C` = connected (automatic), `S` = static (you typed it), `S*` = candidate default.

## Static vs. dynamic — SMB decision

| Situation | Choice | Why |
|---|---|---|
| Single site, one exit | Statics only (often just the default) | Nothing to converge; simplest to debug |
| 2–3 sites over VPN | Statics per site subnet | Manageable by hand |
| Many sites / dual WAN failover | Dynamic (OSPF) or an [[SD-WAN Appliance]] | Hand-maintained statics stop scaling |

Setting a static (Cisco / pfSense → [[CLI Quick Reference]]):
```text
ip route 10.2.0.0 255.255.0.0 10.255.0.2      ! dest-network  mask  next-hop
ip route 0.0.0.0 0.0.0.0 203.0.113.1          ! default route
```
pfSense: *System → Routing → Static Routes* (define the gateway first).

## The #1 routing bug: asymmetric / missing return route

Traffic is **two one-way trips**. If A reaches B but B's gateway has no route *back* to A's subnet, pings die silently. Whenever you add a subnet behind a second router (e.g., a [[Layer 3 Switch]] doing inter-VLAN routing behind the firewall), the firewall needs a static route **to those VLANs via the switch**, or replies go nowhere.

## Gateway of last resort on L3 switches

An L3 switch routes between VLANs itself but still needs `ip route 0.0.0.0 0.0.0.0 <firewall>` for the internet — and the firewall needs the return statics above. This pair of routes is missed in most first L3-switch deployments.

## Diagnostics

| Tool | Shows |
|---|---|
| `traceroute` / `tracert` | Each L3 hop — where the path dies |
| `show ip route` | What the router will actually do |
| `ping` from *the router itself* (specify source interface) | Isolates forward vs. return path problems |

Related: [[Subnetting and CIDR]] · [[NAT]] · [[VPN Fundamentals]] · [[No Internet Playbook]]
