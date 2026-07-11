---
tags: [concept, layer2]
aliases: [STP, RSTP, Loop Prevention, BPDU, PortFast]
---

# Spanning Tree Protocol

STP (802.1D; modern **RSTP** 802.1w) prevents Layer-2 **loops**. Without it, one looped cable = broadcast storm = the entire network down in seconds. It works by electing a *root bridge* and blocking redundant paths until a live path fails.

## Why you care in an SMB

1. **Storm protection** — the intern who plugs both wall jacks into the same desk switch takes down nothing (STP blocks the loop) instead of everything.
2. **Cheap redundancy** — two uplinks between switches: one forwards, one stands by, automatic failover → [[High Availability]].
3. **The 30-second penalty** — classic STP holds a new port in listening/learning for ~30 s. PCs that boot faster than that miss [[DHCP]]. Fix: `spanning-tree portfast` on **edge ports only** (never on switch↔switch links — PortFast on an uplink defeats loop protection).

## Minimum competent config ([[CLI Quick Reference]])

```text
spanning-tree mode rapid-pvst            ! RSTP: sub-second convergence
spanning-tree vlan 1-4094 priority 4096  ! ON THE CORE SWITCH ONLY — forces it to be root
interface range gi0/1-20                 ! edge ports
 spanning-tree portfast
 spanning-tree bpduguard enable          ! port shuts if a switch appears on it
```

**Why set root manually:** default priorities make the *oldest/lowest-MAC* switch root — often some desk switch — producing absurd traffic paths. The root must be your core ([[Layer 3 Switch]] / [[Managed Switch]]).

**BPDU Guard:** edge ports should never receive BPDUs (switch hello packets). If one arrives — rogue switch or loop via an [[Unmanaged Switch]] — the port err-disables itself. Recover: `shutdown` / `no shutdown` after removing the cause.

## Recognizing STP problems

| Symptom | Suspect |
|---|---|
| Whole network dies; switch LEDs all blinking in sync; CPU pegged | Loop where STP is disabled or bypassed (unmanaged gear!) → pull uplinks one at a time until it stops |
| Devices work ~30 s after link-up, not before | Missing PortFast |
| Periodic 30–50 s freezes network-wide | Topology changes from a flapping port — find it: `show spanning-tree detail` (topology change count) |
| A port is up but passes nothing | `show spanning-tree` — is it BLK (blocking)? That may be *correct* |

> [!warning] Unmanaged switches are STP-invisible
> An [[Unmanaged Switch]] forwards BPDUs blindly. Loops *through* it are still caught, but loops *within* daisy-chained unmanaged gear plus misconfigured edges are how storms happen. Prefer managed everywhere loops are possible.

Related: [[Switching and ARP]] · [[VLANs]] · [[Slow Network Playbook]]
