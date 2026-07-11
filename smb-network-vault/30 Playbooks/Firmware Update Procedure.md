---
tags: [playbook, operations]
aliases: [Firmware Upgrades, Patching Network Devices, Update Procedure]
---

# Firmware Update Procedure

Updates fix actively-exploited CVEs — and occasionally brick devices or change behavior. This procedure captures the upside while containing the downside.

## Cadence and urgency

| Trigger | Action window |
|---|---|
| **Actively-exploited CVE in an internet-facing device** ([[Firewall (NGFW)]], [[VPN Appliance]] SSL-VPN!) | Same week — these are the SMB breach headlines |
| Regular security releases | Monthly–quarterly maintenance window |
| Feature releases | Only if you need the feature; let others find the bugs (wait 2–4 weeks post-release, scan vendor forums) |
| "If it ain't broke" school | Rejected: unpatched network gear is how ransomware arrives → [[Security Incident Basics]] |

Track: vendor security mailing lists / PSIRT feeds for each product in the inventory → [[Documentation Standards]].

## The procedure (per device)

**Before**
1. Read the release notes — breaking changes, config-migration warnings, *and the upgrade path* (some jumps require intermediate versions).
2. **Back up the config** → [[Configuration Backup Procedure]]. Verify the export opens/validates.
3. Note current firmware version (rollback target); confirm the vendor supports downgrade (some don't — raises the stakes).
4. Schedule the window; announce; have console access ready ([[Device Access Methods]]) — if the update kills the network path, the console is your only way in.
5. **Stable power**: device on [[UPS]] — power loss mid-flash is the classic brick.

**During**
6. Upload/verify the image (checksums where offered); apply; **do not power-cycle during flash** even when it looks hung — give it 15+ min.
7. Device reboots → verify: version correct, config intact, *function* intact (not just ping: pass traffic, make a call, check VPN tunnels re-established → per-device verification lists in each device note).

**After**
8. Watch logs/[[SNMP and Monitoring]] for 24 h — memory leaks and interop bugs surface under load, not at 2 AM in the window.
9. Log it: device, old→new version, date, notes → change log.

## Fleet strategy (canary pattern)

Never update a whole class at once. **One canary** ([[Wireless Access Point|AP]], switch, phone) → run 24–48 h → then the fleet. Controllers first, then devices ([[Wireless LAN Controller]] before its APs — version compatibility runs controller≥device).

## Order for a full-stack maintenance window

Furthest-from-you first, so each update's network path stays managed: endpoints/peripherals → APs → access switches → core switch → firewall **last** (its reboot drops everything — and you want everything else already verified when it comes back).

## When it goes wrong

| Situation | Response |
|---|---|
| Device unreachable post-flash | Wait 15 min (image unpacking) → console → boot menu/recovery image (most switches/firewalls have dual-partition rollback: `boot system flash:<old>`) |
| Boots but config mangled | Restore the export from step 2 |
| New firmware misbehaves | Downgrade if supported + restore config; else vendor TAC; note the version as banned in the inventory |
| Bricked | Vendor recovery procedure (TFTP boot etc.) — this is why RMA/spares policy exists → [[High Availability]] |
