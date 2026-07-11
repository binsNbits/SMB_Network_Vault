---
tags: [concept, operations]
aliases: [HA, Redundancy, Failover, Dual WAN, LACP]
---

# High Availability

HA removes single points of failure — **in proportion to what downtime actually costs the business.** An SMB rarely needs everything redundant; it needs the *right* things redundant.

## Find your single points of failure (in typical failure-frequency order)

1. **Power** → [[UPS]] on every network stack (the cheapest, highest-value redundancy there is)
2. **WAN link** → second ISP (different medium: fiber + cable, or 5G backup) with automatic failover on the [[Firewall (NGFW)]] or an [[SD-WAN Appliance]]
3. **The firewall itself** → HA pair (pfSense CARP, FortiGate HA) — two boxes, one virtual IP, stateful failover
4. **Core switch** → stacked pair or at minimum a configured cold spare on the shelf
5. **Uplinks** → LACP bundles / redundant paths (blocked by [[Spanning Tree Protocol]] until needed)
6. **Server/storage** → RAID (disk failure ≠ outage), dual PSUs, VM replication; note **RAID is not backup** → [[Backup Strategy]]
7. **[[DHCP]]/[[DNS]]** → split scopes or failover peering; two DNS resolvers in every lease

## Techniques glossary

| Technique | Layer | What fails over |
|---|---|---|
| Dual PSU | Power | One supply / one circuit |
| UPS + generator | Power | Utility power |
| LACP (802.3ad) | L2 | One cable in a bundle (also adds bandwidth) |
| STP / RSTP | L2 | One switch-to-switch path |
| Stacking / MLAG | L2 | One core switch |
| VRRP / CARP / HSRP | L3 | One gateway (virtual IP shared by two routers) |
| Dual-WAN policy | L3/4 | One ISP |
| RAID 1/5/6/10 | Storage | One or two disks |
| VM HA / replication | Compute | One hypervisor host |

## SMB decision framework

Ask: **what does one hour of downtime cost?** Then:

| Downtime tolerance | Sensible spend |
|---|---|
| A day is annoying but survivable | UPS everywhere, cold-spare switch, config backups ([[Configuration Backup Procedure]]) |
| Hours hurt (most SMBs) | + dual WAN with auto-failover, RAID + tested backups |
| Minutes hurt (POS-dependent retail, medical) | + HA firewall pair, stacked core, dual-PSU everything, 4G/5G out-of-band access |

**Test your failovers on a schedule.** An untested failover is a rumor: pull the primary WAN quarterly, kill a PSU, restore a backup. Discover failures on your terms → [[SNMP and Monitoring]] should page you *during* the test.

Related: [[UPS]] · [[SD-WAN Appliance]] · [[Firewall (NGFW)]] · [[Backup Strategy]]
