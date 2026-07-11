---
tags: [device, infrastructure, layer1]
aliases: [Rack, Patch Panel, Structured Cabling, Cable Management]
---

# Network Rack and Patch Panel

**Role:** the physical layer's skeleton ([[OSI Model]] L1). Structured cabling = permanent horizontal runs from wall jacks terminating on a **patch panel**; short **patch cords** then map panel ports to switch ports. ~50% of real-world faults are Layer 1 — this note is how you prevent them.

## Why the patch panel exists (the part newcomers question)

Terminating wall runs directly into a switch works... until you re-terminate a solid-core cable for the fifth time and it fails intermittently forever. The panel gives you: solid-core runs punched **once** and never touched again; all moves/changes happen on cheap replaceable stranded patch cords; and a fixed labeling reference. **Change patch cords, never the punch-downs.**

## Rack planning

| Decision | Guidance |
|---|---|
| Size | Count RU (firewall 1U, switches 1–2U, panels 1U each, [[UPS]] 2U, [[Server]] 1–2U, shelf, PDU) + 30% growth |
| Type | Wall-mount 6–12U (closet SMB) vs floor 24–42U (server-room SMB) |
| Location | Lockable, ventilated/cooled, near cable-run center; not under plumbing; dedicated circuit(s) |
| Layout convention | Panels adjacent to their switches (short patches); UPS at bottom (weight); horizontal cable managers between panel/switch rows |

## Cabling standards that prevent future pain

- **Category:** Cat6 minimum new-install (Cat6a for 10 GbE runs / heavy [[Power over Ethernet|PoE]] heat → [[PoE Switch]]); solid-core for permanent runs, stranded for patches.
- **100 m rule:** 90 m permanent + 10 m patches total per channel — beyond it, gigabit *and* PoE get flaky.
- **One termination standard: T568B both ends** (A works too — *mixing* them per-cable creates crossover surprises). Punch all 4 pairs.
- Separation from power: don't run data parallel to mains in the same conduit; cross at 90° if unavoidable — induced noise = CRC errors → [[Switching and ARP]].
- No: staples through cables, tight zip-ties (crush = return loss), bend radius < 4× diameter, un-terminated coils "for later" without labels.
- **Test every run** with at least a wiremap tester at install; certify (Fluke-class) if you paid a contractor — make payment contingent on the report.

## Labeling & mapping ([[Documentation Standards]] applied)

```text
Wall jack:  A-14            (zone A, jack 14)
Panel port: A-14            (same ID — the whole trick)
Patch cord: both ends tagged
Switch:     interface gi0/14 description JACK-A-14-Reception
```
The **port map** (jack ↔ panel ↔ switch port ↔ device) lives with the site docs → [[Site Note Template]]. `show lldp neighbors` + `show mac address-table` rebuild it when inherited undocumented → [[CLI Quick Reference]].

## Diagnosing Layer 1 (the checks before any config work)

| Tool/check | Finds |
|---|---|
| Link LEDs both ends | Dead run, dead port |
| Swap patch cord | Bad patch (commonest fault — keep known-good spares) |
| Wiremap tester | Miswire, split pair, open |
| `show interfaces` CRC/input errors climbing | Noise, damaged run, connector |
| Cable tester TDR / switch cable-diag | Distance-to-fault on a broken run |
| Try adjacent panel port | Isolates panel termination vs the run |

**Split pairs deserve mention:** wiremap looks fine, gigabit fails or drops to 100 Mb — pairs punched to the right *pins* but not preserving the twisted *pairs*. A proper tester catches it; visual inspection doesn't.

Ops: photograph the rack after every change (before/after) · annual: retire abandoned patches (the rat's nest grows one "temporary" cable at a time) · door locked, access list short → physical access = total network access ([[Security Incident Basics]]).

Related: [[Power over Ethernet]] · [[UPS]] · [[Managed Switch]] · [[Documentation Standards]]
