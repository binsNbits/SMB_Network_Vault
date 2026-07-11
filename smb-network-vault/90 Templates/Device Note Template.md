---
tags: [template]
aliases: []
---

# Device Note Template

> Copy this structure when documenting a **specific physical device** you manage (an *instance*, e.g. `HQ-SW-CORE-01`), linking back to its class note (e.g. [[Managed Switch]]). Fill every row — blanks are future outages.

---

# <HOSTNAME per [[Documentation Standards]]>

**Class:** [[<device class note>]]
**Site:** [[<site note>]]

## Identity

| Field | Value |
|---|---|
| Model / HW rev | |
| Serial | |
| Firmware (current) | |
| Purchase date / warranty / EoL | |
| Physical location (rack + RU) | → [[Network Rack and Patch Panel]] |
| Management IP / VLAN | → IPAM row |
| MAC (mgmt) | |
| Credentials | (password-manager entry name — never the password itself) |
| Access methods enabled | SSH / HTTPS / console / cloud → [[Device Access Methods]] |

## Connections

| Local port | Connects to | Remote port | VLAN/purpose |
|---|---|---|---|
| | | | |

## Configuration highlights

- Deviations from the class-note baseline and *why*:
- Config backup location + last export date → [[Configuration Backup Procedure]]:

## History

| Date | Change | By | Notes |
|---|---|---|---|
| | | | |

## Open issues / quirks

- (e.g., "port 7 flaky since 2026-03 — re-terminate on next visit")
