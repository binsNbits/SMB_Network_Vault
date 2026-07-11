---
tags: [device, wireless]
aliases: [Mesh Wi-Fi, Range Extender, Repeater, Wireless Uplink]
---

# Mesh Node and Range Extender

**Role:** extend Wi-Fi where Ethernet can't go. Two very different technologies share this aisle — know which you're holding.

## Extender vs. mesh — not the same thing

| | Classic repeater/extender | Mesh system / wireless-uplink AP |
|---|---|---|
| How | Rebroadcasts on the *same radio* | Dedicated backhaul band (tri-band) or managed wireless uplink |
| Throughput cost | **≥50% per hop** (half-duplex re-transmit → [[Wi-Fi Fundamentals]]) | Modest with dedicated backhaul; ~50% if backhaul shares the client band |
| SSID/roaming | Often a second SSID (`_EXT`), dumb roaming | One SSID, coordinated roaming |
| Management | Standalone web UI | Part of the [[Wireless LAN Controller|controller]] fleet |
| SMB verdict | **Avoid** — troubleshooting tax exceeds the price | Acceptable where cable is genuinely impossible |

**Decision tree:**
1. Can you pull cable? → wired [[Wireless Access Point]]. Always first choice.
2. Power outlet near coax/existing wire? → MoCA/powerline backhaul + normal AP (often beats mesh).
3. Truly no wire (detached shop, warehouse corner, historic walls)? → mesh node from the **same ecosystem** as your APs (UniFi wireless uplink, Omada mesh), placed per below.
4. Outdoor span between buildings? → that's a **point-to-point wireless bridge** (directional, e.g. UBNT building-to-building) — a different, better tool.

## Placement — the rule everyone violates

Put the node at **half-way between the wired AP and the dead zone, where signal from the wired AP is still ≥ −65 dBm** — *not* in the dead zone itself. A node placed in weak signal relays garbage at high volume: strong bars, terrible speeds. Verify backhaul RSSI in the controller after mounting.

## Configuration (UniFi-flavor mesh)

1. First connect the node **by wire** near the controller: adopt + firmware update. *Why: wireless adoption of un-updated firmware is flaky.*
2. Enable wireless uplink/meshing on the site (Settings → Wi-Fi → advanced) and on the uplink AP.
3. Unplug, mount at the planned midpoint, power (PoE injector/outlet), wait — it re-joins via the strongest wired-AP uplink.
4. Verify: controller shows uplink = wireless, RSSI ≥ −65 dBm, hop count = 1. **Cap hops at 1–2** — each hop halves shared-band throughput.
5. Same SSIDs/[[VLANs]] flow over the mesh automatically (that's the point of staying in-ecosystem).

## Troubleshooting

| Symptom | Cause → fix |
|---|---|
| Full bars, dial-up speeds | Node placed in the dead zone → move to midpoint |
| Node drops nightly | Marginal backhaul + interference; check RSSI, change backhaul channel |
| Clients stick to the far node | Roaming: enable min-RSSI/802.11r on the controller → [[Wi-Fi Fundamentals]] |
| Extender's `_EXT` SSID confusion, devices on wrong one | Retire the extender; deploy proper mesh or wire |
| Worked, then broke after adding a node | >2 hops or a mesh loop — re-check topology in the controller |

Ops: minimal — firmware with the fleet ([[Firmware Update Procedure]]); *revisit quarterly whether cable has become possible* — every mesh node is a standing reminder of a missing wire ([[Documentation Standards]]).

Related: [[Wireless Access Point]] · [[Wi-Fi Fundamentals]] · [[Wireless LAN Controller]] · [[Wi-Fi Problems Playbook]]
