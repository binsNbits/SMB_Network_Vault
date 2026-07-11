---
tags: [playbook, operations]
aliases: [Config Backup, Config Export, Device Backups]
---

# Configuration Backup Procedure

Device configs are hours-to-days of accumulated decisions. A dead [[Firewall (NGFW)]] with a current export = 30-minute recovery onto spare hardware; without = a rebuild from memory, wrong in ways you'll discover for weeks. **This procedure is a precondition in every other playbook.**

## When

| Trigger | Backup |
|---|---|
| **Before any change** | Always — it's the rollback |
| After any change (verified working) | Always — the new "known good" |
| Before [[Firmware Update Procedure|firmware updates]] | Always, and verify the file |
| Scheduled | Weekly automated sweep catches undocumented drive-by changes |

## How, per device class

| Device | Method |
|---|---|
| Cisco-style [[Managed Switch]]/[[Router]] | `copy running-config startup-config` first (save!), then `copy running-config tftp:`/`scp:` — or automate: **RANCID/Oxidized** pulls all configs nightly into git (diffs show *what changed and when* — invaluable) |
| pfSense/OPNsense | Diagnostics → Backup/Restore → XML export; AutoConfigBackup service for automatic on-change cloud copies |
| FortiGate | `execute backup config` / GUI; FortiManager at fleet scale |
| [[Wireless LAN Controller]] | Its backup **contains every AP's config** — controller settings → auto-backup + export off-box |
| [[NAS]] (DSM/QTS) | Configuration export (users/shares/settings) — *separate from data backup* → [[Backup Strategy]] |
| [[UPS]] NMC, [[Network Printer|printers]], [[IP Cameras and NVR|NVR]], PBX | Each has an export buried in settings — find it once, note the path in the device's inventory row |
| [[Unmanaged Switch]], [[Modem]] (bridged) | Nothing to back up — the exceptions |

## Where exports live (rules)

1. **Off the device** (a backup stored on the thing that dies with it isn't one) — a `configs/` share on the [[NAS]], itself backed up per [[Backup Strategy]].
2. **Versioned, dated, named**: `HQ-FW-01_2026-07-11_pre-vlan-change.xml` — never `backup-final-v2-REAL.xml`.
3. **Encrypted or access-controlled** — configs contain password hashes, PSKs, SNMP strings; they're credential material → treat like the password manager ([[Documentation Standards]]).
4. One copy reachable **when everything is down**: an encrypted USB in the safe or offline copy — restoring the firewall requires its config *before* the NAS is reachable.

## Restore — practice it

- Quarterly: open an export and confirm it parses/imports to a VM or spare (a corrupt export discovered during a disaster is worthless).
- Know each platform's restore quirk: pfSense XML restores cleanly cross-hardware *if interface names match* (edit the XML's interface assignments otherwise); Cisco configs paste per-section; controller restores must match software version.
- Restore drill = half of [[High Availability|cold-spare]] strategy: spare + current config + this procedure = your switch RMA plan.

Related: [[Firmware Update Procedure]] · [[Backup Strategy]] · [[Documentation Standards]]
