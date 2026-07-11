---
tags: [device, core, layer2]
aliases: [Dumb Switch, Desktop Switch]
---

# Unmanaged Switch

**Role:** plug-and-play port multiplication. Learns MACs and forwards frames ([[Switching and ARP]]) — nothing else. No config, no IP, no VLAN awareness, no [[Spanning Tree Protocol|STP]] participation, no visibility.

**Position:** the far edge — under a desk, in a conference room, behind a TV. Fed from one [[Managed Switch]] access port.

## When it's the right tool

| Use it | Don't use it |
|---|---|
| 3 devices at one desk, 1 wall jack | Anywhere a [[VLANs|VLAN]] boundary is needed (it can't tag) |
| Temporary event/lab setups | As a building's main switch (no visibility, no security) |
| Truly tiny office (≤5 devices, flat network) | Where [[Power over Ethernet|PoE]] devices hang off it (most are PoE pass-nothing) |
| Budget is zero and needs are simple | Anywhere loops are plausible → invisible to STP debugging |

**Rule of thumb:** the moment you want guest isolation, VoIP, cameras, or any troubleshooting visibility, the answer is a managed (or at least "smart") switch. The price gap is small now.

## Configuration

None — that's the definition. Setup: plug in power, plug in cables. Verify: link LEDs on.

## Behavior facts that matter during troubleshooting

- **VLAN-blind but tag-transparent (usually):** it floods 802.1Q-tagged frames like any frame — several devices *behind* it will all land in whatever VLAN the feeding managed port assigns. All-or-nothing.
- **STP-transparent:** forwards BPDUs without participating. A loop *through* it is caught by upstream STP; a loop *between two of its own ports* creates a storm the upstream may only see as a flood. If BPDU guard is enabled upstream ([[Managed Switch]]), plugging this switch in may **err-disable the feeding port** — by design.
- **Invisible:** no LLDP, no MAC table you can query, no counters. From upstream you just see many MACs on one port — which is exactly how you *find* undocumented ones: `show mac address-table interface gi0/x` showing 6 MACs on a "PC port" = hidden desk switch.
- **DHCP snooping interaction:** upstream port must be untrusted; a rogue router plugged into the desk switch is still filtered upstream → [[DHCP]].

## Troubleshooting

1. Power LED? (wall-wart failures are common — spare PSUs cheap)
2. Link LEDs both ends? No → cable → [[Network Rack and Patch Panel]].
3. One device dead, others fine → move to another port (single-port failures happen).
4. All dead → uplink err-disabled upstream? (BPDU guard / port-security MAC limit exceeded — a desk switch adds MACs!)
5. Everything slow network-wide → suspect a loop within/through it → [[Spanning Tree Protocol]].

Ops: none needed — no firmware, no config, no backup. Just **document its existence and location** ([[Documentation Standards]]) so the MAC-count mystery above never costs an hour.

Related: [[Managed Switch]] · [[Switching and ARP]] · [[OSI Model]]
