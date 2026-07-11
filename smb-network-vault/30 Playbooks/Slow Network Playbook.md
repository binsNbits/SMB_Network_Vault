---
tags: [playbook, troubleshooting]
aliases: [Network Slow, Performance Issues, Bandwidth Problems]
---

# Slow Network Playbook

"Slow" is the vaguest ticket in networking. First job: **turn "slow" into a measurement**, then find the bottleneck.

## Step 0 — Quantify and scope

1. **What exactly is slow?** One app / file transfers to the [[NAS]] / websites / video calls / everything? Since when? For whom? (Same scoping table as [[No Internet Playbook]] Step 0.)
2. **Measure:** speedtest from a *wired* client (removes Wi-Fi as a variable) vs. the ISP contract; `ping -t <gateway>` for jitter/loss; large-file copy to the NAS timed (≈110 MB/s = healthy gigabit).
3. **Latency vs bandwidth:** pages lag but downloads fine = latency/[[DNS]]; downloads crawl = bandwidth/duplex. Video calls choppy = jitter/loss → [[QoS]] territory.

## Decision tree

### A. Slow for everyone, everywhere → shared bottleneck
1. **WAN saturated?** Firewall dashboard/[[SNMP and Monitoring]] graphs: uplink pegged → *who?* NetFlow/traffic insights name the host → backup running in business hours ([[Backup Strategy]] scheduling), cloud sync, one PC uploading suspiciously (→ [[Security Incident Basics]] if unexplained)
2. **Firewall CPU pegged?** IDS/IPS beyond hardware capacity → [[Firewall (NGFW)]] sizing
3. **ISP degraded?** Wired speedtest far under contract with quiet LAN → [[Modem]] signal stats → ISP call
4. **Bufferbloat** (fast speedtest, terrible calls while loaded): shaper on the firewall → [[QoS]]

### B. Slow on one switch / one area → path problem
1. `show interfaces` on the uplink: **errors/CRCs climbing = cable/duplex** → [[Switching and ARP]] duplex section, [[Network Rack and Patch Panel]] L1 checks
2. Uplink utilization ≥ wire speed → LACP or 10G uplift → [[Managed Switch]]
3. Daisy-chained [[Unmanaged Switch|unmanaged switches]] funneling a department through 1 Gb
4. Periodic network-wide freezes → STP topology changes from a flapping port → [[Spanning Tree Protocol]]

### C. Slow on Wi-Fi only → [[Wi-Fi Problems Playbook]]
(Interference, coverage, co-channel, one AP overloaded — the controller's client/RF views → [[Wireless LAN Controller]])

### D. Slow for one user → endpoint
Their link speed negotiated at 100 Mb (cable pair damage!), their position/RSSI, their machine → [[Desktops and Laptops]] masquerade table. `Get-NetAdapter` shows negotiated speed instantly.

### E. One application/server slow → not the network until proven
Server-side disk/CPU ([[Server]] / [[NAS]] health), app database, or DNS timeouts on the client. Prove the network innocent with a parallel raw transfer: fast copy + slow app = app problem.

## The classics (check-first list)

| Finding | Frequency |
|---|---|
| Port negotiated 100/half — damaged pair or ancient patch cord | Very common — `show interfaces status` |
| Backup/sync jobs in business hours | Very common |
| Wi-Fi channel overlap after someone "fixed" an AP | Common |
| Firewall shaper set for an old, slower WAN plan after ISP upgrade | Sneaky classic — caps everyone at the old speed → [[QoS]] |
| Broadcast storm brewing (loop) | Rare but catastrophic → [[Spanning Tree Protocol]] |

## Afterward

Baseline what "normal" looks like in [[SNMP and Monitoring]] (utilization, latency, error counters) — next time, the graphs answer Step 0 in seconds. Document the fix → [[Documentation Standards]].
