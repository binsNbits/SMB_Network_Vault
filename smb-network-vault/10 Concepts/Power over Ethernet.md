---
tags: [concept, layer1]
aliases: [PoE, PoE+, 802.3af, 802.3at, 802.3bt, PoE Budget]
---

# Power over Ethernet

PoE delivers DC power and data over one Ethernet cable — why [[Wireless Access Point|APs]], [[VoIP Phones and PBX|phones]], and [[IP Cameras and NVR|cameras]] need no wall outlet. Supplied by a [[PoE Switch]] or midspan injector.

## Standards — match device class to switch capability

| Standard | Name | Power at port | Powers |
|---|---|---|---|
| 802.3af | PoE | 15.4 W | Phones, basic APs, small cameras |
| 802.3at | PoE+ | 30 W | Wi-Fi 5/6 APs, PTZ cameras |
| 802.3bt | PoE++ (Type 3/4) | 60 / 90–100 W | Wi-Fi 6E/7 APs, video phones, thin clients, lighting |
| — | "Passive PoE" | fixed, unnegotiated | Some Ubiquiti/Mikrotik gear — **can damage non-matching devices**; never mix with standard PoE blindly |

Negotiation is automatic and safe in the 802.3 standards: the switch probes (25 kΩ signature) before applying power — plugging a laptop into a PoE port is harmless.

## The PoE budget — the trap everyone hits

A 24-port PoE+ switch does **not** supply 24×30 W. It has a total budget (e.g., 195 W). Devices negotiate up to their class; when the sum exceeds the budget, **the switch silently refuses power to lowest-priority ports** — usually noticed as "the newest camera won't boot."

**Planning:** sum realistic draws (AP ≈ 13–25 W, phone ≈ 4–7 W, camera ≈ 5–13 W, PTZ ≈ 25 W), add 25% headroom, check `show power inline` ([[CLI Quick Reference]]) against the datasheet budget. Count PoE load into [[UPS]] runtime too — a PoE switch under load draws serious wattage.

## Conditionals

- **Only 1–2 PoE devices, non-PoE switch?** → per-device injectors; cheaper than replacing the switch.
- **Device >100 m from the switch?** → PoE extender or fiber+local power; Ethernet's 100 m limit includes power.
- **AP reboots under load?** → It negotiated af but needs at; check `show power inline`, move to a higher-budget port or upgrade switch.
- **Remote hard-reboot needed?** → PoE gives you it free: `shutdown`/`no shutdown` (or web-UI power-cycle) on the port power-cycles the device — invaluable for hung cameras/APs.

## Cabling notes ([[Network Rack and Patch Panel]])

Cat5e handles PoE+, but Cat6/6a runs cooler in large bundles (heat rises with .bt loads). Avoid PoE over old Cat5 or couplers; voltage drop causes flaky far-end behavior.

Related: [[PoE Switch]] · [[Wireless Access Point]] · [[IP Cameras and NVR]] · [[UPS]]
