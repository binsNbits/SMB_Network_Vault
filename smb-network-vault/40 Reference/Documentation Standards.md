---
tags: [reference, operations]
aliases: [IPAM, Labeling, Network Documentation]
---

# Documentation Standards

Undocumented networks are unmanageable networks. Every playbook in this vault assumes you can answer: *what is this device, where is it, what IP does it have, and what plugs into it?*

## What to document (minimum viable set)

| Artifact | Contents | Update trigger |
|---|---|---|
| **IPAM sheet** | Every static IP: address, device, MAC, VLAN, location, owner | Any IP assignment |
| **Network diagram** | Physical (what plugs into what) + logical ([[VLANs]], subnets) | Any topology change |
| **Device inventory** | Model, serial, firmware, purchase/EoL date, credentials location | New device / firmware update ([[Firmware Update Procedure]]) |
| **Port maps** | Switch port → patch panel port → wall jack → device ([[Network Rack and Patch Panel]]) | Any re-patch |
| **Config backups** | Exports of every device config | Before/after every change → [[Configuration Backup Procedure]] |
| **Change log** | Date, who, what, why, rollback plan | Every change |

## Naming convention (pick one, never deviate)

```text
<SITE>-<ROLE>-<NN>
HQ-FW-01      main firewall at HQ
HQ-SW-CORE-01 core switch
HQ-SW-ACC-02  access switch #2
HQ-AP-12      access point #12
BR1-RTR-01    branch-1 router
```

**Why:** the hostname appears in logs, [[SNMP and Monitoring|monitoring alerts]], LLDP neighbor tables, and CLI prompts — a good name tells you what and where without a lookup.

## IP allocation convention

Reserve predictable ranges inside each subnet (example for a /24):

| Range | Use |
|---|---|
| .1 | Gateway ([[Router]] / [[Layer 3 Switch]] SVI) |
| .2–.9 | Network infrastructure (switches, [[Wireless Access Point|APs]], [[UPS]]) |
| .10–.49 | Servers, [[NAS]], [[Network Printer|printers]] (static/reserved) |
| .50–.239 | [[DHCP]] pool |
| .240–.254 | Static exceptions / testing |

## Labeling

- Label **both ends** of every cable (`P12↔SW01:g0/12`).
- Label every device with hostname + management IP.
- Label patch panel ports to wall-jack IDs.

## Credentials

- Password manager (shared team vault) — one entry per device, linked to inventory row.
- Never default creds ([[Vendor Default IPs and Logins]]), never reused passwords, never in plaintext files.

Related: [[New Site Deployment]] · [[Site Note Template]] · [[Device Note Template]]
