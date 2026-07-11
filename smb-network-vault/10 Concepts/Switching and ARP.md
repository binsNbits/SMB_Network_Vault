---
tags: [concept, layer2]
aliases: [ARP, MAC Address, MAC Table, Ethernet Switching]
---

# Switching and ARP

How frames actually move inside a subnet (Layer 2 of the [[OSI Model]]).

## MAC addresses

48-bit hardware IDs (`AA:BB:CC:DD:EE:FF`); first half identifies the vendor (OUI) — useful for identifying mystery devices in the [[DHCP]] lease list. MACs matter only on the local segment; every [[Routing|routed]] hop rewrites them while IPs stay constant end-to-end.

## How a switch forwards

1. Frame arrives on port 3 with source MAC `A` → switch **learns**: `A` lives on port 3 (MAC address table).
2. Destination MAC known? → forward out **only** that port (unicast).
3. Unknown / broadcast (`FF:FF:…`)? → **flood** out all ports in that [[VLANs|VLAN]].

`show mac address-table` ([[CLI Quick Reference]]) answers "which physical port is this device on?" — indispensable for hunting rogue devices and mapping undocumented cabling ([[Documentation Standards]]).

## ARP — gluing L3 to L2

Host knows the destination *IP* but needs its *MAC*:

```text
"Who has 10.0.20.1? Tell 10.0.20.51"   (broadcast)
"10.0.20.1 is at aa:bb:cc:11:22:33"    (reply, cached in ARP table)
```

View caches: `arp -a` (Windows) / `ip neigh` (Linux) / `show ip arp` (Cisco).

## Failure modes & security

| Issue | Symptom | Response |
|---|---|---|
| Stale ARP after IP move | New device unreachable for minutes | Clear ARP cache on hosts/gateway |
| Duplicate IP | "Address conflict" popups; two MACs answer one IP | Find both via `show mac address-table`, fix assignment ([[IP Addressing]]) |
| ARP spoofing (MITM) | Attacker answers ARP for the gateway, intercepts traffic | Dynamic ARP Inspection + DHCP snooping on [[Managed Switch]]; segment untrusted gear → [[Network Segmentation]] |
| Unicast flooding | Random slowness on large flat networks | Symptom of asymmetric paths or tiny MAC table timeouts — another argument for [[VLANs]] |
| MAC flapping (log message) | Same MAC seen on two ports rapidly | A loop ([[Spanning Tree Protocol]]) or a device moving between AP and cable |

## Duplex/speed (physical-adjacent)

Modern gigabit auto-negotiates. A **duplex mismatch** (one side forced full, other auto → half) still causes brutal, intermittent slowness with late collisions in `show interfaces`. Rule: leave both ends on auto, or force **both** ends.

Related: [[Managed Switch]] · [[Unmanaged Switch]] · [[OSI Model]] · [[Slow Network Playbook]]
